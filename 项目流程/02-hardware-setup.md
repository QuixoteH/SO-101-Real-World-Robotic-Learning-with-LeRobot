# 2. 硬件连接、校准与安全检查

## 推荐顺序

先确认从臂、主臂和校准，再进行无相机遥操作，最后接入双相机。相机故障不应干扰机械臂联动的定位。

## 设备身份

本机记录使用稳定设备名：`/dev/ttySO101_LEADER`、`/dev/ttySO101_FOLLOWER`、`/dev/videoSO101_ENV`、`/dev/videoSO101_GRIPPER`。不要把 `/dev/ttyACM0` 或 `/dev/video2` 当成长期身份；重插 USB 后编号可变化。

每次启动前在采集环境执行：

```bash
ls -l /dev/ttySO101_LEADER /dev/ttySO101_FOLLOWER
ls -l /dev/videoSO101_ENV /dev/videoSO101_GRIPPER
ls -l /dev/serial/by-id/
```

再确认 leader/follower 校准文件存在。端口、ID 或校准任一项不匹配时，不启动录制或策略控制。

## 校准与安全

校准后先以低速、短行程验证每个关节方向和夹爪开合。回放与策略推理会真实驱动从臂；执行前移走障碍物，固定相机与线缆，并确保操作者能立即断电。

完整的设备序列号、udev 规则、相机格式与已验证命令见 [硬件测试报告](./02-hardware-verification.md)。
