# 技术路径与文件职责

## 流程

单人照片 → BiRefNet ONNX 透明抠图 → YOLOv8n-Pose ONNX 17 点检测 → SCHP Person-Part ONNX 七类人体解析 → 部件/关节融合 → 连续蒙皮 → 关节动作预览与校正 → 独立桌面宠物。

BiRefNet 与 YOLO 为 FP32 ONNX，SCHP 为静态 INT8 ONNX，均在本机 CPU ONNX Runtime 推理。生成后播放器仅运行 C# 动画，不再加载模型。

## 项目结构

| 文件/目录 | 职责 |
|---|---|
| PetGenerator.exe | 生成器入口 |
| src/UI/Generator.cs | WinForms 界面、异步任务、取消、动作预览、宠物库 |
| src/Shared/Common.cs | 配置、照片读取、相对路径验证、完整宠物包导出 |
| src/Inference/InferenceWorker.cs | C# 后台入口、抠图管线和 JSON 行进度协议 |
| src/Inference/NativeInference.cs | ONNX Runtime 会话、张量输入输出和资源释放 |
| src/Inference/ImageProcessing.cs | BGRA 像素、双线性/Lanczos 缩放、透明度、RGB CHW 归一化 |
| src/Inference/ModelStore.cs | 三个模型的固定版本、下载续传、SHA-256 校验 |
| src/Inference/PoseBuilder.cs | YOLO letterbox、NMS、坐标还原、肢体分层、蒙皮权重 |
| src/Inference/HumanParser.cs | SCHP 归一化、七类像素 argmax、尺寸还原和部件预览 |
| src/Shared/RigData.cs | UI、推理程序、播放器共同使用的骨架数据契约 |
| src/Animation/PoseRig.cs | 骨架读取、父子变换、两骨 IK、动作曲线、渲染调度 |
| src/Animation/PhotoMesh.cs | 连续网格蒙皮、双线性透明纹理采样、CPU 三角形光栅化 |
| src/UI/JointEditor.cs | 拖动校正关节，保存后重新分层 |
| src/Player/PetPlayer.cs | 透明置顶窗口、鼠标交互、重力、菜单与托盘 |
| models/birefnet-portrait.onnx | BiRefNet 人像模型 |
| models/model.json | BiRefNet 固定版本和校验信息 |
| models/yolov8n-pose.onnx | YOLO 姿态模型，13,484,153 字节 |
| models/pose-model.json | YOLO 固定版本和 SHA-256 |
| models/schp-pascal-7-int8.onnx | SCHP 七类人体部件分割模型，69,148,800 字节 |
| models/parser-model.json | SCHP 固定提交、预处理、类别和 SHA-256 |
| bin/inference/ | C# InferenceWorker.exe、ONNX Runtime 原生/托管 DLL、依赖清单、许可证 |
| bin/PetPlayer.exe | 导出时复制的播放器模板 |
| tools/setup.ps1 | 本地依赖与模型准备 |
| tools/build.ps1 | 使用系统 C# 编译器构建 |
| tools/test.ps1 | C# 回归测试 |
| tools/build-common.ps1 | 统一编译输入与托管依赖白名单 |
| tools/acceptance.ps1 | 无 Python 的真实模型端到端验收 |
| tests/InferenceTests.cs | 预处理、透明度、姿态解码、NMS、新旧实图对比与骨架权重测试 |
| tests/CoreTests.cs | 物理、边界、配置安全、图片资源测试 |
| tests/RigTests.cs | 旋转中心、父子连接、IK 与动作预览图 |
| outputs/ | 完整可分享宠物包 |
| work/jobs/ | 每次照片处理/关节校正的独立工作区 |
| logs/ | 本地任务日志 |

## 姿态与分层

YOLO 输入为 640×640 RGB，保持比例补灰边，除以 255，NCHW。输出 [1,56,8400]：框 4 + 人物置信度 1 + 17×3 关节点。人物过滤阈值 0.3、NMS IoU 0.45。补边/缩放逆变换回抠图坐标；多人物明确拒绝。

17 点：鼻、左右眼、左右耳、左右肩、左右肘、左右腕、左右髋、左右膝、左右踝。左右按人物自身方向。低置信度肢体保持静止，不编造检测结果。脖子锚点取两肩中点，骨盆锚点取两髋中点，是推导点而非模型直接检测点。

SCHP 输出背景、头、躯干、上臂、前臂、大腿、小腿七类像素。左右侧由像素到 YOLO 对应骨段的距离决定；模型部件与关节半径互相约束，减少袖子侵入躯干或左右腿串层。SCHP 漏分的透明人物像素保留原几何结果，因此深色衣服漏检不会导致肢体消失。最终仍为 10 层：躯干、头、左右上臂/前臂/大腿/小腿。手脚随前臂和小腿，不模拟手指。rig/parts.png 保存彩色部件预览供诊断。

## 关节中心和连续蒙皮

每层保存源图片坐标 pivot、end、parent。计算式：

世界点 = 世界关节 + 旋转矩阵 ×（源点 − 源关节）

子关节位置由父骨骼变换得到，因此肩旋转带动肘，前臂绕肘旋转，而不是绕裁片中心旋转。

当前采用混合渲染：躯干顶点只绑定根骨骼，手臂从身体纹理分离，以肩、肘为中心独立绘制；肩部保留源纹理重叠区。身体网格不接受手臂变换，避免摆手时胸腹膨胀缩小。腿部保留连续蒙皮，权重只在相邻关节附近混合，不向无关骨骼扩散。大幅遮挡仍可能露出缺失部位，因此限制摆幅，不补画不可见身体。

## 动作与物理

呼吸轻微起伏，走路由速度驱动步相，手臂反相摆动。可用站姿腿使用两骨 IK，支撑半周期保持脚目标高度，摆动半周期抬脚。舞蹈采用肩肘躯干错相摆动并停止水平追随。跳跃结合弹道与抬臂/屈腿，动作权重指数平滑。

重力固定 1/120 秒步长，位置包含 1/2·g·dt²，具有地面碰撞、水平加速、屏幕约束和跳跃冷却。不是完整三维人体动力学，二维脚目标不能保证所有照片完全无滑步。

## 导出和搬移

包内包含 Pet.exe、pet.json、pet.png、rig/rig.json、rig/texture.png、10 张肢体 PNG 和说明。只使用相对路径，配置最后写入；未完成包不进入宠物库。分享整个文件夹即可，播放器不需要模型或 Python。

生成器运行必须携带 PetGenerator.exe、models、bin（含 inference 全部 DLL 和许可证）；src 只在开发时需要。需要 .NET Framework 4.8+ 和 Visual C++ x64 运行库。所有照片处理离线，准备模型时才联网。原照片不会覆盖。校正使用新工作目录，取消时保留上一份有效骨架。

## 模块边界

UI 只通过 UTF-8 JSON 行协议调用独立 C# 推理进程，不直接加载大型模型；取消会终止当前工作进程并释放内存。Inference 只依赖 Shared 数据契约和 ONNX Runtime，不依赖 UI 或动画渲染。Player 只依赖 Shared + Animation，不包含推理 DLL 或模型。各任务使用独立 work/jobs 目录，源照片不覆盖。
