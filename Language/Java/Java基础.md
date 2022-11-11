# Java 基础

[TOC]

## 1 前置准备

### 1.1 命令提示符

DOS(Disk Operating System)

在windows系统上，为命令提示符（cmd）

```shell
# 切换盘符 打开文件夹 同 Linux 命令行语法
C:
cd 文件夹

# 显示文件夹内容 Linux 命令行语法为 ls
dir

# 清除屏幕
cls


```



### 1.2 Java虚拟机——JVM

* <B>JVM</B>  (Java Virtual Machine)：JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。
* <B>跨平台</B> ：Java虚拟机本质上就是一个程序，当它在命令行上启动的时候，就开始执行保存在某字节码文件中的指令。Java语言的可移植性正是建立在Java虚拟机的基础上。任何平台只要装有针对于该平台的Java虚拟机，字节码文件（.class）就可以在该平台上运行。这就是“一次编译，多次运行”。

### 1.3 JRE 和 JDK

* <B>JRE</B> ：Java运行环境（Java Runtime Environment，简称JRE）是一个软件，由太阳微系统所研发，JRE可以让计算机系统运行Java应用程序（Java Application）。
* <B>JDK</B> ：JDK是 Java 语言的软件开发工具包（Java Development Kit，简称JDK），主要用于移动设备、嵌入式设备上的java应用程序。JDK是整个java开发的核心，它包含了JAVA的运行环境（JVM+Java系统类库）和JAVA工具。

JDK包含的基本组件包括：

>javac – 编译器，将源程序转成字节码
>
>jar – 打包工具，将相关的类文件打包成一个文件
>
>javadoc – 文档生成器，从源码注释中提取文档
>
>jdb – debugger，查错工具
>
>java – 运行编译后的java程序（.class后缀的）
>
>appletviewer：小程序浏览器，一种执行HTML文件上的Java小程序的Java浏览器。
>
>Javah：产生可以调用Java过程的C过程，或建立能被Java程序调用的C过程的头文件。
>
>Javap：Java反汇编器，显示编译类文件中的可访问功能和数据，同时显示字节代码含义。
>
>Jconsole: Java进行系统调试和监控的工具



### 1.4 入门程序

java程序开发三步：编写、编译、运行。

编写：HelloWorld.java

```java
public class HelloWorld {
	public static void main(String[] args){
		System.out.println("Hello World!");
	}
}
```

编译：会生成 HelloWorld.class 文件

```shell
javac HelloWorld.java
```

运行：

```shell
java HelloWorld
```



### 1.5 简单注释



```java

//单行注释

/*
	多行注释
*/
/** 文档注释 */
```



## 2 关键字与标识符

* 关键字的特点：

1. 全部为小写字母；
2. 在编辑器内有特殊颜色。

* 标识符的命名规则：

> 硬性要求：
>
> * 标识符可以包含：英文字母（区分大小写）、数字、$ 、_  ；
> * 不能以数字开头；
> * 不能是关键字。
>
> 软性建议：
>
> * 类名规范：首字母大写，后面每个单词首字母大写（大驼峰式）；
> * 变量名规范：首字母小写，后面每个单词首字母大写（小驼峰式）；
> * 方法名：同变量名。

## 3 常量与变量

* 常量

> 分类：
>
> 1. 字符串常量：“asd”
> 2. 整数常量： 1， 32，  324， -100
> 3. 浮点数常量：23.4， 324.5， -234.55
> 4. 字符常量：'s' ， '的'
> 5. 布尔常量：ture、false
> 6. 空常量：NULL



