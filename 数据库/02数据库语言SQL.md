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

> 元组新增Insert：新增一个或一些元组到数据库中的Table中
>
> 元组更新Update：对某些元组中的某些属性值进行重新设定
>
> 元组删除Delete：删除某些元组

SQL-DML既能单一记录操作，也能对记录集合进行批更新操作。

#### 元组新增Insert命令

> 单一元组新增命令形式：插入一条指定元组值的元组
>
> ​				insert into 表名 [(列名[,列名]...)]  value (值[,值]...);
>
> 批量数据新增命令形式：插入子查询结果中的若干条元组。待插入的元组由子查询给出。
>
> ​				insert into 表名 [(列名[,列名]...)]  子查询;
>
> 示例：
>
> Insert Into Teacher (Tnumber, Tname, Dnumber, Salary)  value ("005", "张三", "03", "1250");
>
> Insert Into **St (Snumber, Sname)**  Select **Snumber, Sname**  From **Student**  Where **Sname like '%伟'**;

#### 元组删除Delete命令

> 元组删除Delete命令：删除满足指定条件的元组
>
> ​				Delete From 表名 [Where 条件表达式]；
>
> 如果Where条件省略，则删除所有元组。
>
> 示例：删除自动控制系的所有同学。
>
> Delete From **Student** Where **Dnumber** in (**Select Dnumber From Dept Where Dname = '自动控制'**);
>
> 删除有四门不及格课程的所有同学
>
> Delete From **Student** Where **Snumber** in (Select **Snumber** From **SC** Where **Score < 60** Group by **Snumber** Having **Count(*) >= 4**);

#### 元组修改Update命令

> 元组更新Update命令：用指定要求的值更新指定表中满足指定条件的元组的指定列的值。
>
> Update 表名 Set 列名 = 表达式|（子查询）[[， 列名=表达式|（子查询）]...]  [Where 条件表达式]；
>
> 如果Where条件省略，则更新所有的元组。
>
> 示例：将所有教师工资上调5%
>
> Update Teacher Set Salary = Salary * 1.05;
>
> 将所有计算机系的教师工资上调10%
>
> Update **Teacher** Set **Salary = Salary * 1.1** Where **Dnumber** in (Select **Dnumber** From **Dept** Where **Dname  = '计算机'** );



### SQL语言DDL之撤销与修改

修正数据库：修正数据库的定义，主要是修正表的定义

修正基本表的定义

> alter table tablename
>
> [add  {colname datatype, ... }]  //增加新列
>
> [drop {完整性约束名}]  //删除完整性约束
>
> [modify {colname dataype, ...}]  //修改列定义
>
> 示例：在学生表**Student**(**Snumber** char(8) not null, **Sname** char(8), **Ssex** char(2)， **Sage** integer, **Dnumber** char(2), **Sclass** char(6)) 基础上增加两列Saddr，PID：
>
> Alter Table **Student** Add **Saddr char(40), PID char(18)**;  
>
> 将上例表中Sname列的数据类型增加两个字符
>
> Alter Table **Student** Modify **Sname char(10)**;
>
> 删除学生姓名必须取唯一值的约束
>
> Alter Table **Student** Dorp Unique (Sname);

撤销基本表

> Drop Table 表名

* 注意，SQL-Delete语句只是删除表中的元组，而册小基本表Drop Table的操作是册小包括表格式、表中所有元组、由该表到处的视图等相关的所有内容，所以使用要特别注意。

同样可以撤销数据库

> Drop Database 数据库名

### SQL语言DDL之指定与关闭

指定当前数据库

> Use 数据库名；

关闭当前数据库

> Close 数据库名；

 ### SQL语言DDL之其他

数据库授权

>语法形式：
>
>Grant 权限 on 表名 to 用户名；
>
>权限有：Select，Update，Insert，Delete，Exec，Dri;
>
>对呗授权的用户，要先称为该数据库的使用者，即要把用户加到数据库里，才能授权。



## SQL语言之复杂查询



现实中很多情况需要进行下述条件的判断：

> * 集合成员资格：某一元素是否是某一集合的成员。
> * 集合之间的比较：某一个集合是否包含另一个集合。
> * 集合基数的测试：测试集合是否为空；测试集合是否存在重复元组。

子查询：出现在Where子句中的Select语句被称为子查询（subquery），子查询返回了一个集合，可以通过与这个集合的比较来确定另一个查询集合。

> 三种类型的子查询：（NOT）IN-子查询；$\theta$-Some/$\theta$-All子查询；（NOT）EXISTS子查询。

### (NOT) IN子查询

> 基本语法：
>
> ​				表达式 [Not] In (子查询)
>
> 语法中，表达式的最简单形式就是列名或常数。
>
> 语义：判断某一表达式的值是否在子查询的结果中。
>
> 示例：列出张三和王三同学的所有信息。
>
> Select ***** From **Student** Where **Sname** in **('张三', '王三')**;
>
> 列出选修了001号课程的学生的学号和姓名
>
> Select **Snumber, Sname** From **Student** Where **Snumber** in**(Select Snumber From SC where Cnumber = '001')**;

**非相关子查询**：内层查询独立进行，没有色剂任何外层查询相关信息的子查询。

**相关子查询**：内层查询需要依靠外层查询的某些参量作为限定条件才能进行的子查询，外层向内层传递的参量需要使用外层的表名或表别名来限定。

> 示例：求学过001号课程同学的姓名
>
> Select **Sname** From **Student Stud** Where Snumber (Select Snumber From SC Where Snumber=**Stud**.Snumber and Cnumber=’001‘)；



### $\theta$-Some/$\theta$-All子查询

> 基本语法：
>
> ​				表达式  $\theta$ Some （子查询）
>
> ​				表达式  $\theta$ All  （子查询）
>
> 语法中，$\theta$是比较运算符: `<`, `>`, `>=`, `<=`, `=`, `<>`。
>
> 语义：将表达式的值与子查询的结果进行比较：
>
> * 如果表达式的值至少与子查询结果的某一值相比较满足比较关系，则表达式   $\theta$ Some （子查询）的结果便为真。
> * 如果表达式的值与子查询结果的所有值相比较满足比较关系，则表达式   $\theta$ All （子查询）的结果便为真。
>
> 示例：找出工资最低的教师姓名。
>
> Select **Tname** From **Teacher** Where **Salary <= all (Select Salary From Teacher);**
>
> 找出001号课程成绩不是最高的所有学生的学号。
>
> Select **Snumber** From **SC** Where **Cnumber = "001**" and **Score < Some (Select Score From SC Where Cnumber =  "001")**;

需要注意的“等价变换”：

> 如下两种表达方式含义是相同的
>
> ​		表达式 **= Some** (子查询)
>
> ​		表达式 **in** (子查询)
>
> 另外
>
> ​		表达式 **<> all** (子查询)
>
> ​		表达式 **not in** (子查询)
>
> 是等价的。



### (NOT) EXISTS子查询

> 基本语法：
>
> ​			[not] Exists (子查询)
>
> 语义：子查询结果中由于元组存在
>
> 示例：检索选修了赵三老师主讲课程的所有同学的姓名
>
> Select DISTINCT **Sname** From **Student** Where  exists (Select ***** From **SC, Course, Teacher** Where **SC.Cnumber = Course.Cnumber and SC.Snumber = Student.Snumber and Course.Tnumber = Teacher.Tnumber and Tname = "赵三"**);
>
> 然而not exists却可以实现很多新功能，
>
> 示例：检索学过001号教师主讲的所有课程的所有同学的姓名
>
> ```sql
> SELECT Sname From Student
> Where not exists  --不存在
> 	(
>         SELECT * From Course --有一门001教师的主讲课程
>         Where Course.Tnumber='001' and not exists --该同学没学过
>         (
>         	SELECT * From SC
>             Where Snumber=Student.Snumber and Cnumber=Course.Cnumber
>         )
> 	);
> ```
>
> 上述语句的意思：不存在有一门001号教师主讲的课程该学生没有学过



### SQL语言的结果计算与聚集计算



#### 结果计算

> Select-From-Where语句中，Select子句后面不仅可是列名，而且可是一些计算表达式或聚集函数，表明再投影的同时直接进行一些运算
>
> ```sql
> Select 列名|expr|agfunc(列名) [[, 列名|expr|agfunc(列名)]...]
> From 表名1 [,表名2...]
> [Where 检索条件];
> ```
>
> * expr可以是常量、列名或由常量、列名、特殊函数及算术运算符构成的算术运算式。特殊函数的使用需要结合子DBMS的说明书。
> * agfunc()是一些聚集函数。

示例：求有差额（差额>0）的任意两位教师的薪水差额

```sql
Select T1.Tname as TR1, T2.Tname as TR2, t1.Salary-T2.Salary
From Teacher T1, Teacher T2
Where T1.Salary > T2.Salary;
```

示例：依据学生年龄求出学生的出生年份，当前是2022年

```sql
Select S.Snumber, S.Sname, 2022-S.Sage+1 as Syear
From Student S;
```



#### 聚集函数

> * SQL提供了5个作用再简单列值集合上的内置聚集函数agfunc，分别是：
>
>   COUNT（求个数）、SUM（求和）、AVG（求平均）、MAX（求最大）、MIN（求最小）

示例：求教师的工资总额。

```sql
Select Sum(Salary) From Teacher;
```

示例：求数据库课程的平均成绩

```sql
Select AVG(Score) From Course C, SC 
Where C.Cname='数据库' and C.Cnumber = SC.Cnumber
```



#### 分组查询与分组过滤

**分组**：SQL可以将检索到的元组按照某一条件进行分类，具有相同条件值的元组划到一个组或一个集合中，同时处理多个组或集合的聚集运算。

> 基本语法：
>
> ```sql
> Select 列名|expr|agfunc(列名) [[, 列名|expr|agfunc(列名)]...]
> From 表名1 [,表名2...]
> [Where 检索条件]
> [Group by 分组条件];
> ```
>
> 分组条件可以是
>
> * 列名1，列名2 ...
>
> 示例：求每个学生的平均成绩
>
> ```sql
> Select Snumber, AVG(Score) From SC Group by Snumber;
> --按照学号进行分组，即学号相同的元组划到一个组中并求平均值。
> ```
>
> 求每一门课程的平均成绩
>
> ```sql
> Select Cnumber, AVG(Score) From SC Group by Cnumber;
> --按照课程号进行分组，即课程号相同的元组划到一个组中并求平均值。
> ```

聚集函数是不允许用于Where子句中的：Where子句是对每一元组进行条件过滤，而不是对集合进行条件过滤。

分组过滤：若要对集合（分组）进行条件过滤，即满足条件的集合/分组留下，不满足条件的集合/分组剔除。

> **Having子句，又称分组过滤子句。**需要Group by子句支持，换句话没有Group by子句，便不能有Having子句。
>
> ```sql
> Select 列名|expr|agfunc(列名) [[, 列名|expr|agfunc(列名)]...]
> From 表名1 [,表名2...]
> [Where 检索条件]
> [Group by 分组条件 [Having 分组过滤条件]];
> ```
>
> 示例：求不及格课程超过两门的同学的学号
>
> ```sql
> Select Snumber From SC
> Where Score < 60
> Group by Snumber Having Count(*) > 2;
> ```

**Where子句中不能出现求最大求最小统计相关的东西。**

> 求有两门以上不及格课程同学的学号及其平均成绩
>
> ```sql
> Select Snumber, AVG(Score) From SC
> Where Snumber in --找出符合条件的学生学号
> 	(Select Snumber From SC
>      Where Score < 60
>      Group by Snumber Having Count(*)>2)
> Group by Snumber;--通过学号分组计算
> ```



### SQL语言实现关系代数操作



#### 并-交-差的处理

> SQL语言：并运算UNION，交运算INTERSECT，差运算EXCEPT。
>
> 基本语法形式：
>
> ```sql
> 子查询 {Union [ALL] | Intersect [ALL] | Except [ALL] 子查询}
> ```
>
> 通常情况下自动删除重复元组（不带ALL）。若要保留重复元组，则需要带ALL。	

示例：求学过002号课或003号课的同学的学号。

```sql
Select Snumber From SC Where Cnumber='002'
UNION
Select Snumber From SC Where Cnumber='003';
--或
Select Snumber From SC Where Cnumber='002' or Cnumber='003'; 
```

但有时也不能完全转换成不用UNION的方式

如，已知两个表Customers（CID，Cname，City，Discnt），Agents（AID，Aname，City，Percent）求客户所在的或者代理商所在的城市。

```sql
Select City From Customers
UNION
Select City From SC Where Agents;
```

#### SQL语言的空值处理

> 空值是其值步子到、不确定、不存在的值；
>
> 数据库中有了空值，会影响许多方面，如影响聚集函数运算的正确性，不能参与算术、比较或逻辑运算等。
>
> 在SQL标准中，空值被用一种特殊的符号Null来标记，使用特殊的空值检测函数来获得某列的值是否为空值。
>
> 空值检测： is [not] null （测试指定列的值是否为空值）

示例：找出年龄值为空的学生姓名

```sql
Select Sname From Student Where Sage is null;
--为什么不写成=null，因为null不能进行运算和比较。
```

先行DBMS的空值处理方案：

> * 除is [not] null之外，空值不满足任何查找条件；
> * 如果null参与算术运算，则该算术表达式的值为null；
> * 如果null参与比较运算，则结果可视为false，部分可以看成unknown；
> * 如果null参与聚集运算，则除count(*)之外其他聚集函数都忽略null。



#### 内连接与外连接

> 关系代数中，有连接运算，又分为$\theta$连接和外连接
>
> 标准SQL语言中连接通常是采用
>
> ```sql
> Select 列名 [[, 列名]...]
> From 表名1,表名2, ...
> [Where 检索条件];
> ```
>
> 即相当于采用$$\pi_{列名,...,列名}(\sigma_{检索条件}(表名1\times 表名2\times ...))$$  。

SQL的高空语法中引入了内连接于外连接运算，具体形式：

> ```sql
> Select 列名 [[, 列名]...]
> From 表名1[NATURAL] 
> 		 [INNER | {LEFT|RIGHT|FULL} [OUTER]] JOIN 表名2
> 	{ON 连接条件|Using (Colname{,Colname ...})}
> [Where 检索条件];
> ```
>
> 上书的连接运算有两部分构成：连接类型和连接条件
>
> | 连接类型（四选一） |     连接条件（三选一）      |
> | :----------------: | :-------------------------: |
> |     inner join     |           natural           |
> |  left outer join   |        on <连接条件>        |
> |  right outer join  | using(Col1, Col2, ...,Coln) |
> |  full outer join   |              /              |



* Inner Join：即关系代数中的$\theta$-连接运算。
* Left Outer Join， Right Outer Join， Full Outer Join：即关系代数中的外连接运算。

> （1）连接中使用 natural
>
> 出现在结果关系中的两个连接关系在公共属性上取值相等，且公共属性只出现一次。
>
> （2）连接中使用 on<连接事件>
>
> 出现在结果关系中的两个连接关系的原则取值满足连接条件，且公共属性出现两次。
>
> （3）连接中使用 using(Col1, Col2, ...,Coln)
>
> (Col1, Col2, ...,Coln)是两个连接关系的公共属性的子集，元组在(Col1, Col2, ...,Coln)上取值相等，且(Col1, Col2, ...,Coln)只出现一次。

示例：求所有教师的任课情况并按教师号排序（没有课程的教师也需要列在表中）

```sql
--（Inner Join ）错误的 因为会丢失没有课程的老师
Select Teacher.Tnumber, Tname, Cname 
From Teacher Inner Join Course 
	ON Teacher.Tnumber = Course.Tnumber
Order By Teacher.Tnumber ASC;

--（Left Outer Join ）左连接不会丢失没有课程的老师
Select Teacher.Tnumber, Tname, Cname 
From Teacher Left Outer Join Course 
	ON Teacher.Tnumber = Course.Tnumber
Order By Teacher.Tnumber ASC;
```



## SQL语言之视图及其应用

三级模式两层映像结构

> 对应概念模式的数据在SQL中被称为**基本表（Table）**，而对应外模式的数据称为视图（View）。**视图不仅包含外模式，而且还包含其E-C映像。**

### SQL数据库结构-视图

* 基本表是实际存储于存储文件中的表，基本表中的数据是需要存储的。
* 视图在SQL中只存储其由基本表到处视图所需要的公式，即由基本表产生视图的影响信息，其数据并补存储，而是在运行过程中动态产生与维护的。
* 对视图数据的更改最终要反应在对基本表的更改上。

#### 视图的定义

视图需要“先定义，再使用”，视图的**定义**：

> 定义视图：
>
> ```sql
> Create view 视图名 [[, 列名]...]
> 	as 子查询 [with check option];
> ```
>
> * 如果视图的属性名缺省，则默认为子查询结果中的属性名；也可以显示知名其所拥有的列名。
> * with check option 指明当对视图进行insert、update、delete时，要检测进行的对应操作的元组是否满足视图定义中子查询定义的条件表达式。

示例：定义一个视图CompStud为计算机系的学生，通过该视图可以将Student表中其他系的学生屏蔽掉。

```sql
Create View CompStud AS
	(Select * From Student 
     Where Dnumber in (Select Dnumber From Dept 
                       Where Dname='计算机' ));
```

示例：定义一个视图Teach为教师任课的情况，吧Teacher表中的个人隐私方面的信息，如工资等屏蔽掉，仅反应其教纳闷可学分多少。

```sql
Create View Teach AS 
	(Select T.Tname C.Cname Credit
    	From Teacher T, Course C
    	Where T.Tnumber = C.Tnumber);
```

#### 视图的查询

**使用视图：定义号的视图，可以像Table一样，再SQL各种语句中使用。**

示例检索主讲数据库课程的教师姓名，我们可以适应Teach。

```sql
Select T.Tname From Teach T
Where T.Cname='数据库';
```

==对视图查询的本质实际上是将对视图的查询转化成对应的基本表的查询。==

#### 视图的更新

SQL视图的更细是比较复杂的问题，因视图不保存数据，对视图的更新最终要反映到对基本表的更新上，而有时，视图定义的映射是不可逆的。

**示例1**：

```sql
Create View S_G(Snumber, Savg)
	AS (Select Snumber, AVG(Score)
       	From SC Group by Snumber);
```

如何要进行下面的更新操作？

```sql
Update S_G Set Savg = 85 Where Snumber='98030101';
```

显然上述映射是不可逆的，是无法通过视图对基本表进行更新的，所以无法实现。

**示例2**：

```sql
Create View ClassStud(Sname,Sclass)
	AS (Select Sname, Sclass From Student);
```

如何进行下面的更新操作？

```sql
Insert Into ClassStud Values ('张三', '980301');
```

上述更新操作仍然是不可实现的，因为缺少Student的主键Snumber。

SQL视图更新的可执行性

> * 如果视图的select目标列包含聚集函数，则不能更新。
> * 如果视图的select子句中适应了unique或distinct，则不能更新。
> * 如果视图中包括了group by子句，则不能更新。
> * 如果视图中包括精算术表达式计算出来的列，则不能更新。
>
> 对于由单一Table子集构成的视图，即如果视图时从单个基本表使用选择、投影操作到处的，并且包含了基本表的主键，则可以更新。



#### 视图的撤销

```sql
--撤销视图
Drop View 视图名;
```



## 数据库的完整性及完整性约束









