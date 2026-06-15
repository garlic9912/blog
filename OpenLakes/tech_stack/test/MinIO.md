# MinIO 的功能定位

`MinIO` 是一个开源、与 `Amazon S3 API` 完全兼容的对象存储系统。它可以部署在本地服务器或私有云环境中，为应用程序提供统一的文件存取接口

**核心作用**：解决传统文件系统在分布式环境下的存储问题，包括容量扩展、数据冗余、高并发访问、权限控制等

**设计目标**：
- 私有化部署，避免公有云厂商锁定
- 高性能，单集群可达数百 `GB/s` 吞吐量
- 与 `AWS S3` 协议完全兼容，现有 `S3` 工具可直接使用

## 核心概念
MinIO 包含以下四个基本概念：

| 概念       | 英文                      | 定义                                                             |
| -------- | ----------------------- | -------------------------------------------------------------- |
| **桶**    | Bucket                  | 用于隔离并管理对象的容器, 每个桶拥有全局唯一的名称.                                    |
| **对象**   | Object                  | 实际存储的文件（图像、文本、视频等）。每个对象通过唯一的键（Key）在桶内定位。                       |
| **端点**   | Endpoint                | MinIO 服务器的网络访问地址（例如 `http://192.168.1.100:9000`），客户端通过该地址连接服务。 |
| **访问凭证** | Access Key / Secret Key | 身份验证信息。Access Key 相当于用户名，Secret Key 相当于密码，用于控制对桶和对象的操作权限。      |
|          |                         |                                                                |
- 对象存储采用扁平命名空间，不依赖传统文件系统的层级目录，但可通过键中的分隔符（如 `/`）模拟目录结构。
- 每个桶可以配置独立的访问策略（私有、只读、公开等）


# 基本操作
## 连接 MinIO
```python
from minio import Minio

client = Minio(
    "localhost:9000",                # 服务地址
    access_key="admin",              # 用户名
    secret_key="password",           # 密码
    secure=False                     # 是否使用 HTTPS（本地测试用 False）
)
```
## 创建桶
```python
bucket_name = "my-bucket"

# 检查桶是否存在，不存在则创建
if not client.bucket_exists(bucket_name):
    client.make_bucket(bucket_name)
```

## 删除桶
```python
# 先删除桶内所有对象
objects = client.list_objects("my-bucket", recursive=True)
for obj in objects:
    client.remove_object("my-bucket", obj.object_name)

# 再删除桶
client.remove_bucket("my-bucket")
```

## 上传文件
```python
# 上传本地文件到桶中
client.fput_object(
    "my-bucket",           # 桶名
    "remote-file.txt",     # 对象键名（相当于文件名）
    "/local/path/file.txt" # 本地文件路径
)
```

## 下载文件
```python
client.fget_object(
    "my-bucket",           # 桶名
    "remote-file.txt",     # 对象键名
    "./downloaded.txt"     # 本地保存路径
)
```

## 列出桶内对象
```python
objects = client.list_objects("my-bucket")

# obj 是一个迭代器, 包含对象属性, 但没有完整的数据
for obj in objects:
    print(obj.object_name, obj.size)
```

## 获得对象数据
```python
response = client.get_object("my-bucket", "photo.jpg")
data = response.read()        # 读取全部字节（bytes）
response.close()              # 必须关闭释放连接```
```
- 支持流式读取：`response.read(1024)` 分块读取
- 使用后**必须关闭**，否则会导致连接泄漏。推荐用 `with` 语句
```python
with client.get_object("my-bucket", "photo.jpg") as response:
    data = response.read()
```
```
## 删除对象
```python
client.remove_object("my-bucket", "remote-file.txt")
```

