# 文件打开关闭
## 打开文件 `fopen`
```c
#include <stdio.h>

FILE *fopen(const char *filename, const char *mode);
```
**参数**：
- `filename`：文件路径（相对或绝对路径）
- `mode`：打开模式（见下表）
**返回值**：
- 成功：返回 `FILE*` 指针
- 失败：返回 `NULL`，并设置 `errno`

## 文件打开模式
| 模式     | 含义        | 文件不存在 | 文件存在 | 初始位置 |
| ------ | --------- | ----- | ---- | ---- |
| `"r"`  | 只读        | 失败    | 打开   | 文件开头 |
| `"w"`  | 清空写       | 创建    | 清空   | 文件开头 |
| `"a"`  | 追加        | 创建    | 打开   | 文件末尾 |
| `"r+"` | 读写（读+写）   | 失败    | 打开   | 文件开头 |
| `"w+"` | 清空读写（写+读） | 创建    | 清空   | 文件开头 |
| `"a+"` | 读写（追加+读）  | 创建    | 打开   | 文件末尾 |

## 关闭文件 `fclose`
```c
int fclose(FILE *stream);
```
- 成功返回 `0`，失败返回 `EOF`
- 关闭前自动刷新缓冲区

# 文件写入
## 写入字节 `fputc`
```c
int fputc(int ch, FILE *stream);
```
- 写入一个字符到文件
- 成功返回写入的字符（`unsigned char` 转换为 `int`）
- 失败返回 `EOF`

## 写入字符串 `fputs`
```c
int fputs(const char *str, FILE *stream);
```
- 写入字符串到文件（**不自动添加换行**）
- 成功返回非负数，失败返回 `EOF`

## 格式化写入`fprintf`
```c
int fprintf(FILE *stream, const char *format, ...);
```
- 和 `printf` 用法相同，但输出到文件
- 成功返回写入的字符数，失败返回负数


# 文件读取
## 读取字节 `fgetc`
```c
int fgetc(FILE *stream);
```
- 从文件中读取一个字符
- 成功返回读取的字符（`unsigned char` 转换为 `int`）
- 到达文件末尾或出错返回 `EOF`
```c
FILE *fp = fopen("test.txt", "r");
int ch;
while ((ch = fgetc(fp)) != EOF) {
    putchar(ch);    // 输出到终端
}
fclose(fp);
```

## 读取字符串 `fgets`
```c
char *fgets(char *str, int n, FILE *stream);
```
- 从文件读取最多 `n-1` 个字符到 `str`，并自动添加 `\0`
- 遇到换行符 `\n` 时停止（包含换行符）
- 到达文件末尾时停止
- 成功返回 `str`，失败或到达末尾返回 `NULL`

## 格式化读取 `fscanf`
```c
int fscanf(FILE *stream, const char *format, ...);
```
- 和 `scanf` 用法相同，但从文件读取
- 成功返回成功匹配并赋值的参数个数
- 失败或到达末尾返回 `EOF`

# 标准输入输出和错误
## 标准输入 `stdin`
默认从键盘读取，可以通过重定向从文件读取。

## 标准输出 `stdout`
默认输出到终端，可以通过重定向输出到文件。

## 标准错误 `stderr`
与 `stdout` 不同：
- **默认无缓冲**（立即输出）
- 用于错误信息，不会被重定向掩盖（`stdout` 可能被重定向到文件，`stderr` 仍输出到终端）