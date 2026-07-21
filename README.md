# SO-101 SmolVLA 服务器离线训练记录

> 本仓库按 `so101-real` 教程仓库的文档结构整理，但内容只覆盖 2026-07-21 完成的 SO-101 双相机 SmolVLA 服务器离线训练。

本次工作完成了 90-episode v3.0 数据的离线校验、200-step CUDA 冒烟和 25,000-step 基线训练。服务器只保存原始训练产物；没有选择最佳 checkpoint、没有实机推理，也没有声称机器人任务成功。

## 目录结构

```text
so101-real-server-training-2026-07-20/
├── README.md
├── ENVIRONMENT_REPORT_2026-07-07.md
├── 踩坑记录/
│   ├── pitfalls.md
│   ├── 01-ubuntu-system.md ... 11-appendix.md
└── 项目流程/
    ├── workflow.md
    ├── 01-task-overview.md ... 08-summary.md
```

## 本次实验

| 项目 | 实际值 |
|---|---|
| GPU | RTX 4090 24GB |
| 数据 | `smolvla90_2color_v1_v30`，90 episodes，17,778 frames，2 tasks |
| 框架 | LeRobot 0.5.1，Python 3.12.13，PyTorch 2.7.1+cu126 |
| 视频 | PyAV；`front -> camera1`，`top -> camera2` |
| 冒烟 | 200 steps，batch 8，成功 |
| 基线 | 25,000 steps，batch 8，AMP，未启用图像增强 |
| checkpoint | 2k 至 24k，每 2k 一份，另有最终 25k，共 13 份 |

## 原始产物

同级目录中的服务器交接包为 `so101-smolvla-experiment-2026-07-20.tar.zst`，SHA256 为 `255e1b2991443599011a4710f94d4f55e184d5966763e8531d0f8fb58306604d`。

推荐阅读顺序：

1. [项目流程/workflow.md](./项目流程/workflow.md)
2. [项目流程/06-model-training.md](./项目流程/06-model-training.md)
3. [踩坑记录/09-model-training.md](./踩坑记录/09-model-training.md)
4. [踩坑记录/11-appendix.md](./踩坑记录/11-appendix.md)
