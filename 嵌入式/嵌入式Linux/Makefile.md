# 核心语法
```makefile
目标 : 依赖
	命令   # 必须以 Tab 开头，不能是空格
```
**规则含义**：要生成 `目标`，必须先有 `依赖`；如果 `依赖` 比 `目标` 新(或目标不存在)，则执行 `命令`

# 变量与自动变量
## 变量的定义与使用
```makefile
# 定义变量
CC = gcc
CFLAGS = -Wall -g -O2
TARGET = myapp
OBJS = main.o utils.o

# 使用变量
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^
```
## 自动变量
| 自动变量 | 含义            |
| ---- | ------------- |
| `$@` | 目标的完整名称       |
| `$<` | 第一个依赖文件       |
| `$^` | 所有依赖文件（去重）    |
| `$+` | 所有依赖文件（不去重）   |
| `$*` | 目标的主文件名（不含后缀） |
| `$?` | 所有比目标新的依赖文件   |
```makefile
main.o: main.c utils.h
	gcc -c $< -o $@
# $< = main.c, $@ = main.o
```

# 函数
## 字符串函数
```makefile
# 替换后缀
SRCS = main.c utils.c
OBJS = $(SRCS:.c=.o)          # 替换 .c 为 .o

# 查找文件
SRCS = $(wildcard *.c)         # 获取所有 .c 文件
OBJS = $(patsubst %.c,%.o,$(SRCS))  # 模式替换

# 去除空格
STR =  a b c 
STR = $(strip $(STR))          # 结果为 "a b c"

# 过滤
C_SRCS = $(filter %.c, $(SRCS))        # 只保留 .c
ASSEMBLY_SRCS = $(filter %.s, $(SRCS))  # 只保留 .s

# 去除重复
UNIQ = $(sort $(LIST))          # 排序并去重
```
## 文件函数
```makefile
# 判断文件是否存在
ifeq ($(wildcard config.h),)
	$(error config.h not found)
endif

# 获取路径
DIR = $(dir $(FILE))            # 获取文件所在目录
BASE = $(basename $(FILE))      # 获取去掉后缀的文件名
```

## 条件判断
```makefile
ifeq ($(CC), gcc)
	CFLAGS += -Wall -g
else
	CFLAGS += -O2
endif

ifdef DEBUG
	CFLAGS += -DDEBUG
endif

ifndef RELEASE
	CFLAGS += -g
endif

# 也可以使用
ifeq ($(ARCH), x86_64)
	CFLAGS += -m64
else ifeq ($(ARCH), arm)
	CFLAGS += -marm
endif
```

# 模式与隐含规则
## 模式规则
```makefile
# 把任何 .c 编译成对应的 .o
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 静态模式规则（更精确）
$(OBJS): %.o: %.c %.h
	$(CC) $(CFLAGS) -c $< -o $@
```

## 隐含规则
```makefile
# 默认：
# .c -> .o : $(CC) -c $(CFLAGS) $(CPPFLAGS)
# .c -> 可执行文件 : $(CC) $(LDFLAGS) $(CFLAGS) $^ $(LDLIBS)
```

# 伪目标
```makefile
.PHONY: clean install all test

# 总是执行，不检查依赖是否新
clean:
	rm -f $(OBJS) $(TARGET)

install:
	cp $(TARGET) $(INSTALL_DIR)

all: $(TARGET)

test: $(TARGET)
	./$(TARGET) --test
```