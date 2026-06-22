# 为什么不用 print
`print` 的问题：
- 没有级别，不能按严重程度过滤
- 没有时间戳、文件名、行号
- 生产环境想关掉所有调试输出，得一个个删
- 不能同时输出到文件和终端

# Log 基本操作
## 简单示例
```python
import logging

logging.basicConfig(level=logging.DEBUG)

logging.debug("调试信息")
logging.info("普通信息")
logging.warning("警告")
logging.error("错误")
logging.critical("严重错误")
```
输出:
```python
DEBUG:root:调试信息
INFO:root:普通信息
WARNING:root:警告
ERROR:root:错误
CRITICAL:root:严重错误
```

## 格式化输出
```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s  %(name)s  %(levelname)s  %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
```
输出:
```python
2024-03-15 14:23:01  api  INFO  收到文本查询: 夕阳
2024-03-15 14:23:01  api  INFO  FAISS 返回 3 个结果
2024-03-15 14:23:02  api  INFO  Trino 查询完成
```

- 常用的格式变量
	- `%(asctime)s`时间
	- `%(name)s`logger 名字（模块名）
	- `%(levelname)s`级别名
	- `%(message)s`日志内容
	- `%(filename)s`文件名
	- `%(lineno)d`行号

# 五个级别

| 级别       | 数值  | 用途         |
| -------- | --- | ---------- |
| DEBUG    | 10  | 开发时的详细信息   |
| INFO     | 20  | 正常运行的关键节点  |
| WARNING  | 30  | 不影响运行但需要注意 |
| ERROR    | 40  | 出错了但程序还能跑  |
| CRITICAL | 50  | 严重错误，程序可能崩 |
`level=logging.INFO` 意味着 INFO 以上的都输出，DEBUG 被过滤掉。这就是不用 print 的核心价值——**一行配置控制所有输出**


# Logging 在项目中的使用
不要直接用 `logging.info()`，那是根 logger，所有模块混在一起分不清来源。
- 正确的用法：每个模块一个 logger
```python
# 每个文件顶部这一行
logger = logging.getLogger(__name__)

# 然后用 logger 而不是 logging
logger.info("开始检索")
logger.error("Trino 连接失败")
```
`__name__` 在 `api.py` 里就是字符串 `"api"`，在 `search.py` 里就是 `"search"`，这样配置因此在日志里能看到是哪个模块(文件)写的。

- 格式化输出
```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s  %(name)s  %(levelname)s  %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
```

- 写入日志
```python
try:
    cur.execute(sql)
except Exception as e:
    logger.error("Trino 查询失败", exc_info=True)
    # exc_info=True 会把完整的堆栈信息打进日志
```