# time
```python
start = time.time()   # 记录开始时间，返回 Unix 时间戳（秒，浮点数）
# ... 做一些事 ...
elapsed = time.time() - start   # 现在的时间 - 开始时间 = 经过了多少秒
```
`time.time()` 返回的是类似 `1710000000.234` 这样的数，两个相减就是时间差，单位秒

# request
Python 发 HTTP 请求的标准库，对应在终端手打的 `curl`
```python
# curl -X POST "http://...?text=夕阳"
r = requests.post(URL, params={"text": QUERY})
```
`params` 会自动把字典拼成查询字符串并处理编码不一致，等价于：
```
http://127.0.0.1:8000/query/text?text=%E5%A4%95%E9%98%B3
```
## 常用方法
```python
requests.get(url)                          # GET
requests.post(url, params={})              # POST + 查询参数
requests.post(url, json={"key": "val"})    # POST + JSON body
requests.post(url, files={"file": open("img.png", "rb")})  # POST + 文件上传
```
返回的 `r` 是响应对象：
```python
r.status_code   # 200, 404, 500 ...
r.json()        # 把响应体解析成 Python dict
r.text          # 响应体原始字符串
r.elapsed       # 这次请求耗时
```
断言确保正常 :
```python
assert r.status_code == 200
```