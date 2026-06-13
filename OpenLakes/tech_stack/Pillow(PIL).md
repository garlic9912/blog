# 概念
**PIL（Python Imaging Library）** 是 Python 的图像处理标准库。由于其开发停滞，目前广泛使用的是其活跃分支 **Pillow**。
**核心功能**：读取、编辑、保存多种格式的图像（如 JPEG、PNG、BMP、GIF 等）。支持的操作包括：调整尺寸、旋转、裁剪、颜色转换、滤镜、图像叠加等。

# 使用方法
## 打开图像
```python
from PIL import Image

img = Image.open("photo.jpg")   # 打开图片文件
img.show()                       # 用默认图片查看器打开
```
## 获取图像基本信息
```python
print(img.format)        # 格式，如 'JPEG'
print(img.size)          # (宽度, 高度) 单位像素
print(img.mode)          # 颜色模式：'RGB', 'L' (灰度), 'RGBA' 等
```
## 转换图像模式
```python
gray_img = img.convert("L")   # 转为灰度图
rgb_img = img.convert("RGB")  # 转为 RGB
```
## 调整尺寸, 旋转, 裁剪
```python
# 调整尺寸
resized = img.resize((300, 200))   # 新尺寸 (宽, 高)

# 旋转
rotated = img.rotate(45)          # 逆时针旋转 45 度
rotated_expand = img.rotate(45, expand=True)  # 扩大画布以容纳全部内容

# 裁剪区域 (left, top, right, bottom)
cropped = img.crop((100, 100, 400, 400))
```
## 保存图像
```python
resized.save("output.jpg")          # 自动根据扩展名确定格式
cropped.save("cropped.png", "PNG")  # 指定格式
```
## 复制粘贴
```python
img_copy = img.copy()                     # 复制图像
region = img.crop((10, 10, 100, 100))    # 取一块区域
img_copy.paste(region, (50, 50))          # 把 region 粘贴到指定位置
```