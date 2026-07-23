# SO-101 硬件测试报告

## 1. 测试目标

- 验证 `JereoZero/so101-real` 实机路线的硬件链路是否打通
- 固定主臂、从臂、环境相机、夹爪相机的设备身份
- 解决重复插拔后的权限和设备名变化问题
- 完成不带视觉和带视觉的主从联动测试

## 2. 测试环境

- 测试日期：2026-07-15
- 操作系统：Ubuntu
- 项目目录：`/home/quixoteh/coding/so101-real`
- LeRobot 目录：`/home/quixoteh/coding/so101-real/lerobot`
- Conda 环境：`/home/quixoteh/anaconda3/envs/lerobot`
- 参考硬件文档：`SO-ARM101开源6轴机械臂使用文档.pdf`

## 3. 设备识别结果

### 3.1 机械臂串口

本次按“先插从臂，再插主臂”完成首次身份绑定，识别结果如下：

- 从臂序列号：`5C82111087`
- 主臂序列号：`5C82108763`

已通过 udev 固定为：

- `/dev/ttySO101_FOLLOWER`
- `/dev/ttySO101_LEADER`

对应规则文件：

- `/etc/udev/rules.d/99-so101.rules`

规则内容：

```udev
SUBSYSTEM=="tty", ENV{ID_SERIAL_SHORT}=="5C82111087", SYMLINK+="ttySO101_FOLLOWER", GROUP="dialout", MODE="0666"
SUBSYSTEM=="tty", ENV{ID_SERIAL_SHORT}=="5C82108763", SYMLINK+="ttySO101_LEADER", GROUP="dialout", MODE="0666"
```

### 3.2 相机身份

先插环境相机，再插夹爪相机，识别结果如下：

- 环境相机：`/dev/videoSO101_ENV -> /dev/video2`
- 夹爪相机：`/dev/videoSO101_GRIPPER -> /dev/video4`

对应规则文件：

- `/etc/udev/rules.d/99-so101-cameras.rules`

规则内容：

```udev
SUBSYSTEM=="video4linux", ENV{ID_SERIAL}=="icSpring_icspring_camera", ATTR{index}=="0", SYMLINK+="videoSO101_ENV"
SUBSYSTEM=="video4linux", ENV{ID_SERIAL}=="icSpring_icspring_camera_202404160005", ATTR{index}=="0", SYMLINK+="videoSO101_GRIPPER"
```

## 4. 测试流程

### 4.1 机械臂身份固定与权限处理

执行思路：

1. 通过 `ls -l /dev/serial/by-id/` 记录主从臂序列号
2. 写入 udev 规则，固定为主臂/从臂别名
3. 处理串口权限，避免每次插拔后手动授权

验证结果：

- `/dev/ttySO101_FOLLOWER` 可稳定指向从臂
- `/dev/ttySO101_LEADER` 可稳定指向主臂
- Python 可直接打开两个设备，不再出现 `Permission denied`

### 4.2 相机身份固定与权限处理

执行思路：

1. 先插环境相机，再插夹爪相机，完成第一次身份绑定
2. 通过 `/dev/v4l/by-id/` 和 `udevadm info` 确认唯一身份
3. 写入 udev 规则，固定为环境相机和夹爪相机别名
4. 将用户加入 `video` 组，避免重复授权

验证结果：

- `/dev/videoSO101_ENV` 和 `/dev/videoSO101_GRIPPER` 均可直接打开
- 重复插拔后可通过固定别名直接识别身份

### 4.3 不带视觉主从联动测试

测试目标：

- 验证主臂转动时，从臂能否稳定跟随
- 验证夹爪开合
- 验证连续运行时是否掉线

测试结果：

- 主臂转动时，从臂能跟随
- 各关节方向基本一致
- 无明显反向、突跳、抽搐、乱转
- 夹爪能开合
- 连续运行 3 到 5 分钟未掉线

结论：不带视觉联动测试通过。

### 4.4 主从臂校准

在首次带视觉联动前，程序自动检测到缺少或不匹配的校准文件，因此分别对主臂和从臂执行了校准。

校准文件保存路径：

- 主臂：`/home/quixoteh/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/my_leader.json`
- 从臂：`/home/quixoteh/.cache/huggingface/lerobot/calibration/robots/so101_follower/my_follower.json`

结论：主臂、从臂校准均成功完成。

### 4.5 带视觉联动测试

先检查相机支持格式：

- 环境相机 `/dev/videoSO101_ENV`：仅支持 `YUYV`
- 夹爪相机 `/dev/videoSO101_GRIPPER`：支持 `MJPG` 和 `YUYV`

最终可用命令：

```bash
python -m lerobot.teleoperate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --robot.cameras="{ front: {type: opencv, index_or_path: \"/dev/videoSO101_GRIPPER\", width: 640, height: 480, fps: 30, fourcc: MJPG}, top: {type: opencv, index_or_path: \"/dev/videoSO101_ENV\", width: 640, height: 480, fps: 30, fourcc: YUYV} }" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttySO101_LEADER \
  --teleop.id=my_leader \
  --display_data=true
```

测试现象：

- 主臂连接成功
- 从臂连接成功
- 夹爪相机连接成功
- 环境相机连接成功
- Rerun 窗口可正常显示两路图像
- 主从协同正常
- 图像清晰

结论：带视觉联动测试通过。

## 5. 遇到的问题与解决办法

### 问题 1：机械臂设备名不稳定

现象：

- 直接依赖 `/dev/ttyACM0`、`/dev/ttyACM1` 无法保证重插后身份不变

解决：

- 通过 `/dev/serial/by-id/` 获取唯一序列号
- 使用 udev 规则固定为 `/dev/ttySO101_FOLLOWER` 和 `/dev/ttySO101_LEADER`

### 问题 2：串口权限不足，出现 `Permission denied`

现象：

- Python 无法直接打开固定后的串口

解决：

- 在 udev 规则中补充 `GROUP="dialout", MODE="0666"`
- 重新加载规则后，串口设备可直接访问

### 问题 3：带视觉联动时环境相机读取失败

现象：

- 首次带视觉联动时，夹爪相机能连接，环境相机报：

```text
RuntimeError: OpenCVCamera(/dev/videoSO101_ENV) read failed (status=False)
```

原因：

- 环境相机只支持 `YUYV`
- 夹爪相机支持 `MJPG`/`YUYV`
- 默认配置无法正确匹配两台相机的格式

解决：

- 使用 `v4l2-ctl --list-formats-ext` 分别确认格式
- 最终确定：
  - `front`（夹爪相机）使用 `MJPG`
  - `top`（环境相机）使用 `YUYV`

### 问题 4：项目文档写了 `fourcc`，但本地代码不支持

现象：

- 直接按 `so101-real` 项目文档传 `fourcc` 参数时，报错：

```text
DecodingError: `robot.cameras.front`: The fields `fourcc` are not valid for OpenCVCameraConfig
```

原因：

- 当前本地 `lerobot` 代码与 `so101-real` 文档不一致
- 文档使用了 `fourcc`，但本地 `OpenCVCameraConfig` 原始实现中没有该字段

解决：

- 本地补充 `fourcc` 字段到 `OpenCVCameraConfig`
- 在 `camera_opencv.py` 中补充 `cv2.CAP_PROP_FOURCC` 设置逻辑
- 修复后，文档中的命令可以正常执行

## 6. 最终测试结果

### 已通过项目

- 主臂 / 从臂固定识别
- 主臂 / 从臂免重复授权
- 环境相机 / 夹爪相机固定识别
- 相机免重复授权
- 不带视觉主从联动测试
- 主臂校准
- 从臂校准
- 带视觉主从联动测试

### 当前可复用的稳定设备名

- 主臂：`/dev/ttySO101_LEADER`
- 从臂：`/dev/ttySO101_FOLLOWER`
- 环境相机：`/dev/videoSO101_ENV`
- 夹爪相机：`/dev/videoSO101_GRIPPER`

## 7. 下一步建议

建议进入下一阶段：

1. 进行最小数据采集测试（先录 1 到 2 条短 episode）
2. 验证数据能否正常落盘
3. 再进入正式批量数据采集
