# SO-101 完整流程与阶段门槛

## 总览

```text
硬件身份固定与校准
  -> 无相机主从联动
  -> 双相机预检
  -> 两条 smoke 录制、可视化和回放
  -> 正式数据采集与归档
  -> 合并数据集、静态加载
  -> 服务器 200 step 冒烟
  -> 25k 基线训练与产物归档
  -> 固定条件真机评估
  -> 独立复位复验与失败分析
```

每一箭头都是阻断门槛，不是建议顺序。后续阶段的成功不能证明前一阶段正确。

## 阶段、输入与验收

| 阶段 | 主要输入 | 通过条件 | 失败时的动作 |
|---|---|---|---|
| 设备身份 | 串口、相机、校准文件 | 稳定符号链接存在，主从关节方向正确 | 重新识别设备与校准，不录数据。 |
| smoke 采集 | 两条短 episode | 视频、Parquet、动作与时间同步；episode 0 可安全回放 | 修复相机、串口或录制参数。 |
| 正式数据 | 已定义的任务和场景 | 每条成功、连续、无长静止尾段；元数据连续 | 删除无效条并更新元数据，不混入训练。 |
| 数据合并 | 六组原始数据集 | 90 条、17,778 帧、双图像字段和任务文本可静态读取 | 修正字段、视频或 metadata。 |
| CUDA 冒烟 | 本地模型缓存、合并数据 | GPU 可用、loss 有限、无 OOM、checkpoint 可加载 | 先解决版本和 feature 映射。 |
| 正式训练 | 已通过的冒烟配置 | 日志、环境锁定、checkpoint 与校验记录齐全 | 保留现状，不以最后一步替代选择。 |
| 真机评估 | 指定 checkpoint、固定协议 | 每条可追溯，终态成功标准一致 | 记录失败类型；不从 loss 推断成功率。 |

## 文档导航

| 主题 | 文档 |
|---|---|
| 任务、数据和判据 | [01-task-overview.md](./01-task-overview.md) |
| 实机设备与校准 | [02-hardware-setup.md](./02-hardware-setup.md)、[硬件测试报告](./02-hardware-verification.md) |
| 环境与版本边界 | [03-dev-environment.md](./03-dev-environment.md) |
| 采集与质检 | [05-data-collection.md](./05-data-collection.md)、[采集实操](./05-data-collection-practice.md) |
| 服务器训练 | [06-model-training.md](./06-model-training.md)、[训练准备评估](./06-server-training-readiness.md) |
| 推理和实机证据 | [07-model-inference.md](./07-model-inference.md)、[真实评估总报告](./09-real-world-evaluation.md) |

## 当前状态

服务器训练和首轮六组真实运行已完成；独立随机复位评估、逐阶段失败标注、仿真闭环以及长期部署稳定性仍未完成。下一轮最有价值的工作是对绿色 10k checkpoint 进行独立复位复验，并用相同协议重测紫色任务。
