`os` 模块是 Python 标准库中**最核心、最常用**的模块之一，它提供了**与操作系统进行交互**的统一接口。通过 `os` 模块，你可以执行文件操作、目录管理、进程控制、环境变量读取等几乎所有与底层操作系统相关的任务，而且代码可以跨不同操作系统运行。

> **注意**：`os` 模块包含子模块 `os.path`（专门处理路径字符串），但自 Python 3.4 起，`pathlib` 提供了更现代、面向对象的路径操作方式，官方推荐在可能的情况下使用 `pathlib`。不过 `os` 仍广泛应用于脚本、系统管理、文件处理等场景。


# 主要功能分类
## 文件与目录操作
这是最常用的功能，包括创建、删除、重命名、遍历目录等。

| 函数 | 作用 | 示例 |
|------|------|------|
| `os.getcwd()` | 获取当前工作目录 | `os.getcwd()` → `'/home/user/project'` |
| `os.chdir(path)` | 改变当前工作目录 | `os.chdir('/tmp')` |
| `os.listdir(path)` | 列出目录下的所有文件和子目录 | `os.listdir('.')` → `['file1.txt', 'dir2']` |
| `os.mkdir(path)` | 创建单级目录（父目录必须存在） | `os.mkdir('new_folder')` |
| `os.makedirs(path)` | 递归创建多级目录（父目录自动创建） | `os.makedirs('a/b/c')` |
| `os.rmdir(path)` | 删除空目录 | `os.rmdir('empty_folder')` |
| `os.removedirs(path)` | 递归删除空目录链 | `os.removedirs('a/b/c')`（逐级删除空的父目录） |
| `os.remove(path)` | 删除文件（非目录） | `os.remove('temp.txt')` |
| `os.unlink(path)` | 与 `remove` 相同（Unix 风格） | `os.unlink('temp.txt')` |
| `os.rename(src, dst)` | 重命名文件或目录，也可用于移动 | `os.rename('old.txt', 'new.txt')` |
| `os.renames(src, dst)` | 递归重命名，会自动创建目标路径的父目录 | `os.renames('a/b/old.txt', 'c/d/new.txt')` |
| `os.replace(src, dst)` | 类似于 rename，但会覆盖已存在的目标（跨平台更安全） | `os.replace('temp.txt', 'final.txt')` |
| `os.scandir(path)` | 返回迭代器，比 `listdir` 更高效（包含文件属性） | `with os.scandir('.') as it: ...` |
| `os.walk(top)` | 递归遍历目录树，生成 `(dirpath, dirnames, filenames)` 三元组 | `for root, dirs, files in os.walk('.'): ...` |
**示例：递归统计文件数量**
```python
count = 0
for root, dirs, files in os.walk('.'):
    count += len(files)
print(f"文件总数: {count}")
```


## 路径操作（`os.path` 子模块）
`os.path` 提供了大量操作路径字符串的函数，虽然现在推荐用 `pathlib`，但它在旧代码中仍广泛存在。

| 函数                         | 作用                    | 示例                                                                                  |
| -------------------------- | --------------------- | ----------------------------------------------------------------------------------- |
| `os.path.join(a, *p)`      | 智能拼接路径（自动适应操作系统分隔符）   | `os.path.join('dir', 'file.txt')` → `'dir/file.txt'`（Linux）或 `'dir\\file.txt'`（Win） |
| `os.path.basename(path)`   | 返回路径最后一部分（文件名或目录名）    | `os.path.basename('/a/b/c.txt')` → `'c.txt'`                                        |
| `os.path.dirname(path)`    | 返回路径的目录部分             | `os.path.dirname('/a/b/c.txt')` → `'/a/b'`                                          |
| `os.path.split(path)`      | 同时返回 (目录, 文件名) 元组     | `os.path.split('/a/b/c.txt')` → `('/a/b', 'c.txt')`                                 |
| `os.path.splitext(path)`   | 分离文件名和扩展名             | `os.path.splitext('image.png')` → `('image', '.png')`                               |
| `os.path.exists(path)`     | 判断路径是否存在（文件或目录）       | `os.path.exists('config.yaml')` → `True`                                            |
| `os.path.isfile(path)`     | 是否为文件                 | `os.path.isfile('data.txt')`                                                        |
| `os.path.isdir(path)`      | 是否为目录                 | `os.path.isdir('src')`                                                              |
| `os.path.islink(path)`     | 是否为符号链接               | `os.path.islink('/usr/bin/python')`                                                 |
| `os.path.getsize(path)`    | 获取文件大小（字节）            | `os.path.getsize('large.dat')`                                                      |
| `os.path.getmtime(path)`   | 获取最后修改时间（时间戳）         | `os.path.getmtime('file.txt')`                                                      |
| `os.path.abspath(path)`    | 返回绝对路径                | `os.path.abspath('..')` → `'/home/user'`                                            |
| `os.path.realpath(path)`   | 解析符号链接并返回真实路径         | `os.path.realpath('/usr/bin/python3')`                                              |
| `os.path.normpath(path)`   | 规范化路径（消除多余的分隔符和 `..`） | `os.path.normpath('a/../b/./c')` → `'b/c'`                                          |
| `os.path.expanduser(path)` | 展开 `~` 为当前用户 home 目录  | `os.path.expanduser('~/data')` → `'/home/user/data'`                                |
| `os.path.expandvars(path)` | 展开环境变量（如 `$HOME`）     | `os.path.expandvars('$HOME/data')`                                                  |

## 环境变量
通过 `os.environ` 字典式对象可以获取和修改环境变量。
```python
import os

# 获取环境变量
path = os.environ.get('PATH', '')
home = os.environ['HOME']  # 如果不存在会抛 KeyError

# 设置环境变量（影响当前进程及其子进程）
os.environ['MY_VAR'] = 'hello'

# 删除环境变量
del os.environ['TEMP']
```
**注意**：修改环境变量不会影响系统全局，只对当前 Python 进程及其启动的子进程有效。


## 进程管理
`os` 模块提供了创建、管理进程的相关函数（但更高级的进程管理通常使用 `subprocess` 模块）。

| 函数 | 作用 |
|------|------|
| `os.system(command)` | 执行 shell 命令，返回退出码（不推荐，用 `subprocess`） |
| `os.popen(command)` | 执行命令并返回文件对象（已过时，用 `subprocess`） |
| `os.exec*(path, args)` | 执行新程序，替换当前进程（exec 族函数） |
| `os.fork()` | 创建子进程（仅 Unix） |
| `os.getpid()` | 获取当前进程 ID |
| `os.getppid()` | 获取父进程 ID（Unix） |
| `os.kill(pid, sig)` | 向进程发送信号（Unix） |
| `os.wait()` | 等待子进程结束（Unix） |
| `os.spawn*(mode, path, ...)` | 创建新进程（跨平台简化版） |

## 系统信息与常数
`os` 模块提供了大量与操作系统相关的常量和信息。

| 函数/属性 | 作用 |
|-----------|------|
| `os.name` | 操作系统名称：`'posix'`（Linux/macOS）、`'nt'`（Windows）、`'java'`（Jython） |
| `os.uname()` | 返回系统信息（Unix-only，如内核版本、主机名等） |
| `os.cpu_count()` | CPU 核心数（逻辑核心） |
| `os.getloadavg()` | 系统负载（Unix） |
| `os.linesep` | 当前系统的换行符：`\n`（Unix）、`\r\n`（Windows） |
| `os.sep` | 路径分隔符：`/` 或 `\` |
| `os.devnull` | 空设备路径：`/dev/null` 或 `NUL` |
| `os.environ` | 环境变量字典（已说明） |
| `os.getenv(key, default)` | 获取环境变量，不存在返回默认值（字符串） |
| `os.putenv(key, value)` | 设置环境变量（不建议，推荐 `os.environ`） |


## 权限与属性
修改文件权限、所有者、时间戳等。

| 函数 | 作用 |
|------|------|
| `os.chmod(path, mode)` | 修改文件权限（mode 为八进制，如 `0o755`） |
| `os.chown(path, uid, gid)` | 修改文件所有者（Unix） |
| `os.utime(path, times)` | 修改文件的访问和修改时间 |
| `os.stat(path)` | 获取文件状态信息（大小、权限、时间戳等） |
| `os.lstat(path)` | 同 `stat`，但不跟随符号链接 |
**示例：读取文件元数据**
```python
stat_info = os.stat('file.txt')
print(f"大小: {stat_info.st_size} 字节")
print(f"最后修改时间: {stat_info.st_mtime}")
print(f"权限 (八进制): {oct(stat_info.st_mode & 0o777)}")
```

## 文件描述符与底层 I/O
一些高级操作直接操作文件描述符，用于底层编程。

| 函数                           | 作用                   |
| ---------------------------- | -------------------- |
| `os.open(file, flags, mode)` | 底层打开文件（返回文件描述符）      |
| `os.read(fd, n)`             | 从文件描述符读取数据           |
| `os.write(fd, str)`          | 写入数据到文件描述符           |
| `os.close(fd)`               | 关闭文件描述符              |
| `os.dup(fd)`                 | 复制文件描述符              |
| `os.pipe()`                  | 创建管道（返回读写文件描述符对）     |
| `os.fdopen(fd, mode)`        | 将文件描述符转为 Python 文件对象 |
大部分场景用内置的 `open()` 函数更简单。

## 其他实用功能
- `os.get_terminal_size()`：获取终端尺寸（列、行）。
- `os.getlogin()`：获取当前登录用户名（有时不准确）。
- `os.path.getatime()`、`os.path.getctime()`：获取访问时间、创建时间（平台差异）。
- `os.symlink(src, dst)`：创建符号链接（Unix）。
- `os.readlink(path)`：读取符号链接目标。
- `os.umask(mask)`：设置文件创建掩码。

# 跨平台注意事项
虽然 `os` 模块尽量屏蔽差异，但某些功能在不同平台上行为不同：
- **路径分隔符**：使用 `os.path.join` 而不是手工拼接 `/` 或 `\`。
- **符号链接**：Windows 需要管理员权限或开发模式才支持。
- **权限模型**：Windows 的权限模型不同于 Unix，`os.chmod` 效果有限。
- **进程管理**：`fork` 仅在 Unix 上可用，Windows 上应使用 `multiprocessing`。
- **编码问题**：文件路径可能包含非 ASCII 字符，建议使用 `os.fsencode()` 和 `os.fsdecode()` 处理。
