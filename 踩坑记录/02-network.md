# 2. 网络与离线边界

模型、tokenizer 和数据训练阶段禁止重新从 Hugging Face 下载。训练使用：

```bash
export HF_HUB_OFFLINE=1
export TRANSFORMERS_OFFLINE=1
export HF_HUB_CACHE="$SO101_BUNDLE_ROOT/models/hf_cache"
```

环境安装时，官方 PyTorch CUDA wheel 约八分钟只下载 17,563,648 bytes，因速度不足停止。使用已配置的清华 PyPI 镜像重试后，最终运行时仍验证为 `torch 2.7.1+cu126`，且 CUDA 运算成功。该切换只发生在依赖安装阶段，不发生在模型或数据加载阶段。
