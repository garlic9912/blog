# 知识点

## function<Return(args...)>

### 基本语法

```cpp
#include <functional>
std::function<返回类型(参数类型列表)>  变量名;
```

### 与传统传递函数的方法有何区别

| 场景    | 问题                | `std::function` 的解决                         |
| ----- | ----------------- | ------------------------------------------- |
| 函数指针  | 不能捕获原场景的上下文，类型不灵活 | 可存储捕获状态的 Lambda                             |
| 类成员函数 | 调用时需要对象实例，语法复杂    | 用 `std::function` + `std::bind` 或 Lambda 包装 |
| 设计模式  | 需要类型擦除，统一接口       | 用作多态函数容器                                    |

```cpp
if (table == "__mock_table_1") {
     return [plan](size_t cursor) { // 注意这里的 [plan]
     std::vector<Value> values{};
     // ... 使用了捕获进来的 plan 对象 ...
     return Tuple{values, &plan->OutputSchema()};
   }
}
```

- 函数指针：不能“随身携带”额外的变量（比如 plan 指针）。如果想在 C 语言里实现类似功能，你必须手动传递一个 `void*` 上下文。
- `std::function + Lambda`：可以把当前的上下文（如 plan 变量）“捕获”到函数内部。这样当你以后调用这个函数时，它内部已经记住了它属于哪个 plan。

## C++ 拷贝控制与赋值运算符设计

### 为何 = delete 的函数声明可以省略参数名？

```cpp
class TableIterator {
public:
    // 禁止拷贝构造，参数名省略
    TableIterator(const TableIterator&) = delete;
    auto operator=(const TableIterator&) -> TableIterator& = delete
};
```

这个函数参数只有参数类型, 却并没有写参数名字  

**核心原因**

- `= delete` 只是**声明**该函数存在但禁止使用，**不会定义函数体**。

- 既然没有函数体，参数名在函数体内永远不会被用到，因此参数名是可选的。

- 编译器只需要知道参数类型（签名），就能在有人试图拷贝时进行拦截。

**习惯约定**

- 省略参数名是 `= delete` 声明的标准写法，简洁且表明该函数永远不会被实现。

- 即使对于普通函数，如果参数未被使用，也可以省略参数名（避免编译器的“未使用参数”警告）。但在 `= delete` 场景下省略是普遍做法。

### 为什么 operator= 返回的是 TableIterator& 引用类型

```cpp
auto operator=(const TableIterator& other) -> TableIterator& {
    if (this != &other) {
        // 执行拷贝逻辑
    }
    return *this;   // 返回左侧对象的引用
}
```

- 返回类型是 **`T&`**（非 `const` 引用），允许对结果继续赋值或修改

- 返回 `*this` 是引用，不会拷贝，效率高

# Bustub 表结构解析

## Bustub TablePage 设计

![](/home/garlic/snap/marktext/9/.config/marktext/images/2026-06-09-20-02-32-image.png)

一种基于`slotted page`设计出的`POD`页面对象

- `Header` 中存放了`TablePage`页面的基本信息, 页面`id`, 所链接的下一个页面的`id`, 页面中逻辑删除的元组数目

- `Tuples Info Array`就是`slots`, 记录了插入在尾部的那些元组的信息. 包括在页面中的起始偏移量, 元组长度, 元组其他信息等

- 页面尾部就是插入元组的地方

## Table Heap

`TableHeap`是由 `TablePage` 组成的双向链表, 负责管理一个表 Table 在内存中的逻辑布局

### 为何要在 TablePage 基础上在进行一次封装

`TablePage`仅能处理自己一个页面的逻辑, 但对于数据库的表, 在物理内存上可以跨越多个页面, 也就是需要多个`TablePage`进行组合; 并且`TablePage`作为纯数据类型, 是通过强转来覆盖缓冲池的页面, 不会创建对象, 因此更加不好管理. 从而引出`Table Heap`进行再次封装, 方便调用者, 不需要调用者关心和处理`TablePage`的布局问题

### Table Heap 的职责

| 职责            | 说明                                                        |
| ------------- | --------------------------------------------------------- |
| **寻找空闲空间**    | 遍历链表（或通过 `last_page_id_` 快速定位），找到有足够空间的页面                 |
| **动态扩展表**     | 若当前页满，调用 `BPM->NewPage()` 创建新页，并维护链表指针                    |
| **持有 BPM 引用** | 所有页面的获取、释放、Unpin 都由 `TableHeap` 通过 `BufferPoolManager` 完成 |
| **高层操作接口**    | 对外提供 `InsertTuple`、`GetTuple`、`UpdateTuple` 等语义           |
| **并发控制**      | 通过内部的 `latch_` 保护表结构变更（如分配新页、更新 `last_page_id_`）          |

### 封装布局

1.`Page`(缓冲池层面): 由`bpm`进行管理, 是最原始的页面类型

2.`TablePage`(数据结构层面): 定义了表页面该有的内部成员和方法

3.`TablePage`(逻辑结构层): 管理`Table Page`, 处理页面溢出等其他情况, 无需调用者关心表页面的底层细节
