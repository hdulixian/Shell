---

---
[TOC]
#本机课程目标

- 掌握for循环语句的基本语法结构
- 掌握while和until循环语句的基本语法结构

# 一、for循环语句

## 1. for循环语法结构

### ㈠ 列表循环

> 列表for循环：将一组命令执行若干次

- **基本语法格式**

```powershell
for variable in {list}
     do
          commands
     done
或者
for variable in a b c
     do
         commands
     done
```

- **举例说明**

```powershell
for var in {1..10}; do echo $var; done
for var in 1 2 3 4 5; do echo $var; done
for var in `seq 10`; do echo $var; done
for var in $(seq 10); do echo $var; done
for var in {0..10..2}; do echo $var; done
for var in {2..10..2}; do echo $var; done
for var in {10..1}; do echo $var; done
for var in {10..1..-2}; do echo $var; done # 列表的步长放在最后
for var in `seq 10 -2 1`; do echo $var; done # seq命令中步长放在中间
for var in $(seq 10 -2 1); do echo $var; done
```

### ㈡ 不带列表循环

>  不带列表的for循环执行时由**用户指定参数和参数的个数**

- **基本语法格式**

```powershell
for variable
	do
        commands
	done
```

- **举例说明**

```powershell
#!/bin/env bash
for var
	do
		echo $var
	done
echo "脚本后面有$#个参数"
```

###  ㈢ 类C风格的for循环

- **基本语法结构**

```powershell
for (( expr1;expr2;expr3 ))
	do
		commands
	done

其中：
expr1：定义变量并赋初值
expr2：决定是否进行循环（条件）
expr3：决定循环变量如何改变，决定循环什么时候退出
如：
for (( i=1;i<=5;i++))
	do
		echo $i
	done
```

- **举例说明**

```powershell
 for ((i=1;i<=5;i++)); do echo $i; done
 for ((i=1;i<=10;i+=2)); do echo $i; done
 for ((i=2;i<=10;i+=2)); do echo $i; done
```

## 2. 应用案例

### ㈠ 计算1-100之间所有奇数的和

#### ① 思路

1. 定义一个变量用于保存奇数的和
2. 找出1-100之间的奇数，保存到另一个变量里
3. 从1-100中找出奇数后，再相加，然后将和赋值给变量
4. 遍历完毕后，将sum的值打印出来

#### ② 落地实现（条条大路通罗马）

```powershell
#!/bin/env bash
# 计算1-100的奇数和

sum=0 # 定义变量来保存奇数和
# for循环遍历1-100的奇数，并且相加，把结果重新赋值给sum
for i in {1..100..2}
	do
		let sum=$sum+$i
	done
echo "1-100的奇数和为:$sum" # 打印所有奇数的和


方法1：
#!/bin/bash
sum=0
for i in {1..100..2}
	do
		sum=$[$i+$sum]
	done
echo "1-100的奇数和为:$sum"

方法2：
#!/bin/bash
sum=0
for ((i=1;i<=100;i+=2))
	do
		let sum=$i+$sum
	done
echo "1-100的奇数和为:$sum"

方法3：
#!/bin/bash
sum=0
for ((i=1;i<=100;i++))
	do
		if [ $[$i%2] -ne 0 ]; then
			let sum=$sum+$i
		fi
done
或者
test $[$i%2] -ne 0 && let sum=$sum+$i

echo "1-100的奇数和为:$sum"

方法4：
sum=0
for ((i=1;i<=100;i++))
	do
		if [ $[$i%2] -eq 0 ]; then
			continue
		else
			let sum=$sum+$i
		fi
	done
echo "1-100的奇数和为:$sum"

#!/bin/bash
sum=0
for ((i=1;i<=100;i++))
	do
		test $[$i%2] -eq 0 && continue || let sum=sum+$i
	done
echo "1-100的奇数和是:$sum"
```

#### ③ 循环控制语句

- continue：循环体内continue语句下面的代码不执行，重新开始下一次循环
- break：跳出循环，执行循环体后面的代码
- exit：直接跳出本程序

```powershell
[root@server ~]# cat for5.sh 
#!/bin/bash
for i in {1..5}
	do
		test $i -eq 2 && break || touch /tmp/file$i
	done
echo hello hahahah
```

### ㈡ 判断输入的整数是否为质数

**质数(素数)：**只能被1和它本身整除的数，如2 3 5 7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71 73 79 83 89 97

#### ① 思路

1. 输入一个数，保存到一个变量里
2. 如果能被其他数整除就不是质数——>`$num%$i `是否等于0	`$i=2到$num-1`
3. 如果输入的数是1或者2取模根据上面判断又不符合，所以先排除1和2
4. 测试序列从2开始，输入的数是4——>得出结果`$num`不能和`$i`相等，并且`$num`不能小于`$i`

#### ② 落地实现

```powershell
#!/bin/bash
read -p "请输入一个正整数: " number
[ $number -eq 1 ] && echo "$number不是质数" && exit
[ $number -eq 2 ] && echo "$number是质数" && exit

for i in `seq 2 $[$number-1]`
	do
	 [ $[$number%$i] -eq 0 ] && echo "$number不是质数" && exit
	done
echo "$number是质数" && exit

-----------------------------------------------------------------------------

#!/bin/bash
read -p "输入一个正整数: " number
[ $number -eq 1 ] && echo "${number}不是质数" && exit
[ $number -eq 2 ] && echo "${number}是质数" && exit

for i in $(seq 2 $[number-1])
	do
		if [ $[number%i] -eq 0 ]; then
			echo "${number}不是质数" && exit
		fi
	done
echo "${number}是质数"
```

### ㈢ 批量创建用户

**需求：**批量创建5个新用户，以u1到u5命名，并统一加入一个新组，组名为class，统一修改用户密码为123

#### ① 思路

1. 添加用户的命令
2. 判断class组是否存在
3. 循环5次创建用户并设置用户密码

#### ② 落地实现

useradd或adduser命令可用于创建用户帐号。

语法格式：

```powershell
useradd [-d home] [-s shell] [-c comment] [-m [-k template]] [-f inactive] [-e expire] [-p passwd] [-r] name
```

参数选项：

```
-c：加上备注文字，备注文字保存在passwd的备注栏中。
-d：指定用户登入时的主目录，默认值为/home/<用户名>
-D：变更预设值。
-e：指定账号的失效日期，日期格式为MM/DD/YY，例如06/30/12。缺省表示永久有效。
-f：指定在密码过期后多少天即关闭该账号。如果为0账号立即被停用；如果为-1则账号一直可用。默认值为-1.
-g：指定用户所属的群组。值可以使组名也可以是GID。用户组必须已经存在的，期默认值为100，即users。
-G：指定用户所属的附加群组。
-m：自动建立用户的登入目录。
-M：不要自动建立用户的登入目录。
-n：取消建立以用户名称为名的群组。
-r：建立系统账号。
-s：指定用户登入后所使用的shell，默认值为/bin/bash。
-u：指定用户ID。该值在系统中必须是唯一的。0~499默认是保留给系统用户账号使用的，所以该值必须大于499。
```

代码实现：

```powershell
方法一：
#!/bin/bash
# 判断class组是否存在
grep -w class /etc/group &> /dev/null
[ $? -ne 0 ] && groupadd class
# 批量创建用户
for i in {1..5}
	do
		useradd -G class u$i
		echo 123 | passwd --stdin u$i  # --stdin表示接受echo后面的字符串作为密码
	done
```

## 3. 课堂练习

### ㈠ 批量创建用户

**需求1：**批量创建5个用户u1~u5，要求这几个用户的家目录都在/rhome中。

```powershell
#!/bin/bash
# 判断/rhome是否存在
[ -f /rhome ] && mv /rhome /rhome.bak
test ! -d /rhome && mkdir /rhome

for ((i=1;i<=5;i++))
	do
		useradd -d /rhome/u$i u$i
		echo 123 | passwd --stdin u$i
	done
```

### ㈡ 局域网内脚本检查主机网络通讯

**需求2：**写一个脚本，把局域网内能ping通的IP和不能ping通的IP分类，并保存到两个文本文件中。

以10.1.1.1~10.1.1.10为例：

```powershell
#!/bin/bash
IP=10.1.1
for ((i=1;i<=10;i++))
  do
    ping -c1 $IP.$i &> /dev/null
    if [ $? -eq 0 ]; then
      echo "$IP.$i is ok" >> /tmp/1.txt
    else
      echo "$IP.$i is down" >> /tmp/2.txt
    fi
    或者
    [ $? -eq 0 ] && echo "$IP.$i is ok" >> /tmp/1.txt || echo "$IP.$i is down" >> /tmp/2.txt
  done
```

**延伸扩展：shell脚本并发**

```powershell
{commands}&表示将程序放到后台并行执行，如果需要等待后台程序执行完毕再执行后续的命令，需要加wait

#!/bin/bash
IP=10.1.1
for ((i=1;i<=10;i++))
	do
	{
        ping -c1 $IP.$i &> /dev/null
        if [ $? -eq 0 ]; then
						echo "$IP.$i is ok" >> /tmp/ip_up.txt
        else
						echo "$IP.$i is down" >> /tmp/ip_down.txt
        fi
	}&
	done
wait
echo "done..."
```

### ㈢ 判断闰年

**需求3：**输入一个年份，判断是否是润年（能被4整除但不能被100整除，或能被400整除的年份即为闰年）

```powershell
#!/bin/bash
read -p "Please input year: " year
if [ $[$year%4] -eq 0 -a $[$year%100] -ne 0 ]; then
	echo "$year is leap year"
elif [ $[$year%400] -eq 0 ]; then
	echo "$year is leap year"
else
	echo "$year is not leap year"
fi
```

##4. 总结

- for循环语法结构
- for循环可以结合条件判断和流程控制语句
  - do......done  循环体
  - 循环体里可以是命令集合，再加上条件判断以及流程控制
- 控制循环语句
  - continue  跳过本次循环，继续执行下一次循环
  - break       跳出循环，执行循环体外的代码
  - exit          直接退出程序

#二、while循环语句

**特点：**条件为真就进入循环，条件为假就退出循环

##1. while循环语法结构

~~~powershell
while condition
	do
		commands
	done
如：
while  [ 1 -eq 1 ] 或者 (( 1 > 2 ))
  do
     commands
	done
~~~

**循环打印1-5数字**

```powershell
for ((i=1;i<=5;i++))
	do
		echo $i
	done

i=1
while [ $i -le 5 ]
	do
		echo $i
		let i++
	done
```

## 2. 应用案例

### ㈠ 脚本计算1-50偶数和

~~~powershell
#!/bin/bash
sum=0
i=2
while [ $i -le 50 ]
  do
    let sum=sum+i
    let i+=2
  done
echo "1-50的偶数和为: $sum"
~~~

### ㈡ 脚本同步系统时间

#### ① 具体需求

1. 写一个脚本，30秒同步一次系统时间，时间同步服务器10.1.1.1
2. 如果同步失败，则进行邮件报警,每次失败都报警
3. 同步成功也进行邮件通知，但是成功100次才通知一次

#### ② 思路

1. 每个30s同步一次时间，该脚本是一个死循环

2. 同步失败发送邮件

3. 同步成功100次发送邮件

#### ③ 落地实现

~~~powershell
#!/bin/bash
count=0
ntp_server=10.1.1.1
while true
  do
  	# rdate -s IP_Address 表示把远程主机的日期和时间回存到本地主机的系统时间。
    rdate -s $ntp-server &> /dev/null
    if [ $? -ne 0 ]; then
    	# 下条命令表示发送主题为 check system date，内容为 system date failed 的邮件到 root@localhost
      echo "system date failed" | mail -s 'check system date' root@localhost
    else
      let count++
      if [ $[$count%100] -eq 0 ]; then
      	echo "system date successfull" | mail -s 'check system date' root@localhost && count=0
      fi
    fi
  	sleep 30
  done
~~~

#三、until循环

**特点**：条件为假就进入循环，条件为真就退出循环

## 1. until语法结构

```shell
until condition  [ 1 -eq 1 ] 或 (( 1 >= 1 ))
	do
		commands
	done
```

**打印1-5数字**

```powershell
i=1
while [ $i -le 5 ]
	do
		echo $i
		let i++
	done
或者
i=1
until (( $i > 5 ))
  do
    echo $i
    let i++
  done
```

## 2. 应用案例

###㈠ 具体需求

1. 使用until语句批量创建10个用户，要求u1—u5的用户ID分别为1001—1005；
2. u6~u10用户的家目录分别在/rhome/u6—/rhome/u10
3. u1~u10的用户密码为123

### ㈡ 思路

### ㈢ 落地实现

```powershell
#!/bin/bash
i=1
until [ $i -gt 10 ]
  do
    if [ $i -le 5 ]; then
      useradd -u $[1000+$i] u$i && echo 123 | passwd --stdin u$i
    else
      [ ! -d /rhome ] && mkdir /rhome
      useradd -d /rhome/u$i u$i && echo 123 | passwd --stdin u$i		
    fi
  	let i++
  done
```

# 四、课后作业

1. 判断/tmp/run目录是否存在，如果不存在就建立，如果存在就删除目录里所有文件
2.  输入一个路径，判断路径是否存在，而且输出是文件还是目录，如果是链接文件，还得输出是  有效的连接还是无效的连接 
3. 交互模式要求输入一个ip，然后脚本判断这个IP 对应的主机是否 能ping 通，输出结果类似于：
   Server  10.1.1.20 is Down! 最后要求把结果邮件到本地管理员root@localhost mail01@localhost
4. 写一个脚本/home/program，要求当给脚本输入参数hello时，脚本返回world,给脚本输入参数world时，脚本返回hello。而脚本没有参数或者参数错误时，屏幕上输出“usage:/home/program hello or world”
5. 写一个脚本自动搭建nfs服务

