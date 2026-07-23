# SO-101 真机机器人学习：SmolVLA 离线训练与实机评估记录

> 本仓库记录一个已经完成数据采集、服务器离线训练和首轮真机评估的 SO-101 项目。它不是通用安装手册，也不承诺机器人已稳定部署；所有结论均区分为“已验证”“可复现步骤”和“仍需验证”。

项目目标是根据语言指令，让 SO-101 从臂将指定颜色的小方块放入白盘。策略为在 `lerobot/smolvla_base` 上进行行为克隆微调，输入双相机图像、机器人状态和任务文本，输出连续动作。

## 已验证结论

| 项目 | 已验证事实 |
|---|---|
| 真实数据 | 90 条 episode、17,778 帧、两种颜色、三种光照条件、双相机、20 Hz。 |
| 服务器训练 | LeRobot 0.5.1、RTX 4090 24 GB；200 step CUDA 冒烟和 25,000 step 基线训练完成。 |
| 训练产物 | 2k 至 24k 每 2k 一个 checkpoint，外加最终 25k，共 13 份。 |
| 真机评估 | 绿色任务的 10k checkpoint 在连续 10 条演示中为 9/10；紫色 10k 与 20k 均为 5/10。 |
| 不可声称 | 现有 9/10 是连续展示，不是独立随机复位实验；不能据此宣称已可稳定部署或已完成 sim-to-real。 |

## 项目边界

- 语言任务为 `put small green block in plate` 与 `put small purple block in plate`。
- 训练数据来自真机专家遥操作；训练损失、checkpoint 存在和视频落盘不能替代真实任务成功率。
- 当前仓库只保存文档、命令、元数据与校验信息，不包含原始数据、模型权重或实验压缩包。
- 所有绝对路径均是本项目历史记录。复现前必须替换为自己的设备路径、校准 ID、数据集路径与 checkpoint 路径。

## 阅读路线

1. 从 [项目流程/workflow.md](./项目流程/workflow.md) 获取完整阶段和准入门槛。
2. 需要搭建或复核硬件时，阅读 [硬件测试报告](./项目流程/02-hardware-verification.md)；先验证主从臂，再接入相机。
3. 需要采集或检查数据时，按 [数据采集实操与质检](./项目流程/05-data-collection-practice.md) 执行，不跳过 smoke 数据集。
4. 需要复现服务器训练时，先读 [服务器训练准备评估](./项目流程/06-server-training-readiness.md)，再读 [模型训练](./项目流程/06-model-training.md) 和 [训练踩坑](./踩坑记录/09-model-training.md)。
5. 需要判断模型效果时，以 [真实评估总报告](./项目流程/09-real-world-evaluation.md) 为准，并遵守其中的证据边界。

## 目录

```text
.
├── ENVIRONMENT_REPORT_2026-07-07.md  # 服务器环境的原始核验摘要
├── 项目流程/
│   ├── workflow.md                    # 全流程、输入输出与阶段门槛
│   ├── 01-task-overview.md            # 任务、数据与成功判据
│   ├── 02-hardware-setup.md           # 硬件连接和校准流程
│   ├── 02-hardware-verification.md    # 本机硬件测试实录
│   ├── 03-dev-environment.md          # 两套环境的职责与版本边界
│   ├── 04-planning.md                 # 数据和评估设计
│   ├── 05-data-collection.md          # 采集阶段总览
│   ├── 05-data-collection-practice.md # 录制、回放与质量门槛
│   ├── 06-model-training.md           # 服务器训练的实际命令与检查
│   ├── 06-server-training-readiness.md# 训练前资源与兼容性评估
│   ├── 07-model-inference.md          # 安全推理流程
│   ├── 07-inference-and-evaluation-commands.md # 已执行的推理命令记录
│   ├── 08-summary.md                  # 当前结论与下一轮最小工作
│   └── 09-real-world-evaluation.md    # 六组真实评估的汇总证据
└── 踩坑记录/                           # 按故障域组织的可复用排查记录
```

## 数据与训练概览

| 维度 | 实际配置 |
|---|---|
| 目标物 | 绿色、葡萄紫小方块；同一白盘。 |
| 条件 | 每种颜色包含标准光、部分拉窗帘、强窗边侧逆光三组。 |
| 相机 | `camera1` 为夹爪视角，`camera2` 为环境视角；采集原始名称可为 `front`/`top`，训练侧必须显式映射。 |
| 数据版本 | 采集端 LeRobot 0.3.4；服务器训练端 LeRobot 0.5.1。两者不能混用命令。 |
| 基线 | batch size 8、AMP、未启用图像增强；25,000 step。 |

## 最小复现原则

复现应按“设备安全 -> 数据静态检查 -> 200 step 冒烟 -> checkpoint 加载 -> 固定初始位真机评估”的顺序进行。任何一层失败，都应停在该层排查，不能用更长训练或更多数据掩盖接口错误。

## 参考与致谢

文档结构参考 [JereoZero/so101-real](https://github.com/JereoZero/so101-real)。本仓库只使用本项目已记录的硬件、数据、训练和评估证据；上游教程中的硬件配置、数据量和超参数不能直接套用。
