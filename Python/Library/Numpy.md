`NumPy` 是 `Python` 科学计算的核心库，提供了高性能的多维数组对象和丰富的数学函数

## 核心概念: ndarray
`ndarray` 是 `NumPy` 的 N 维数组对象，具有以下特点：
- 同质（所有元素类型相同）
- 固定大小（创建后不能动态增长）
- 向量化操作（无需显式循环）
```python
import numpy as np

# 创建一个一维数组
arr1 = np.array([1, 2, 3, 4, 5])
print(arr1)        # [1 2 3 4 5]
print(type(arr1))  # <class 'numpy.ndarray'>
print(arr1.shape)  # (5,)
print(arr1.dtype)  # int64（取决于系统）

# 二维数组
arr2 = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2.shape)  # (2, 3)
print(arr2.ndim)   # 2
print(arr2.size)   # 6（总元素个数）
```
## 创建 np 数组的多种方式
### 从列表/元组中创建
```python
a = np.array([1, 2, 3])
b = np.array([(1, 2), (3, 4)], dtype=float)  # 指定类型
c = np.array([[1,2],[3,4]], dtype=np.float32)
```
### 便捷创建函数
```python
# 快速创建
np.arange(10)               # [0 1 2 3 4 5 6 7 8 9]

# 全零
np.zeros(5)                 # [0. 0. 0. 0. 0.]
np.zeros((2, 3))            # 2x3 全零矩阵

# 全一
np.ones((2, 3))             # 2x3 全一矩阵
np.ones_like(arr2)          # 和 arr2 形状相同的全一数组

# 随机数组
np.random.rand(3, 2)        # 均匀分布 [0,1)，3x2
np.random.randn(3, 2)       # 标准正态分布 N(0,1)
np.random.randint(0, 10, size=(2, 3))  # 随机整数 [0,10)
np.random.random((2, 3))    # 同 rand，但参数为 shape 元组

# 空数组（内容未初始化）
np.empty((2, 2))            # 速度快，值随机
```

## 索引与切片
```python
arr = np.array([[1, 2, 3, 4],
                [5, 6, 7, 8],
                [9,10,11,12]])

# 获取元素
arr[0, 2]        # 3 (第0行第2列)
arr[1][1]        # 6

# 切片 [start:stop:step]
arr[:2, 1:3]     # 前两行，第1-2列 → [[2,3],[6,7]]
arr[1:, :2]      # 从第1行开始，前两列 → [[5,6],[9,10]]
arr[:, ::2]      # 所有行，每隔一列 → [[1,3],[5,7],[9,11]]

# 负索引
arr[-1, -2]      # 最后一行倒数第二列 → 11
```
### 注意事项

| 表达式            | 实际含义                       | 结果            |
| -------------- | -------------------------- | ------------- |
| `arr[:2, 1:3]` | 取前两行，再取每行的第1-2列            | shape (2,2)   |
| `arr[:2][1:3]` | 先取前两行，再取结果中的第1-3行（但只有1行可取） | shape (1, 列数) |
- NumPy 的多维数组索引**必须在一个方括号内用逗号分隔**不同维度，例如 `arr[行切片, 列切片]`
- 链式索引 `arr[...][...]` 是**先对第一个维度切片，返回一个新数组，再对这个新数组的**（也是第一个维度）**再次切片**。它无法同时控制在两个维度上的切片，因为第二次切片已经是在降维后的数组上操作了

## 数组操作
### 1.形状操作
```python
a = np.arange(12)

# 改变形状
a.reshape(3, 4)        # 返回新数组，不改变原数组
a.resize(3, 4)         # 原地修改
a.shape = (3, 4)       # 直接设置 shape

# 展平
a.flatten()            # 返回一维副本

# 转置
b = np.array([[1,2],[3,4]])
b.T                    # 转置 → [[1,3],[2,4]]
b.transpose()          # 同上
```

### 2.拼接与分裂
```python
a = np.array([[1,2], [3,4]])
b = np.array([[5,6], [7,8]])

# 垂直拼接
np.vstack((a, b))      # [[1,2],[3,4],[5,6],[7,8]]
# 水平拼接
np.hstack((a, b))      # [[1,2,5,6],[3,4,7,8]]

# 更通用的拼接
np.concatenate((a, b), axis=0)   # 同 vstack
np.concatenate((a, b), axis=1)   # 同 hstack

# 分裂
arr = np.arange(12).reshape(3,4)
np.split(arr, 3)       # 按行分为3个数组
np.hsplit(arr, 2)      # 按列分为2个 → [ (3,2), (3,2) ]
np.vsplit(arr, 3)      # 按行分为3个 → [ (1,4), (1,4), (1,4) ]
```