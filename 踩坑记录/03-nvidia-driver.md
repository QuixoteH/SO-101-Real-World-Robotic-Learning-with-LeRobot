# 3. NVIDIA 驱动与 CUDA

实际 GPU 是 RTX 4090 24GB，驱动 550.144.03，`nvidia-smi` 报告 CUDA 12.4。训练环境中的 PyTorch 是 `2.7.1+cu126`。

不要仅凭 NVIDIA-SMI 的 CUDA 文本推断 PyTorch wheel 不可用。本次实际导入后 `torch.cuda.is_available()` 为真，并完成 CUDA 矩阵运算；后续 25,000-step 训练无 OOM。
