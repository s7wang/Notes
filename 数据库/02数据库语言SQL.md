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

数据库完整性（DB Integrity）是指DBMS应保证DB的一种特性——在任何情况下的正确性、有效性和一致性。

> 广义完整性：语义完整性、并发控制、安全控制、DB故障恢复等。
>
> 狭义完整性：专指语义完整性，DBMS通常有专门的完整性管理机制与程序来处理语义完整性问题。

数据库完整性管理的作用：

> * 防止和避免数据库中不合理数据的出现
> * DBMS应尽可能地自动防止DB中语义不合理现象

完整性约束条件（完整性约束规则）的一般形式

> **Integrity Constraint ::= (O, P, A, R)**
>
> * O:数据集合：约束的对象？
> * P:谓词条件：什么样的约束？
> * A:触发条件：什么时候检测？
> * R:响应动作：不满足时怎么办？

### 数据库完整性分类

**按约束对象分类**

* 域完整性约束条件

> 施加于某一列上，对于给定列上所要更新的某一后选址是否可以接受进行约束条件判断，这是独立进行的。（说人话，约束每一列的取值）

* 关系完整性约束条件

> 施加于关系/table上，对于给定table上所要更新的某一候选元组时否可以接收进行约束条件判断，或是对一个关系中的若干元组和另一个关系中的若干元组间的联系是否可以接受进行约束条件判断。（说人话，同一行不同列之间的值可能会存在某种约束关系）



**按约束来源分类**

* 结构约束

> 来自于模型的约束，例如函数依赖约束、主键约束（实体完整性）、外键约束（参照完整性），只关心数值相等与否、是否允许空值等；

* 内容约束

> 来自于用户的约束，如用户自定义完整性，关心元组或属性的取值范围。例如：Student表中的Sage属性值在15和40之间等。



**按约束状态分类**

* 静态约束

> 要求DB在任一时候均满足的约束；例如Sage在任何时候都应该大于0而小于150.

* 动态约束

> 要求DB从一状态变为另一状态时应满足的约束；例如工资只能升，不能降。



**SQL语言支持如下几种约束**

* 静态约束

> 列完整性——域完整性约束
>
> 表完整性——关系完整性约束

* 动态约束

> 触发器



### SQL语言实现数据库的静态完整性

静态约束形式：

```txt
Integrity Constraint ::= (O, P, A, R)
O:列或者表
P:需要定义
A:更新时检查（默认）
R:拒绝（默认）
```

在SQL中使用`Create Table`实现静态约束的定义，`Create Table`有三种功能：定义关系模式、**定义完整性约束**和定义物理存储特性。

> 定义完整性约束条件：
>
> * 列完整性
> * 表完整性

```SQL
CREATE TABLE 表名
( (列名 数据类型 [DEFAULT{default_constant|NULL}] 
   		[col_constr{col_constr ...}] |,table_constr 
  {,{列名 数据类型 [DEFAULT{default_constant|NULL}] 
   		[col_constr{col_constr ...}] |,table_constr}
   }));
   
--col_constr：列约束
--table_constr：表约束
```

#### Col_constr列约束

一种约束类型，对单一列的值进行约束

```sql
{NOT NULL|							--列值非空
	[CONSTRAINT constraintname]		--为约束名，以便后续撤销
	{UNIQUE							--列值唯一
	 |PRIMARY KEY					--列为主键
	 |CHECK(search_cond)			--列值满足条件，条件只能使用当前列值
	 |REFERENCES tablename [(colname)] --定义为外键，指向另一个表的某一列
	 	[ON DELETE{CASCADE|SET NULL}] } }
--引用另一表tablename的列colname的值，如果有ON DELETE CASCADE或ON DELETE SET NULL语句，则删除被引用表的某列值v时，要将本表该列值为v的行删除或列值更新为null，缺省为无操作。
```

**Col_constr列约束**：只能应用在单一列上，其后面的约束如UNIQUE，PRIMARY KEY及search_cond只能是单一列唯一、单一列为主键、单一列相关。

示例：假设Ssex只能取{“男”，“女”}，1<= Sage<=150，Dnumber是外键。

```sql
CREATE TABLE Student(Snumber char(8) not null unique, Sname char(10), 
	Ssex char(2) CONSTRAINT ctssex CHECK (Ssex="男" or Ssex="女"),
    Sage integer CHECK(Sage>=1 and Sage<150),
    Dnumber char(2) REFERENCES Dept(Dnumber) ON DELETE CASCADE,
    Sclass char(6));
```



#### Table_constr表约束

一种关系约束类型，对多列或元组的值进行约束



```sql
[CONSTRAINT constraintname]		--为约束名，以便后续撤销
	{UNIQUE	(colname{,colname...})	--几列混合在一起的值是唯一的
	 |PRIMARY KEY (colname{,colname...}) --几列联合为主键
	 |CHECK(search_cond) --元组多列值共同满足条件，仅为同一元组的不同列值
     |FOREIGN KEY (colname{,colname...}) --定义为外键，指向另一个表的某一列
	 |REFERENCES tablename [(colname{,colname...})] 
	 	[ON DELETE CASCADE] }
```

**Table_constr列约束**：是应用在关系上，及对关系的多元或元组进行约束，列约束时其特例。

示例：在上面基础上添加表约束，Snumber为主键。

```sql
CREATE TABLE Student(Snumber char(8) not null unique, Sname char(10), 
	Ssex char(2) CONSTRAINT ctssex CHECK (Ssex="男" or Ssex="女"),
    Sage integer CHECK(Sage>=1 and Sage<150),
    Dnumber char(2) REFERENCES Dept(Dnumber) ON DELETE CASCADE,
    Sclass char(6)，
    
    PRIMARY KEY (Snumber));
```

#### Create Table

示例：CHECK种的条件可以是SELECT-FROM-WHERE内任何WHERE后的语句，包含了子查询。

```sql
CREATE TABLE SC
	(Snumber char(8) CHECK(Snumber IN (SELECT Snumber FROM Student)), 
     Cnumber char(3) CHECK(Cnumber IN (SELECT Cnumber FROM Course)), 
     Score float(1) CONSTRAINT ctscore CHECK (Score>=0.0 and Score<=100.0));
--前两个CHECK相当外键的效果     
```

Create Table中定义的表约束可以在以后根据需要进行册小或追加。撤销或最佳约束的语句时Alter Table（不同系统可能存在差异）

```sql
ALTER TABLE tblname
	[ADD ({colname datatype [DEFAULT{default_const|NULL}]
          [col_constr{col_constr...}] |,table_constr}
          {, col_name ...}) ]
    [DROP {COLUMN colname|(colname{,colname...})}]
    [MODIFY(colname datatype
           [DEFAULT {default_const|NULL}] [[NOT] NULL]
           {,colname...} )]
	[ADD CONSTRAINT constr_name] 
    [DROP CONSTRAINT constr_name]
    [DROP PRIMARY KEY];
```

示例：撤销SC表中ctscore约束（未命名的约束是不能撤销的）

```sql
ALTER TABLE SC
	DROP CONSTAINT ctscore;
```

示例：若要再对SC表中的score进行约束，比如分数在0-150之间，则可新增加一个约束。在Oracle中增加新约束，需要通过修改列的定义来完成

```sql
ALTER TABLE SC
	MODIFY(Score float(1) CONSTRAINT nctscore CHECK(Score>=0.0 and Score<=150) );
```

有些DBMS支持独立追加约束，注意书写格式可能有差异，示例：

```sql
ALTER TABLE SC
	ADD CONSTRAINT nctscore CHECK(Score>=0.0 and Score<=150) );
```



#### 断言ASSERTION

> * 一个断言就是一个谓词表达式，它表达了希望数据库总能满足的条件；
> * 表约束和列约束就是一些特殊的断言；
> * SQL还提供了复杂表达的断言。其语法形式为：
>
> ```sql
> CREATE ASSERTION<assertion-name> CHECK<predicate>
> ```
>
> * 当一个断言创建后，系统将检测器有效性，并在每一次更新中测试更新是否违反改断言。

**断言测试增加了数据库维护的负担，小心使用复杂的断言。**

示例：

> TABLE:
>
> borrower(customer_name,loan_number,...)  //客户及其贷款
>
> account(account_number,...,balance)  //账户及其余额
>
> depositor(account_number, customer_name) //客户及其账户
>
> loan(loan_number, amount) //每一笔贷款

“每一笔贷款，要求至少一位借款者账户中存有最低数目的余额，如1000元。”

```sql
CREATE ASSERTION balance CONSTRAINT CHECK
	(NOT EXISTS(
    	SELECT * FROM loan
    	WHERE NOT EXISTS(
        	SELECT * FROM borrower, depositor, account
        	WHERE loan.loan_number=borrower.loan_number
        		AND borrower.customer_name=depositor.customer_name
        		AND depositor.account_number=account.account_number
        		AND account.account_number>=1000 )));
```

### SQL语言实现数据库的动态完整性

SQL语言实现数据库的静态完整性，需要通过触发器来实现，其约束形式如下：

```txt
Integrity Constraint ::= (O, P, A, R)
O:需要定义
P:需要定义
A:需要定义
R:需要定义
```

#### 触发器Trigger

Create Table中的表约束和列约束基本上都是静态的约束，也基本上都是对单一列或单一元组的约束（尽管有参照完整性），为实现动态约束以及多个元组之间的完整性约束，就需要触发器技术Trigger。

> **Trigger**是一种过程完整性约束（相比之下啊，Create Table中定义的都是非过程性约束），是一段程序，该程序可以在特定的时刻被自动触发执行，比如再一次更新操作之前执行，或在更新操作之后执行。

基本语法：

```sql
CREATE TRIGGER trigger_name BEFORE|AFTER
	{INSERT|DELETE|UPDATE [OF colname{,colname...}]}
	ON tablename [REFERENCING corr_name_def{,corr_name_def ...}]
	[FOR EACH ROW|FOR EEACH STATEMENT]--[对更新操作的每一条结果|整个更新操作完成]
	[WHEN (search_condition)]
		{statement
		 |BEGIN ATOMIC statement;{statement;...} END}
--FOR EACH ROW：对每一行；FOR EEACH STATEMENT对每一条SQL语句。
```

触发器Trigger意义：当某一时间发生（Before|After），对该事件产生的结果（或是每一元组，或者是整个操作的所有元组），检查条件search_condition，如果满足条件，则执行后面对程序。条件或程序段中引用的变量可用corr_name_def来限定。



事件：`BEFORE|AFTER{INSERT|DELETE|UPDATE...}` 

* 当一个事件（INSERT|DELETE|UPDATE）发生之前Before或发生之后After触发
* 操作发生，执行触发器操作需要处理两组值：更新前的值和更新后的值，这两个值由corr_name_def的使用来区分

>* corr_name_def的定义
>
>```sql
>{ OLD [ROW] [AS] old_row_corr_name --更新前的旧元组命名别名为
>| NEW [ROW] [AS] new_row_corr_name --更新后的新元组命名别名为
>| OLD TABLE [AS] old_table_corr_name --更新前的旧Table命名别名为
>| NEW TABLE [AS] new_table_corr_name --更新后的新Table命名别名为
>}
>```
>
>* corr_name_def将在检测条件或后面的动作程序段中被引用处理。



示例一：设计一个触发器当进行Teacher表更新元组时，使其工资只能升不能降

```sql
CREATE TRIGGER teacher_chgsal BEFORE UPDATE OF salary
	ON Teacher
	REFERENCING NEW x, OLD y
	FOR EACH ROW WHEN(x.Salary<y.Salary)
		BEGIN 
	raise_application_error(-20003,'invalid salary on update');
	--词条语句为Oracle的错误处理函数
		END;
--#################################################
Integrity Constraint ::= (O, P, A, R)
O:约束对象为Teacher表中的Salary对象
P:谓词是x.Salary<y.Salary
A:触发时间 BEFORE UPDATE 更新前
R:触发动作，BEGIN和END闭包内容
```

示例二：假设Student（Snumber，Sname， SumCourse），SumCourse为该同学已学习课程的门数，初始值为0，以后每选修一门都要对其增1，设计一个触发器自动完成上述功能。

```sql
CREATE TRIGGER sumc AFTER INSERT ON SC
	REFERENCING NEW ROW newi
	FOR EACH ROW
		BEGIN
	UPDATE Student SET SumCourse=SumCourse+1
	WHERE Snumber=:newi.Snumber;
		END;
```

示例三：假如Student（Snumber，Sname，Sage，Ssex，Sclass）中某一学生要变更其主码Snumber的值，如使原来的98030101变更为98030131，此时SC表中该同学已选课记录的Snumber也需要随其改变。设计一个触发器完成上述功能

```sql
CREATE TRIGGER updSnumber AFTER UPDATE OF Snumber ON Student
	REFERENCING OLD oldi, NEW newi
	FOR EACH ROW
		BEGIN
	UPDATE SC SET Snumber=newi.Snumber WHERE Snumber=oldi.Snumber;
		END;
```



## 数据库的安全性

数据库安全性是指DBMS应该保证的数据库的一种特性（机制或手段）：免受非法、非授权用户的使用、泄露、更改或破坏。数据库安全性管理涉及许多方面（法律、伦理、制度、数据安全等）



### DBMS的安全机制

> * 自主安全性机制：存取控制（Access Control）
>
>   通过权限再用户之间的传递，使用户自主管理数据库安全性。
>
> * 强制安全性机制：
>
>   通过对数据和用户强制分类，使得不同类别用户能过访问不同类别的数据
>
> * 推断控制机制：
>
>   防止通过历史信息推断除不该被其知道的信息；
>
>   防止通过公开信息（通常是一些聚集信息）推断出私密信息（个体信息），通常再一些由个体数据构成的公共数据库中此问题尤为重要。
>
> * 数据加密存储机制：
>
>   通过加密、解密保护数据，密钥、加密/解密方法与传输。

















