# Makefile 基础

[TOC]

## 1 简介

名称：Makefile

```makefile
# Makefile 工程管理工具：帮助我们实现项目的自动编译。分析文件的依赖和被依赖关系。
# 命名规则 Makefle 或 makefile 两种都可以，如果同时存在优先读取 makefile

target:files
	cmd:
```



示例：

```makefile
# 最简单的写法

mytool:main.o tool1.o tool2.o
	gcc main.o tool1.o toll2.o -o mytoll
	
main.o:main.c
	gcc main.c -c -Wall -g -o main.o

tool1.o:tool1.c
	gcc tool1.c -c -o tool1.o

tool2.o:tool2.c
	gcc tool2.c -c -o tool2.o

clean:
	rm *.o mytool -rf
```

```makefile
# 改进写法1

OBJS=main.o tool1.o tool2.o
CC= gcc
# RM = rm -rf

CFLAGS+=-c -Wall -g

target:$(OBJS)
	$(CC) $(OBJS) -o mytool

main.o:main.c
	$(CC) main.c $(CFLAGS) -o main.o

tool1.o:tool1.c
	$(CC) tool1.c $(CFLAGS) -o tool1.o

tool2.o:tool2.c
	$(CC) tool2.c $(CFLAGS) -o tool2.o

clean:
	$(RM) *.o mytool 
	
```

```makefile
# 改进写法2

OBJS=main.o tool1.o tool2.o
CC= gcc
# RM = rm -rf

CFLAGS+=-c -Wall -g

mytool:$(OBJS)
	$(CC) $^ -o $@

main.o:main.c
	$(CC) $^ $(CFLAGS) -o $@

tool1.o:tool1.c
	$(CC) $^ $(CFLAGS) -o $@

tool2.o:tool2.c
	$(CC) $^ $(CFLAGS) -o $@

clean:
	$(RM) *.o mytool 

############################################################

# 改进写法2

OBJS=main.o tool1.o tool2.o
CC= gcc
# RM = rm -rf

CFLAGS+=-c -Wall -g

mytool:$(OBJS)
	$(CC) $^ -o $@

%.o:%.c
	$(CC) $^ $(CFLAGS) -o $@


clean:
	$(RM) *.o mytool 
```



## 2 Linux 微观编译

​	-E	预编译

​	-S	汇编

​	-c	二进制

​	-o	输出



win		.exe

Linux	.elf



1. 预编译处理：xxx.i 预编译结果文件。 gcc -E hello.c -o hello.i
2. 汇编处理：xxx.S 汇编结果。 gcc -S hello.i -o hello.S
3. 编译：xxx.o 二进制文件。 gcc -c hello.S -o hello.o
4. 链接：gcc hello.o -o hello





## 3 Makefile 脚本语法

使用 Makefile 对多文件 c 工程快速编译。

### 3.1

```makefile
# 1.#是注释
# 第一次：显式规则
# 语法格式
# 目标文件：依赖文件


#Makefile 的伪目标：
.PHONY:clean
clean:
	rm -rf *.o target



```







