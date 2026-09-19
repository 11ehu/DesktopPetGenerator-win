# Desktop Pet Generator for Windows

A lightweight AI-powered desktop pet generator for Windows. Turn photos into interactive desktop companions with automatic background removal, local ONNX inference, transparent rendering, and customizable animations.

这是 Desktop Pet Generator V1 的 Windows 发布仓库。

完整的可运行程序请从 GitHub 的 **Releases** 页面下载 `DesktopPetGenerator-v1-win-x64.zip`，解压后运行 `Setup.exe`（首次配置）或 `PetGenerator.exe`。

本仓库仅保留发布说明、校验清单和第三方许可证；ONNX 模型、运行时文件和完整 Windows 程序均以 Release 附件发布，以避免 GitHub 普通 Git 仓库的单文件大小限制。

## 校验

下载后可使用发布包内的 `release-manifest.sha256` 校验文件完整性。

## 许可证

发布包内包含 BiRefNet、YOLO 和 SCHP 的第三方许可证文件。
# 桌面宠物生成器 V1

把一张真人照片制作成可以在 Windows 桌面上行走、跟随鼠标、跳跃和跳舞的桌面宠物。

项目采用真人照片的二维骨骼动画路线：先保留原照片的真实外观，再利用姿态和人体部件信息驱动关节运动，不依赖视频生成或三维重建。

## 技术路线

1. **BiRefNet**：对输入照片进行人物抠图，生成透明背景的人物图层。
2. **YOLOv8-Pose**：检测人体关键点，得到肩、肘、腕、髋、膝、踝等关节位置。
3. **SCHP Person-Part**：进行人体部件分割，将头部、躯干、手臂和腿部拆成可独立变换的图层。
4. **关节动作系统**：以真实关节为旋转中心，结合骨骼层级、步态周期、支撑脚和重力状态生成走路、呼吸、跳舞、跳跃等动作。
5. **ONNX Runtime**：在本机直接运行三个 ONNX 模型，推理链路全部由 C# 调用，无需 Python、PyTorch 或网络连接。

## 功能

- 上传 JPG、PNG 或 BMP 单人照片并自动抠图。
- 姿态检测、人体部件分层和关节校正。
- 真人风格二维骨骼动画：走路、呼吸、跳舞、跳跃。
- 宠物跟随鼠标移动；鼠标位于头顶附近时会尝试跳起触碰。
- 可开关重力、跟随鼠标、关节动画、骨架和调试预览。
- 生成可单独分发的宠物文件夹，播放器不依赖 Python、模型或独立显卡。

## 快速开始

### 使用发布包

从 GitHub Releases 下载 `DesktopPetGenerator-v1-win-x64.zip`，解压后运行 `PetGenerator.exe`。首次运行需要 Windows 10/11 x64、.NET Framework 4.8+ 和 Microsoft Visual C++ 2015–2022 x64 运行库。

### 从源码构建

```powershell
powershell -ExecutionPolicy Bypass -File tools/setup.ps1
powershell -ExecutionPolicy Bypass -File tools/build.ps1
powershell -ExecutionPolicy Bypass -File tools/test.ps1
```

`tools/setup.ps1` 负责准备 ONNX Runtime、模型和许可证；后续构建与测试不需要 Python 或 .NET SDK。

## 分发生成的宠物

将 `outputs` 中生成的宠物文件夹整体复制给其他 Windows 用户，保留 `Pet.exe`、`pet.json`、`pet.png` 和 `rig` 目录即可。生成后的宠物运行时只使用 C# 播放器和图片数据，不需要携带三个 ONNX 模型。

## 项目结构

```text
src/       C# 生成器、推理、姿态、分层和播放器代码
models/    ONNX 模型配置与权重
tools/     环境准备、构建、测试和验收脚本
tests/     核心逻辑与推理回归测试
docs/      技术路径、分发、验收和第三方许可说明
outputs/   本地生成的宠物（不提交到源码仓库）
```

## 已知限制

这是基于单张照片的二维近似动画，不会自动补全照片中不可见的背面或遮挡肢体。坐姿、交叉手臂、宽袖、长发和长裙可能需要手动校正关节或使用“保守动作”。

欢迎提交 Issue 反馈模型兼容性、动作效果和分发问题。

## 许可证

项目中的第三方模型和运行库分别遵循其原始许可证，详见 `LICENSE-*.txt` 和 `docs/THIRD_PARTY.md`。
