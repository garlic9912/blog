# 概念
torch 是 PyTorch 深度学习框架的核心库，主要用于：张量计算：提供类似 NumPy 的多维数组，但支持 GPU 加速。自动微分：自动计算梯度，方便训练神经网络。构建和训练神经网络：提供 `torch.nn` 模块，包含各种层、损失函数、优化器等。
## 对 CLIP 的作用
在 CLIP / Chinese-CLIP 的使用场景中，`torch` 负责：
- 存储模型参数和输入数据（张量）。
- 执行模型的前向推理（如 `model.get_image_features()`）。
- 管理设备（CPU / GPU）和数据传输。

# 基本操作
## 设备管理
```python
import torch

device = "cuda" if torch.cuda.is_available() else "cpu"
```
- 将张量或模型移到设备：`tensor.to(device)`、`model.to(device)`

## 禁用梯度
```python
with torch.no_grad():
    ...
```
`with torch.no_grad():` 是一个上下文管理器，用于**临时禁用梯度计算**. 在 PyTorch 中，默认会记录张量的操作历史，以便后续调用 `.backward()` 计算梯度（用于训练）。但在推理（如 CLIP 提取特征）时，不需要梯度。
- 减少内存占用（不再保存中间计算图）
- 加快推理速度

## 结果转 NumPy
```python
embedding = embedding.cpu().numpy()
```
- `torch` 张量转 `NumPy`，用于后续存储或处理