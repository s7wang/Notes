# Shell

[TOC]

特点

* 命令的堆积
* 基于标准化之上的工具化
* 特定的语法+系统的命令=文件
* 简化操作步骤，减少人为干预，减少系统故障
* 自动化的完成基础配置（系统初始化操作）、部署业务。
* 自动化定期备份恢复程序、自动化信息采集（硬件、系统、服务、网络、等）
* 自动化安装程序、自动化调整配置文件、自动化日志收集（ELK）
* 日志分析（取值->排序->去重->统计->分析）
* 自动化扩（缩）容（zabbix + shell）
* Shell什么都能做，但是要符合需求。



## Shell 基础语法

1. 变量

   自定义变量

   系统环境变量

   预先定义变量

   位置参数变量

2. 条件判断

   if  else

3. 循环语句

   for  while

4. 流程控制

   case

5. 函数

   function

6. 数组

   array

7. 正则表达

## Shell 格式

每个程序都有自己的解释器，一般都在开头写明解释器的路径。

```shell
#!解释器路径
例：
#！/bin/sh # shell 解释器
#！/usr/bin/python # python 解释器
```



## Shell 脚本特性

1. 命令补全、文件路径补全  table

2. 命令历史记忆功能 history

3. 别名、取消别名 alias/unalias

4. 常用快捷键 ctrl + u k a e l c z d w r y 

5. 前后台作业控制 bg，fg，job，screen

6. 输入输出重定向 >,>>,1>,2>>,&>, cat < 

7. 管道 | tee（| 将前者命令的标准输出交给后者命令的输入 tee）

8. 命令排序  ； &&  ||（；无逻辑先后执行， &&必须前者成功后者才执行， ||前面不成功在执行后者）

9. 通配符

   >`*` 匹配任意多个字符
   >
   >`?` 匹配任意一个字符
   >
   >`[]` 匹配括号中的任意一个字符，a-z, 0-9, A-Z, a-z
   >
   >`()` 在子shell中执行，相当于在当前sell中创建子shell执行，不改变当前shell环境
   >
   >`{}` 集合 touch file{1..9}
   >
   >`\` 转义符

10. echo输出颜色、printf（与C基本一致）格式化输出文本

    ```shell
    # 字体颜色
    echo -e "\e[30m test content黑 \e[0m"
    echo -e "\e[31m test content红 \e[0m"
    echo -e "\e[32m test content绿 \e[0m"
    echo -e "\e[33m test content黄 \e[0m"
    echo -e "\e[34m test content蓝 \e[0m"
    echo -e "\e[35m test content紫 \e[0m"
    echo -e "\e[36m test content天蓝 \e[0m"
    echo -e "\e[37m test content白 \e[0m"
    
    # 背景颜色
    echo -e "\e[40m test content黑 \e[0m"
    echo -e "\e[41m test content红 \e[0m"
    echo -e "\e[42m test content绿 \e[0m"
    echo -e "\e[43m test content黄 \e[0m"
    echo -e "\e[44m test content蓝 \e[0m"
    echo -e "\e[45m test content紫 \e[0m"
    echo -e "\e[46m test content天蓝 \e[0m"
    echo -e "\e[47m test content白 \e[0m"
    
    ```

    例

    ```shell
    #!/bin/bash
    # 定义颜色变量，\033、\e、\E是等价的，都是转义起始符
    RED='\e[1;31m'  # 红
    GREEN='\e[1;32m'  # 绿
    YELLOW='\033[1;33m'  # 黄
    BLUE='\E[1;34m'  # 蓝
    PINK='\E[1;35m'  # 粉红
    RES='\033[0m'  # 清除颜色
    
    
    echo -e "${RED} Red ${RES}"
    echo -e "${YELLOW} Yellow ${RES}"
    echo -e "${BLUE} Blue ${RES}"
    echo -e "${GREEN} Green ${RES}"
    echo -e "${PINK} Pink ${RES}"
    ```

    tput 命令会利用 terminfo 数据库中的信息，来控制和更改我们的终端，比如控制光标、更改文本属性、控制屏幕，以及为文本涂色。

    ```shell
    #!/bin/bash
    # 定义颜色变量，\033、\e、\E是等价的，都是转义起始符
    # 0 黑
    RED=$(tput setaf 1)  # 红
    GREEN=$(tput setaf 2)  # 绿
    YELLOW=$(tput setaf 3) # 黄
    BLUE=$(tput setaf 4)  # 蓝
    PINK=$(tput setaf 5)  # 粉红
    # 6 黄
    # 7 白
    RES=$(tput sgr0)  # 清除颜色
    
    
    echo -e "${RED} Red ${RES}"
    echo -e "${YELLOW} Yellow ${RES}"
    echo -e "${BLUE} Blue ${RES}"
    echo -e "${GREEN} Green ${RES}"
    echo -e "${PINK} Pink ${RES}"
    ```



## 1 Shell 变量

### 1.1 自定义变量

```shell
# 定义变量 `变量名=变量值`
# 查看变量 `echo $变量名` set显示所有变量，包括定义变量和环境变量
# 取消变量 `unset 变量名` 作用范围仅在当前shell中有效
a=1
b=2
echo $a
echo $b

```

### 1.2 环境变量

```shell
# 定义环境变量 `export 变量名`
# 引用环境变量 `$变量名 或 ${变量名}`
# 查看环境变量 `echo $变量名 或 env | grep Name`
# 取消环境变量 `unset 变量名`
# 变量作用范围 `当前shell和子shell`
```

永久生效改写 `/etc/profile` 在文本最后追加 `export 变量名=值` 。

```shell
export PATH = $PATH:/xxx/xxx
# 保存后执行 source /etc/profile
```



### 1.3 位置参数

```shell
$1 # 第一个参数
$2 # 第二个参数
# 最多到10 ${10}
```





### 1.4 预先定义变量

```shell
$0 # 脚本文件名 获取的是执行的路径 要获取文件名需要 `echo "$(basename $0)"`
$* # 所有的参数
$@ # 所有的参数
$# # 参数个数
$$ # 当前进程 PID
$! # 上一个后台进程的 PID
$? # 上一个命令的返回值 0 表示成功

```



### 1.5 变量赋值

```shell
# 显式赋值（变量名=变量值）
IP=192.168.56.11
str="Hello World"
today1=`data +%F` # 不常用
today2=$(data +%F) # 常用命令解析

# `read` 从键盘读入变量值 read 变量名
read -p "提示信息：" 变量名
read -t 5 -p "提示信息：" 变量名 #等待5秒自动退出
read -n 2 变量名

# 注意事项：定义定义或引用变量时注意事项：" " 弱引用  '' 强引用
school=xidian
echo "${school} is good!" # 显示 xidian is good！会解析变量
echo '${school} is good!' # 显示 ${school} is good! 所见即所得

# 命令替换 `命令` 等价于 $(命令)
today1=`data +%F` # 不常用
today2=$(data +%F) # 常用命令解析
```



### 1.6 变量数值运算

* 整数运算

```shell
# 整数数值运算 `expr + - \* / %`
expr 1 + 2 # 输出 3
expr $num1 + $num2
expr 3 \* 2 # 输出 6

# 整数运算 $((表达式)) + - * /
echo $((1+2)) # 输出 3
echo $((3**2)) # 3^2 输出 9

# 整数运算 $[表达式]
# 整数运算 `let 变量名=表达式`
let sum=2+3 # $sum 的值为 5
```

* 小数运算

```shell
# 小数运算 `bc + - * / %`
echo "2+4" | bc
echo "2^4" | bc
echo "scale=2;2/3" | bc # 控制输出两位小数
awk 'BEGIN{print 1/2}'
echo "print 5.0/2" | python
```



### 1.7 变量替代

```shell
# 变量替代 ${变量名-默认值}  -后边的相当于默认值
# 变量没有被赋值：会使用"默认值"替代
# 变量有被赋值（包括空值）：使用原值

unset var1
echo ${var1}
echo ${var1-test1} # 输出 test1

var2=111
echo ${var2-test2} # 输出 111

var3=
echo ${var3-test3} # 输出 "空值"

# 变量替代 ${变量名:-默认值}  -后边的相当于默认值
# 变量没有被赋值（包括空值）：会使用"默认值"替代
# 变量有被赋值：使用原值
var4=
echo ${var4:-test4} # 输出 test4
```



### 1.8 变量自增

```shell
# 变量自增
i=1
while [ $i -le 5 ];do # -le 相当于 <=
	# 语句块
	let i++ # 或 let ++i
done
```



## 2 Shell 条件测试

```shell
# Shell 条件测试 [ 是一个命令 后边要接空格
# 格式1：test 条件表达式
# 格式2：[ 条件表达式 ]
# 格式3：[[ 条件表达式 ]]
```



### 2.1 文件测试

```shell
[ -e dir|file ] # 校验dir|file是否存在
[ -d dir ] # 是否存在，而且是目录
[ -f file ] # 是否存在，而且是文件
[ -r file ] # 读权限校验
[ -w file ] # 写权限校验
[ -x file ] # 执行权限校验
[ -L file ] # 是否为符号连接文件
```



### 2.2 数值比较

```shell
# 数值比较 [ 数值1 操作符 数值2 ]
[ 1 -gt 10 ] # 大于
[ 1 -lt 10 ] # 小于
[ 1 -eq 10 ] # 等于
[ 1 -ne 10 ] # 不等于
[ 1 -ge 10 ] # 大于等于
[ 1 -le 10 ] # 小于等于
```

#### 实例 1

```shell
#!/usr/bin/sh
# 查看/磁盘使用率 如果大于 80 执行 if 的内容
Disk_Free=$(df -h | grep "/$" | awk '{print $(NF-1)}' |awk -F '%' '{print $1}')

if [ $Disk_Free -ge 80 ];then
	echo "/ Disk is used ${Disk_Free}%" > /tmp/disk_use.log
fi
```



```shell
#!/usr/bin/sh
# 打印当前系统的系统版本、内核、虚拟平台、主机名、内网IP、外网IP

System=$(hostnamectl |grep "System" |awk -F ':' '{print $2}')
kernel=$(hostnamectl |grep "Kernel" |awk -F ':' '{print $2}')
Vt=$(hostnamectl |grep "Virtualization" |awk -F ':' '{print $2}')
Static_Hostname=$(hostnamectl |grep "Static hostname" |awk -F ':' '{print $2}')
Domain_Hostname=$(hostnamectl |grep "Icon name" |awk -F ':' '{print $2}')

Network_Total=$(ls /etc/sysconfig/network-scripts/ |grep ifcfg|wc -l)
Network_sum=$(ls /etc/sysconfig/network-scripts/ |grep ifcfg|awk -F '-' '{print $2}' |xargs)
Network_Eth0=$(ifconfig eth0|awk 'NR==2{print $2}')
Network_lo=$(ifconfig lo|awk 'NR==2{print $2}')
Network_w=$(curl -s icanhazip.com)

echo "当前系统版本是： $System"
echo "当前内核版本是： $kernel"
echo "当前虚拟平台是： $Vt"
echo "当前静态主机名是： $Static_Hostname"
echo "当前动态主机名是： $Domain_Hostname"
echo "当前总网络IP是： $Network_Total"
echo "当前网卡名称分别是： $Network_sum"
echo "当前eth0网卡的IP地址是： $Network_Eth0"
echo "当前lo网卡IP地址是： $Network_lo"
echo "当前外网IP地址是： $Network_w"
```



```shell
#!/usr/bin/sh
# 打印CPU负载
load_1=$(w|awk 'NR==1'|awk -F ':' '{print $4}'|awk -F ',' '{print $1}')
load_5=$(w|awk 'NR==1'|awk -F ':' '{print $4}'|awk -F ',' '{print $2}')
load_15=$(w|awk 'NR==1'|awk -F ':' '{print $4}'|awk -F ',' '{print $3}')

echo "\n====================================="
echo "当前系统1分钟负载是： $load_1"
echo "当前系统5分钟负载是： $load_5"
echo "当前系统15分钟负载是： $load_15"
```



```shell
#!/usr/bin/sh
# 打印当前磁盘的分区状态
df -h|grep "/$"
# 以此命令为基础截取字段
echo "\n====================================="
echo ""
```



```shell
#!/usr/bin/sh
# 批量创建用户，配置固定密码 123
# 用户需要输入用户名
# 用户需要输入前缀
# 不管是否存在该用户都创建
# 给用户配置123密码

read -p "请输入用户数量： " usr
while true
do
    if [[ !($usr =~ ^[0-9]+$) ]]; then
        echo "错误，请输入数字。"
    else
		break
    fi
    read -p "请输入用户数量： " usr
done

read -p "请输入用户前缀： " pre

while true
do
    if [[ !($pre =~ ^[a-Z]+$) ]]; then
        echo "错误，请输入字母。"
    else
		break
    fi
    read -p "请输入用户前缀： " pre
done


for i in $(seq $usr);do
	usrname=$pre$i
	useradd $usrname &> /dev/null
	
	echo "123"|passwd --stdin $usrname &> /dev/null
	if [ $? -eq 0 ];then
		echo "Create $usrname is ok!"
	else
		echo "Create $usrname is fail!"
	fi
	
done
```



### 2.3 Shell 流程控制 if

```shell
# 流程控制 if

if [ 条件 ];then
	语句
elif [ 条件 ];then
	语句
else
	语句
fi

```



### 2.4 Shell 流程控制 case

```shell
# Shell 流程控制 case
case $1 in
	redhat)
		echo "centos"
		;;
	centos
		echo "redhat"
		;; 
	*)
		echo "USAGE: $(basename $0) [redhat|centos]"
		exit 1
esac

# 打印菜单
print_menu(){
    cat <<-EOF
    ============================
    menu
    ============================
    EOF
}

print_menu #执行

# 并发执行
{
	命令块
}& # 挂到后台

wait # 等待子shell执行完毕
 
```



```shell
#!/bin/bash
# 获取主机IP地址
>ip.txt

for i in {1..5}
do
	ip=127.0.0.$i
	{
        ping -c1 -W1 $ip &> /dev/null
        if [ $? -eq 0 ];then
            echo "$ip" >> ip.txt
        fi
	}&
done
	wait
	echo "Get IP is OK!"

# 生成对应的密钥
if [ ! -f ~/.ssh/id_rsa ];then
	ssh-keygen -P "" -f ~/.ssh/id_rsa
fi

# 批量分发
while read line
do

	/usr/bin/expect <<-EOF
		set timeout 5            
    	spawn ssh-copy-id  $line
    	expect {
        	"yes/no"	{send "yes\r"; exp_continue}
        	"password"	{send "1\r"}
    	}
    	# 当出现 #时执行如下命令
    	expect "#"
    	send "useradd bgx\r"
    	send "pwd\r"
    	send "exit\r"
    	expect eof
	EOF
done<ip.txt
```



### 2.5 Shell 循环语句

* for

```shell
# Shell 循环语句 for
for 变量名 in [ 取值列表 ]
do
	循环体
done
```

```shell
#!/bin/bash
# 例 判断单台主机是否成功
for i in {1..5}
do
    {
    ip=127.0.0.$i
    ping -c1 -W1 $ip &> /dev/null
    if [ $? -eq 0 ];then
    echo "$ip" | tee -a ip.txt
    fi
    }&
done
    wait
    echo "Get IP is OK!"
```

```shell
#!/bin/bash
# 从文件读入用户名 批量创建用户

case $1 in
	cr) # 创建
		for i in $(cat users.txt)
		do
			id $i &> /dev/null
			if [ $? -ne 0 ];then
				useradd $i && \
				echo "123" |passwd --stdin $i &> /dev/null
				echo "$i Is OK!"
		    	else
		    		echo "$i Is Ex"
			fi
		done
		;;

	rm) # 删除
		for i in $(cat users.txt)
		do
			userdel -r $i &>/dev/null
			rm -rf /home/$i 
			if [ $? -eq 0 ];then
				echo "$i delete successfully!"
			else
				echo "$i delete fail!"		
			fi 
		done
		;;
	*) # 其它
		echo "USAGE:$(basename $0) [cr|rm]"
esac

```



* while

```shell
# Shell 循环语句 while
while 条件测试 # : 和 true 为死循环 （命令执行成功为真）
do
	循环体
done
```



```shell
#!/bin/bash
# 从文件读入用户名 批量创建

while read user
do
    id $user &> /dev/null
    if [ $? -ne 0 ];then
        useradd $user && \
        echo "123" |passwd --stdin $user &> /dev/null
        echo "$user Is OK!"
    else
        echo "$user Is Ex"
    fi
done < users.txt
```

```shell
#!/bin/bash
# 从文件读入用户名和密码 批量创建

#!/bin/bash
# 从文件读入用户名 批量创建

while read line
do
	user=$(echo $line|awk '{print $1}')
	pwd=$(echo $line|awk '{print $2}')
    id $user &> /dev/null
    if [ $? -ne 0 ];then
        useradd $user && \
        echo "$pwd" |passwd --stdin $user &> /dev/null
        echo "$user Is OK!"
    else
        echo "$user Is Ex"
    fi
done < users2.txt


```



## 3 Shell 内置命令

```shell
exit # 退出整个程序
break # 退出当前循环
continue # 忽略本次循环后边的代码

```



## 4 Shell 关联数组

```shell
# 定义关联数组
declare -A tt_array_1
declare -A tt_array_2

# 关联数组赋值 
# 数组名[索引]=变量
tt_array_1[index1]=pear
tt_array_1[index2]=apple
tt_array_1[index3]=orange
tt_array_1[index4]=peach
# 数组名=([索引]=变量 [索引]=变量 [索引]=变量)
tt_array_2=([index1]=pear [index2]=apple [index3]=orange [index4]=peach)
# 查看关联数组
declare -A
# 访问所有值
${tt_array_1[@]} # 或 ${tt_array_1[*]}
# 访问所有索引
${!tt_array_1[@]} # 或 ${!tt_array_1[*]}

echo ${tt_array_1[indexi]} # 打印值
echo ${!tt_array_1[indexi]} # 打印索引
```



```shell
#!/bin/bash
#统计 cat < /etc/passwd |awk -F ':' '{print $NF}' | sort | uniq -c
declare -A array_passwd

while read line
do
	type=$(echo $line|awk -F ':' '{print $NF}')
	let array_passwd[$type]++
	
done < /etc/passwd

for i in ${!array_passwd[@]}
do
	echo -e "$i\t${array_passwd[$i]}"
done
```



## 5 Shell 函数

### 5.1 Shell 函数的定义和调用

函数是 Shell 脚本中自定义的一系列执行命令，一般来说函数应该设置有返回值（正确返回0，错误返回非0）使用函数最大的好处是可避免出现大量重复代码，增强脚本的可读性。

在 Shell 中定义函数的方法如下（其中 function 为定义函数的关键字，可以省略）：

```shell
# Shell 中定义函数
function FUNCTION_NAME() {
	command1 # 函数体中可以有多个语句，不允许有空语句
	command2 
    . . . 
}
```



```shell
#!/bin/bash
function hello() {
	echo "第一种函数定义方式"
}
hello2() {
	echo "第一种函数定义方式"
}

# 调用 
hello 
hello2

# 计算指定文件的行数
FILE=/etc/passwd # 指定要检查的文件

# 定义计数函数
function countLine() {
	local i=0
	while read line
	do
		let i++
	done<$FILE
	echo "$FILE have $i lines."
}

# 函数调用
countLine

```



### 5.2 Shell 函数返回值（状态码）

```shell
#!/bin/bash
# 判断文件是否存在

FILE=$1

function checkFileExist() {
	if [ -f $FILE ];then
		return 0
	else
		return 1
	fi
}

checkFileExist
if [ $? -eq 0 ];then
	echo "$FILE exist."
else
	echo "$FILE not exist."
fi

```



### 5.3 Shell 指定位置参数值

除了在脚本运行时给脚本传入位置参数外，还可以使用内置命令 set命令给脚本指定位置参数的值（又叫重置）。一旦使用 set 设置了传入参数的值，脚本将忽略运行时传入的位置参数，实际上是被 set 命令重置了位置参数的值。

```shell
#!/bin/bash
# 这里相当于把位置参数赋值
# $1<--5 $2<--1 $3<--2 $4<--3 $5<--4 $6<--5 $7<--6
set  5 1 2 3 4 5 6
count=0
for i in $*
do
	echo -e "$i\t\t$1\t$2\t$3\t$4\t$5\t$6\t$7"
	let count++
done	
```



## 6 Shell 正则表达



正则表达式

> Linux 正则表达式：`grep` , `awk` , `sed` 
>
> PHP , JAVA , PERL , PYTHON ( PERL 兼容正则 PCRE )

正则表达式和通配符是有本质区别的

特定字符

```shell
[[:space:]] # 空格
[[:digit:]] # 0-9
[[:lower:]] # a-z
[[:upper:]] # A-Z
[[:alpha:]] # a-Z
```

### 6.1 基础正则表达式

元字符意义 BRE ，正则表达式实际上就是一些特殊字符，赋予了特殊意义

> `^word` 	匹配以 word开头的内容。vi/vim ^代表一行开头。
>
> `word$` 	匹配以 word结尾的内容。vi/vim $代表一行结尾。
>
> `^$` 		  表示空行
>
> `.` 			代表且仅能代表任意一字符
>
> `\` 			转义符，脱马甲，还原原型
>
> `*` 			重复前面的一个字符0次或者多次
>
> `?` 			匹配之前的项1次或0次
>
> `+` 			匹配之前的项1次或多次
>
> `()` 		  匹配表达式，创建用于匹配的子串
>
> `|` 			交替匹配|两边的任意一项 ab(c|d) 匹配abc或abd
>
> `.*` 		  匹配所有字符。延伸`^.*`以任意多个字符开头；`.*$`以任意多个字符结尾
>
> `[abc]` 	匹配字符集合内任意一个字符
>
> `[^abc]` 		不匹配字符集合内任意一个字符，`^[abc]` 匹配以字符集合内任意一个字符开头的
>
> `a\{n,m\}`  	匹配前一个字符重复n到m次。如果用`egrep/sed -r`可以去掉斜线
>
> `a\{n\}`  	匹配前一个字符重复至少n次。如果用`egrep/sed -r`可以去掉斜线



```shell
grep "^m" file.txt # 以m开头行
grep "m$" file.txt # 以m结尾行
grep -vn "^$" file.txt # 过滤空行 -v排除匹配内容 -n打印行号
grep "." file.txt # 匹配字符（不包括空行）
grep ".*" file.txt # 匹配所有（包括空行
grep "sdda.sd" file.txt # 匹配任意字符串sdda（未知）sd
grep "\.$" file.txt # 匹配以.结尾行
grep -o "8*" file.txt # 精确匹配 -o 只看匹配结果
grep "[abc]" file.txt # 匹配a b c 中任意一个
grep "[^a-z]" file.txt # 匹配除a-z的内容
grep "^[a-z]" file.txt # 匹配a-z开头的行
grep "8\{3\}" file.txt # 匹配重复8三次（匹配888）/grep -E "8{3}" file.txt
grep "8\{3,7\}"file.txt # 匹配重复8三到七次
grep "8\{1,\}" file.txt # 匹配重复8至少一次
```



### 6.2 sed 流编辑器

sed 是一个流编辑器，非交互式的编辑器

> sed 是一种在线的、非交互的编辑器，他一次处理一行内容。处理时的行存储在临时缓冲区中，称为”模式空间“（pattern space）接着用 sed 命令处理缓冲区的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，直到文件结尾。
>
> sed 不会改变文件内容，除非使用重定向。
>
> sed 主要用来自动编辑一个或多个文件；简化对文件的反复操作、编写转换程序等。

```shell
## sed 命令格式
# sed [options] 'command' file(s)
# sed [options] -f scriptfile file(s)
# 注：sed 和 grep 不同，sed不管是否找到指定模式，退出状态都为 0 ，自由当出现命令语法错误时才返回非 0 状态。

# sed 正则使用
# 正则表达式是扩在斜杠间的模式，用于查找和替换，一下是sed支持的元字符
# 使用基本元字符集 ^, $, ., *, [], [^], \< \>, \( \), \{ \}
# 使用扩展元字符集 ?, +, { }, |, ( )
# 使用扩展元字符的方式 转义 或  sed -r

## sed 基本用法
sed -r '' /etc/passwd # 
sed -r 'p' /etc/passwd # 打印所有内容 不取消默认打印
sed -r -n '2p' /etc/passwd # 取消默认输出 并打印第二行
sed -r -n '/root/p' /etc/passwd # 
sed -r 's/root/alice/' /etc/passwd # 
sed -r 's/root/alice/g' /etc/passwd # 
sed -r 's/root/alice/gi' /etc/passwd # 
sed -r '/root/d' /etc/passwd # 
sed -r '\crootcd' /etc/passwd # 
sed -r '' /etc/passwd # 
sed -r '' /etc/passwd # 


```

```shell
## sed 命令
# sed 对指定文件进行操作，包括打印、删除、修改、追加等

## 选项 功能
# -e <script> 以选项中指定的script来处理输入的文本文件
# -f <script文件> 以选项中指定的script文件来处理输入的文本文件
# -n 取消默认的输出 仅显示script处理后的结果
# -i 直接修改对应文件
# -r 支持扩展元字符

## 命令
# a	新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
# c 取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
# d 删除， d 后面通常不接任字符
# i 插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)
# l 列出非打印字符
# p 打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行
# n 读入下一输入行，从下一条命令开始处理 
# q 结束或退出sed
# ! 对所选行以外的所有行应用命令
# s 取代，可以直接进行取代的工作 通常这个s的动作可以搭配正规表示法！
# g 在行内进行全局替换
# i 忽略大小写
# r 从文件中读
# w 将行写入文件
# y 将字符转换为另一个字符（不支持正则表达）
# h 把模式空间里的内容复制到暂存缓冲区（覆盖）
# H 把模式空间里的内容追加到暂存缓冲区
# g 取出暂存缓冲区的内容，将其复制到模式空间，覆盖该处原有内容
# G 取出暂存缓冲区的内容，将其复制到模式空间，追加在原有内容后面
# x 交换暂存缓冲区于模式空间的内容

```



```shell
##  sed 后面接的动作，请务必以 '' 两个单引号括住
# 在testfile文件的第四行后添加一行内容为"newLine",并将结果输出到标准输出
sed -e '4a\newLine' testfile

# 将 /etc/passwd 的内容列出并且列印行号，同时，请将第 2~5 行删除
nl /etc/passwd | sed '2,5d'
# 只要删除第 2 行
nl /etc/passwd | sed '2d'
# 要删除第 3 到最后一行
nl /etc/passwd | sed '3,$d'

# 在第二行后(亦即是加在第三行)加上『drink tea?』字样
nl /etc/passwd | sed '2a drink tea'
# 那如果是要在第二行前
nl /etc/passwd | sed '2i drink tea'
# 如果是要增加两行以上，在第二行后面加入两行字 
# 每一行之间都必须要以反斜杠『 \ 』来进行新行的添加
# 例如 Drink tea or ..... 与 drink beer?
nl /etc/passwd | sed '2a Drink tea or ......\
> drink beer ?'

# 将第2-5行的内容取代成为『No 2-5 number』
nl /etc/passwd | sed '2,5c No 2-5 number'

# 仅列出 /etc/passwd 文件内的第 5-7 行
nl /etc/passwd | sed -n '5,7p'


## 数据的搜寻并显示
# 搜索 /etc/passwd有root关键字的行
nl /etc/passwd | sed -n '/root/p'

## 数据的搜寻并删除
# 删除/etc/passwd所有包含root的行，其他行输出
nl /etc/passwd | sed  '/root/d'
## 数据的搜寻并执行命令
# 搜索/etc/passwd,找到root对应的行，执行后面花括号中的一组命令，
# 每个命令之间用分号分隔，这里把bash替换为blueshell，再输出这行 
nl /etc/passwd | sed -n '/root/{s/bash/blueshell/;p;q}' # 最后的q是退出。

## 数据的搜寻并替换
# 除了整行的处理模式之外， sed 还可以用行为单位进行部分数据的搜寻并取代。
# 基本上 sed 的搜寻与替代的与 vi 相当的类似
sed 's/要被取代的字串/新的字串/g'

## 多点编辑
# 一条sed命令，删除/etc/passwd第三行到末尾的数据，并把bash替换为blueshell
nl /etc/passwd | sed -e '3,$d' -e 's/bash/blueshell/'
# -e表示多点编辑，第一个编辑命令删除/etc/passwd第三行到末尾的数据，
# 第二条命令搜索bash替换为blueshell。

## 直接修改文件内容(危险动作)
## sed 可以直接修改文件的内容，不必使用管道命令或数据流重导向 -i
sed -i 's/\.$/\!/g' regular_express.txt
```















