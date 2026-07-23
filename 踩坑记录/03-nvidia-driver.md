# 3. NVIDIA 驱动与 CUDA

不要仅凭 `nvidia-smi` 成功就开始正式训练。它只能证明驱动与 GPU 管理接口可用，不能证明当前 Python 环境中的 PyTorch 可调用 CUDA。

```bash
nvidia-smi
python - <<'PY'
import torch
print(torch.__version__)
print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'no cuda')
PY
```

服务器环境的验证值为 PyTorch `2.7.1+cu126` 和 RTX 4090 24 GB。本机 RTX 3060 Laptop 只有 6 GB，不能默认承担 SmolVLA 正式训练；本机职责是设备验证、固定命令推理和结果检查。
