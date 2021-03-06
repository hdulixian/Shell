---

---
[TOC]
#课程目标

- 熟悉awk**命令行模式**的基本语法结构
- 熟悉awk的相关内部变量
- 熟悉awk常用的打印函数print
- 能够在awk中匹配正则表达式打印相关的行

# 一、awk介绍

## 1. awk概述

- awk是一种编程语言，也是Linux/Unix下的一个工具，主要用于在Linux/Unix下对文本和数据进行处理，其中数据可以来自标准输入、一个或多个文件，或其它命令的输出。
- awk的处理文本和数据的方式：**逐行扫描文件**，默认从第一行到最后一行，寻找匹配特定模式的行，并在这些行上进行你想要的操作。
- awk分别代表其作者姓氏的第一个字母。因为它的作者是三个人，分别是Alfred Aho、Brian Kernighan、Peter Weinberger。
- gawk是awk的GNU版本，它提供了Bell实验室和GNU的一些扩展。


- 下面介绍的awk是以GNU的gawk为例的，在Linux系统中已把awk链接到gawk，所以下面全部以awk进行介绍。

## 2. awk能干啥?

1. awk用来处理文件和数据的，是类Unix下的一个工具，也是一种编程语言
2. awk可以用来统计数据，如网站的访问量、访问的IP量等等
3. awk支持条件判断，支持for和while循环

#二、awk使用方式

## 1. 命令行模式使用

### ㈠ 语法结构

```powershell
awk 选项 '命令部分' 文件名  # 语法格式与sed一致
特别说明：sed和awk中命令部分需要引用shell变量时，需要将单引号换为双引号
```

###㈡ 常用选项介绍

- -F  定义字段分割符（默认为空格）
- -v  定义变量并赋值

###㈢  **命令部分**

- 正则表达式，地址定位

```powershell
'/root/{awk语句}'						sed中： '/root/p'
'NR==1,NR==5{awk语句}'			sed中： '1,5p'
'/^root/,/^ftp/{awk语句}'  	sed中：'/^root/,/^ftp/p'
```

- {awk语句1**;**awk语句2**;**...}

```powershell
'{print $0;print $1}'				sed中：'p'
'NR==5{print $0}'						sed中：'5p'
注：awk命令语句之间用分号间隔
```

- BEGIN...END...

```powershell
'BEGIN{awk语句};{处理中};END{awk语句}'
'BEGIN{awk语句};{处理中}'
'{处理中};END{awk语句}'
```

## 2. 脚本模式使用

### ㈠ 脚本编写

```powershell
#!/bin/awk -f 		定义魔法字符
以下是awk引号中的命令清单，不要用引号保护命令，一行若有多个命令则用分号间隔
BEGIN{FS=":"}
NR==1,NR==3{print $1"\t"$NF}
...
```

### ㈡ 脚本执行

```powershell
方法1：
awk [选项] -f awk脚本文件  要处理的文本文件
awk -f awk.sh filename
对比sed脚本
sed -f sed.sh -i filename

方法2：
./awk脚本文件	要处理的文本文件
./awk.sh filename
对比sed脚本
./sed.sh filename
```

#三、 awk内部相关变量

| 变量              | 变量说明                           | 备注                             |
| ----------------- | ---------------------------------- | -------------------------------- |
| $0            | 当前处理行               |  |
| \$1,\$2,\$3…,\$n | 文件中每行以分隔符分割的不同字段 | awk -F: '{print \$1,\$3}' |
| NF            | 当前处理行的字段数（列数）        | awk -F: '{print NF}' |
| $NF           | 最后一列                           | $(NF-1)表示倒数第二列            |
| FNR/NR        | 行号                               |                                  |
| FS            | 定义输入分隔符                      | 'BEGIN{FS=":"};{print \$1,$3}' |
| OFS           | 定义输出字段分隔符，默认空格   | 'BEGIN{OFS="\t"};print \$1,$3}' |
| RS                | 输入记录分割符，默认换行           | 'BEGIN{RS="\t"};{print $0}'      |
| ORS               | 输出记录分割符，默认换行           | 'BEGIN{ORS="\n\n"};{print \$1,$3}' |
| FILENAME          | 当前输入的文件名                   |                                  |

## 1、常用内置变量举例

```powershell
awk -F: '{print $1,$(NF-1)}' 1.txt  # 以:为分隔符，打印第1列和倒数第2列
awk -F: '{print $1,$(NF-1),$NF,NF}' 1.txt  # 以:为分隔符，打印各行第1列、倒数第2列，最后1列以及列数
awk '/root/{print $0}' 1.txt  # 打印包含root的行
awk '/root/' 1.txt   # 打印包含root的行（{print $0}为默认操作）
awk -F: '/root/{print $1,$NF}' 1.txt  # 打印包含root的行的第1列和最后1列
awk 'NR==1,NR==5' 1.txt  # 打印1~5行
awk 'NR==1,NR==5{print $0}' 1.txt

awk 'NR==1,NR==5;/^root/{print $0}' 1.txt  # 打印1-5行以及包含root的行
root:x:0:0:root:/root:/bin/bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

## 2、内置变量分隔符举例

```powershell
FS 和 OFS
[root@server shell07]# awk 'BEGIN{FS=":"};/^root/,/^lp/{print $1,$NF}' 1.txt
root /bin/bash
bin /sbin/nologin
daemon /sbin/nologin
adm /sbin/nologin
lp /sbin/nologin

[root@server shell07]# awk -F: 'BEGIN{OFS="\t\t"};/^root/,/^lp/{print $1,$NF}' 1.txt  
或
[root@server shell07]# awk 'BEGIN{FS=":";OFS="\t\t"};/^root/,/^lp/{print $1,$NF}' 1.txt 
root            /bin/bash
bin             /sbin/nologin
daemon          /sbin/nologin
adm             /sbin/nologin
lp              /sbin/nologin

[root@server shell07]# awk -F: 'BEGIN{OFS="@@@"};/^root/,/^lp/{print $1,$NF}' 1.txt     
root@@@/bin/bash
bin@@@/sbin/nologin
daemon@@@/sbin/nologin
adm@@@/sbin/nologin
lp@@@/sbin/nologin


RS 和 ORS:
修改源文件前2行，增加制表符和内容：
vim 1.txt
root:x:0:0:root:/root:/bin/bash	hello	world	
bin:x:1:1:bin:/bin:/sbin/nologin	test1	test2	

[root@server shell07]# awk 'BEGIN{RS="\t"};{print $0}' 1.txt
[root@server shell07]# awk 'BEGIN{ORS="\t"};{print $0}' 1.txt
```

#四、 awk工作原理

`awk -F: '{print $1,$3}' /etc/passwd`

1. awk使用一行作为输入，并将这一行赋给内部变量$0，每一行也可称为一个记录，以换行符（RS）结束。

2. 每行被分隔符（默认为空格或制表符）分解成字段，每个字段存储在已编号的变量中，从$1开始。

   问：awk如何知道用空格来分隔字段的呢？

   答：因为有一个内部变量FS来确定字段分隔符（FS默认为空格或制表符）。

3. awk使用print函数打印字段，打印出来的字段默认以空格分隔（可以通过OFS设置输出字段分隔符）。$1,\$3之间有一个逗号，逗号比较特殊，它映射为另一个内部变量，称为输出字段分隔符OFS（OFS默认为空格）。

4. awk处理完一行后，将从文件中获取下一行，并将其存储在$0中，覆盖原来的内容，然后将新的字符串分隔成字段并进行处理。该过程将持续到所有行处理完毕为止。

# 五、awk使用进阶

## 1. 格式化输出 print 和 printf

```powershell
print函数类似echo		默认换行
[root@server ]# date | awk '{print "Month: "$2 "\nYear: "$1}'
Month: 8月16日
Year: 2020年

printf函数类似echo -n		默认不换行
[root@server ]# awk -F: '{printf "%-15s %-10s %-15s\n", $1,$2,$3}'  /etc/passwd
[root@server ]# awk -F: '{printf "|%15s| %10s| %15s|\n", $1,$2,$3}' /etc/passwd
[root@server ]# awk -F: '{printf "|%-15s| %-10s| %-15s|\n", $1,$2,$3}' /etc/passwd
[root@server ]# awk 'BEGIN{FS=":"};{printf "%-15s %-15s %-15s\n",$1,$6,$NF}' a.txt

注意：%-15s
  s 字符类型
  d 数值类型	
  15 占15个字符
  - 左对齐（默认右对齐）
  printf默认不会在行尾自动换行
```

## 2. awk变量定义

~~~powershell
[root@server ]# awk -v -F: NUM=3 '{ print $NUM }' /etc/passwd  # 错误的调用方式
[root@server ]# awk -v -F: NUM=3 '{ print NUM }' /etc/passwd   # 正确的调用方式
[root@server ]# awk -v num=1 'BEGIN{print num}'		# 正确的调用方式

注意：awk中调用定义的变量不需要加$
~~~

##3. awk中BEGIN...END的使用

​	①BEGIN：表示在程序开始前执行

​	②END ：表示所有文件处理完后执行

​	③用法：`'BEGIN{处理之前};{处理中};END{处理之后}'`

#### ㈠ 举例说明1

**以以下格式打印最后一列和倒数第二列（登录shell和家目录）**

~~~powershell
Login_shell		Login_home
************************

************************

[root@server ]# awk -F: 'BEGIN{ print "Login_shell\t\tLogin_home\n*******************"};{print $NF"\t\t"$(NF-1)""};END{print "************************\n"}' 1.txt
或者
[root@server ]# awk 'BEGIN{ FS=":"; print "Login_shell\t\tLogin_home\n*******************"};{print $NF"\t\t"$(NF-1)""};END{print "************************"}' 1.txt

Login_shell		Login_home
************************
/bin/bash			root
/sbin/nologin	bin
/sbin/nologin	daemon
/sbin/nologin	adm
/sbin/nologin	lp
************************
~~~

#### ㈡ 举例说明2

**以以下格式打印/etc/passwd里的用户名、家目录及登录shell**

```powershell
u_name      h_dir       shell
***************************

***************************

[root@server ]# awk -F: 'BEGIN{OFS="\t\t"; print"u_name\t\th_dir\t\tshell\n*****************"};{printf "%-20s %-20s %-20s\n",$1,$(NF-1),$NF};END{print "*****************"}' a.txt
或者
[root@server ]# awk -F: 'BEGIN{print "u_name\t\th_dir\t\tshell" RS "*****************"};{printf "%-15s %-20s %-20s\n",$1,$(NF-1),$NF};END{print "*****************"}'  a.txt

格式化输出：
echo		print
echo -n	printf
{printf "%-15s %-20s %-20s\n",$1,$(NF-1),$NF}
```

###4. awk和正则的综合运用

| 运算符 | 说明     |
| ------ | -------- |
| ==     | 等于     |
| !=     | 不等于   |
| >      | 大于     |
| <      | 小于     |
| >=     | 大于等于 |
| <=     | 小于等于 |
| ~      | 匹配     |
| !~     | 不匹配   |
| !      | 逻辑非   |
| &&     | 逻辑与   |
| \|\|   | 逻辑或   |

### ㈠ 举例说明

~~~powershell
awk -F: 'NR==1,/^lp/{print $0}' passwd  	# 打印从第1行到第一个以lp开头的行
awk -F: 'NR==1,NR==5{print $0}' passwd   	# 打印第1行到第5行
awk -F: '/^lp/,NR==10{print $0}' passwd   # 打印以lp开头的行到第10行 
awk -F: '/^root/,/^lp/{print $0}' passwd  # 打印以root开头的行到以lp开头的行  

awk -F: '/^root/ || /^lp/{print $0}' passwd  # 打印以root开头的行和以lp开头的行
awk -F: '/^root/{print $0};/^lp/{print $0}' passwd  # 打印以root开头的行和以lp开头的行
awk -F: '/^root/;/^lp/{print $0}' passwd  # 打印以root开头的行和以lp开头的行

awk -F: 'NR>=5 && NR<=10 {print $0}' passwd   # 打印5-10行  
awk -F: 'NR<10 && NR>5 {print $0}' passwd   # 打印6-9行

打印30-39行中以bash结尾的行：
[root@MissHou shell06]# awk 'NR>=30 && NR<=39 && $0 ~ /bash$/{print $0}' passwd
或者
[root@MissHou shell06]# awk 'NR>=30 && NR<=39 && /bash$/{print $0}' passwd
stu1:x:500:500::/home/stu1:/bin/bash
yunwei:x:501:501::/home/yunwei:/bin/bash
user01:x:502:502::/home/user01:/bin/bash
user02:x:503:503::/home/user02:/bin/bash
user03:x:504:504::/home/user03:/bin/bash

打印文件中1-5行中以root开头的行
[root@MissHou shell06]# awk 'NR>=1 && NR<=5 && $0 ~ /^root/{print $0}' 1.txt
root:x:0:0:root:/root:/bin/bash
打印文件中1-5行中不以root开头的行
[root@MissHou shell06]# awk 'NR>=1 && NR<=5 && $0 !~ /^root/{print $0}' 1.txt
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

理解;号和||的含义
打印3-8行以及以bash结尾的行
[root@MissHou shell06]# awk 'NR>=3 && NR<=8 || /bash$/{print $0}' 1.txt
[root@MissHou shell06]# awk 'NR>=3 && NR<=8; /bash$/{print $0}' 1.txt

打印IP地址
[root@MissHou shell06]# ifconfig eth0 | awk 'NR>1 {print $2}' | awk -F':' 'NR<2 {print $2}'    
[root@MissHou shell06]# ifconfig eth0 | grep Bcast | awk -F':' '{print $2}'| awk '{print $1}'
[root@MissHou shell06]# ifconfig eth0 | grep Bcast | awk '{print $2}'| awk -F: '{print $2}'

[root@MissHou shell06]# ifconfig eth0 | awk NR==2 | awk -F '[ :]+' '{print $4RS$6RS$8}'
[root@MissHou shell06]# ifconfig eth0 | awk -F '[ :]+' '/inet addr:/{print $4}'
~~~

## 4. 课堂练习

1. 显示可以登录操作系统的用户所有信息     从第7列匹配以bash结尾，输出整行（当前行所有的列）

```powershell
[root@MissHou ~] awk '/bash$/{print $0}' /etc/passwd
[root@MissHou ~] awk '/bash$/' /etc/passwd
[root@MissHou ~] awk -F: '$7 ~ /bash/' /etc/passwd
[root@MissHou ~] awk -F: '$NF ~ /bash/' /etc/passwd
[root@MissHou ~] awk -F: '$0 ~ /bash/' /etc/passwd
[root@MissHou ~] awk -F: '$0 ~ /\/bin\/bash/' /etc/passwd
```

2. 显示可以登录系统的用户名 

```powershell
[root@MissHou ~] awk -F: '$0 ~ /\/bin\/bash/{print $1}' /etc/passwd
```

3. 打印出系统中普通用户的UID和用户名

```powershell
500	stu1
501	yunwei
502	user01
503	user02
504	user03

[root@MissHou ~]# awk -F: 'BEGIN{print "UID\tUSERNAME"} {if($3>=500 && $3 !=65534 ) {print $3"\t"$1} }' /etc/passwdUID	USERNAME

[root@MissHou ~]# awk -F: '{if($3 >= 500 && $3 != 65534) print $1,$3}' a.txt 
redhat 508
user01 509
u01 510
YUNWEI 511
```

##5. awk的脚本编程

awk脚本语法格式： awk   [选项]   '地址定位{awk语句}'   文件名

### ㈠ 流程控制语句

#### ① if结构

```powershell
if [ xxx ]; then
	xxx
fi

格式：{ if(表达式)｛语句1;语句2;...｝}

[root@MissHou ~]# awk -F: '{ if($3>=500 && $3<=60000) {print $1,$3} }' passwd

[root@MissHou ~]# awk -F: '{ if($3==0) {print $1"是管理员"} }' passwd 
root是管理员
[root@MissHou ~]# awk 'BEGIN{ if($(id -u)==0) {print "是管理员"} }'
是管理员
```

#### ② if...else结构

```powershell
if [ xxx ]; then
	xxxxx
else
	xxx
fi

格式：{ if(表达式)｛语句;语句;...｝else｛语句;语句;...} }

[root@MissHou ~]# awk -F: '{ if($3>=500 && $3!=65534) {print $1"是普通用户"} else {print $1"不是普通用户"} }' passwd 
[root@MissHou ~]# awk 'BEGIN{ if($(id -u)>=500 && $(id -u)!=65534) {print "是普通用户"} else {print "不是普通用户"} }'
```

#### ③ if...elif...else结构

```powershell
if [xxxx]; then
	xxxx
elif [xxxx]; then
	xxxx
else
	xxxx
fi

格式：{ if(表达式1)｛语句;语句;...｝else if(表达式2)｛语句;语句;...｝else if(表达式3)｛语句;语句;...｝else｛语句;语句;...｝}

[root@MissHou ~]# awk -F: '{ if($3==0) {print $1,"是管理员"} else if($3>=1 && $3<=499 || $3==65534) {print $1,"是系统用户"} else {print $1,"是普通用户"}}' a.txt 

[root@MissHou ~]# awk -F: '{ if($3==0) {i++} else if($3>=1 && $3<=499 || $3==65534 ) {j++} else {k++}};END{print "管理员个数为:"i"\n系统用户个数为:"j"\n普通用户的个数为:"k }' a.txt 

[root@MissHou ~]# awk -F: '{if($3==0) {print $1,"is admin"} else if($3>=1 && $3<=499 || $3==65534) {print $1,"is sys users"} else {print $1,"is general user"} }' a.txt 
root is admin
bin is sys users
daemon is sys users
adm is sys users
lp is sys users
redhat is general user
user01 is general user
named is sys users
u01 is general user
YUNWEI is general user

[root@MissHou ~]# awk -F: '{ if($3==0) {print $1":管理员"} else if($3>=1 && $3<500 || $3==65534) {print $1":是系统用户"} else {print $1":是普通用户"} }' /etc/passwd

[root@MissHou ~]# awk -F: '{ if($3==0) {i++} else if($3>=1 && $3<500 || $3==65534){j++} else {k++} };END{print "管理员个数为:" i RS "系统用户个数为:"j RS "普通用户的个数为:"k}' /etc/passwd
管理员个数为:1
系统用户个数为:28
普通用户的个数为:27

[root@MissHou ~]# awk -F: '{ if($3==0) {print $1":是管理员"} else if($3>=500 && $3!=65534) {print $1":是普通用户"} else {print $1":是系统用户"}}' passwd 

[root@MissHou ~]# awk -F: '{if($3==0){i++} else if($3>=500){k++} else{j++}} END{print i; print k; print j}' /etc/passwd

[root@MissHou ~]# awk -F: '{if($3==0){i++} else if($3>999){k++} else{j++}} END{print "管理员个数: "i; print "普通用个数: "k; print "系统用户: "j}' /etc/passwd 

如果是普通用户打印默认shell，如果是系统用户打印用户名
[root@MissHou ~]# awk -F: '{if($3>=1 && $3<500 || $3==65534) {print $1} else if($3>=500 && $3<=60000 ) {print $NF}}' /etc/passwd
```

### ㈡ 循环语句

#### ① for循环

```powershell
打印1~5
[root@MissHou ~]# for ((i=1;i<=5;i++)); do echo $i; done
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=5;i++) {print i} }'

打印1~10中的奇数
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=10;i+=2) {print i} }'
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=10;i+=2) print i }'

计算1-10中奇数的和
[root@MissHou ~]# sum=0; for ((i=1;i<=10;i+=2)) do let sum+=i; done; echo $sum
[root@MissHou ~]# awk 'BEGIN{ sum=0; for(i=1;i<=10;i+=2) {sum+=i}; {print sum} }'
[root@MissHou ~]# awk 'BEGIN{ sum=0; for(i=1;i<=10;i+=2) sum+=i; print sum }'
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=10;i+=2) sum+=i; print sum }'
[root@MissHou ~]# for ((i=1;i<=10;i+=2)); do echo $i; done | awk '{sum+=$0}; END{print sum}'
```

#### ② while循环

```powershell
打印1-5
[root@MissHou ~]# i=1; while (($i<=5)); do echo $i; let i++; done
[root@MissHou ~]# i=1; while [ $i -le 5 ]; do echo $i; let i++; done

[root@MissHou ~]# awk 'BEGIN { i=1; while(i<=5) {print i; i++} }'
打印1~10中的奇数
[root@MissHou ~]# awk 'BEGIN{ i=1; while(i<=10) {print i; i+=2} }'
计算1-5的和
[root@MissHou ~]# awk 'BEGIN{ i=1; sum=0; while(i<=5) {sum+=i; i++}; print sum }'
[root@MissHou ~]# awk 'BEGIN{ i=1; while(i<=5) {(sum+=i); i++}; print sum }'
[root@MissHou ~]# i=1; while (($i<=5)); do echo $i; let i++; done | awk '{sum+=$0}; END{print sum}'
```

#### ③ 嵌套循环

~~~powershell
#!/bin/bash
for ((y=1;y<=5;y++))
do
	for ((x=1;x<=$y;x++))
	do
		echo -n $x	
	done
	echo
done

[root@MissHou ~]# awk 'BEGIN{ for(y=1;y<=5;y++) { for(x=1;x<=y;x++) {printf x}; print } }'
1
12
123
1234
12345
[root@MissHou ~]# awk 'BEGIN{ y=1; while(y<=5) { for(x=1;x<=y;x++) {printf x}; y++; print } }'
1
12
123
1234
12345

打印99口诀表：
[root@MissHou ~]# awk 'BEGIN{ for(y=1;y<=9;y++) { for(x=1;x<=y;x++) { printf x"*"y"="x*y"\t" }; print } }'
[root@MissHou ~]# awk 'BEGIN{ for(y=1;y<=9;y++) { for(x=1;x<=y;x++) printf x"*"y"="x*y"\t"; print } }'
[root@MissHou ~]# awk 'BEGIN{ i=1; while(i<=9) {for(j=1;j<=i;j++) { printf j"*"i"="j*i"\t" }; print; i++ } }'
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=9;i++) {j=1; while(j<=i) { printf j"*"i"="i*j"\t"; j++ }; print } }'

循环控制：
break		条件满足的时候中断循环
continue	条件满足的时候跳过循环
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=5;i++) { if(i==3) break; print i } }'
1
2
[root@MissHou ~]# awk 'BEGIN{ for(i=1;i<=5;i++) { if(i==3) continue; print i } }'
1
2
4
5
~~~

##6. awk算数运算

~~~powershell
+ - * / %(模) ^(幂2^3)
可以在模式中执行计算，awk都将按浮点数方式执行算术运算

[root@MissHou ~]# awk 'BEGIN{print 1+1}'
1
[root@MissHou ~]# awk 'BEGIN{print 2*3}'
6
[root@MissHou ~]# awk 'BEGIN{print 2/3}'
0.666667
[root@MissHou ~]# awk 'BEGIN{print 2%3}'
2
[root@MissHou ~]# awk 'BEGIN{print 2^3}'
8
~~~

# 六、awk统计案例

## 1、统计系统中各种类型的shell

```powershell
[root@MissHou ~]# awk -F: '{ shells[$NF]++ }; END{ for (i in shells) { print i, shells[i]} }' /etc/passwd

books[linux]++
books[linux]=1
shells[/bin/bash]++
shells[/sbin/nologin]++

/bin/bash 5
/sbin/nologin 6

shells[/bin/bash]++			a
shells[/sbin/nologin]++		b
shells[/sbin/shutdown]++	c

books[linux]++
books[php]++
```

## 2、统计网站访问状态

```powershell
[root@MissHou ~]# ss -antp | grep 80 | awk '{ states[$1]++ }; END{ for(i in states) { print i, states[i] } }'
TIME_WAIT 578
ESTABLISHED 1
LISTEN 1

[root@MissHou ~]# ss -an | grep :80 | awk '{ states[$2]++ }; END{ for(i in states) { print i, states[i] } }'
LISTEN 1
ESTAB 5
TIME-WAIT 25

[root@MissHou ~]# ss -an | grep :80 | awk '{ states[$2]++ }; END{ for(i in states) { print i, states[i] } }' | sort -k2 -rn
TIME-WAIT 18
ESTAB 8
LISTEN 1
```

## 3、统计访问网站的每个IP的数量

```powershell
[root@MissHou ~]# netstat -ant | grep :80 | awk -F: '{ ip_count[$8]++ }; END{ for(i in ip_count) { print i, ip_count[i] } }' | sort

[root@MissHou ~]# ss -an | grep :80 | awk -F":" '!/LISTEN/{ip_count[$(NF-1)]++}; END{ for(i in ip_count) { print i,ip_count[i] } }' | sort -k2 -rn | head
```

## 4、统计网站日志中PV量

```powershell
统计Apache/Nginx日志中某一天的PV量 　<统计日志>
[root@MissHou ~]# grep '27/Jul/2017' mysqladmin.cc-access_log | wc -l
14519

统计Apache/Nginx日志中某一天不同IP的访问量　<统计日志>
[root@MissHou ~]# grep '27/Jul/2017' mysqladmin.cc-access_log | awk '{ ips[$1]++}; END{ for(i in ips) { print i,ips[i] } }' | sort -k2 -rn | head

[root@MissHou ~]# grep '07/Aug/2017' access.log | awk '{ ips[$1]++ }; END{ for(i in ips) { print i,ips[i] } }' | awk '$2>100' | sort -k2 -rn
```

**名词解释：**

网站浏览量（PV）
名词：PV=PageView (网站浏览量)
说明：指页面的浏览次数，用以衡量网站用户访问的网页数量。多次打开同一页面则浏览量累计。用户每打开一个页面便记录1次PV。

名词：VV = Visit View（访问次数）
说明：从访客来到您网站到最终关闭网站的所有页面离开，计为1次访问。若访客连续30分钟没有新开和刷新页面，或者访客关闭了浏览器，则被计算为本次访问结束。

独立访客（UV）
名词：UV= Unique Visitor（独立访客数）
说明：1天内相同的访客多次访问您的网站只计算1个UV。

独立IP（IP）
名词：IP=独立IP数
说明：指1天内使用不同IP地址的用户访问网站的数量。同一IP无论访问了几个页面，独立IP数均为1

#七、课后作业

**作业1：**
1、写一个自动检测磁盘使用率的脚本，当磁盘使用空间达到90%以上时，需要发送邮件给相关人员
2、写一个脚本监控系统内存和交换分区使用情况

**作业2：**
输入一个IP地址，使用脚本判断其合法性：
必须符合ip地址规范，第1、4位不能以0开头，不能大于255不能小于0

#八、企业实战案例

#### 1. 任务/背景

web服务器集群中总共有9台机器，上面部署的是Apache服务。由于业务不断增长，每天每台机器上都会产生大量的访问日志，现需要将每台web服务器上的apache访问日志**保留最近3天**的，3天以前的日志**转储**到一台专门的日志服务器上，已做后续分析。如何实现每台服务器上只保留3天以内的日志？

#### 2. 具体要求

1. 每台web服务器的日志对应日志服务器相应的目录里。如：web1——>web1.log（在日志服务器上）
2. 每台web服务器上保留最近3天的访问日志，3天以前的日志每天凌晨5:03分转储到日志服务器
3. 如果脚本转储失败，运维人员需要通过跳板机的菜单选择手动清理日志

#### 3. 涉及知识点

1. shell的基本语法结构
2. 文件同步rsync
3. 文件查找命令find
4. 计划任务crontab
5. apache日志切割
6. 其他
