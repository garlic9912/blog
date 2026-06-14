# 核心概念
`Faiss` 做一件事：**给定一个查询向量，从海量向量中快速找出最相似的 k 个。**
```
向量库: [v1, v2, v3, ... v1000000]
查询:   q
输出:   距离 q 最近的 k 个向量的索引和距离
```

# 索引类型
## 精确搜索
- `IndexFlatL2` / `IndexFlatIP`
- 暴力扫描全部向量，结果100%准确。数据量小（`<100`万）首选
```python
index = faiss.IndexFlatL2(d)   # L2 欧氏距离，精确但慢
index = faiss.IndexFlatIP(d)   # 内积/余弦相似度，精确但慢
```
## 分桶近似搜索
- `IndexIVFFlat`
- 适合`100 ~ 1000`万数据量使用
```python
nlist = 100   # 分成多少个桶
quantizer = faiss.IndexFlatL2(d)
index = faiss.IndexIVFFlat(quantizer, d, nlist)

index.train(vectors)   # IVF 系列需要先训练
index.add(vectors)

index.nprobe = 10      # 查询时搜索多少个桶，越大越准但越慢
D, I = index.search(query, k=5)
```

# 基本操作
## 最简单的流程
```python
import faiss
import numpy as np

d = 128          # 向量维度
n = 10000        # 向量数量

# 造一批假数据
vectors = np.random.random((n, d)).astype("float32")  # faiss 只接受 float32
query_text   = np.random.random((1, d)).astype("float32")  # 查询也要是二维

# 建索引
index = faiss.IndexFlatL2(d)   # 最简单的 L2 距离索引
index.add(vectors)             # 添加向量

# 查询
D, I = index.search(query_text, k=5)   # D=距离, I=索引编号
print(I)   # [[3721, 88, 4412, 991, 7023]]
print(D)   # [[0.23, 0.31, 0.35, 0.38, 0.41]]
```
- `faiss` 只接受 `float32`
- `D, I` 中的元素顺序是: 越相似越靠前
- `FAISS` 的  `index.search(embedding, k)` 意思是"找最相似的 k 个" 但如果索引里存的图片数量不足 k 个，FAISS 凑不够 k 个结果，它不会报错，而是用  -1  来填充空位：
```python
    I[0] = [0, 1, -1]   ← 第三个位置凑不够，填 -1
    D[0] = [0.12, 0.34, inf]
```

## 保存和加载
```python
faiss.write_index(index, "my.index")
index = faiss.read_index("my.index")
```

## 易错点
```python
# ❌ 忘记转 float32
vectors = np.random.random((n, d))          # float64，会报错
vectors = vectors.astype("float32")         # ✅

# ❌ 查询向量是一维的
query = np.random.random(d)                 # shape (128,)，会报错
query = np.array([query])                   # ✅ shape (1, 128)

# ❌ IVF 系列忘了 train
index = faiss.IndexIVFFlat(quantizer, d, 100)
index.add(vectors)                          # 报错：index not trained
index.train(vectors)                        # ✅ add 之前先 train
index.add(vectors)

# ❌ nprobe 忘了设，默认是 1，召回率很低
index.nprobe = 10                           # ✅ 查询前设置
```