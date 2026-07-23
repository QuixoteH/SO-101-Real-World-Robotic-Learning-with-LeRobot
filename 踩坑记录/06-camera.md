# 6. 相机、fourcc 与 USB 稳定性

本项目两台相机格式不同：夹爪相机为 MJPG，环境相机为 YUYV。相机字段身份、分辨率、方向和 fourcc 是数据契约的一部分，任何一项变化都应新建数据集或明确记录。

## 已遇到的直接原因

环境相机曾出现 `read failed (status=False)`；内核同时记录 USB Hub 端口被禁用并重枚举。该问题是 USB 断连，不是 Rerun、EGL 或相机旋转问题。解决方向是将相机接回独立电脑 USB 端口，先做持续读帧测试，再录制。

## 运行前检查

```bash
ls -l /dev/videoSO101_ENV /dev/videoSO101_GRIPPER
v4l2-ctl --list-formats-ext -d /dev/videoSO101_ENV
v4l2-ctl --list-formats-ext -d /dev/videoSO101_GRIPPER
```

录制前用 30 至 60 秒带视觉遥操作检查图像不黑屏、不冻结、身份不反转。仅能打开设备不等于连续采集稳定。
