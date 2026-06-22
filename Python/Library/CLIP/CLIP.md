# 定义
CLIP（全称 Contrastive Language-Image Pre-training，对比语言-图像预训练）。其核心目标是让模型学会图像和文本的语义对应关系。模型采用双塔架构，包含一个图像编码器和一个文本编码器. **CLIP 通过对比学习将图像和文本映射到同一特征空间，实现了强大的零样本识别能力。**

| 组件    | 架构                                          | 输入                  | 输出维度       |
| ----- | ------------------------------------------- | ------------------- | ---------- |
| 图像编码器 | ResNet（50/101/152）或 ViT（Vision Transformer） | 224×224 像素的 RGB 图像  | 1024 维特征向量 |
| 文本编码器 | 基于 Transformer                              | 最多 77 个 token 的文本序列 | 1024 维特征向量 |
# sentence_transformers 使用方法
## 加载模型
```python
from sentence_transformers import SentenceTransformer

# 加载一个多模态模型
model = SentenceTransformer("clip-ViT-B-32")
```
只要用 `SentenceTransformer("模型名")` 即可, 它会自动去网上下载，下次再用就直接从本地加载。也可以指定用 CPU 还是 GPU 来跑，不写的话它会自己选最快的

| 模型类型      | 示例模型               | 输入支持    | 输出用途            |
| --------- | ------------------ | ------- | --------------- |
| 纯文本模型     | `all-MiniLM-L6-v2` | 仅文本     | 文本语义相似度、聚类、检索   |
| 多模态模型（图文） | `clip-ViT-B-32`    | 文本 + 图像 | 图文匹配、图像搜索、零样本分类 |
如果只做文本任务（如句子相似度、QA 检索），加载图文模型是浪费资源且效果不一定好；如果任务涉及图像，必须使用多模态模型。

## 编码数据
用 `encode` 方法把数据变成向量
```python
from PIL import Image

# 1. 编码文本
sentences = ["今天天气真好。", "She sells sea shells."]
text_embeddings = model.encode(sentences)

# 2. 编码图像 (用 CLIP 模型)
img_model = SentenceTransformer("clip-ViT-B-32")
img = Image.open("my_cat.jpg")
image_embedding = img_model.encode(img)
```
图片和文本由`CLIP`处理后, 会落入同一个特征空间, 关联性强的图文会更接近