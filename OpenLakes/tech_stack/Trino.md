# 概念
**Trino**是一个开源的**分布式 SQL 查询引擎**，专门用于对大型数据集进行**交互式分析**。
## 核心特点
- **非存储系统**：不存储数据，只负责查询计算。数据存储在外部源（HDFS,S3,MySQL,Kafka,Iceberg 等）。
- **联邦查询**：单个 SQL 可以跨多个异构数据源联合查询。
- **高性能**：基于内存并行处理，支持多租户，延迟低（亚秒到分钟级）。
- **ANSI SQL 兼容**：支持标准 SQL，以及复杂查询（聚合、多表 Join、窗口函数等）。
## 典型使用场景
- 数据湖分析（查询 Hive,Iceberg,Delta Lake 上的 PB 级数据）
- 跨数据库联邦（统一查询 MySQL,PostgreSQL,MongoDB,Elasticsearch）
- BI 报表与 Ad-hoc 分析（连接 Tableau,Superset）

# 使用方法
- 导入模块
```python
from trino.dbapi import connect
```
- 建立连接
```python
conn = connect(
    host="localhost",    # Trino 服务地址
    port=8080,           # Trino HTTP 端口（默认 8080）
    user="admin",        # 用户名
    catalog="iceberg",   # 目录（数据源配置 iceberg / hive / mysql）
    schema="default"     # 数据库/模式 (default / public)
)
```
- 执行查询
```python
cur = conn.cursor()  # 创建游标对象
cur.execute("SELECT id, s3_path, cardinality(vector) FROM images")
```
- 获取结果
```python
print(cur.fetchall())  # 获取所有查询结果行
```
	fetchall() 返回所有行的列表
	每行是一个元组：(id, s3_path, vector_dimension)
## 完整示例
```python
from trino.dbapi import connect

# 1. 连接配置
conn = connect(
    host="trino-server.example.com",
    port=8080,
    user="analyst",
    catalog="iceberg",
    schema="image_db"
)

# 2. 执行查询
cur = conn.cursor()

# 查询图片信息和向量维度
cur.execute("""
    SELECT 
        id, 
        s3_path, 
        cardinality(vector) as vector_dim,
        created_at
    FROM images 
    WHERE cardinality(vector) = 512
    LIMIT 10
""")

# 3. 处理结果
results = cur.fetchall()
for row in results:
    img_id, s3_path, dim = row
    print(f"Image {img_id}: {s3_path} (dim: {dim})")

# 4. 关闭连接
cur.close()
conn.close()
```