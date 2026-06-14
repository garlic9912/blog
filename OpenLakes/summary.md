- 在项目一开始就要配置 `log`
- `app`配置
```
Java 后端
    │
    │  POST /query/text?text=月亮
    │  POST /query/image  (multipart file)
    │  GET  /health
    ▼
FastAPI (api.py, port 8000)
    │
    ├── Chinese-CLIP 编码
    ├── FAISS 检索
    └── Trino 查路径
```
`FastAPI` 在这里扮演的角色就是**语言边界适配器**——Java 不能直接调 Python 函数，但所有语言都能发 HTTP 请求。