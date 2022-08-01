# 02 数据库语言SQL



## SQL语言概述

SQL语言是集DDL、DML和DCL于一体的数据库语言，SQL语言主要由以下9个单纯引导的操作语句来构成，但是每一种语句都能表达复杂的操作请求。

> * DDL语句引导词：Create（建立），Alter（修改），Drop（撤销）：模式的定义和删除，包括定义Database，Table，View，Index，完整性约束条件等，也包括定义对象（RowType行对象，Type列对象）。
> * DML语句引导词：Insert，Delete，Update，Select：各种方式的更新与检索操作，如直接输入记录，从其他Table（由SubQuery建立）输入；各种复杂条件的检索，如连接查找，模糊查找，分组查找，嵌套查找等；各种聚集操作，求平均、求和、等，分组聚集，分组过滤等。
> * DCL语句引导词：Grant、Revoke：安全控制的授权和撤销授权。





## SQL语言之DDL-定义数据库

### 建立数据库

包括两件事：**定义数据库和表（使用DDL）**，**向表中最佳元组（使用DML）**

> DDL：Data Definition Language
>
> * 创建数据库（DB）——Create Database
> * 创建DB中的Table（定义关系模式）——Create Table

DDL通常由DBA来使用，也有经DBA授权后由应用程序员来使用。

**创建Database**：

> 数据库可以看作是一个集中存放若干Table的大型文件，创建语法形式如下：
>
> ​			create database 数据库名；
>
> 示例：创建学习数据库SCT
>
> ​			create database SCT；

**创建Table**：

> create table简单语法形式：
>
> ​			create table 表名（列名 数据类型 [Primary key|Unique] [Not null]，列名 数据类型 [Not null]，...）；
>
> * []中的内容可以省略，|表示隔开的两项可取其一；
>
> * Primary key：主键约束，每个表只能创建一个主键约束；
>
> * Unique：唯一性约束（候选键），可以有多个唯一性约束；
>
> * Not null：非空约束，是指该列不允许有空值出现。
>
> 示例：定义学生表 Student
>
> ​			Create Table **Student**(**Snumber** char(8) not null, **Sname** char(8), **Ssex** char(2)， **Sage** integer, **Dnumber** char(2), **Sclass** char(6));
>
> 示例：定义课程Course
>
> ​			Create Table Course(Cnumber char(3), Cname char(12), Chours integer, Credit float(1), Tnumber char(3));





## SQL语言之DML-操作数据库

> DML：Data Manipulation Language
>
> * 向Table中追加新的元组：Insert
> * 修改Table中某些元组的某些属性的值：Update
> * 删除Table中的某些元组：Delete
> * 对Table中的数据进行各种条件的检索：Select

DML通常由用户或应用程序员使用，访问近授权的数据库。

### 向表中追加元组

> insert into简单语法形式：
>
> ​			insert into 表明[(列名[,列名]...)]
>
> ​						values (值 [,值]， ...)
>
> * values后面值的排列，须与into子句后面的列名排列一致
> * 若表名后的所有列名省略，则values后的值的排列须与该表存储中的列名排列一致
>
> 示例：最佳学生表中的元组
>
> Insert Into Student
>
> Values ('98030101', '张三', '男', 20, '03', '980301');
>
> Insert Into Student (Snumber, Sname, Ssex, Sage, Dnumber, Sclass)
>
> Values ('98030102', '张四', '女', 20, '03', '980301');

### 查询数据 

**（1）单表查询 SELECT-FROM-WHERE**

> 语义：从表名所给出的表中，查询除满足检索条件的元组，并按给定的列名及顺序进行投影显示。
>
> Select 列名 [(列名[,列名]...)]    From 表名 [Where 检索条件];
>
> 示例：检索学生表中所有学生的信息
>
> Select **Snumber, Sname, Ssex, Sage, Sclass, Dnumber** From **Student**;
>
> Select ***** From **Student**;
>
> 示例：检索学生表中所有年龄小于等于19岁的学生的年龄及姓名
>
> Select Sname,Sage From Student Where Sage <= 19;

**结果唯一性问题**：关系模型不允许出现重复元组。但现实DBMS却允许出现重复元组，但也允许无重复元组。在Table中要求无重复元组是通过定义Primary key和Unique来保证的；而在检索结果中要求无重复元组，是通过**DISTINCT**保留字使用来实现的。

例如：Select **DISTINCT** Snumber From ... ...

**结果排序问题**：DBMS可以对检索结果进行排序，可以升序排列，也可以降序排列。

> Select语句中结果排序是通过增加order by子句实现的
>
> ​		Order by 列名 [asc|desc]
>
> * 以为检索结果按指定列名进行排序，若后跟asc或省略，则为升序；若后跟desc，则为降序。
>
> 示例：按学号由小到大的现实所有学生的学号和姓名
>
> Select **Snumber, Sname** From **Student** Order By **Snumber**;



**模糊查询问题**：找出匹配给定字符串的字符串。其中给定字符串中可以出先%, _等匹配符

> 匹配规则：
>
> * “%”匹配零个或多个字符
> * “_”匹配任意单个字符,一个汉字两个字符，匹配汉字一个字需要两个下划线
> * “\”转义字符
>
> 含有like运算符的表达式
>
> ​			列名 [Not]Like "字符串"
>
> 示例：检索所有姓张的学上学号及姓名
>
> Select **Snumber, Sname** From **Student** Where **Sname** Like **'张%'**;

**（2）多表联合查询**

多表联合检索可以通过连接运算来完成，而链接运算又可以通过**广义笛卡尔积**后在进行选择运算来实现。

> Select的多表联合检索语句：
>
> ​		Select 列名 [(列名[,列名]...)]  From 表名1, 表名2, ...  Where 检索条件;
>
> * 检索条件中要包含连接条件，通过不同的连接条件可以实现等值连接、不等值连接以及各种$\theta$-连接。
>
> 示例：按”001“号课成绩由高到低顺序显示所有学生的姓名（二表连接）
>
> Select Sname From Student, SC Where **Student.Snumber = SC.Snumber and SC.Cnumber = '001'** Order By Score DESC;



**重名处理**：连接运算涉及到重名的问题，如果两个表中的属性重名，连接两个表重名（同一表的连接）等，因此需要使用**别名**以便区分。

> Select中采用别名的方式
>
> ​		Select 列名 as 列别名 [[列名 as 列别名]...] From 表名1 as 表别名1, 表名2 as 表别名2, ... Where 检索条件;
>
> * 上述定义中的as可以省略
> * 当定义了别名后，在检索条件中可以使用别名来限定属性
>
> 示例：求有薪水擦额的任意两位教师
>
> Select **T1.Tname as t1, T2.Tname as t2** From **Teacher T1, Teacher T2** Where **T1.Salary > T2.Salary**;



### SQL语言的增-删-改























