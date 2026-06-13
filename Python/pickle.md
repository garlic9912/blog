# 核心概念
pickle 是 Python 的**对象序列化协议**
- 把任意 Python 对象转成字节流（序列化）
- 从字节流还原回对象（反序列化）。
```
Python 对象  ──pickle.dump──▶  字节流（可存文件/传网络）
字节流       ──pickle.load──▶  Python 对象（完全还原）
```

# 基本用法
## 存到文件 / 从文件读
```python
import pickle

data = {"model": "resnet", "weights": [0.1, 0.2, 0.3], "epoch": 42}

# 序列化
with open("checkpoint.pkl", "wb") as f:   # 注意是 "wb"，二进制写
    pickle.dump(data, f)

# 反序列化
with open("checkpoint.pkl", "rb") as f:   # "rb"，二进制读
    loaded = pickle.load(f)

print(loaded)  # {'model': 'resnet', 'weights': [0.1, 0.2, 0.3], 'epoch': 42}
```
- 注意是`wb`, `rb`, 二进制写和二进制读
- 不要反序列化来路不明的数据

## 为什么用 with
其实 pickle **不需要必须用 `with`**
用 `with` 只是因为涉及文件操作，`with` 管的是**文件句柄**，跟 pickle 本身没关系
```python
with open("model.pkl", "wb") as f:
    pickle.dump(model, f)
#  ↑ with 在这里保证 f 用完自动关闭
```
- 涉及文件操作, 使用`with`是一个好习惯