# Windows 分发说明

## 两种文件不要混淆

### 已生成的宠物

发送 `outputs/某只宠物/` 整个目录即可，包含 `Pet.exe`、`pet.json`、`pet.png` 和 `rig/`。它不加载模型和 ONNX Runtime，不需要 Python、网络或 VC++ 运行库；目标为 Windows 10/11 x64 与 .NET Framework 4.8+。

### 桌面宠物生成器

运行 `tools/package-generator.ps1` 生成最小分发目录。它只包含生成器、播放器模板、C# 推理程序、三个 ONNX 模型、许可证和文档，不包含私人照片、已有宠物、源码、日志或测试缓存。当前约 1.02 GiB，不能只发送单独一个 `PetGenerator.exe`。

生成器还依赖 Microsoft Visual C++ 2015–2022 Redistributable x64，因为 `onnxruntime.dll` 是用 Microsoft C++ 工具链构建的原生组件。`.NET Framework` 与 VC++ 运行库是两套不同组件；缺少后者时，文件明明存在也可能出现“找不到指定模块”或无法加载 DLL。

官方最新版下载入口：<https://aka.ms/vc14/vc_redist.x64.exe>。当前便携目录不会静默安装系统组件。正式安装器应先检测，再经用户确认调用官方安装包；这通常需要管理员权限。

## SmartScreen 是什么

通过浏览器、邮件或聊天软件下载的文件会带互联网来源标记。Microsoft Defender SmartScreen 会综合检查文件哈希下载信誉、发布者签名及安全情报。当前 EXE 没有受信任的 Authenticode 签名，新版本也没有公众下载信誉，因此陌生电脑可能显示“Windows 已保护你的电脑 / 未知发布者”。这不等于已经发现病毒，而是 Windows 无法确认发布者身份与文件声誉。

面向熟人测试，可以提供 HTTPS 下载地址、发布包 SHA-256 和明确版本，让对方只在确认来源后运行。面向陌生用户，不应把“点击仍要运行”作为正式方案：应制作 MSIX/MSI/EXE 安装器，使用稳定的受信任代码签名身份对安装器和全部 PE 文件签名，并建立发布信誉；Microsoft Store 分发最容易避免 SmartScreen 下载警告。企业策略或 Windows 11 Smart App Control 可能完全禁止未签名程序。

代码签名需要经过身份验证的证书或 Microsoft Artifact Signing，无法在源码里免费伪造；自签名证书对陌生电脑没有信任价值。签名后也不保证首日完全没有 SmartScreen 提示，但能显示已验证发布者并让发布者信誉跨版本积累。

## 正式发布前检查

1. 在没有开发工具的全新 Windows 10/11 虚拟机安装并测试。
2. 处理 YOLOv8n-Pose 权重的 AGPL-3.0 分发与商用授权问题。
3. 建立产品版本号，生成 SBOM、第三方公告与 `release-manifest.sha256`。
4. 制作能够检测/安装 .NET Framework 与 VC++ x64 运行库的安装器。
5. 对安装器、`PetGenerator.exe`、`InferenceWorker.exe`、`PetPlayer.exe` 和原生 DLL 签名并加时间戳。
6. 从最终下载渠道再次测试 SmartScreen、杀毒软件、升级和卸载。
