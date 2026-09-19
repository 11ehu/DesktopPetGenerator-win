# DesktopPetGenerator v1 Windows x64 发布说明

## 版本概览

本包为 Windows 10 / 11 x64 的最小分发目录，适合在已安装依赖的环境中直接测试和使用。

## 包含内容

- `PetGenerator.exe`：桌面宠物生成器主程序
- `bin/`：播放器、C# 推理程序和 ONNX Runtime 依赖
- `models/`：三个主模型权重与配置文件
- `docs/`：第三方许可证和说明文件
- `README.md`：使用说明
- `release-manifest.sha256`：完整文件 SHA-256 校验清单

## 运行前提

- Windows 10 / 11 x64
- .NET Framework 4.8+
- Microsoft Visual C++ 2015-2022 Redistributable x64

> 若系统缺少 VC++ x64 运行库，原生 DLL 可能加载失败。

## 使用方式

1. 解压本目录。
2. 先确认已安装 .NET Framework 4.8+。
3. 安装 Microsoft Visual C++ Redistributable x64。
4. 运行 `PetGenerator.exe`。
5. 如需要，可使用 `bin/PetPlayer.exe` 进行播放器测试。

## 重要提醒

- 本包不包含源码、日志、测试缓存、个人照片及已生成宠物产物。
- 本包不是最终签名安装器，不适合直接当作公开发布的正式安装包。
- 公开发布前建议再做代码签名、安装器包装和 Windows SmartScreen 检查。

## 校验方式

可在 Windows PowerShell 中执行：

```powershell
Get-ChildItem -Recurse -File | Get-FileHash -Algorithm SHA256
```

或核对 `release-manifest.sha256` 中的哈希值。
