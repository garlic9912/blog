# 核心概念
`BytesIO` 本质是**内存里的一个虚拟文件**。它把一段 `bytes` 包装成"看起来像文件"的对象，支持所有文件操作（`read`, `write`, `seek`、`tell`……），但数据从不碰磁盘。
```
真实文件:   磁盘 ──── open() ──── 文件对象 ──── read() ──── bytes
BytesIO:   bytes ─── BytesIO() ── 文件对象 ──── read() ──── bytes
```

# 使用场景
很多库`(PIL, pandas, zipfile……)`的 API 设计是"接受一个文件对象"，而不是"接受 bytes"：
```python
Image.open("photo.jpg")        # 接受路径
Image.open(some_file_object)   # 接受文件对象
Image.open(b"\xff\xd8...")     # ❌ 直接传 bytes 不行
```
从网络/数据库拿到的是 `bytes`，但如果写回磁盘再读回来, 性能开销很大
`BytesIO` 就是中间的适配器：
```python
image_bytes = response.read()       # bytes
buf = io.BytesIO(image_bytes)       # 包装成文件对象
img = Image.open(buf)               # PIL 开心了
```

# 虚拟文件读写
- **读模式**：
```python
buf = io.BytesIO(b"hello world")
buf.read()      # b"hello world"
buf.read()      # b""  ← 指针已经在末尾了！
buf.seek(0)     # 指针归零
buf.read()      # b"hello world"  ← 又能读了
```
- **写模式**：
```python
buf = io.BytesIO()
buf.write(b"part1")
buf.write(b"part2")
buf.seek(0)             # 写完必须 seek(0)，否则读到空
data = buf.read()       # b"part1part2"

# 或者直接取底层 bytes，不需要 seek
data = buf.getvalue()   # b"part1part2"  ← 更常用
```
- `seek`指针
`BytesIO` 内部维护一个**读写指针**，跟真实文件完全一样, 因此读写之后要恢复指针