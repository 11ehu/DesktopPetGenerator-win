# C# / ONNX 迁移验收 · 2026-09-19

## 已完成

- BiRefNet、YOLOv8n-Pose 和 SCHP Person-Part 均通过 C# 调用官方 ONNX Runtime 1.23.2 CPU 执行，不依赖 Python。
- 预处理、归一化、sigmoid、透明度合成、裁剪、YOLO NMS、坐标还原、肢体分层和蒙皮权重全部由 C# 完成。
- 已移除 runtime/、旧 .py 源码、requirements.txt、解释器安装包、pip 缓存及字节码。便携运行环境和下载缓存进入回收站，可恢复；项目目录内没有 Python 解释器、DLL、脚本或安装包。
- src 按 Shared / Inference / Animation / UI / Player 分层。推理模块仅依赖 Shared，不编译动画或界面代码；播放器不带模型和推理库。
- 版本与依赖 SHA-256 清单：bin/inference/dependencies.json；三个模型通过 prepare 完整 SHA-256 校验。
- 推理目录含许可证、测试程序共约 14.85 MiB；三个模型共约 1006 MiB。清理的便携环境和构建/下载缓存合计约 639 MiB，不是全由 Python 本体构成。

## 本机实际验证

- tools/setup.ps1 -SkipModel：依赖文件哈希校验、三个可执行程序构建、原生运行库 health 均通过。
- tools/test.ps1：物理、边界、配置安全、旋转中心、父子骨骼、两骨 IK、支撑/摆动脚步周期测试通过；C# 推理测试含 1290 次断言通过。
- 西装照片新旧抠图尺寸均为 440×1385，裁剪框完全相同；alpha > 128 的前景 IoU 为 0.999599，alpha 平均绝对误差 0.2826/255。
- 17 个关节点与旧结果的最大偏差 0.852 像素。SCHP 七类部件分割、独立手臂、躯干根骨骼权重和站姿步态通过；rig/parts.png 已生成。
- 删除 Python 后 tools/acceptance.ps1 实跑通过：GUI → BiRefNet → YOLO → SCHP 部件 → 融合分层 → 导出 → 启动 → 关闭。
- 取消测试通过：当前工作进程停止，界面恢复，不生成半成品宠物包。
- 手动关节 JSON 输入后重新分层通过。
- 新导出包移到 work/acceptance-export-*，从 C:\Windows 工作目录运行 Pet.exe --smoke-test，70 帧后正常退出，smoke-pass.txt 为 PASS。
- 人物右键菜单改为播放器生命周期内的持久对象；连续两次显示/关闭后 menu-pass.txt 为 PASS，不再在 Closed 事件中提前释放。现有西装宠物包的 Pet.exe 已同步更新。
- C# 新骨架六动作预览已检查：work/bodyparts-acceptance/动作预览.png。渲染 60 帧平均约 6.75 ms（仅本机测试数据）。
- outputs 保持原有一个西装人物包；验收产物放 work 下，不覆盖已有宠物。

## 边界

以上是本机 Windows x64 验收，不代表已经完成全新 Windows 虚拟机测试。生成器需要 .NET Framework 4.8+ 和 Microsoft Visual C++ 2015–2022 x64 运行库，不需要安装 Python、CUDA 或 .NET SDK。模型/依赖首次缺失时需联网下载；下载中断续传实现已保留，但本次未重新下载全部约 1006 MiB 模型做网络故障注入测试。

单照片二维骨骼仍受遮挡限制：较大摆幅可能出现腋下或衣物接缝空隙，不会补画照片中不可见的身体；本次迁移不等于升级为三维人体动画。
