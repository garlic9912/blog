# 基本语法
- 注释的多种写法
```python
# 这是一行注释

"""
这是多行注释
"""

'''
这也是多行注释
'''
```

- 声明变量
```python
# 简单数据类型
a = 123
b = 1.2
name = "name"
flag = True
```

```python
# 复杂数据类型
a = [1, 3, 4]  # 列表
b = {1, 3, 4}  # 集合
c = (1, 3, 4)  # 元组
d = {'n':1, 'b':2}  # 字典
```

- 基本运算
  ```python
  # 幂次
  a = 2
  a = a ** 10  # a 的 10 次幂
  
  # 求余
  a = 10
  b = a % 2
  ```

- 输入输出
```python
a = input()
a = input("请输入: ")
```

```python
print("123")

a = 15
print(a)

# 格式化输出
print(f"hello {a}")

# 特殊字符
print("Hello\nWorld")  # "\n"
print("Hello\tWorld")  # "\t"
print("Hello World, end="")  # 消除输出的自动换行
```

- 变量类型转换
```python
a = 15
b = str(a)
c = float(b)
```

- 随机数
  ```python
  import random
  a = random.randint(1, 100)  # [1, 100] 之内的随机数
  b = random.uniform(1, 100)  # [1, 100] 之内的随机小数
  c = random.random()  # [0, 1] 之内的随机小数
  ```

## 字符串
- 字符串基本操作
```python
"""
原字符串一旦创建便不可以修改
"""
s1 = "O"
s2 = "H"

# 字符串拼接
oh = s1 + s2

# 字符串复制
hh = s2 * 2
ohhh = s1 + s2 * 3
```

- 字符串切割
```python
"""
字符串正着从 0 开始编号, 反着从 -1 开始编号
1.str[a:b] 切割字符串下标[a, b)的字符串
2.str[1:] 切割字符串下标 1 及其之后的字符串
3.str[:5] 切割字符串从头到下标 4 的字符
4.str[a:b:2] 前两个是下标, 最后一个是步长
5.str[-5:-1] 从[倒数第五个, 倒数第一个)
6.str[::-1] 把字符串反过来
"""
a = "my name is xxx"
str1 = a[1:5]  # 切割下标 [1, 5) 的字符串
print(str1)  # "y na"

str2 = a[1::2]
print(str2)  # "ynm sxx"
```

- 字符串替换/分割/拼接
```python
# 替换字符串
a = "my name is xxx"
str1 = a.replace("xxx", "king")  # my name is king

# 根据"分隔符"将字符串分割成列表

arr = a.split(" ")  # ["my", "name", "is", "xxx"]

# 将字符串列表按照"拼接符"拼接成字符串

str2 = "-".join(arr)  # "my-name-is-xxx"
```

- 常用函数
```python
# 字符串常用函数
str = "  abcde     "

# 字符串改大写
str1 = str.upper()

# 字符串改小写
str2 = str1.lower()

# 删除两侧的空格
str3 = str.strip()
```

## 分支与循环
- 分支语句 
  ```python
  a = 11
  b = a % 2
  c = b == 0
  if c:
    print("是偶数")
  else:
    print("不是偶数")
  ```

- 循环语句
```python
# while 循环
a = 0
while a < 10:
	a += 1
	print(a)

if a == 5:
	break
	
# for 循环
for i in range(10):  # range(10) 代表 0 到 9 的一个列表
	print("Hello World")

# 等价于
for i in [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]:
	print("Hello World")

# 只要后面可以迭代就可以以作为循环对象
string = "ABCDEFG"
	for i in string:
	print(i)
```

- 循环语句配合 `else`
```python
"""
若循环正常结束则会进入 else 语句, 若中途退出则不会进入 else 语句
"""
a = 0
while a < 10:
    a += 1
else:
    print("end")


arr = range(10)  # [0, 10)
for i in arr:
    if a == 5:
        break  # 从此处退出则不会进入 else
else:
    print("end")
```

## 列表操作
- 基本操作
```python
# 列表可以存放不同类型的元素
a = 12
b = [1, False, "happy", a, [1, 2, 3]]
  
# 下标访问
elm1 = b[1]
elm2 = b[3]

# 下标修改
b[0] = "lalala"

# 引用赋值
c = b  # b, c 指向的同一块内存

# 复制赋值
c = b.copy()  # c 是新的一块内存
```

- 常用函数
```python
a = [1, False, "happy", [1, 2, 3]]

# 末尾插入
a.append("abc")

# 指定插入
a.insert(1, 't')  # 在下标 1 的位置插入 't'

# 指定下标删除
a.pop(1)  # 删除下标 1 位置上的元素

# 指定元素删除
a.remove("happy")

# 清空列表
a.clear()

# 排序列表
a.sort()  # 直接修改 a 进行排序
b = sorted(a)  # 使用 python 内置函数, 返回新的列表
```

## 元组, 集合, 字典
- 元组
  ```python
  tuple = (1, 2, "22")
  """
  1.不可修改
  2.可以迭代访问
  3.元素类型可以不同
  """
  ```

- 集合
```python
set = {1, 3, 3, 4}
print(set)  # {1, 3, 4}
"""
1.无序性, 不可用下标随机访问
2.不重复性, 元素各不相同
"""

# 空集合与空字典
s = set()  # 空集合
d = {}  # 空字典
```

- 字典
```python
dir = {"name": "King", "age": 20}

# 获取特定键对应的值
val = dir["name"]

# 获取所有键
keys = dir.keys()  # ["name", "age"]

# 获取所有值
values = dir.values()  # ["King", 20]

# 获取所有键值对
pairs = dir.items()  # [("name", "King"), ("age", 20)]

# 遍历字典
for k, v in dir.items():
    print(f"Key: {k}, Val: {v}")
```

## 函数
- 函数定义
```python
  # 基本定义
  def fun():
    print("lalalalala")
    return True

# 多返回值
def fun():
    return 1, 2

tuple = fun()  # tuple = (1, 2), 多返回值返回的是元组
a, b = fun()  # a = 1, b = 2
```

- 函数参数
```python
# 默认参数
def fun(a, b=2):
    print(a)
    print(b)

# 不定长参数 *args
"""
普通参数要写在不定长参数前面, 否则会冲突
"""
def fun(a, *args):
    for i in args:
        print(i)

fun(1, 2, 3, 4, 5)

# 不定长参数 **kwargs
"""
参数顺序: 1.普通参数, 2.*args, 3.**kwargs
"""
def fun(a, *args, **kwargs):
    print(a)

    for i in args:
        print(i)

    print(kwargs["key1"])
    print(kwargs["key2"])

fun("lalala", 1, 2, 3, key1=11, key2=13, key3=14)
```

- 函数注释的编写
  ```python
  def put_color(img, x, y, long, color):
    """
    在一个图片中的一个正方形位置填充对应颜色
    :param img: 图片名(路径)
    :param x: 在图片中的横坐标位置
    :param y: 在图片中的纵坐标位置
    :param long: 正方形的边长
    :param color: 填充的颜色
    :return: 无返回值
    """
  
    # ...
  ```

- 读写外部变量
```python
  DAY = 0

# 仅读
def day1():
    print(DAY)

# 涉及写
"""
修改外部变量时, 要声明 global, 否则会报错
"""
def day2():
    global DAY
    DAY += 1
```

## 异常处理
### try-except 结构
#### try-except
```python
try:
    # 可能抛出异常的代码
    num = int(input("输入数字: "))
    result = 10 / num
    print(f"结果是: {result}")
except ZeroDivisionError:
    print("错误：不能除以零！")
except ValueError:
    print("错误：请输入有效的数字！")
```
- `try` 块中的代码被监控, 如果发生 `ZeroDivisionError`，执行对应 `except` 块, 如果发生 `ValueError`，执行对应 `except` 块
- 如果没有异常，跳过所有 `except`

#### try-except-else
- `else`在`try`没有发生异常时执行
```python
try:
    result = 10 / divisor
except ZeroDivisionError:
    print("除数不能为零")
else:
    # 仅在 try 块没有抛出异常时执行
    print(f"计算成功: {result}")
```

#### try-except-finally
- `finally`语块中的内容, 无论是否发生异常都会执行
```python
f = None
try:
    f = open("data.txt", "r")
    content = f.read()
except FileNotFoundError:
    print("文件不存在")
finally:
    # 无论是否有异常，都会执行（通常用于清理资源）
    if f:
        f.close()
        print("文件已关闭")
```

### 捕获多个异常
```python
try:
    risky_code()
except (ZeroDivisionError, ValueError, TypeError) as e:
    print(f"发生错误: {e}")  # e 是异常对象
```
- **慎用捕获所有异常**
```python
try:
    dangerous_operation()
except Exception as e:
    print(f"捕获到异常: {e}")
    # 这能捕获几乎所有异常，但会掩盖你未预料到的错误（如 KeyboardInterrupt、SystemExit）
```
- 这会导致无法接收由代码逻辑所导致的异常, 从而无法察觉代码问题

### 抛出异常
```python
def validate_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄超出合理范围")
    return age

try:
    validate_age(-5)
except ValueError as e:
    print(e)  # 年龄不能为负数
```

# 其他语法
## 上下文管理器 with
`with` 是 Python 的**上下文管理器**语法，核心作用是：**自动处理资源的获取和释放**，不管中间有没有异常
```python
with open("file.txt", "r") as f:
    data = f.read()
# 出了这个块，文件自动关闭——哪怕 read() 抛了异常
```
- 不用`with`的等价写法
```python
f = open("file.txt", "r")
try:
    data = f.read()
finally:
    f.close()  # 必须手动写，还容易忘
```
	`with` 就是把这个 try/finally 模式封装起来了


## 列表推导式
### 基本形式
```python
[表达式 for 变量 in 可迭代对象]
```
等价于:
```python
result = []
for 变量 in 可迭代对象:
    result.append(表达式)
```
简单的例子:
```python
squares = [x**2 for x in range(5)]
# [0, 1, 4, 9, 16]
```
### 条件过滤
```python
[表达式 for 变量 in 可迭代对象 if 条件]
```
简单的例子:
```python
evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]
```

## 生成器表达式
### 和列表推导式的区别
```python
list_comp = [x**2 for x in range(10)]  # 立刻算完，全部放内存
gen_expr  = (x**2 for x in range(10))  # 什么都没算，只是记住了"怎么算"
```
```python
print(list_comp)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
print(gen_expr)   # <generator object <genexpr> at 0x7f3a1b2c3d40>
```

### 使用方法
- `next`取值
```python
gen = (x**2 for x in range(5))

next(gen)  # 0  ← 算第一个
next(gen)  # 1  ← 算第二个
next(gen)  # 4
next(gen)  # 9
next(gen)  # 16
next(gen)  # StopIteration 异常，没了
```
- `for`循环取值
```python
for val in (x**2 for x in range(5)):
    print(val)
```

### 存在的意义
假设有 1000 万条数据：
```python
# 列表推导式：先把 1000 万个结果全算出来，全塞进内存
total = sum([x**2 for x in range(10_000_000)])

# 生成器：每次只算一个，sum 消费一个扔一个，内存里始终只有一个值
total = sum(x**2 for x in range(10_000_000))
```

### 注意事项
#### 一次性的
- 生成器用完就空了，不能重置：
```python
gen = (x**2 for x in range(3))

list(gen)  # [0, 1, 4]
list(gen)  # []  ← 已经耗尽，再迭代得到空
```
需要重复使用就得重新创建，或者用列表

#### 作为函数唯一参数时可省略括号
```python
sum((x**2 for x in range(10)))  # 两层括号
sum(x**2 for x in range(10))    # 省一层，完全等价

",".join((f"'{n}'" for n in names))
",".join(f"'{n}'" for n in names)   # 省一层，完全等价
```

## lambda 表达式
- 语法
```python
lambda 参数: 返回值表达式
```
只能写**一个表达式**，没有函数体，没有 `return`，**表达式的值就是返回值**

- 具体例子
```python
# 普通函数
def add(x, y):
    return x + y

# 等价的 lambda
add = lambda x, y: x + y
```

# 内置库函数
## map(函数, 可迭代对象)
- 把函数依次应用到每个元素上
```python
map(str, [1, 2, 3])
# 等价于
(str(x) for x in [1, 2, 3])
```
- 两者几乎完全一样，都是惰性的，都返回一个迭代器。
```python
result = map(str, [1, 2, 3])
print(result)        # <map object at 0x...>，还没算

list(result)         # ["1", "2", "3"]，强制求值
",".join(result)     # "1,2,3"，join 会迭代它
```
- 可传入自定义函数
```python
def double(x):
    return x * 2

list(map(double, [1, 2, 3]))  # [2, 4, 6]

# 简单函数通常用 lambda
list(map(lambda x: x * 2, [1, 2, 3]))  # [2, 4, 6]
```
