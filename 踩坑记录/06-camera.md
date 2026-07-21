# 6. 相机与视频解码

训练数据有两路视频键：`observation.images.front` 与 `observation.images.top`。预训练策略期望 `camera1` 与 `camera2`，因此训练始终传入：

```json
{"observation.images.front":"observation.images.camera1","observation.images.top":"observation.images.camera2"}
```

v3.0 数据的 AV1 视频必须用 PyAV。TorchCodec 曾不能可靠识别该拼接视频流，因此不能把训练后端改回默认 TorchCodec。
