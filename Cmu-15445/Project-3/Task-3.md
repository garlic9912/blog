# 编码规范

---

- 原版代码风格
  
  - 太过冗余, 需要`if-else`配合
  
  - 不够精简, 可读性较差

```cpp
if (right_itr_ == right_hash_table.End()) {
    right_finish_ = true;        
} else {
    right_finish_ = false;
}
```

- 改版代码风格

```cpp
right_finish_ = (right_itr_ == right_hash_table_.End());
```

---

# Intermediate_result_page 的设计动机

## 为什么是 Page 而不是堆内存 Buffer

`IntermediateResultPage` 用于存放 Hash Join 和外部排序产生的中间元组。选择 Page 而非堆内存 Buffer，核心原因是**绕开 OS，由 DBMS 自主管理内存**。

## 使用 Page（Buffer Pool）的优势

| 优势       | 说明                                                   |
| -------- | ---------------------------------------------------- |
| **统一管理** | Page 由 BufferPoolManager 统一分配、缓存、淘汰、刷盘，遵守 DBMS 的内存规范 |
| **内存可控** | Buffer Pool 大小固定，DBMS 知道自己还有多少可用内存                   |
| **换入换出** | 中间结果太大时可以自然溢出到磁盘，由 Buffer Pool 自动调度                  |
| **无碎片**  | Page 是固定 4KB 的数组，不会产生堆内存碎片                           |

## 使用堆内存 Buffer（std::vector/ new）的劣势

| 劣势          | 说明                                                |
| ----------- | ------------------------------------------------- |
| **频繁陷入内核态** | 大块堆内存分配（`new` / `malloc`）需要通过系统调用向 OS 申请，上下文切换代价高 |
| **堆内存碎片**   | 频繁分配/释放大块内存导致堆空间碎片化，OS 管理越来越复杂                    |
| **不可控**     | DBMS 不知道 OS 还剩多少可用内存，容易触发 OS 的 swap               |
| **无法换出**    | 堆内存是"匿名"的，DBMS 无法像 Page 一样按策略将其换出到磁盘              |

## 核心原则

> **DBMS 不应该依赖 OS 来管理自己的内存。** 使用 Buffer Pool + Page 是对内存的自主掌控，使用堆内存 Buffer 是把命运交给 OS。对于中间结果这种"临时大对象"，Page 是唯一正确的选择。

# POD 对象

Page 类是 **POD（Plain Old Data）** 对象，即纯数据容器，没有任何复杂的 C++ 特性

## 规定

| 规则                      | 禁止                                              | 原因                                                       |
|:----------------------- |:----------------------------------------------- |:-------------------------------------------------------- |
| **1. 不准有虚函数**           | `virtual void Foo()`                            | 虚函数会产生 vtable 指针，占用 Page 空间，破坏磁盘布局                       |
| **2. 不准有指向外部堆内存的指针**    | `std::vector`, `std::string`, `char*` 等         | Page 必须自包含，数据直接落在 4KB 内。外部指针在序列化到磁盘后失效                   |
| **3. 不准有绝对指针，只能存相对偏移量** | 存储任何绝对内存地址（如 `reinterpret_cast<uint32_t>(ptr)`） | Buffer Pool 中 Page 每次读入的 Frame 地址不同，绝对地址在 Page 被换出再读入后失效 |
| **4. 不准有复杂的构造/析构函数**    | 构造函数里 `new`、析构里 `delete`                        | Page 通过 `reinterpret_cast` 强转而来，不经过正常的构造/析构流程            |
| **5. 所有数据必须落在 4KB 之内**  | 向外部分配的内存存数据                                     | Page 是磁盘 I/O 的基本单位，必须能整体写入/读出磁盘                          |

# 零长度数组

## 页面对象不是 new 出来的

数据库中的页面（Page）不是通过 `new TablePage()` 创建的，而是从 Buffer Pool **申请一块原始内存**，然后通过 `reinterpret_cast` 将这块内存"看作"页面对象：

```java
char *raw_data = bpm->GetPage(page_id)->GetData();  // 拿到 4KB 原始内存
TablePage *page = reinterpret_cast<TablePage *>(raw_data);  // 扣上结构体
```

此时，类的第一个成员变量的起始地址就是页面在内存中的起始地址

## 为什么不用 char* page_start_

如果用一个**指针**来记录起始地址

```java
class TablePage {
    char *page_start_;   // 在 64 位系统上占 8 字节
    // ... 其他成员
};
```

| 问题        | 说明                             |
| --------- | ------------------------------ |
| **占用空间**  | 指针本身占 8 字节，挤占页面空间              |
| **需要初始化** | 必须在构造函数中初始ua                   |
| **破坏布局**  | 8 字节的指针会推后后续成员偏移量，破坏磁盘页面内部布局结构 |

## char page_start_[0]

```java
class TablePage {
    char page_start_[0];        // Offset: 0, Size: 0
    page_id_t next_page_id_;    // Offset: 0, Size: 4
    uint16_t num_tuples_;       // Offset: 4, Size: 2
};
```

**核心特性**：

- 长度为 0 的数组**不占用任何空间**。

- 数组名可以**作为地址标签**使用，代表 `this` 指针所在的位置。

当访问 `page->page_start_` 时，编译器计算：

```java
Address = this + offset(0) = 页面内存起始地址
```

## 为什么不用 int / long 来定义数组

无论是`char arr[0]`, `int arr[0]`还是`long arr[0]`, 他们本身在数据结构中占据的内存大小都是0,  但是在代码中, 对于这种充当"内存占位符"的数组, 几乎都是强制使用`char`或`uint8_t`

### 原因1: 指针运算步长

在 C/C++ 中，对指针进行加减运算时，移动的字节数是 `偏移量 × 类型大小`

| 类型         | `sizeof` | `ptr + 100` 实际移动字节 | 结果              |
| ---------- | -------- | ------------------ | --------------- |
| **`char`** | 1        | `100 × 1 = 100`    | ✅ 正确：移动 100 字节  |
| **`int`**  | 4        | `100 × 4 = 400`    | ❌ 越界：飞到 400 字节外 |
| **`long`** | 8        | `100 × 8 = 800`    | ❌ 越界：飞到 800 字节外 |

数据库中的偏移量以**字节**为单位。用 `char` 才能保证 `page_start_ + 100` 恰好移动 100 字节。

### 原因 2：内存对齐与结构体填充

编译器会在变量间插入填充字节（Padding）以满足 CPU 对齐要求

| 类型                    | 对齐要求 | 影响                                  |
| --------------------- | ---- | ----------------------------------- |
| **`char`**            | 1 字节 | 数组紧贴前一个变量，不产生 Padding               |
| **`int`**             | 4 字节 | 若前面成员占 12 字节，中间插入 4 字节 Padding 才能对齐 |
| **`long` / `double`** | 8 字节 | 更多 Padding，破坏预期的磁盘布局                |

`char` 对齐要求为 1，能最大限度避免编译器自作主张地插入 Padding。

# 复合键

SQL 中经常出现多列等值连接:

```sql
SELECT * FROM A JOIN B ON A.a1 = B.b1 AND A.a2 = B.b2;
```

要求**同时匹配**才算连接成功

## 为什么不能只按一列分区？

| 分区方式                  | 能保证什么                     | 无法保证什么                |
| --------------------- | ------------------------- | --------------------- |
| **只按 `a1` 分区**        | 同桶内 `a1 = b1`             | `a2` 可能乱七八糟，还需在桶内费力挑选 |
| **按 `[a1, a2]` 整体分区** | 同桶内 `a1=b1` **且** `a2=b2` | —                     |

**结论**：必须将连接列组成一个 **复合键**，整体计算哈希值。

## 复合键哈希的优势

| 优势           | 说明                                                 |
| ------------ | -------------------------------------------------- |
| **极高的过滤效率**  | `a1=1, a2=2` 和 `a1=1, a2=3` 在分区阶段彻底隔离              |
| **减少内存对比次数** | 大量不匹配数据提前被分到不同分区                                   |
| **统一处理**     | 1 列、2 列还是 10 列连接，逻辑完全一样：提取 Value → 算 Hash → 丢进 Map |

# 哈希表自定义 Key 条件

## operator() 必须有 const 限定符

```java
// ❌ 错误：缺少 const
size_t operator()(const JoinKey &key) { ... }

// ✅ 正确：必须有 const
size_t operator()(const JoinKey &key) const { ... }
```

## 特化必须放在 namespace std 内

- 编译器从上往下, 从左往右的去读代码, 因此需要在定义特殊键的哈希表上面进行特化

```java
namespace std {
  template <>
  struct hash<bustub::JoinKey> {
    auto operator()(const bustub::JoinKey &key) const -> std::size_t {
      // 哈希逻辑
    }
  };
}


// ...

namespace bustub {
    // ...
    std::unordered_map<JoinKey, ...> ht_;
}
```

## Key 必须定义 operator==

`unordered_map` 处理哈希冲突时需比较 Key 是否相等

```java
struct JoinKey {
  std::vector<Value> keys_;

  auto operator==(const JoinKey &other) const -> bool {
    for (uint32_t i = 0; i < other.keys_.size(); i++) {
      if (keys_[i].CompareEquals(other.keys_[i]) != CmpBool::CmpTrue) {
        return false;
      }
    }
    return true;
  }
};
```

# 工程反思

## Page 遍历中的死锁分析(AB-BA 死锁)

**错误代码**：

```cpp
// guard 正持有 Page A 的写锁
guard = bpm->CheckedWritePage(next_page_id);  // 尝试获取 Page B
```

在 C++ 中，`=`赋值操作符会先完全执行等号右边的表达式，然后再将结果赋给左边（复制赋值或移动赋值, 此代码是移动赋值）。这意味着，系统会先通过`CheckWritePage()`请求 Page B 的写锁。此时线程同时持有着 Page A 的锁，并且在等待 Page B 的锁。如果在并发环境下，另一个线程刚好持有着Page B 的锁，且想要访问 Page A，就会发生最经典的 AB-BA 死锁。

```
线程 1：持有 Page A 锁 → 等待 Page B 锁
线程 2：持有 Page B 锁 → 等待 Page A 锁
→ AB-BA 死锁
```

## 状态隔离

### 旧写法："向前看"循环

```text
page_id = 首页
while (page_id 有效) {
    拿着当前页的锁 → 读出 next_page_id

    if (next_page_id 有效) {
        guard 释放当前锁 → 获取下一页锁   // 状态切换
    }

    删除当前页 (此时 guard 已经指向下一页!)
    page_id = next_page_id
}
```

| 问题          | 说明                                              |
|:----------- |:----------------------------------------------- |
| **状态撕裂**    | 删除 `page_id` 时，`guard` 已经指向下一页，代码在"现在"和"未来"之间跳跃 |
| **锁生命周期混乱** | 当前迭代被迫为下一次迭代准备锁，循环边界不隔离                         |

### 正确写法："当前项"循环

```text
curr_id = 首页
while (curr_id 有效) {
    1. 拿 curr_id 的锁
    2. 读出 next_id 保存为局部变量
    3. 释放 curr_id 的锁（干脆利落）
    4. 删除 curr_id

    curr_id = next_id   // 交出接力棒
}
```

| 优势          | 说明                                |
|:----------- |:--------------------------------- |
| **循环不变量清晰** | 每次进入循环时，手里没有任何锁，只有一把钥匙（`curr_id`） |
| **作用域隔离**   | `guard` 生命周期严格限制在"拿锁→读数据→释放锁"三步内  |
| **异常安全**    | 任何步骤失败直接 `break`，绝对不会导致死锁         |

> 遍历数据库 Page 时，永远遵循：**只处理当前节点，取出下一节点 ID 后立刻释放当前节点。** 一次迭代只做一件事，<mark>不替下一次迭代做任何准备工作</mark>。
