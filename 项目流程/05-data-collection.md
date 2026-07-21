# 5. 数据准备与验证

服务器没有采集新数据。主训练数据为 `smolvla90_2color_v1_v30`：90 episodes、17,778 frames、180 个视频、20 FPS、两个任务。

用 LeRobot 0.5.1 与 `video_backend=pyav` 读取了每个 episode 的首尾样本，共 180 样本和 360 路图像。`front` 图像 shape 为 `[3,640,480]`，`top` 为 `[3,480,640]`；state/action 都是 6 维。所有 index、episode、frame、task 和有限值检查通过。
