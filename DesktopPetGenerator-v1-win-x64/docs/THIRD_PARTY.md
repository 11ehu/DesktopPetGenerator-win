# 第三方组件与许可

- BiRefNet：[官方实现](https://github.com/ZhengPeng7/BiRefNet)，MIT；[ONNX 转换模型](https://huggingface.co/onnx-community/BiRefNet-portrait-ONNX)。版本和哈希见 models/model.json，许可文本见 LICENSE-BiRefNet.txt。
- YOLOv8n-Pose：[Ultralytics](https://github.com/ultralytics/ultralytics)，[姿态格式说明](https://docs.ultralytics.com/tasks/pose/)。
- 部署权重：[Xenova/yolov8-pose-onnx](https://huggingface.co/Xenova/yolov8-pose-onnx)，仓库标注 AGPL-3.0；固定版本与哈希见 models/pose-model.json。完整生成器分发和商业集成应核对该授权，不应假设允许无条件闭源商用。
- SCHP Pascal Person-Part：原始 Self-Correction for Human Parsing 与 ONNX 模型仓库均标注 MIT；固定提交、INT8 模型哈希、输入和七类输出见 models/parser-model.json，许可文本见 LICENSE-SCHP.txt。只加载固定 ONNX，不执行仓库里的 Python 远程代码。
- ONNX Runtime：Microsoft 官方 1.23.2 C# API 与 Windows x64 原生 CPU DLL，MIT；许可证与第三方声明保存在 bin/inference/licenses/。推理引擎内部是原生代码，不是重新用 C# 编写神经网络算子。
- .NET 配套库：System.Memory 4.5.5、System.Buffers 4.5.1、System.Numerics.Vectors 4.5.0、System.Runtime.CompilerServices.Unsafe 4.5.3；许可证在 bin/inference/licenses/，版本和文件哈希见 bin/inference/dependencies.json。
- Python、NumPy、Pillow 及其安装器已移除，不再作为项目运行或开发依赖。

导出播放器只依赖 .NET Framework，不包含 Python、推理代码和模型权重。照片与人物肖像的使用授权由素材提供者负责。
