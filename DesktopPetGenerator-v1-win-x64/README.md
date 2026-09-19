# 桌面宠物生成器 V1 · 真人关节版

双击 PetGenerator.exe。照片只在本机处理，不上传，不重绘脸部和服装。

当前只保留西装人物，启动生成器会自动加载其预览。原图在 inputs/西装人物.jpg，独立宠物在 outputs 内唯一文件夹。旧角色素材和旧产物已移入 Windows 回收站，源码、模型和无关合同文档保留。

本版修正了手臂影响躯干的问题：躯干绑定根骨骼，手臂改为独立关节图层；脚步采用 60% 支撑、40% 抬脚回摆，并与左右摆臂配合。正面照片只能提供二维迈步近似，不会自动变出真实侧身走路素材。

## 使用

1. 选择清晰的单人全身照，最好双脚完整、手臂离开身体。
2. 点击“抠图 + 识别关节”，使用 BiRefNet、YOLO Pose 和 SCHP Person-Part 三个 ONNX 模型。SCHP 将人物分为头、躯干、上下臂和上下腿，再与关节几何融合；漏分区域会回退到骨骼规则。
3. 下拉框预览走路、呼吸、跳舞和跳跃，勾选“显示关节旋转轴”检查骨架。
4. 点击“校正关节 / 重新分层”，将圆点拖到真实关节位置，保存。
5. 长裙、坐姿、遮挡建议勾选“保守动作”，重新处理或校正后生效。
6. 点击“生成宠物文件夹”，然后“启动宠物”。

桌面上：左键拖动，双击跳跃，鼠标靠近头顶自动跳起。右键菜单提供跳舞、关节动画开关、骨架、跟随、重力、暂停、隐藏和退出。托盘也可操作。

## 分享与限制

发送 outputs 中整个宠物文件夹，必须保留 Pet.exe、pet.json、pet.png、rig 目录。对方运行宠物不需要 Python、模型、网络或独立显卡。支持 Windows 10/11 x64、.NET Framework 4.x。

生成器已全部迁移到 C#：BiRefNet、YOLO 和 SCHP 由 ONNX Runtime 1.23.2 CPU 推理，图片预处理、透明度处理、关节识别和肢体分层均不再调用 Python。项目已删除便携解释器、pip、NumPy、Pillow 和旧 Python 脚本。

分享生成器需保留 PetGenerator.exe、bin/（包括 inference 内全部 DLL 和许可证）、models/、docs/。inputs/ 和 outputs/ 按需要携带，src/、tests/、tools/ 是开发文件，work/、logs/ 不必分享。模型未齐全时点击“准备处理环境”。需要 Windows 10/11 x64、.NET Framework 4.8+ 和 Microsoft Visual C++ 2015–2022 x64 运行库；本机已验证，尚未在全新 Windows 虚拟机验证。三个 ONNX 模型共约 1006 MiB。

这是单照片二维骨骼动画，不是视频生成或三维重建。坐姿不会自动变成自然站姿，长裙不会被强行拆成两条腿；不可见肢体不凭空补画。复杂交叉、宽袖、长发遮挡仍可能出现局部拉伸，建议校正或换图。

## 开发

运行 tools/setup.ps1 准备/校验 C# 依赖和模型，tools/build.ps1 离线编译，tools/test.ps1 执行 C# 回归测试。无需 Python 或 .NET SDK。tools/acceptance.ps1 可重复验证真实模型、界面取消、导出和异地工作目录启动。

[技术路径与文件职责](docs/TECHNICAL_PATH.md) · [开发说明](docs/DEVELOPMENT.md) · [验收](docs/ACCEPTANCE.md) · [第三方许可](docs/THIRD_PARTY.md)

[工程化审查与建议优先级](docs/ENGINEERING_REVIEW.md)

[生成器、宠物包、SmartScreen 与 VC++ 运行库分发说明](docs/DISTRIBUTION.md)
