Shell 脚本就是把一系列 Shell 命令写入一个文件，并赋予执行权限，从而实现**批量处理、自动化任务**

==**注意**:必须以`#!/bin/bash`开头==

# 简单`Shell`脚本
```bash
#!/bin/bash
# 这是第一个脚本
echo "Hello, Embedded World!"
```
执行方式:
```bash
# 方法一：赋予执行权限后直接运行
chmod +x hello.sh
./hello.sh

# 方法二：使用解释器执行（无需执行权限）
bash hello.sh
```
- `chmod`赋予权限, `./`执行
- 直接用解释器`bash`执行


# 交互式`Shell`脚本
**交互式 Shell** 是指用户通过终端直接与 Shell 进行实时命令交互的工作模式。它的核心特征是：**等待用户输入 → 立即执行 → 显示结果 → 再次等待输入**，形成一个连续的“对话”循环。

## `read`命令
`read` 是 `Shell` 中用于从标准输入（键盘）读取用户输入的内置命令。
```bash
# 最基础的用法
read name                # 等待用户输入，存入变量 name
echo "Hello, $name"
```
### 常用选项
| 选项          | 作用                   |
| ----------- | -------------------- |
| `-p "提示文字"` | 显示提示信息，输入在同一行        |
| `-s`        | 静默模式（输入不显示，适合密码）     |
| `-t 秒数`     | 超时时间（秒），超时后自动退出      |
| `-n N`      | 只读 N 个字符，输入 N 个后自动继续 |
| `-r`        | 禁止反斜杠转义（保留原始输入）      |
| `-d 字符`     | 以指定字符作为结束符（默认是换行）    |
| `-a 数组名`    | 将输入存入数组              |

## 简单示例
```bash
# 带提示（-p），输入和提示在同一行
read -p "Enter your name: " name
echo "Hello, $name"

# 静默输入（-s），常用于密码
read -s -p "Enter password: " pwd
echo    # 换行
echo "Password entered"

# 超时（-t），5秒内无输入则退出
read -t 5 -p "You have 5 seconds to answer: " ans
if [ -z "$ans" ]; then
    echo "Timeout!"
fi

# 只读1个字符（-n 1），按任意键继续
read -n 1 -p "Press any key to continue..."
echo "Continuing..."

# 读取整行原始输入（-r，不解析反斜杠）
read -r line
```

# `Shell`脚本的数值计算
## 语法
`$((...))` : 这是`POSIX`兼容的算术扩展语法
```bash
result=$((表达式))
```

## 支持的运算符
|类型|运算符|
|---|---|
|加减乘除|`+` `-` `*` `/`|
|取模（余数）|`%`|
|幂运算|`**`|
|自增/自减|`++` `--`|
|复合赋值|`+=` `-=` `*=` `/=` `%=`|
|位运算|`&` `\|` `^` `~` `<<` `>>`|
|逻辑运算|`&&` `\|` `!`|
|比较运算|`<` `>` `<=` `>=` `==` `!=`|
|三目运算|`条件 ? 值1 : 值2`|

## 简单示例
```bash
#!/bin/bash

a=10
b=3

# 基本运算
echo $((a + b))      # 13
echo $((a - b))      # 7
echo $((a * b))      # 30
echo $((a / b))      # 3（整数除法，向下取整）
echo $((a % b))      # 1（取余）

# 幂运算
echo $((2 ** 10))    # 1024

# 复合赋值
((a += 5))           # a = 15
((b *= 2))           # b = 6

# 自增自减
((i++))              # 先使用后自增
((++i))              # 先自增后使用
((i--))
((--i))

# 比较（返回 1 为真，0 为假）
echo $((a > b))      # 1
echo $((a == b))     # 0

# 逻辑运算
echo $((a > 0 && b > 0))   # 1

# 三目运算
max=$((a > b ? a : b))
echo $max
```


# `test`命令
`test` 是 Shell 中用于**条件判断**的内置命令。它通常不会产生任何输出，而是通过**退出状态码（`$?`）** 来告诉脚本：**条件为真（返回 0）还是为假（返回 1）**. 它是 `if`, `while`, `until` 等流程控制语句的**核心基础**

## 基本语法
`test` 命令有三种等价写法：

| 写法          | 说明                        |
| ----------- | ------------------------- |
| `test 表达式`  | 标准写法                      |
| `[ 表达式 ]`   | 最常用，注意**括号和表达式之间必须有空格**   |
| `[[ 表达式 ]]` | Bash 增强版（支持更多特性，但仅限 Bash） |
```bash
# 下面三种写法完全等价
test -f /etc/passwd
[ -f /etc/passwd ]
[[ -f /etc/passwd ]]   # Bash 特有
```
**重要**：`[` 实际上是一个命令（`/usr/bin/[`），所以它后面必须加空格
```bash
#!/bin/bash
read str1
read str2
test $str1 == $str2
[ $str1 == $str2 ]  # 与上面等价
```

## 退出状态码
`test` 通过 `$?` 返回结果：
- `0`：条件为真（True）
- `1`：条件为假（False）
```bash
[ -f /etc/passwd ]
echo $?        # 输出 0，表示文件存在

[ -f /nonexist ]
echo $?        # 输出 1，表示文件不存在
```
**在实际脚本中，通常直接在 `if` 中使用，不需要手动检查 `$?`**：
```bash
if [ -f /etc/passwd ]; then
    echo "File exists"
fi
```
**类三目运算符**
```bash
#!/bin/bash
read -p "please input your filename: " filename
test -e $filename && echo "$filename exist" || echo "$filename does not exists"

```

## `test`文件常用选项
| 选项          | 含义              | 示例                            |
| ----------- | --------------- | ----------------------------- |
| `-e`        | 文件或目录存在         | `[ -e /etc/passwd ]`          |
| `-f`        | 存在且为普通文件        | `[ -f /etc/passwd ]`          |
| `-d`        | 存在且为目录          | `[ -d /home ]`                |
| `-h` / `-L` | 存在且为符号链接        | `[ -L /usr/bin/python ]`      |
| `-b`        | 存在且为块设备文件       | `[ -b /dev/sda ]`             |
| `-c`        | 存在且为字符设备文件      | `[ -c /dev/tty ]`             |
| `-S`        | 存在且为 Socket     | `[ -S /var/run/docker.sock ]` |
| `-p`        | 存在且为管道（FIFO）    | `[ -p /tmp/mypipe ]`          |
| `-r`        | 存在且可读           | `[ -r /etc/shadow ]`          |
| `-w`        | 存在且可写           | `[ -w /tmp ]`                 |
| `-x`        | 存在且可执行          | `[ -x /bin/ls ]`              |
| `-s`        | 存在且文件大小 > 0     | `[ -s /var/log/syslog ]`      |
| `-O`        | 文件所有者是当前用户      | `[ -O /home/user/file ]`      |
| `-G`        | 文件组是当前用户组       | `[ -G /home/user/file ]`      |
| `-nt`       | 文件1 比 文件2 新（时间） | `[ file1 -nt file2 ]`         |
| `-ot`       | 文件1 比 文件2 旧     | `[ file1 -ot file2 ]`         |
| `-ef`       | 两个文件是同一个（硬链接）   | `[ file1 -ef file2 ]`         |


# 默认变量
在 `Shell` 脚本中，有一类变量是**系统预定义**的，它们不需要手动赋值，而是由 `Shell` 自动维护，用于获取脚本运行时的各种信息（如参数、进程 `ID`、退出状态等）。这类变量称为**默认变量**或**特殊变量**

## 位置参数
用于获取从命令行传递给脚本或函数的参数。

|变量|含义|
|---|---|
|`$0`|脚本或函数的**名称**（执行路径）|
|`$1`|第 1 个参数|
|`$2`|第 2 个参数|
|`$3` ... `$9`|第 3 到第 9 个参数|
|`${10}`|第 10 个参数（9 以上必须用花括号）|
|`${N}`|第 N 个参数（N 为任意正整数）|
```bash
#!/bin/bash
# 脚本名为 test.sh

echo "脚本名称: $0"
echo "第一个参数: $1"
echo "第二个参数: $2"
echo "第三个参数: $3"
echo "第十个参数: ${10}"
```
```bash
$ ./test.sh a b c d e f g h i j
脚本名称: ./test.sh
第一个参数: a
第二个参数: b
第三个参数: c
第十个参数: j
```

# 参数数量与展开
| 变量   | 含义                  |
| ---- | ------------------- |
| `$#` | 参数的**个数**（不包括 `$0`） |
| `$*` | 所有参数作为一个**整体字符串**   |
| `$@` | 所有参数作为**独立的多个字符串**  |
- `"$*"`：将所有参数视为**一个整体**，常用于打印全部参数
- `"$@"`：将每个参数视为**独立个体**，常用于遍历参数（推荐使用）
```bash
#!/bin/bash
# test.sh

echo "--- 使用 \$* ---"
for arg in "$*"; do
    echo "  $arg"
done

echo "--- 使用 \$@ ---"
for arg in "$@"; do
    echo "  $arg"
done
```
执行结果: 
```bash
$ ./test.sh hello "world of" shell

--- 使用 $* ---
  hello world of shell          # 所有参数合并成一个字符串

--- 使用 $@ ---
  hello                         # 每个参数独立处理
  world of
  shell
```
## 进程与状态变量
|变量|含义|
|---|---|
|`$$`|当前 Shell 进程的 PID|
|`$!`|最近一个**后台进程**的 PID|
|`$?`|上一条命令的**退出状态码**（0 表示成功，非 0 表示失败）|
|`$-`|当前 Shell 的**选项标志**（如 `i` 表示交互式 Shell）|
|`$_`|上一条命令的**最后一个参数**|


# 条件判断
## `if`语句
```bash
if 条件; then
	# 条件为真时执行
fi
```
- `if` 和 `then` 可以写在同一行（用 `;` 分隔），也可以分两行写
- `fi` 是 `if` 的反写，表示结束
**完整结构** : 
```bash
if 条件1; then
    # 条件1为真时执行
elif 条件2; then
    # 条件1为假且条件2为真时执行
elif 条件3; then
    # 前面条件都为假且条件3为真时执行
else
    # 所有条件都为假时执行
fi
```

### `if`条件的写法
#### test 与 [ ]
```bash
if [ -f "/etc/passwd" ]; then
    echo "文件存在"
fi

if [ "$name" = "root" ]; then
    echo "是 root 用户"
fi
```
- 变量一定要加双引号: "$name"

#### 算术表达式 (( ))
```bash
a=10
b=20

if (( a > b )); then
    echo "a > b"
elif (( a == b )); then
    echo "a == b"
else
    echo "a < b"
fi
```

## `case` 语句
当有多个固定值需要匹配时，`case` 比 `if-elif` 更清晰。
### 语法
```bash
case 变量 in
    模式1)
        命令
        ;;
    模式2)
        命令
        ;;
    模式3)
        命令
        ;;
    *)
        默认命令
        ;;
esac
```
- 每个分支以 `;;` 结束
- `*)` 是默认分支，匹配所有未被匹配的值
- 模式支持通配符：`*`、`?`、`[abc]` 等

### 简单示例
```bash
#!/bin/bash

read -p "输入一个选项 (start/stop/restart): " action

case $action in
    start|Start|START)
        echo "启动服务..."
        # 启动命令
        ;;
    stop|Stop|STOP)
        echo "停止服务..."
        # 停止命令
        ;;
    restart|Restart|RESTART)
        echo "重启服务..."
        # 重启命令
        ;;
    *)
        echo "无效选项: $action"
        echo "可用: start, stop, restart"
        exit 1
        ;;
esac
```

## `do - done`语句
### `for`循环
```bash
for i in 1 2 3 4 5; do
    echo "数字: $i"
done
```
输出: 
```bash
数字: 1
数字: 2
数字: 3
数字: 4
数字: 5
```

### `while` 循环
```bash
count=0
while [ $count -lt 5 ]; do
    echo "计数: $count"
    ((count++))
done
```
输出: 
```bash
计数: 0
计数: 1
计数: 2
计数: 3
计数: 4
```

### 循环控制命令
#### `break`- 立刻退出循环
```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo "数字: $i"
done
# 输出：1 2 3 4（到 5 时退出）
```

#### `continue` -  跳过本次循环
```bash
for i in {1..5}; do
    if [ $i -eq 3 ]; then
        continue
    fi
    echo "数字: $i"
done
# 输出：1 2 4 5（跳过了 3）
```

#### `exit` - 退出整个脚本
```bash
for i in {1..5}; do
    if [ $i -eq 3 ]; then
        echo "遇到 3，退出脚本"
        exit 1
    fi
    echo "数字: $i"
done
echo "这行不会被执行"
```
