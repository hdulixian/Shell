[TOC]
#课程目标

- 掌握for循环语句的基本语法结构
- 掌握while和until循环语句的基本语法结构
- 能会使用RANDOM产生随机数
- 理解嵌套循环

# 一、随机数

## 1. 如何生成随机数？

**系统变量RANDOM**，默认产生0~32767之间的随机整数

**前言：**要想调用变量，不管你是什么变量都要给钱，而且是美元:heavy_dollar_sign:

~~~powershell
echo $RANDOM	# 打印一个随机数
set s| grep RANDOM	# 查看系统上一次生成的随机数
RANDOM=28325

echo $[$RANDOM%2]				# 产生0~1之间的随机数
echo $[$RANDOM%3]				# 产生0~2之间的随机数
echo $[$RANDOM%4]				# 产生0~3内的随机数
echo $[$RANDOM%10]			# 产生0~9内的随机数
echo $[$RANDOM%101]			# 产生0~100内的随机数
echo $[$RANDOM%51+50]		# 产生50-100之内的随机数
echo $[$RANDOM%900+100]	# 产生三位数的随机数
~~~

## 2. 实战案例

### ㈠ 随机产生以139开头的电话号码

**具体需求：**写一个脚本，产生一个phonenum.txt文件，用于存储随机产生的1000个以139开头的手机号，每行一个。

#### ① 思路

1. 产生1000个电话号码，脚本需要循环1000次 `FOR WHILE UNTIL`
2. 139+8位，后8位随机产生，可以让每一位数字都随机产生  `echo $[$RANDOM%10]`
3. 将随机产生的数字分别保存到变量里，然后加上139保存到文件里

#### ② 落地实现

~~~powershell
#!/bin/bash
file=/shell03/phonenum.txt
for ((i=1;i<=1000;i++)) # for i in {1..1000}
do
  n1=$[$RANDOM%10]
  n2=$[$RANDOM%10]
  n3=$[$RANDOM%10]
  n4=$[$RANDOM%10]
  n5=$[$RANDOM%10]
  n6=$[$RANDOM%10]
  n7=$[$RANDOM%10]
  n8=$[$RANDOM%10]
  echo "139${n1}${n2}${n3}${n4}${n5}${n6}${n7}${n8}" >> $file
  或者
  echo "139${n1}${n2}${n3}${n4}${n5}${n6}${n7}${n8}" | tee -a phonenum.txt
done

#!/bin/bash
file=/shell03/phonenum.txt
i=1
while [ $i -le 1000 ]
do
  n1=$[$RANDOM%10]
  n2=$[$RANDOM%10]
  n3=$[$RANDOM%10]
  n4=$[$RANDOM%10]
  n5=$[$RANDOM%10]
  n6=$[$RANDOM%10]
  n7=$[$RANDOM%10]
  n8=$[$RANDOM%10]
  echo "139${n1}${n2}${n3}${n4}${n5}${n6}${n7}${n8}" >> $file
  let i++
done
~~~

### ㈡ 随机抽出5位幸运观众

**具体需求：**从phonenum.txt存储的1000个手机号中抽取5个幸运观众并打印（只显示前3位及最后4位，其余用*代替）。

#### ① 思路

1. 确定幸运观众所在的行	`0-1000  随机找出一个数字   $[$RANDOM%1000+1]`
2. 将电话号码提取出来      `head -随机产生行号 phonenum.txt | tail -1`
3. 显示前3个和后4个数到屏幕   `echo 139****xxxx`

#### ② 落地实现

~~~powershell
#!/bin/bash
file=/shell03/phonenum.txt
for ((i=1;i<=5;i++))
do
  # 定位幸运观众所在行号
  linenum=`wc -l $file | cut -d' ' -f1`  # wc -l file 可用于打印文件的总行数及文件名
  luck_line=$[RANDOM%$linenum+1]
  # 取出幸运观众所在行的电话号码
  luck_num=`head -$luck_line $file | tail -1`
  # 显示到屏幕
  echo "139****${luck_num:7:4}"
  echo $luck_num >> luck.txt
  # 删除已经被抽取的幸运观众号码
  # sed -i "/$luck_num/d" $phone
done

#!/bin/bash
file=/shell03/phonenum.txt
for i in {1..5}
do
  linenum=`wc -l $file | cut -d' ' -f1`
  line=$[$RANDOM%$linenum+1]
  luck=`head -n $line $file | tail -1`
  echo "139****${luck:7:4}" && echo $luck >> /shell04/luck_num.txt
done
~~~

### ㈢ 批量创建用户（随机生成用户密码）

**需求：**批量创建5个用户，每个用户的密码为一个随机数

#### ① 思路

1. 循环5次创建用户
2. 产生一个密码文件用于保存用户的随机密码
3. 从密码文件中随机抽取一个为每个用户设置密码

#### ② 落地实现

~~~powershell
#!/bin/bash
# 产生一个保存用户名和密码的文件
echo user0{1..5}:itcast$[$RANDOM%9000+1000] | tr ' ' '\n' >> user_pass.file

for ((i=1;i<=5;i++))
do
  user=`head -$i user_pass.file | tail -1 | cut -d: -f1`
  pass=`head -$i user_pass.file | tail -1 | cut -d: -f2`
  useradd $user
  echo $pass | passwd --stdin $user
done
或者
for i in `cat user_pass.file`
do
  user=`echo $i | cut -d: -f1`
  pass=`echo $i | cut -d: -f2`
  useradd $user
  echo $pass | passwd --stdin $user
done
~~~

```powershell
pwgen命令可用于产生随机密码
语法格式：pwgen [ OPTION ] [ pw_length ] [ num_pw ]
选项：
    -c ：密码中至少包含一个大写字母
    -A ：密码中不包含大写字母
    -n ：密码中至少包含一个数字
    -0 ：密码中不包含数字
    -y ：密码中至少包含一个特殊符号
    -s ：生成完全随机密码
    -B ：密码中不包含歧义字符（例如1,l,O,0）
    -H ：使用SHA1 hash给定的文件作为一个随机种子
    -C ：在列中打印生成的密码
    -1 ：不要在列中打印生成的密码，即一行一个密码
    -v ：不要使用任何元音，以避免偶然的脏话

[root@server shell04]# pwgen -cn1 12
Meep5ob1aesa
[root@server shell04]# echo user0{1..3}:$(pwgen -cn1 12)
user01:Bahqu9haipho user02:Feiphoh7moo4 user03:eilahj5eth2R
[root@server shell04]# echo user0{1..3}:$(pwgen -cn1 12) | tr ' ' '\n'
user01:eiwaShuZo5hi
user02:eiDeih7aim9k
user03:aeBahwien8co
```

# 二、嵌套循环

1. 一个循环体内又包含另一个**完整**的循环结构，称为循环的嵌套。
2. 每次外部循环都会触发内部循环，直至内部循环完成，才接着执行下一次的外部循环。
3. for循环、while循环和until循环可以**相互**嵌套。

```powershell
#!/bin/env bash
for (())
do
  commands
  for (())
  do
  	commands
  done
  commands
done
```

##1. 应用案例

### ㈠ 打印指定图案

```powershell
1
12
123
1234
12345

5
54
543
5432
54321
```

### ㈡ 落地实现1

~~~powershell
#!/bin/bash
for i in {1..5}
do
	for j in $(seq 1 $i)
	do
		echo -n $j
	done
	echo
done

#!/bin/bash
for ((i=5; i>=1; i--))
do
	for ((j=5; j>=i; j--))
		do
			echo -n $j
		done
		echo
done
~~~

### ㈢ 落地实现2

~~~powershell
#!/bin/bash
for i in {1..5}
do
	j=1
	while [ $j -le $i ]
	do
		echo -n $j
		let j++
	done
	echo
done

#!/bin/bash
for i in $(seq 5 1)
do
	j=5
	while [ $j -ge $i ]
	do
		echo -n $j
		let j--
	done
	echo
done
~~~

##2. 课堂练习

**打印九九乘法表（三种方法）**

~~~powershell
1*1=1
1*2=2   2*2=4
1*3=3   2*3=6   3*3=9
1*4=4   2*4=8   3*4=12  4*4=16
1*5=5   2*5=10  3*5=15  4*5=20  5*5=25
1*6=6   2*6=12  3*6=18  4*6=24  5*6=30  6*6=36
1*7=7   2*7=14  3*7=21  4*7=28  5*7=35  6*7=42  7*7=49
1*8=8   2*8=16  3*8=24  4*8=32  5*8=40  6*8=48  7*8=56  8*8=64
1*9=9   2*9=18  3*9=27  4*9=36  5*9=45  6*9=54  7*9=63  8*9=72  9*9=81


#!/bin/bash
for ((i=1; i<=9; i++))
do
	for ((j=1; j<=i; j++))
	do
		echo -ne "$j*$i=$[j*i]\t"
	done
	echo
done

或者
#!/bin/bash
for i in `seq 9`
do
	for j in $(seq $i)
	do
		echo -ne "$j*$i=$[j*i]\t"
	done
	echo
done

或者
#!/bin/bash
i=1
while [ $i -le 9 ]
do
	j=1
	while [ $j -le $i ]
	do
		echo -ne "$j*$i=$[j*i]\t"
		let j++
	done
	echo
	let i++
done

或者
#!/bin/bash
i=1
until [ $i -gt 9 ]
do
	j=1
	until [ $j -gt $i ]
	do
		echo -ne "$j*$i=$[j*i]\t"
		let j++
	done
	echo
	let i++
done
~~~

# 三、阶段性补充总结

## 1. 变量定义

```
1）变量名=变量值
	echo $变量名
	echo ${变量名}
2）read -p "提示用户信息: " 变量名
3）declare -i/-x/-r 变量名=变量值
```

## 2. 流程控制语句

```powershell
1) if [ condition ]; then
		 commands
	 fi
	
2) if [ condition ]; then
		 commands
	 else
   	 commands
   fi
   
 3) if [ condition1 ];then
 		  commands
 	  elif [ condition2 ];then
 	 	  commands
 	  else
    	commands
    fi
```

## 3. 循环语句

```powershell
for / while / until
```

## 4. 影响shell程序的内置命令

~~~powershell
exit			退出整个程序
break		  结束当前循环，或跳出本层循环
continue 	忽略本次循环剩余的代码，直接进行下一次循环
shift			使位置参数向左移动，默认移动1位，可以使用shift 2
~~~

**举例说明：**用户自定义输入数字，然后计算和

```powershell
#!/bin/bash
sum=0
while [ $# -ne 0 ]
do
	let sum=$sum+$1
	shift
done
echo sum=$sum

#!/bin/bash
sum=0
for i
do
	let sum=$sum+$i
done
echo sum=$sum
```

##5. 补充扩展expect

有时候我们使用命令行进行交互时，不想频繁的做一些重复的事情，例如：每次ssh远程登录时都需要输入密码。使用spawn和expect可以自动完成一些交互。如以下为一个免密登录远程服务器的脚本：

```powershell
#!/usr/bin/expect
# 免密登录远程服务器
set login_name  "username"
set login_host  "host's ip"
set password    "guess what"

spawn ssh $login_name@$login_host
expect {
		"(yes/no)" { send "yes\r"; exp_continue }
		"password:" { send "$password\r" }
}
#expect $login_name@*   {send "ls\r" }  ;
interact
```

**需求1：**A远程登录到server上什么都不做

~~~powershell
#!/usr/bin/expect
# 开启一个程序
spawn ssh root@10.1.1.1
# 捕获相关内容
expect {
        "(yes/no)" { send "yes\r"; exp_continue }
        "password:" { send "123456\r" }
}
interact   # 交互

脚本执行方式：
# ./expect1.sh
# /shell04/expect1.sh
# expect -f expect1.sh

1）定义变量
#!/usr/bin/expect
set ip 10.1.1.1
set pass 123456
set timeout 5
spawn ssh root@$ip
expect {
	"(yes/no)" { send "yes\r"; exp_continue }
	"password:" { send "$pass\r" }
}
interact

2）使用位置参数
#!/usr/bin/expect
set ip [ lindex $argv 0 ]
set pass [ lindex $argv 1 ]
set timeout 5
spawn ssh root@$ip
expect {
	"(yes/no)" { send "yes\r"; exp_continue }
	"password:" { send "$pass\r" }
}
interact
~~~

**需求2：**A远程登录到server上操作

~~~powershell
#!/usr/bin/expect
set ip 10.1.1.1
set pass 123456
set timeout 5
spawn ssh root@$ip
expect {
	"(yes/no)?" { send "yes\r"; exp_continue }
	"password:" { send "$pass\r" }
}

expect "#"
send "rm -rf /tmp/*\r"
send "touch /tmp/file{1..3}\r"
send "date\r"
send "exit\r"
expect eof
~~~

**需求3：**结合shell脚本和expect，在多台服务器上创建1个用户

~~~powershell
[root@server shell04]# cat ip.txt 
10.1.1.1 123456
10.1.1.2 123456

1. 循环 useradd username
2. 登录远程主机—>ssh—>从ip.txt文件里获取IP和密码分别赋值给两个变量
3. 使用expect程序来解决交互问题

#!/bin/bash
# 循环在指定的服务器上创建用户和文件
while read ip pass
do
	/usr/bin/expect << -END &> /dev/null
	spawn ssh root@$ip
	expect {
		"(yes/no)" { send "yes\r"; exp_continue }
		"password:" { send "$pass\r" }
	}
	expect "#" {
		send "useradd yy1; rm -rf /tmp/*; exit\r" 
	}
	expect eof
	END
	echo "$ip服务器用户创建完毕"
done < ip.txt

或者
#!/bin/bash
cat ip.txt | while read ip pass
do
	{
		/usr/bin/expect << -HOU &> /dev/null
		spawn ssh root@$ip
    expect {
    	"yes/no" { send "yes\r"; exp_continue }
    	"password:" { send "$pass\r" }
    }
		expect "#"
		send "hostname\r"
		send "exit\r"
		expect eof
		HOU
	}&
done
wait
echo "user is ok...."

或者
#!/bin/bash
while read ip pass
do
	{
		/usr/bin/expect << -HOU &> /dev/null
		spawn ssh root@$ip
		expect {
			"yes/no" { send "yes\r"; exp_continue }
			"password:" { send "$pass\r" }
		}
		expect "#"
		send "hostname\r"
		send "exit\r"
		expect eof
		HOU
	}&
done < ip.txt
wait
echo "user is ok...."
~~~

#四、综合案例

##1. 实战案例1

### ㈠ 具体需求

写一个脚本，将跳板机上yunwei用户的**公钥推送**到局域网内可以ping通的所有机器上

说明：主机和密码文件已经提供

​	10.1.1.1:123456

​	10.1.1.2:123456

### ㈡ 案例分析

1. 跳板机上的yunwei用户生成秘钥对
   - 判断账号是否存在  (id yunwei)，不存在的话使用useradd命令创建yunwei用户
   - 判断yunwei用户是否有密钥对文件  [ -f xxx ]，没有的话使用ssh_keygen命令生成密钥对
   
2. 判断expect程序是否安装

3. 判断局域网内主机是否ping通
   - 循环判断  for  while
   - 循环体do......done    ping 主机  如果ping通，则调用expect程序自动应答推送公钥
   
4. 测试验证是否免密登录成功

   

   - 检查服务器上ssh服务端口号
   - 把公钥推送成功的主机信息保存到文件
   - 关闭防火墙和selinux
   - 日志记录
   - 推送公钥需要自动应答expect

### ㈢ 落地实现

#### ① 代码拆分

功能1：管理员root创建yunwei用户并安装expect软件包

```powershell
#!/bin/env bash
# 实现批量推送公钥

{
	id yunwei  # 判断跳板机上yunwei账号是否存在
	[ $? -ne 0 ] && useradd yunwei && echo 123 | passwd --stdin yunwei
} &> /dev/null
rpm -q expect  # 判断expect程序是否安装
[ $? -ne 0 ] && yum -y install expect && echo "expect安装完成..."
```

功能2：yunwei判断远程主机是否能够ping通并将yunwei用户的公钥推送到能ping通的机器上

```powershell
#!/bin/env bash

# 判断yunwei用户密钥对是否存在
home_dir=/home/yunwei
[ ! -f $home_dir/.ssh/id_rsa.pub ] && ssh-keygen -P '' -f id_rsa &> /dev/null

# 循环检查远程主机是否能够ping通并进行公钥推送
ip_txt=$home_dir/ip.txt
for i in `cat $ip_txt`
do
	ip=`echo $i | cut -d: -f1`
	pass=`echo $i | cut -d: -f2`
	ping -c1 $ip &> /dev/null
	if [ $? -eq 0 ]; then
		echo $ip >> ~/ip_up.txt
		/usr/bin/expect << -END &> /dev/null
		spawn ssh-copy-id root@$ip
		expect {
        "(yes/no)"  { send "yes\n"; exp_continue }
        "password:"  { send "$pass\n" }
		}
		expect eof
		END
	else
		echo $ip >> $home_dir/ip_down.txt
	fi
done
```

或者

~~~powershell
#!/bin/bash

# 判断公钥是否存在
[ ! -f /home/yunwei/.ssh/id_rsa ] && ssh-keygen -P '' -f ~/.ssh/id_rsa

# 循环判断主机是否ping通，如果ping通推送公钥
tr ':' ' ' < /shell04/ip.txt | while read ip pass
do
{
	ping -c1 $ip &> /dev/null
	if [ $? -eq 0 ]; then
		echo $ip >> ~/ip_up.txt
		/usr/bin/expect <<-END &>/dev/null
		spawn ssh-copy-id root@$ip
		expect {
			"yes/no" { send "yes\r"; exp_continue }
			"password:" { send "$pass\r" }
		}
		expect eof
    END
	fi
}&
done
wait
echo "公钥已经推送完毕，正在测试...."
~~~

测试验证：

```powershell
remote_ip=`head -1 ~/ip_up.txt`
ssh root@$remote_ip hostname
[ $? -eq 0 ] && echo "公钥推送成功"
```



##2. 实战案例2

编写一个脚本，用于统计web服务不同连接状态的个数

1. 找出查看网站连接状态的命令  `ss -natp | grep :80`
2. 如何统计不同的状态   循环去统计，需要计算

~~~powershell
#!/bin/bash
# count_http_80_state
# 统计每种状态web服务的个数

declare -A array1
states=`ss -ant | grep 80 | cut -d' ' -f1`

for i in $states
do
	let array1[$i]++
done

#通过遍历数组里的索引和元素打印出来
for j in ${!array1[@]}
do
	echo $j:${array1[$j]}
done
~~~

#五、课后实战

1、将/etc/passwd里的用户名分类，分为管理员用户，系统用户，普通用户。
2、写一个倒计时脚本，要求显示离2019年1月1日（元旦）的凌晨0点，还有多少天，多少时，多少分，多少秒。
3、写一个脚本把一个目录内的所有==空文件==都删除，最后输出删除的文件的个数。