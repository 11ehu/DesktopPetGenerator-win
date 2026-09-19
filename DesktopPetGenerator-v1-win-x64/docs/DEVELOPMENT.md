# 开发说明

Windows 10/11 x64，.NET Framework 4.8+，C# 5 兼容。使用系统 Framework 编译器，不需要 .NET SDK。ONNX Runtime 依赖 Microsoft Visual C++ 2015–2022 x64 运行库；不需要 Python。

## 构建与测试

在项目目录执行：

```powershell
.\tools\setup.ps1
.\tools\build.ps1
.\tools\test.ps1
```

准备脚本按固定版本下载 C# 托管依赖与官方 ONNX Runtime 原生 DLL，并按 bin/inference/dependencies.json 校验。随后编译三个入口并校验 BiRefNet、YOLO Pose、SCHP 三个 ONNX 模型。已携带依赖和模型时全程离线；缺文件时需联网下载。不会安装 pip 或解释器。

## 后端命令

```powershell
.\bin\inference\InferenceWorker.exe health
.\bin\inference\InferenceWorker.exe prepare
.\bin\inference\InferenceWorker.exe cutout --input "照片.jpg" --output "work\demo\photo.png" --rig
.\bin\inference\InferenceWorker.exe rig --input "work\demo\photo.png" --conservative
.\bin\inference\InferenceWorker.exe rig --input "work\demo\photo.png" --joints "work\demo\corrected.json"
```

输出 UTF-8 JSON 行：progress 含 percent/message，error 附非零进程退出码，result 表示阶段产物。UI 同时检查进程成功与完整 rig.json，不把抠图阶段成功当成骨架生成成功。取消只停止当前任务进程树。

## 开发验收入口

- PetGenerator.exe --pipeline-test 照片路径：真实模型→GUI→预览→导出→播放器→关闭，写 work/pipeline-test.txt。
- PetGenerator.exe --ui-smoke 原照片路径 抠图路径：界面截图 work/generator-preview.png。
- PetGenerator.exe --cancel-test 照片路径：取消任务，写 work/cancel-test.txt。
- PetGenerator.exe --export 抠图路径 名字：携带同名 .rig 目录导出，写 work/last-export.txt。
- 导出包的 Pet.exe --smoke-test：约 70 帧后自动退出并写 smoke-pass.txt。

单元测试使用模拟模型输出与合成照片，验证算法不代表真实模型准确率。真实照片效果需另外检查。修改后重新 build 并重新导出；旧 outputs 里的 Pet.exe 不会自动更新。

下一步应优先完善复杂遮挡的部件分割和图层编辑，不以大幅旋转掩盖单照片缺少隐藏身体的限制。

## 模块边界

UI 只通过 UTF-8 JSON 行协议调用独立 C# 推理进程，不直接加载大型模型；取消会终止当前工作进程并释放内存。Inference 只依赖 Shared 数据契约和 ONNX Runtime，不依赖 UI 或动画渲染。Player 只依赖 Shared + Animation，不包含推理 DLL 或模型。各任务使用独立 work/jobs 目录，源照片不覆盖。
