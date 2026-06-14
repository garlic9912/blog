# 核心概念
`FastAPI` 是 Python 的 Web 框架，专门用来写 HTTP 接口，`FastAPI`负责把函数变成网络服务。

# 简单示例
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "hello world"}
```

启动：
```bash
uvicorn main:app --reload
# main = 文件名 main.py
# app  = FastAPI() 实例的变量名
# --reload = 代码改动自动重启，开发时用
```

访问 `http://localhost:8000/hello`，返回：
```json
{"message": "hello world"}
```

# 接收参数的三种方式

**路径参数**——写在 URL 里
```python
@app.get("/user/{user_id}")
def get_user(user_id: int):
    return {"id": user_id}

# GET /user/42  →  {"id": 42}
```

**查询参数**——URL `?` 后面的
```python
@app.get("/search")
def search(keyword: str, limit: int = 10):
    return {"keyword": keyword, "limit": limit}

# GET /search?keyword=hello&limit=5
```

**请求体**——POST 的 `JSON` body，用 `Pydantic` 定义结构
```python
from pydantic import BaseModel

class QueryRequest(BaseModel):
    text: str
    k: int = 3

@app.post("/query")
def query(req: QueryRequest):
    return {"text": req.text, "k": req.k}

# POST /query
# Body: {"text": "月亮", "k": 5}
```

# 上传文件

```python
from fastapi import UploadFile

@app.post("/query/image")
async def query_image(file: UploadFile):
    data = await file.read()   # 读取文件二进制
    ...
```
注意这里有 `async` 和 `await`，文件 `IO` 是异步操作。
`async/await` 的意思是：等文件传完的这段时间，不阻塞，可以先去处理别的请求。

# 自动文档
`FastAPI` 启动后自带两个文档页面，不需要额外配置：
```
http://localhost:8000/docs    ← Swagger UI，可以直接在浏览器测试接口
http://localhost:8000/redoc   ← ReDoc，更适合阅读
```
这是 `FastAPI` 最实用的功能之一。调试时可以直接开浏览器点，不需要写 `curl` 命令。