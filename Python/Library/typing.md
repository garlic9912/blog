`typing` 是 Python 标准库中用于**类型提示（Type Hints）** 的核心模块。它允许开发者为变量、函数参数和返回值标注预期的类型，从而提高代码的可读性、可维护性，并支持静态类型检查工具（如 `mypy`）进行早期错误发现。

# 基本用法
类型提示本质上是**注释**，不会影响运行时行为

## 变量注解
```python
name: str = "Alice"
age: int = 30
scores: list[int] = [95, 87, 76]   # Python 3.9+ 推荐写法
```

## 函数注解
```python
def greet(name: str) -> str:
    return f"Hello, {name}"

def add(a: int, b: int) -> int:
    return a + b
```

# 常用类型

除去基本类型之外, 还有一下的类型

| 类型    | 写法                                   | 说明             |
| ----- | ------------------------------------ | -------------- |
| 列表    | `list[int]`                          | 整数列表           |
| 字典    | `dict[str, int]`                     | 键为字符串，值为整数     |
| 元组    | `tuple[int, str]`                    | 固定长度元组（整数和字符串） |
| 集合    | `set[int]`                           | 整数集合           |
| 可迭代对象 | `collections.abc.Iterable[int]`（需导入） | 支持迭代的整数序列      |
| 序列    | `collections.abc.Sequence[int]`      | 有长度且可下标访问      |
