# 概念
`cn_clip` 是 Chinese-CLIP 项目的核心库，由智源研究院开源。它是 `OpenAI CLIP` 的中文版本，使用大规模中文图文对（约 2 亿对）进行预训练，实现了**中文场景下的图文跨模态理解**。
- **核心功能**：给定中文文本和图像，计算它们的相似度，或将图文映射到同一语义向量空间。

# 使用方法
## 导入依赖
```python
from transformers import ChineseCLIPModel, ChineseCLIPProcessor
from PIL import Image
import torch
```
- `ChineseCLIPModel`：CLIP 模型类，包含图像编码器和文本编码器
- `ChineseCLIPProcessor`：处理器，负责图像的预处理（缩放、归一化）和文本的 `tokenize`
- `torch`：用于推理时的梯度关闭和张量操作

## 加载预训练模型与处理器
```python
model_dir = "../clip-model"                      # 本地模型路径
model = ChineseCLIPModel.from_pretrained(model_dir)
processor = ChineseCLIPProcessor.from_pretrained(model_dir)
```
- `from_pretrained` 可接受本地目录或 Hugging Face 模型名称（如 `"OFA-Sys/chinese-clip-vit-base-patch16"`）

## 处理图像特征向量
```python
image = Image.open("../image/image_00.png")      # 加载图片
inputs = processor(images=image, return_tensors="pt")

with torch.no_grad():
    embedding = model.get_image_features(**inputs).pooler_output.numpy().flatten()
```
- `model.get_image_features(**inputs)`：输入处理器返回的字典（包含 `pixel_values` 等），输出图像特征
- `.pooler_output`：取池化后的特征向量（通常为 `[batch_size, feature_dim]`）
- `.numpy().flatten()`：转为 `NumPy` 一维数组，便于后续存储或检索

## 处理文本特征向量
```python
# text 需要的是二维 numpy 数组
inputs = processor(text=[search_text], return_tensors="pt", padding=True) 
with torch.no_grad(): 
 text_embedding = model.get_text_features(**inputs).pooler_output.numpy().flatten()
```

## 完整实例
```python
from transformers import ChineseCLIPModel, ChineseCLIPProcessor
from PIL import Image
import torch

# 加载模型
model = ChineseCLIPModel.from_pretrained("OFA-Sys/chinese-clip-vit-base-patch16")
processor = ChineseCLIPProcessor.from_pretrained("OFA-Sys/chinese-clip-vit-base-patch16")

# 提取特征向量
def get_image_embedding(image_path):
    image = Image.open(image_path)
    inputs = processor(images=image, return_tensors="pt")
    with torch.no_grad():
        embedding = model.get_image_features(**inputs).pooler_output.numpy().flatten()
    return embedding

# 向量化
emb = get_image_embedding("test.png")
print(emb.shape)
```