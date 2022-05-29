# C语言基础




[TOC]

## 1 gcc vim

### 编译器gcc

实例程序：hello.c

```c
#include <stdio.h>
#include <stdlib.h>

int main(void){

	printf("Hello world!\n");
	
	return 0;
}
```
hello.c ：编译器 gcc
C源件—预处理—编译—汇编—链接—可执行文件

 预处理：hello.i 预处理文件
```
[wangs7@c]$gcc -E hello.c > hello.i
```
编译：hello.s 汇编文件
```
[wangs7@c]$gcc -S hello.i
[wangs7@c]$ls
[wangs7@c]$hello.c hello.i hello.s
```

汇编：hello.o 汇编编译文件

```
[wangs7@c]$gcc -c hello.s
[wangs7@c]$ls
[wangs7@c]$hello.c hello.i hello.s hell.o
```

链接：

```
[wangs7@c]$gcc hello.o -o hello
[wangs7@c]$ls
[wangs7@c]$hello hello.c hello.i hello.s hell.o
```

执行：

```
[wangs7@c]$./hello
Hello world!
[wangs7@c]$
```

简化方式

```
[wangs7@c]$gcc hello.c -o hello
```

### make

编译：自动寻找文件名.c的文件

```
[wangs7@c]$make hello
cc	hello.c	-o hello
[wangs7@c]$
```

### 编辑器vim

配置文件目录：/etc/vimrc



## 2 基本概念

### 2.1程序思路

打印出来本程序的所有警告：

```
[wangs7@c]$gcc hello.c -o hello -Wall
```

<B>必须将所有不可解释的警告消除；</B>

1、头文件包含的正确性

输出错误：（案例为段错误，错误为错误处理输出函数）

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
//#includ <string.h> 加入后无误

int main(){
    FILE *p;
    
    fp = fopen("temp","r");//temp为不存在文件
    if(fp == NULL){
        //strerror()错误输出函数
        fprintf(stderr,"fopen():%s\n",strerror(errno));
        exit(-1);
    }
    
    puts("OK!");
    return 0;
}
```



2、以函数为单位实现功能

3、声明部分 + 实现部分

4、return 0;

5、多用空格空行

6、注释

```c
/*
 *多行注释
 *
 */

//单行注释

#if 0//注释代码段

#endif
```



### 2.2 算法

算法：解决问题的方法。

* 流程图
* NS图
* 有限状态机 (FSM)



### 2.3 程序

程序：用某种语言实现算法。

### 2.4 进程

进程：防止写越界，防止内存泄露，谁打开谁关闭，谁申请谁释放。



## 3 数据类型、运算符、表达式



### 3.1 数据类型

略

1.浮点型数据为不精确数据不能进行判断相等。

2.数据类型要相匹配。

### 3.2 变量与常量

常量：执行过程中不发生变化。

分类：整型常量，实型常量，字符常量，字符串常量，标识常量。

> 字符常量：单引号内的单个字符或转义字符。
>
> 字符串常量：双引号内的字符序列。
>
> 标识常量：`#define` 在预编译过程中进行宏替换。！！！可带参，不进行语法检查

变量：随时会发生变化，用来保存特殊内容。

定义： [存储类型]	数据类型	标识符	=	值

​                                       TYPE	NAME	=	VALUE;

 标识符：一个标识序列，尽量见文生义。

数据类型：基本数据类型 + 构造类型

存储类型：`auto` 	`static`	`register`	`extern` （说明型）

> `auto`：默认，自动分配空间，自动回收空间。
>
> `register`：（建议型）寄存器类型，资源少速度快。由gcc决定，只能定义局部变量，不能定义全局变量；大小限制，只能定义32为大小的数据类型；没有地址，所以无法查看地址。
>
> `static`：静态型，自动初始化为0或空值，并变量值有继承性。常用修饰变量或函数。防止变量冲突或防止函数对外扩展。
>
> `extern`：说明型，不能改变说明的变量的值或类型。

变量的生命周期和作用范围：

* 全局变量和局部变量；
* 生命周期和作用范围；内部作用范围屏蔽外部作用范围。

### 3.3 运算符和表达式

表达式与语句的区别：

带分号语句，不带分号表达式。

运算符部分：

> 1. 每个运算符需要参与运算的数量
> 2. 结合性
> 3. 优先级
> 4. 运算符的特殊用法（逻辑及运算符（&&   ||）的短路特性）
> 5. 位运算的重要意义



## 4 输入输出

input & output -> I/O; （标准IO ，文件IO）

### 4.1格式化输入输出

> printf:   
>
> ```c
> int printf(const char* format, ...);
> //printf("%[修饰符]格式字符",输出项); 返回输出字符数量
> \n //刷新缓冲区
> 
> /*
> *	修饰符
> *	m		指定输出宽度，左补空
> *	.n		指定小数为长度，字符串输出位数
> *	-		左对齐右补空格
> *	+		正数自动显示+
> *	#		自动添加前导符0,0x
> *	l		long 或 double
> */
> 
> 5L//long类型的5
> 324LL//long long 类型的324
> 
> ```
>
> scanf:
>
> ```c
> int scanf(const char* format, ...);
> //scanf("%[修饰符]格式字符",地址表);
> //%s 接收十分危险 容易中断输入或者溢出
> //校验返回值 1成功 否则失败
> //* 抑制符
> scanf("%d%*c%c",a,ch);//吃掉两次输入中的空格/回车
> ```
>
> 



### 4.2 字符输入输出

> getchar:
>
> ```c
> int getchar(void);
> ```
>
> putchar:
>
> ```c
> int putchar(int char);
> //C 库函数 int putchar(int char) 把参数 char 指定的字符（一个无符号字符）写入到标准输出 stdout 中。
> ```
>
> gets:
>
> ```c
> char *gets(char *str);
> //不对缓冲区检查，函数不安全  用fgets或getline代替
> ```
>
> puts:
>
> ```c
> //略
> ```
>
> 

## 5 流程控制

顺序，选择，循环

简单结果与复杂结构：自然流程

顺序：语句逐句执行

选择：出现一种以上的情况

循环：在某个条件成立情况下，重复执行某个动作

关键字：

> 选择：`if-else`	`switch-case`
>
> 循环：`while`	`for`	`if-goto`
>
> 辅助控制：`continue`（本次）	`break`（本层）	`default`

杀掉死循环：`ctrl`+`c` 



## 6 数组

### 6.1 一维数组

1. 定义：[存储类型]	数据类型	标识符[下标长度]

```c
#define	M	3
[auto] int arr[M];
```

2. 初始化：

* 不初始化元素内容随机

* 全部初始化

* 部分初始化

3. 元素引用：数组名[下标]

4. 数组名和数组越界：

5. 排序算法：冒泡，选择，快速；

* 冒泡排序

```c
#include <stdio.h>
#include <stdlib.h>

# define	N	10
/*
 *冒泡排序：每次较大的放后边
 *
 */
static void sort1(){
    
    int a[N] = {12,8,45,30,98,67,2,7,68,11}
    inttemp;
    for(int i=0;i<(N-1);i++){
        for(int j=0;j<(N-1-i);j++){
            if(a[j]>a[j+]){
                temp = a[j+1];
                a[j+1] = a[j];
                a[j] = temp;
            }
        }
    }
}

int main(){
    
    sort1();
    
    return 0;
}
```

* 选择排序

```c
#include <stdio.h>
#include <stdlib.h>

# define	N	10
/*
 *选择排序：每次选择最小的数放在 最 前面
 *
 */
static void sort2(){
    
    int a[N] = {12,8,45,30,98,67,2,7,68,11}
    int temp, k;
    for(int i=0;i<N-1;i++){
        k = i;
        for(int j=i+1;j<N;j++){
            if(a[j]<a[j+1]){
                
            }
        }
        if(k != i){
            temp = a[i];
            a[i] = a[k];
            a[k] = temp;
        }
    }
    
}

int main(){
    
    sort2()
    
    return 0;
}
```



### 6.2 二维数组

1. 定义，初始化

```c
[存储类型]	数据类型	标识符[行下标长度][列下标长度]
#define	X	2
#define	Y	3    
[auto] int arr[X][Y]; 
int a[X][Y] = {{1,2,3},{4,5,6}};
int b[X][Y] = {{1,2,3}};//第二行自动赋值0
//可以省略行号，列号不可省
```

2. 元素引用：
3. 存储形式：顺序存储，按行存储
4. 深入理解：如何行列互换，求最大值及其所在位置，矩阵乘积。

```c
arr + 1 //等价arr[1] 而不是 arr[0][1]
```



### 6.3 字符数组

1. 定义，初始化，存储特点。

```c
#define STRSIZE 10
char str[STRSIZE];
char str[STRSIZE] = {'a','s','d'};
char str[STRSIZE] = "hello";
```

2. 输入输出：不要用 `gets()` 函数不检查缓冲区长度，不安全。
3. 常用函数：

```c
#include <string.h>

//返回字符长度不包括尾0
size_t strlen(const char *str);
size_t sizeof(type);//返回空间大小byte

//把 src 所指向的字符串复制到 dest。
char *strcpy(char *dest, const char *src);
char *strncpy(char *dest, const char *src, size_t n)//把 src 所指向的字符串复制到 dest，最多复制 n 个字符。当 src 的长度小于 n 时，dest 的剩余部分将用空字节填充。
    
//把 src 所指向的字符串追加到 dest 所指向的字符串的结尾。
char *strcat(char *dest, const char *src);
char *strncat(char *dest, const char *src, size_t n); //把 src 所指向的字符串追加到 dest 所指向的字符串的结尾，直到 n 字符长度为止。

/*
 *把 str1 所指向的字符串和 str2 所指向的字符串进行比较。
 *如果返回值小于 0，则表示 str1 小于 str2。
 *如果返回值大于 0，则表示 str1 大于 str2。
 *如果返回值等于 0，则表示 str1 等于 str2。
 *返回值为ASCII的差
 */
int strcmp(const char *str1, const char *str2);
int strncmp(const char *str1, const char *str2, size_t n);//只比较前n个。

```



## 7 指针

### 7.1.1 变量与地址

### 7.1.2 指针与指针变量
```c
int i = 1;
&i//指针常量
int *p = &i;
p//指针变量
```
### 7.1.3 直接访问与间接访问
```c
int i = 1;
int *p = &i;
printf("i = %d\n",i);//直接访问
printf("i = %d\n",*p);//间接访问
//指针变量所占空间固定，但取内容运算时与指针类型有关
```
### 7.1.4 空指针与野指针
```c
int i = 1;
int *p = NULL;//空指针
int *q;//野指针
```
### 7.1.5 空类型
```c
void *p;//能接受任何类型的指针
````
### 7.1.6 定义初始化的书写规范

*跟变量，指针初始化必须有效赋值或指空。

### 7.1.7 指针运算

&	* 	++	--

### 7.1.8 指针与数组

```c
//一维数组---------------------------------------------
int a[3] = {1,2,3};//a为起始地址
//(a + i)	i为偏移量，等价于 &a[i]
int *p = (int [3]){1,2,3}//没有数组名但是有指针指向数组空间。
//---------------------------------------------------

//二维数组---------------------------------------------
int a[2][3] = {1,2,3,4,5,6};
int *p = a;//p是a的行指针，加偏移量后先当行移动
a[i][j] = 2;//a[i][j] 的等效为 *(*(p + i) + j)
p = *a;//p为列指针，指向 a[0][0]
//---------------------------------------------------

//字符数组---------------------------------------------
char str[] = "hello";
char *p = "hello";
//sizeof(str)返回数组空间大小 5，sizeof()返回指针所占空间 8
strcpy(str,"world");//str = "world";是错的
p = "world";//strcpy(p,"world");是错的
//----------------------------------------------------
```
### 7.1.9 指针常量与常量指针(const)

```c
/*	const 将变量常量化
 *	const int a; 常量					
 *	int	const a;
 *  //看名字*在前就是指针常量 const 在前就是常量指针
 *	const int *p; 常量指针  指针指向可以变化 指针指向的内容不能变化
 *	int	const *p;
 *
 *	int *const p; 指针常量  指针指向不能发生变化 指针指向的内容可以变化
 *
 *	const int *const p;
 */

#define PI	3.14//定义常量 不检查语法 只进行宏替换

const float pi = 3.14;//保证pi的值一直不变 检查语法，可通过指针修改，有警告
//错误操作
const float *p = &pi;
*p = 3.141592;//实现修改常量内容但是不合法
//正确操作
const float *p = &pi;


{   //可以通过i修改内容 但是不能通过*p修改
    int i = 1;
    const int *p = &i;
    
    //q的指向无法改变
    int *const q = &i;
    
    //cp的指向和内容都不能发生变化
    const int *const cp = &i; 8

}
```
### 7.1.10 指针数组与数字指针

```c
//指针数组：[存储类型]	数据类型	*指针名	[下标]
//数组指针：[存储类型]	数据类型	(*指针名)	[下标]

char *name[3] = {"Follow me","hello world","Tom"};//三个字符串指针

int a[2][3] = {1,2,3,4,5,6};
int (*p)[3] = a;//q相当于a的行指针，+1相当跳转一行（跳转3列） p+1相当于 a+1

```
### 7.1.11 多级指针

```c

```



## 8 函数

### 8.1 函数的定义

定义：数据类型	函数名	( [参数表] )

一个进程的返回状态是给它的父进程看的；

```c

//int argc:传入命令的数量
//char *argv[]:若干个字符串指针
int main(int argc, char *argv[]){
    
}
```



### 8.2 函数的传参

方式：直传递、地址传递、全局变量传递

参数表：( [数据类型 变量名， ……] )

```c
void func1(int a, int b);//直传递
void func2(int *p, int *q);//地址传递
```



### 8.3 函数的调用

* 嵌套调用

```c
#include <stdio.h>
#include <stdlib.h>

int max(int a, int b){
    
}
int min(int a, int b){
    
}

int dist(int a, int b, int c){
    min(max(a,b), c);
}
int main(){
    int a = 3, b = 5, c = 6;
    int res;
    res = dist(a, b, c);
    
    return 0;
}
```



* 递归调用：压栈过程，调用最后一次向外弹栈。

```c
//求阶乘
#include <stdio.h>
#include <stdlib.h>

int func(int n){
    if(n < 0)
        return -1;
    if(n == 1 || n ==0)
        return 1;
    
    return n * func(n-1);
}

int main(){
    int n, res;
    
    scanf("%d", &n);
    
    res = func(n);
    printf("%d! = %d\n", n, res);
    
    return 0;
}
```



### 8.4 函数与数组

* 一维数组：只能传地址，传不了长度。需要附加长度信息。

```c
void print_arr(int *p, int len){//void print_arr(int p[], int len)
    for(int i=0;i<len;i++){
        printf("%d ",p[i]);
    }
    printf("\n");
}

int a[] = {1,2,3};

print_arr(a, sizeof(a)/sizeof(*a));
```



* 二维数组：区分行列指针

```c
#define M	3
#define N	4

void print_douarr(int *p,int n){
    for(int i=0;i<n;i++){
        printf("%d ",*(p+i));
    }
    printf("\n");
}
void print_douarr1(int (*p)[N],int m, int n){//int p[][N]
    for(int i=0;i<m;i++){
        for(int j=0;j<n;j++){
            printf("%4d ", *(*(p+i)+j));
        }
        printf("\n");
    }
    printf("\n");
}

int a[M][N] = {1,2,3,4,5,6,7,8,9,10,11,12}

print_douarr(&a[0][0], M*N);
print_douarr1(a, M, N);


```

* 字符数组

```c
//与数组相同
```



### 8.5 函数与指针

* 指针函数：返回值为指针

```c
返回值 * 函数名(参数表)
int *func();
```

* 函数指针：指向函数的指针

```c
类型 (*指针名)(参数表)
int (*p)(int);

#include <stdio.h>
#include <stdlib.h>

int add(int a, intb){
    return a+b;
}

int main(){
    int a = 3, b = 5;
    int ret;
    
    int （*p）(int, int);//函数指针  或 int (int,int) *p
    p = add;
    
    ret = p(a,b);//等价ret = add(a, b);
    printf("ret = %d\n", ret);
    
    return 0;
}

```



* 函数指针数组

```c
类型 (*指针名[下标])(参数表)
int (*arr[3])(int, int);       
```



## 9 构造类型

### 9.1 结构体

1. 产生及意义

适用复杂类型数据的存储。成员共存。

2. 类型描述：<B>结构体的类型描述是不占用任何内存空间的，无法直接初始化赋值。</B>

```c
struct 结构体名字 {
    数据类型 成员1;
 	数据类型 成员2;
    …………
};
```



3. 嵌套定义

```c
struct birthday_st{
    int year, month, day;
};

struct student_st{
    char name[100];
    struct birthday_st birth;    
};
```



4. 定义变量：

   成员引用：

   * 变量名.成员名

   * 指针->成员名 / (*指针).成员名

```c
struct node_st{
    int i;
    float f;
    char c;
}__attribute__((packed));//不对齐宏

struct node_st a = {1, 3.14, 's'};
struct node_st a = {.f = 2.3, .c = 'x'};
//a.i	a.f	a.c

```



5. 占用内存空间大小

> 指针永远是固定大小
>
> 结构地址对齐问题

### 9.2 共用体

1. 产生及意义

成员共用一个空间，同时只存在一个成员。

2. 类型描述

```c
union 共用体名 {
    数据类型 成员名1；
    数据类型 成员名2；
}；
       
```



3. 嵌套定义

可以和结构体互相嵌套。

4. 定义变量

```c
union test_un{
    int i;
    float f;
    double d;
    char ch;
};

union test_un a;

a.f = 3.14; //此时 a 的其他成员都无意义

```



5. 占用内存大小

成员中最大的成员大小。

6. 函数传参

同结构体

7. 位域

不以字节为单位存储，以位为单位存储。



### 9.3 枚举类型

```c
enum 标识符{
    成员1；
    成员2；
        ……
};

//相当于定义的宏值 预处理不会被替换
enum day{
    MON = 1,
    TUS,
    THR,
    WES,
    FRI,
    SAT,
    SUN
};

enum day a = SAT; //a 为6；


```



## 10 动态内存管理

原则：谁申请谁释放。

* malloc

```c
void *malloc(size_t size);
//申请size大小的空间 返回起始地址
```



* calloc

```c
void *calloc(size_t nmemb, size_t size);
//申请nmemb个size大小的空间 返回起始地址
```



* realloc

```c
void *realloc(void *ptr, size_t size);
//对malloc或calloc的空间从新分配，扩大或缩小
```



* free

```c
free(void *ptr);
//只释放空间，不修改地址，free后最好把指针指空
```







## 11 typdef

typedef：为已有的数据类型改名；

```c
#define INT int
typedef int INT;

#define P int*
typedef int *P;
//区别：第一组效果一样第二组不同
typedef int ARR[6]; ARR a --> int a[6]

typedef struct{
    
}NODE,*NODEP;

typedef int FUNC(int); --> int(int) FUNC;
FUNC f; --> int f(int);

typedef int (*FUNCP)(char);
FUNCP p; --> int (*p)(char);

```





## END



