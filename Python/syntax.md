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

# 字符串
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

# 分支与循环
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

# 列表操作
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

# 元组, 集合, 字典
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

# 函数
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