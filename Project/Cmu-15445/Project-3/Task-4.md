# 知识点

## ORDER BY 中 NULL 值的排序

### SQL 标准的显式控制

SQL 标准（SQL:2003+）提供 `NULLS FIRST` / `NULLS LAST` 子句，显式指定 NULL 的位置：

```sql
SELECT * FROM t ORDER BY col ASC NULLS FIRST;   -- NULL 排最前
SELECT * FROM t ORDER BY col ASC NULLS LAST;    -- NULL 排最后
SELECT * FROM t ORDER BY col DESC NULLS FIRST;  -- NULL 排最前
SELECT * FROM t ORDER BY col DESC NULLS LAST;   -- NULL 排最后
```

| 子句            | 效果                        |
| ------------- | ------------------------- |
| `NULLS FIRST` | NULL 强制排最前，无论 ASC 还是 DESC |
| `NULLS LAST`  | NULL 强制排最后，无论 ASC 还是 DESC |

### 各数据库的默认行为

若未显式指定，不同系统对 NULL 的"大小"认定不同：

| 数据库                             | ASC 默认        | DESC 默认       | 将 NULL 视为 |
| ------------------------------- | ------------- | ------------- | --------- |
| **PostgreSQL / Oracle**         | `NULLS LAST`  | `NULLS FIRST` | 最大值       |
| **MySQL / SQLite / SQL Server** | `NULLS FIRST` | `NULLS LAST`  | 最小值       |

## 内部类

内部类（Nested Class）是指**定义在另一个类的内部的类**。

```cpp
class Outer {
public:
    class Inner {      // 内部类
    public:
        void innerFunc();
    };
};
```

### 内部类相对与外部类权限

- **访问外部类的成员**

内部类可以访问外部类的**所有静态成员**（包括 `private` 的），但**不能直接访问非静态成员**（因为没有外部类对象）

```cpp
class Outer {
private:
    static int static_private;
    int instance_private;

public:
    class Inner {
    public:
        void access() {
            static_private = 10;     // ✅ 可以访问外部类的私有静态成员
            // instance_private = 20;  // ❌ 错误！不能直接访问非静态成员
        }
    };
};

int Outer::static_private = 0;
```

- 访问非静态成员

内部类需要**通过外部类对象**才能访问其非静态成员

```cpp
class Outer {
private:
    int value = 42;

public:
    class Inner {
    public:
        void access(Outer& outer) {
            outer.value = 100;  // ✅ 通过对象访问私有成员
        }
    };
};
```

### 外部类访问内部类成员

- 外部类不能直接访问内部类的私有成员（除非友元）：

```cpp
class Outer {
public:
    class Inner {
    private:
        int inner_private = 10;
    };

    void access() {
        Inner inner;
        // int x = inner.inner_private;  // ❌ 错误！不能访问内部类的私有成员
    }
};
```

- 要让外部类访问内部类的私有成员，需要将外部类声明为友元：

```cpp
class Outer {
public:
    class Inner {
        friend class Outer;  // 允许外部类访问内部类的私有成员
    private:
        int inner_private = 10;
    };

    void access() {
        Inner inner;
        int x = inner.inner_private;  // ✅ 可以
    }
};
```

## 迭代器在遍历页面链表时的优势

类似于访问`B+`树叶子页面链表时的迭代器

### 核心思想

迭代器的本质：**保存当前读取进度**，而不是每次都从头开始找。

可能需要保存的状态：

- 当前在哪一页 (`page_id`)

- 当前页的锁 (`ReadPageGuard`)

- 当前页内读到第几个元组 (`tuple_idx`)

### 迭代器 vs 无迭代器

| 维度               | 无迭代器（每次从头扫）                     | 有迭代器                |
| ---------------- | ------------------------------- | ------------------- |
| **定位下一个元组**      | O(N)：从 Run 的第一页开始找              | **O(1)**：直接在当前页取下一个 |
| **Pin/Unpin 次数** | 每读一个元组就要 Pin/Unpin 从头到当前位置的所有页面 | 每页只 Pin 一次，读完才释放    |
| **全局复杂度**        | O(N²)                           | **O(N)**            |

### 迭代器的三大优势

#### 1. O(1) 定位

获取下一个元组只需在O(1)时间复杂度内更新迭代器，而无需再次遍历页面链表。

#### 2. 大幅减少 Pin/Unpin

每个页面只 Pin 一次，页内所有元组读完再换下一页，避免重复锁操作(每次重头扫页面链表)。

#### 3. K 路归并友好

有了迭代器，可以直接将 K 个迭代器放入 `std::priority_queue`，轻松实现高效多路归并。

## 一定要尽可能避免拷贝赋值

### 拷贝赋值的缺点

#### 性能代价

- **内存分配**

每次拷贝都需要向 OS 申请一块新内存，频繁的内存分配会陷入内核态，速度比纯用户态操作慢几个数量级。

- **逐字节复制**

将源对象的全部数据逐字节复制到新内存，数据越大耗时越长。在数据库中，一个 Page 是 4KB，一次拷贝就是 4KB 的全量复制，积累起来极其昂贵。

- **CPU 缓存失效**

拷贝产生新副本，原先在 CPU 缓存中的数据被新数据冲刷掉，后续访问时缓存命中率下降，需要重新从内存加载。

#### 内存代价

| 问题           | 说明                           |
| ------------ | ---------------------------- |
| **双倍内存占用**   | 原对象 + 副本同时存在，内存占用翻倍          |
| **内存碎片**     | 多次分配/释放后，堆空间碎片化，后续分配更慢、更容易失败 |
| **触发 OS 换页** | 内存不足时 OS 将数据交换到磁盘，性能断崖式下降    |

在数据库中，一个查询可能产生大量中间结果，如果每个中间结果都拷贝一次，及其低效

### 解决方法

#### 移动语义（Move）

**核心：将源对象的资源指针直接转移，避免分配新内存。**

```cpp
// ❌ 拷贝：触发深拷贝
std::vector<int> v2 = v1;

// ✅ 移动：v1 的数据被"掏空"，v2 直接接管
std::vector<int> v2 = std::move(v1);
```

| 要素          | 说明                  |
| ----------- | ------------------- |
| `std::move` | 将左值转为右值引用，触发移动构造/赋值 |
| 移动构造函数      | 接管资源，将源对象指针置空       |
| 适用场景        | 返回局部变量、向容器中插入、传递所有权 |

#### 引用传递

**核心：不传值，传引用，不产生任何副本**

```cpp
// ❌ 值传递：发生拷贝
void Process(std::vector<int> data);

// ✅ const 引用：零拷贝，只读
void Process(const std::vector<int>& data);

// ✅ 非 const 引用：零拷贝，可修改
void Process(std::vector<int>& data);
```

#### 写时复制(Copy-on-Write)

**核心：多个对象共享同一份数据，只在需要修改时才真正拷贝**

```cpp
// 多个 shared_ptr 指向同一数据，引用计数
auto p1 = std::make_shared<Data>();
auto p2 = p1;  // 共享，无拷贝

// 需要修改时，先拷贝一份再改
void Modify(std::shared_ptr<Data> &p) {
    if (p.use_count() > 1) {
        p = std::make_shared<Data>(*p);  // 拷贝一份
    }
    p->Change();  // 安全修改
}
```

#### RVO / NRVO（返回值优化）

**核心：编译器直接在调用方的内存位置构造返回值，不产生临时对象**

```cpp
// ✅ 编译器会直接在调用方位置构造，零拷贝
std::vector<int> Build() {
    std::vector<int> v;
    v.push_back(42);
    return v;  // 不会拷贝，RVO 保证
}
```

| 要求     | 说明                                     |
| ------ | -------------------------------------- |
| 返回局部对象 | 编译器自动优化                                |
| 避免画蛇添足 | **不要** `return std::move(v);`，这会阻止 RVO |

#### emplace_back 代替 push_back

**核心：在容器内原地构造对象，避免临时对象的构造 + 移动**

```cpp
// ❌ 先构造临时 pair，再移入
vec.push_back({key, value});

// ✅ 直接在容器内存中构造
vec.emplace_back(key, value);
```

## 迭代器要避免死区

### 什么是迭代器的“死区”

**死区**：迭代器处于一个**既指向无效元素，又不等于 `end()`** 的状态。

这种状态违反了迭代器的基本契约：

> **<mark>迭代器要么指向一个有效元素，要么等于 `end()</mark>`**

### 死区产生的根本原因

| 原因           | 说明                                               |
| ------------ | ------------------------------------------------ |
| **跨页边界处理不当** | 当迭代器位于页面最后一个元素时，`operator++` 增加了索引，但未检查是否越过当前页边界 |
| **提前返回**     | 增加索引后直接返回，没有继续尝试定位到下一个有效元素                       |
| **状态不一致**    | `page_id` 仍停留在旧页面，但索引已超出该页有效范围                   |

## sort 严格弱序

`std::sort` 要求比较函数 `f(a, b)` 必须满足：

| 规则       | 要求                                                             |
| -------- | -------------------------------------------------------------- |
| **反自反性** | `f(a, a)` 必须为 `false`（元素不能小于自己）  --> 弱序                        |
| **反对称性** | 若 `f(a, b)` 为 `true`，则 `f(b, a)` 必须为 `false`                   |
| **传递性**  | 若 `f(a, b)` 为 `true` 且 `f(b, c)` 为 `true`，则 `f(a, c)` 为 `true` |

因此在全部相等时, 应返回`false`

```cpp
// ✅ 正确：相等时返回 false
auto cmp = [](const Tuple &a, const Tuple &b) -> bool {
    for (...) {
        if (a.key[i] != b.key[i]) return a.key[i] < b.key[i];
    }
    return false;  // 全部相等 → a 不小于 b
};
```

## C++ 循环依赖与未声明错误

### 问题本质：循环依赖

当两个类相互引用时，由于编译器的**单遍解析**特性，会出现“先有鸡还是先有蛋”的问题：

| 问题       | 说明                      |
| -------- | ----------------------- |
| **前向声明** | 告诉编译器“这个类存在”，但不知道它的具体成员 |
| **完整定义** | 需要知道类的具体大小和成员才能使用       |

### 典型场景：页面类与迭代器类

```textile
问题：页面类需要创建迭代器对象
     迭代器类需要调用页面类的成员函数
```

| 顺序     | 问题                                |
| ------ | --------------------------------- |
| 先定义迭代器 | 迭代器里调用页面类的方法 → 页面类未定义             |
| 先定义页面类 | 页面类的 `Begin()` 需要创建迭代器对象 → 迭代器未定义 |

### 正确的代码结构顺序

```textile
1. 前向声明迭代器类
   └─ class InterPageIterator;

2. 完整定义页面类
   └─ 但部分成员函数如 Begin()、End() 只声明不定义

3. 完整定义迭代器类
   └─ 此时页面类已定义，可以放心调用页面类的成员函数

4. 在文件末尾实现页面类的成员函数
   └─ 此时迭代器类已完整定义，可以创建迭代器对象
```

## 移动语义的使用时机

#### 为什么对临时对象用  std::move 是坏的

| 原因         | 说明                                            |
| ---------- | --------------------------------------------- |
| **阻止 RVO** | 编译器本可以在目标位置直接构造对象，`std::move` 强制转为右值引用，反而阻碍优化 |
| **冗余操作**   | 临时对象本身就是右值，不需要显式 `std::move`                  |
| **性能下降**   | 多了一次移动构造（虽然不是大开销，但没必要）                        |

```cpp
// ❌ 错误：对临时对象使用 move
InterPageIterator itr_a = std::move(GetPageIterator(pages_[0]));

// ✅ 正确：直接赋值
InterPageIterator itr_a = GetPageIterator(pages_[0]);
```

### 何时应该用 std::move

| 场景                     | 是否需要 `std::move` |
| ---------------------- | ---------------- |
| 对局部变量 `return`         | ❌ 不需要（自动移动）      |
| 对具名对象（不再使用）            | ✅ 需要             |
| 对 `std::unique_ptr` 传递 | ✅ 需要             |

## 窗口函数

窗口函数允许你对一组与当前行相关的行执行计算，但**不会像 `GROUP BY` 那样将多行合并为一行**

```sql
函数名() OVER (
    PARTITION BY 列名1, 列名2   -- 【可选】分组依据，相当于"分窗"
    ORDER BY 列名3              -- 【可选】组内排序
    ROWS BETWEEN ... AND ...     -- 【可选】窗口范围
)
```

### <mark>ROWS 的作用</mark>

#### 为什么需要 `ROWS`

当你在 `OVER()` 中写了 `ORDER BY` 后，如果不指定 `ROWS` 或 `RANGE`，默认行为可能不是你想的那样。`ROWS` 让你**明确控制**窗口的大小和边界。

#### ROWS 的语法

```sql
ROWS BETWEEN 起点 AND 终点
```

**起点/终点可以是：**

- `UNBOUNDED PRECEDING` — 一直往前到分区第一行

- `n PRECEDING` — 往前 n 行

- `CURRENT ROW` — 当前行

- `n FOLLOWING` — 往后 n 行

- `UNBOUNDED FOLLOWING` — 一直往后到分区最后一行

#### 代码示例

- 移动平均
  
  ```sql
  AVG(salary) OVER (ORDER BY user_name ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
  ```

- 累计求和
  
  ```sql
  SUM(salary) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
  ```

### 常用函数

| **类别**             | **函数**             | **作用**                        |
| ------------------ | ------------------ | ----------------------------- |
| **排名 (Ranking)**   | `RANK()`           | 排名。如果有并列，会跳过后续数字（1, 2, 2, 4）。 |
|                    | `DENSE_RANK()`     | 稠密排名。并列不跳数字（1, 2, 2, 3）。      |
|                    | `ROW_NUMBER()`     | 绝对行号。即使值一样也分先后（1, 2, 3, 4）。   |
| **值获取 (Value)**    | `LAG()` / `LEAD()` | 获取上一行或下一行的数据（常用于计算增长率）。       |
|                    | `FIRST_VALUE()`    | 获取窗口第一行。                      |
| **聚合 (Aggregate)** | `SUM()`, `AVG()`   | 计算累计总和或移动平均值。                 |

### 窗口内部和外部 order by 对比

| 对比维度        | `OVER(ORDER BY A)` 内部排序                | 外部 `ORDER BY B`  |
| ----------- | -------------------------------------- | ---------------- |
| **作用对象**    | 窗口函数内部的计算逻辑                            | 整个 `SELECT` 查询结果 |
| **影响什么**    | `RANK()` 的排名、`SUM()` 的累计值、`LAG()` 取哪一行 | 屏幕上显示的行顺序        |
| **是否改变行数**  | 否                                      | 否                |
| **是否改变数据值** | **是**（排名、累计值依赖顺序）                      | 否（只调换行位置）        |

- 内部的`order by`会影响某些窗口函数的取值

- 外部仅仅是调换行的顺序

## 运行时动态选择模板类型：类型擦除与多态包装器

### 模板类的问题

C++ 模板是**编译时多态**，模板参数必须在编译期确定。但在某些场景下（如根据运行时配置选择不同的模板参数），需要**在运行时决定使用哪个模板实例化**

```cpp
// ❌ 问题：模板参数必须在编译期确定
std::unique_ptr<BPlusTreeIndex<???>> tree_;  // 模板类型运行时才知道
```

### 主要的代码困境

| 问题         | 说明                                                                                                                          |
| ---------- | --------------------------------------------------------------------------------------------------------------------------- |
| **类型不兼容**  | 不同模板类之间并没有继承关系. 无法使用`dynamic_cast`来进行转换例如: `BPlusTreeIndex<GenericKey<4>>` 和 `BPlusTreeIndex<GenericKey<8>>` 是完全不同的类型，无继承关系 |
| **运行时决定**  | 根据运行时`key_size` 的值（4、8、16…）需要选择不同的模板实例                                                                                      |
| **成员变量类型** | 类的成员变量类型必须在编译期固定，无法动态改变. 因此不能在头文件里面定义带模板参数的成员                                                                               |

### 解决方案: 多态包装器

**核心思想**：

1. 定义一个**非模板的抽象基类**（接口）

2. 定义**模板化的实现类**继承该基类

3. 通过基类指针调用虚函数，抹掉具体模板类型

```cpp
// 非模板基类（定义在 .h 中）
class IndexWrapper {
public:
  virtual ~IndexWrapper() = default;
  virtual void InitBegin() = 0;
  virtual void InitEnd() = 0;
  virtual auto IsEnd() -> bool = 0;
  virtual void Advance() = 0;
  virtual auto GetRid() -> RID = 0;
};

// 执行器持有基类指针(成员变量)
std::unique_ptr<IndexWrapper> index_wrapper_;

// 模板实现类（定义在 .cpp 中）
template <typename K, typename V, typename C>
class IndexWrapperImpl : public IndexWrapper {
 public:
  explicit IndexWrapperImpl(Index *index) : tree_(dynamic_cast<BPlusTreeIndex<K, V, C> *>(index)) {}
  void IterBegin() override { iter_ = tree_->GetBeginIterator(); }
  // ...
  void ScanKey(const Tuple &key, std::vector<RID> *result, Transaction *txn) override {
    tree_->ScanKey(key, result, txn);
  }

 private:
  BPlusTreeIndex<K, V, C> *tree_;
  IndexIterator<K, V, C> iter_;
};
```

- 持有非模板的基类指针, 避免了要在编译期间确定模板参数, 同时借助多态访问动态模板参数的子类对象

### 为什么模板实现类要放在.cpp文件而非.h文件

因为这里的模板类 `IndexWrapperImpl` **只在 `.cpp` 文件内部使用**，不会被其他编译单元引用

| 条件         | 说明                                                      |
| ---------- | ------------------------------------------------------- |
| **仅内部使用**  | `IndexWrapperImpl` 只在 `index_scan_executor.cpp` 中实例化和使用 |
| **对外暴露基类** | 头文件只暴露非模板基类 `IndexWrapper`（接口）                          |
| **实现细节隐藏** | 具体模板类型对编译单元外部不可见                                        |

- 减少头文件依赖: 模板实现中依赖的 `BPlusTreeIndex`、`GenericKey` 等头文件不需要暴露给所有包含此 `.h` 的文件

- 加快编译速度: 模板实例化只发生在一个 `.cpp` 中，而不是每个包含头文件的地方

- 隐藏实现细节: 具体模板参数类型对使用者透明

- 避免头文件膨胀: 减少 `#include` 传播

## Vector 中 resize 和 reserve

| 特性                  | `resize(n)`              | `reserve(n)`             |
| ------------------- | ------------------------ | ------------------------ |
| **改变 `size()`**     | ✅ 会                      | ❌ 不会                     |
| **改变 `capacity()`** | 可能会（如果 `n > capacity()`） | 可能会（如果 `n > capacity()`） |
| **构造元素**            | ✅ 会（添加默认构造的元素）           | ❌ 不会                     |
| **访问 `[n-1]`**      | ✅ 可以                     | ❌ 不可以（越界）                |
| **用途**              | 改变实际元素个数                 | 预分配内存，减少扩容               |

- `resize()`预分配内存, 同时在内存中构造默认元素, `vector`长度发生变化

- `reserve()`只分配内存, 不构造元素, `vector`长度保持不变

# 代码反思

## 外部归并时初始归并段的大小选择

一开始我没管太多, 如果一个页面写满了, 我就把他页面号加入到链表中. 但是在线上测试的时候, 由于`disk write`次数过多导致外部归并的测试没过. 因此我引入了一个宏定义 `TUPLES_IN_RUNS`来根据需要动态设置一个归并段中的元组数量,  并根据这个宏定义来编写创建初始归并段的代码, 而不是根据一个页面是否满了来创建归并段. 这样既达到了增大归并段长度的目的, 也将归并段的创建与页面结构解耦

## 磁盘页面释放的时机

第一版代码, 我将所有归并段(页面链表)的回收放在了外部归并排序结束后进行. 这是非常低效的, 因为归并段是页面链表这样的逻辑结构, 如果要回收后面一个页面就需要读入前面一个页面到内存, 这会导致`buffer pool`中同时出现大量仅仅使用一次的页面, 违背空间局部性原理. 非常低效. 正确的处理方法应该是在读取下一个页面的时候, 就删除掉前面的一个旧页面, 因为对于归并排序的中间页面, 有且仅会读取一次, 读取完就应该立刻删除, 避免了二次读入内存

| 做法        | 旧页释放时机     | Buffer Pool 压力         |
| --------- | ---------- | ---------------------- |
| **延迟 删除** | 归并全部结束后才释放 | 同时持有整个输入链表 + 输出链表，必然溢出 |
| **边读边删**  | 当前页读完后立即释放 | 永远只需 3 个页（2 输入 + 1 输出） |

# TODO 补充

- `group by`和窗口函数的区别
  
  - 窗口函数一定要注意窗口区间, 窗口函数与聚合`group by`有很大的区别, `group by`的作用范围是分组后里面的所有元组; 而窗口函数在分区之后作用范围由窗口区间所决定, 也就是`ROWS BETWEEN ... AND ...`, 并非无脑的分组内全部元组
  
  - 两者都是聚合类型的算子, 都要从子节点获取元组数据. 只不过`group by`会对这些元组进行聚合压缩, 向上返回的是"阉割"后的元组集合; 而窗口函数不压缩从子节点获得的元组集合, 在算子内部执行完聚合以及查询之后, 追加到元组集合中, 向上返回一个更庞大的元组集合
