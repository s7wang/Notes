## 1 静态库

静态库在编译时把库转载进来，不占用程序的运行时间，但是容易造成代码的膨胀。一般为私有资源。

静态库

```txt
libxxx.a
xxx 指代库名
    
ar -cr libxxx.a yyy.o

发布到
/usr/local/include		.h存放目录
/usr/local/lib			.a存放目录

gcc -L/usr/local/lib -o main main.o -lxxx

```









## 2 动态库

动态库在声明时只是引用，只有用到库内的函数时才回去寻找库内资源，占用运行时间。一般为公共资源。

动态库

```txt
libxxx.so
    
gcc -shared -fPIC -o libxxx.so yyy.c
    
gcc -shared -fpic -o libxxx.so yyy.c

发布到
/usr/local/include		.h存放目录
/usr/local/lib			.a存放目录 
    
在 /etc/ld.so.conf 中添加路径
/sbin/ldconfig	重读 /etc/ld.so.conf

标准编译：
gcc -I/usr/local/include -L/usr/local/lib -o ... -lxxx

非 root 用户发布
cp libxxx.so ~/lib
export LD_LIBRARY_PATH=~/lib


```

如果有重名的静态库和动态库会优先链接动态库。编译选项 `-lxxx` 被依赖的库永远在后边。



## 3 make

makefile/Makefile工程管理文件，用来分析依赖和被依赖的关系。

Makefile 的变量：

* 自定变量：

自己定义变量代替文本内容，类似C语言的宏定义。

```txt
变量名=变量值	递归变量展开（几个变量共享一个值）
变量名:=变量值 简单变量展开（类似 C++ 的赋值）

$(变量名) 
```

* 自动变量

在使用的时候，自动用特定的值替换。

| 变量  | 说明                                                    |
| ----- | ------------------------------------------------------- |
| $@    | 当前规则的目标文件                                      |
| $<    | 当前规则的第一个依赖文件                                |
| $^    | 当前规则的所有依赖文件                                  |
| $?    | 规则中日期新于目标文件的所有相关文件列表，逗号分隔      |
| $(@D) | 目标文件的目录部分，若\$@为add/add.c，则 \$(@D)为 add   |
| $(@F) | 目标文件的文件部分，若\$@为add/add.c，则 \$(@F)为 add.c |

* 预定义变量 和 环境变量

内部事先定义好的变量，但是它的值是固定的，并且有些值是为空的。（系统设置的自定义变量）

使用 `export` 查看系统变量。

| 变量     | 描述                          |
| -------- | ----------------------------- |
| AS       | 汇编程序，默认为 as           |
| CC       | c 编译器，默认为  cc          |
| CPP      | c 预编译器，默认为 \$(CC)  -E |
| CXX      | c++ 编译器，默认为 g++        |
| RM       | 删除，默认为 rm -f            |
| ARFLAGS  | 库选项，无默认                |
| ASFLAGS  | 汇编选项，无默认              |
| CFLAGS   | c 编译器选项，无默认          |
| CPPFLAGS | c 预编译器选项，无默认        |
| CXXFLAGS | c++ 编译器选项，无默认        |
| LDFLAGS  | 动态链接选项                  |



> （1）隐藏规则：*.o 文件自动依赖 *.c 文件，可以省略 main.o:main.c 等。
>
> （2）模式规则：通过匹配模式找字符串，%匹配1个或多个任意字符串。另外还能将指定的文件编译到指定的目录中：
>
> ```makefile
> SOURCES:=$(wildcard *.c)
> 
> OBJS=$(patsubst %.c,%.o,$(SOURCES))
> 
> CC=gcc # 修改预定义变量
> DIR:=./Debug/
> NEWOBJS=$(addprefix $(DIR),$(OBJS))
> 
> main:$(NEWOBJS)
> 	$(CC)  $^ -o $(DIR)$@
> 
> $(DIR)%.o:%.c
> 	$(CC) -c $^ -o $@
> 	
> .PHONY:clean rebuild
> clean:
> 	$(RM) $(DIR)*.o $(DIR)main
> 
> rebuild:clean main
> ```
>
> （3）函数：
>
> ```makefile
> # 搜索当前目录下文件名，展开成一列所有符合由其参数描述的文件名，文件名间以空格间隔。
> SOURCES=$(wildcard *.c) # 把当前目录中所有的.c文件存入变量 SOURCES 中。
> 
> # 将源字符串（以空格分隔）中的所有要查找的子串替换成目标子串。
> OBJS=$(patsubst %.c,%.o,$(SOURCES))# 把 SOURCES 中的.c都替换为.o然后存入变量 OBJS 中。
> 
> # 将第二个参数列表的每一项前缀加上第一个参数值。
> $(addprefix 前缀,源字符串)
> 
> ```
>
> 

* 命令前面加@不显示命令





```makefile
# 1.#是注释
# 第一次：显式规则
# 语法格式
# 目标文件：依赖文件
############################################################
# 参数
OBJS=main.o tool1.o tool2.o # 自定义变量
CFLAGS+=-Wall -g # 预定义变量
CC=gcc # 修改预定义变量
# RM = rm -f # 预定义变量
DIR:=./Debug/

target:mytool
###############################
# mytool:$(OBJS)
# 	$(CC) $^ -o $@
# %.o:%.c #模式规则
# 	$(CC) $^ $(CFLAGS) -o $@
###############################
# 14-17 行可改为一下内容
$(DIR)mytool:$(DIR)$(OBJS) # 隐含规则 并 生成到指定目录下
	$(CC) $^ $(CFLAGS) -o $@ # 自动变量 $@替换 mytool; $^替换$(OBJS)


#Makefile 的伪目标：
.PHONY:clean rebuild
clean:
	$(RM) $(DIR)*.o $(DIR)mytool

rebuild:clean target
	

```

