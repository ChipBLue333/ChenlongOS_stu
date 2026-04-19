# TPU-MLIR 基本使用指南

本项目记录了学习 [tpu-mlir](https://github.com/sophgo/tpu-mlir) 并完成 `dog.jpg` 目标检测任务的过程。

## 任务目标
跑通 `tpu-mlir` 官方示例中的 `dog.jpg` 识别，并获取带有检测框的结果图片。

---

## 步骤 1：环境准备 (Docker 推荐)

由于 `tpu-mlir` 依赖较多库文件，官方强烈建议使用 Docker 环境。

1. **拉取官方 Docker 镜像**：
   ```bash
   docker pull sophgo/tpumlir:latest
   ```

2. **启动并进入容器**：
   确保你已经在 `TPU-MLIR` 目录或源码目录下，将其挂载到容器内：
   ```bash
   # 假设你在 TPU-MLIR 源码根目录
   docker run -v $PWD:/workspace -it sophgo/tpumlir:latest bash
   ```

## 步骤 2：初始化环境变量

进入容器后，在 `tpu-mlir` 的根目录下执行脚本来配置路径：
```bash
cd /workspace
source ./envsetup.sh
```

## 步骤 3：准备工作目录与模型

我们将以 Yolov5s 为例进行演示。

1. **创建工作路径**：
   ```bash
   mkdir -p model_yolov5s && cd model_yolov5s
   ```

2. **下载或准备模型文件**：
   你需要一个导出为 ONNX 格式的 Yolov5 模型。
   ```bash
   # 如果在官方示例路径下，通常在 regression/model/yolov5s.onnx
   cp ../regression/model/yolov5s.onnx .
   cp ../regression/image/dog.jpg .
   ```

## 步骤 4：模型转换 (ONNX -> MLIR)

将 ONNX 模型转换为 TPU-MLIR 的中间表达格式：
```bash
model_transform.py \
    --model_name yolov5s \
    --model_def yolov5s.onnx \
    --input_shapes [[1,3,640,640]] \
    --mean 0.0,0.0,0.0 \
    --scale 0.0039216,0.0039216,0.0039216 \
    --keep_aspect_ratio \
    --pixel_format rgb \
    --output_names output \
    --mlir yolov5s.mlir
```

## 步骤 5：生成测试结果 (量化前验证)

在生成最终的硬件模型前，我们可以先用 `model_deploy.py` 在 CPU 上模拟运行模型，并处理图片：

```bash
python3 -m tpu_mlir.tools.model_deploy \
    --mlir yolov5s.mlir \
    --quantize F32 \
    --chip bm1684x \
    --test_input dog.jpg \
    --test_result yolov5s_f32_output.jpg \
    --model yolov5s_bm1684x_f32.bmodel
```

*注意：`tpu_mlir.tools.model_deploy` 脚本会自动根据模型类型（如 Yolov5）在检测任务后绘制框图。*

## 步骤 6：确认结果

1. 在当前目录下查找生成的图片文件（如 `yolov5s_f32_output.jpg` 或类似命名的文件）。
2. 将该图片保存并提交至考核目录。

---

## 考核记录
* [运行结果图片](./result_dog.jpg) (待上传)
