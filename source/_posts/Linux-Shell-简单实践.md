---
title: Linux Shell 简单实践
date: 2017-08-18 10:08:12
toc: true
tags:
    - linux
    - shell
    - 代码
  
---

本文为本人在阅读《鸟哥私房菜》，阅读之余、思考之余,动手实践操作
<!--More-->
## sh01.sh > hello world!
```bash
#!/bin/bash
echo -e "hello world! \a \n"
exit 110
```

## sh02.sh > read
```bash
#!/bin/bash
# Program:
#	User inputs his first name and last name.Program show the user full name.

# History:
#	2017/7/11 huangmj First release

# 提示用户输入firstname
read -p"Please input your firstname: " firstname

# 提示用户输入lastname
read -p"Please input your lastname: " lastname

# 结果荧屏输出用户全名
echo -e "\n Your fullname is: $firstname $lastname"
```
## sh03.sh
```bash
#!/bin/bash
# Program:
#	Program create three files,which named by user`s input and date command

# History:
#	2017/7/11 huangmj First release

# 1.让用户输入档案名,并取得fileuser这个变量：
echo -e "I will use 'touch' command to create 3 files."
read -p "Please input your filename: " file_user

# 2.为了避免用户随意按Enter，利用变量功能分析文档开始名是否设定
# 判断有没有设定文档开始名
filename=${file_user:-"filename"}

# 3. 开始利用date指令来取得所需的文档名：
# 前2天的日期.
date1=$(date -d "2 day ago" '+%Y-%m-%d')
# 前1天的日期
date2=$(date -d "1 day ago" '+%Y-%m-%d')
# 今天的日期
date3=$(date +%Y%m%d)
# 设定文件名
file1=${filename}${date1}
file2=${filename}${date2}
file3=${filename}${date3}

# 4.建立文件
touch "$file1"
touch "$file2"
touch "$file3"
```
## sh04.sh > cross two numbers
```bash
#!/bin/bash
# Program:
#	User inputs 2 integer numbers;program will cross these two numbers.

# History:
#	2017/7/11 huangmj First release

echo -e "You shold input 2 numbers,I will cross them!\n"
read -p "fiste number: " first_num
read -p "second number: " second_num
total=$(($first_num*$second_num))
echo -e "The result of $first_num X $second_num is ==> $total"
```
## sh05.sh > check file
```bash
#!/bin/bash
# Program:
#	User input a filename,program will check the flowing:
#	1. exist?
#	2. file/directory?
#	3. file permisssions

# History:
#	2017/7/11 huangmj First release

# 1.让用户输入文件名，并判断用户是否真的有输入字符串
echo -e "Please input a filename,I will check the filename's type and permission.\n "
read -p "input a filename: " file_name
# 如果用户未输入任何字符窜，输入提示语句，退出。
test -z $file_name && echo "you must input a filename." && exit 0

# 2.判断文件是否存在，如果不存在先显示提示语，退出。
test ! -e $file_name && echo "the filename '$file_name' do not exit." && exit 0

# 3.开始判断文件的类型与属性
test -f $file_name && file_type="regulare file"
test -d $file_name && file_type="directory"
test -r $file_name && perm="read able"
test -w $file_name && perm="writ able"
test -x $file_name && perm="execut able"

# 4. 开始输出信息
echo "The filename: $file_name is a $file_type"
echo "And the permission are: $perm"
```
## sh06.sh > if
```bash
#!/bin/bash
# Program:
#	This program shows the user's choice

# History:
#	2017/7/11 huangmj First release

read -p "Please input(Y/N): " yn
[ "$yn" == "Y" -o "$yn" == "y" ] && echo "OK,continue" && exit 1
[ "$yn" == "N" -o "$yn" == "n" ] && echo "Oh,interrupt!" && exit 0
echo "I don't know what your choice($yn) is" && exit 2
```
### sh06-2.sh
```bash
# 使用if [...];then fi
if [ "$yn" == "Y" ]||[ "$yn" == "y" ];then
	echo "OK,continue"
	exit 1
fi
if [ "$yn" == "N" ]||[ "$yn" == "n" ];then
	echo "Oh,interrupt!"
	exit 0
fi
echo "I don't know what your choice($yn) is" && exit 2
```
### sh06-3.sh
```bash
# 使用if elif elif ... else fi
if [ "$yn" == "Y" ]||[ "$yn" == "y" ];then
	echo "OK,continue"
elif [ "$yn" == "N" ]||[ "$yn" == "n" ];then
	echo "Oh,interrupt!"
else
        echo "I don't know what your choice($yn) is"
fi
```
## sh07.sh > script name and parameters
```bash
#!/bin/bash
# Program:
#	Program shows the script name,parameters

# History:
#	2017/7/11 huangmj First release

echo "The script name is         ====>$0"
echo "Total parameters number is ====>$#"
[ "$#" -lt 2 ] && echo "the number of parameter is less than 2. Stop here" && exit 0
echo "Your whole parameter is    ====>$@"
echo "The 1st parameter is       ====>$1"
echo "the 2nd parameter is       ====>$2"
```
## sh08.sh >  if and case
```bash
#!/bin/bash
# Program:
#	Check $1 is equal to "hello"

# History:
#	2017/7/11 huangmj First release

if [ "$1" == "hello" ];then
	echo "Hello,shell world,I am huangmj."
elif [ "$1" == "" ];then
	echo "You must input parameters,example>{$0 someword}"
else
	echo "The only parameter is 'hello',example>{$0 hello}"
fi 
```

### sh08-2.sh
```bash
#!/bin/bash
# Program:
#	Check $1 is equal to "hello".Using case...

# History:
#	2017/7/12 huangmj First release

case $1 in
	"hello")
		echo "Hello,shell world,I am huangmj."
	;;
	"")
		echo "You must input parameters,example>{$0 someword}"
	;;
	*)
		echo "The only parameter is 'hello',example>{$0 hello}"
	;;
esac
```
### sh08-3.sh
```bash
#!/bin/bash
# Program:
#	Using netstat and grep to detect WWW,SSH,FTP and Mail services.

# History:
#	2017/7/11 huangmj First release

# 1.告知一些要执行的操作
echo "Now,I will detect your Linux server's services!"
echo -e "The WWW,ftp,ssh and mail will be detect!\n"

# 2.开始进行一些测试工作，并且罗列出一些资讯
# 检测80端口在否？
testing=$(netstat -tuln | grep ":80")
if [ "$testing" != "" ];then
	echo "WWW is runing in your server system."
fi

# 检测22端口在否？
testing=$(netstat -tuln | grep ":22")
if [ "$testing" != "" ];then
        echo "ssh is runing in your server system."
fi

# 检测21端口在否？
testing=$(netstat -tuln | grep ":21")
if [ "$testing" != "" ];then
        echo "ftp is runing in your server system."
fi
```
## sh10.sh > calculate date
```bash
#!/bin/bash
# Program:
#	User input his demobilization(退伍）date,
#       Program calculate how many days before he demobilize(退伍）.

# History:
#	2017/7/12 huangmj First release

# 1. 提示信息,告知输入信息格式
echo "This program will try to calculate:"
echo "How many days before you demobilization date...Please "
read -p "Input your demobilization date (YYYYMMDD ex>20170711): " demobilize_date

# 2. 判断用户输入的信息是否有错，利用正则表达
# 看看是否是输入8个数字
date_d=$(echo $demobilize_date|grep '[0-9]\{8\}')
echo "date_d:$date_d"
if [ "$date_d" == "" ];then
	echo "Your input the wrong date formate..."
	exit 1
fi

# 3.开始计算时间
# 退伍日期秒数
declare -i date_dem_sec=`date --date="$demobilize_date" +%s`
# 现在时间秒数
declare -i date_now=`date +%s`
# 剩余时间秒数
declare -i date_total_s=$(($date_dem_sec-$date_now))
# 秒数转为日子数
declare -i date_days=$(($date_total_s/60/60/24))
# 判断是否已退伍
if [ "$date_total_s" -lt "0" ];then
	echo "You had been demobilizetion(退伍) befor:"$((-1*$date_days))" ago"
else
	declare -i date_hours=$(($(($date_total_s-$date_days*60*60*24))/60/60))
	echo "You will demobililize(退伍) after $date_days days and $date_hours hours."
fi
```
## sh11.sh > function
```bash
#!/bin/bash
# Program:
#	Using function to repeat information

# History:
#	2017/7/12 huangmj First release

# 1. 定义函数
function print_info(){
	# 加上-n可以不断行继续输出
	# 下面的$1非下下面的$1
	echo -n "Your choice is:$1:"	
}

# 2. 程序开始，调用函数
echo "This program will print your seclection."
case $1 in
	"one")
		# 将参数做大小写转换
		# echo "HELLO WORLD" | tr 'A-Z' 'a-z' 	   >hello world
		# echo "hello 123 world 456" | tr -d '0-9' >hello world 
		# more -> http://man.linuxde.net/tr
		print_info 1;echo $1 | tr 'a-z' 'A-Z'
	;;
	"two")
		print_info 2;echo $1 | tr 'a-z' 'A-Z'
	;;
	"three")
		print_info 3;echo $1 | tr 'a-z' 'A-Z'
	;;
	*)
		echo "Usage $0 {one|two|three}"
	;;
esac
# bash sh11.sh two > Your choice is:2:TWO
```
## sh12.sh > loop
```bash
#!/bin/bash
# Program:
#	Using "while", 
#	Repeat question until user input correct answer.

# History:
#	2017/7/12 huangmj First release

while [ "$yn" != "yes" -a "$yn" != "YES" ]
do
	read -p "Please input yes/YES to stop this program:" yn
done
echo "OK! you input the correct answer."
```

### sh12-2.sh Using "until"
```bash
#!/bin/bash
# Program:
#	Using "until", 
#	Repeat question until user input correct answer.

# History:
#	2017/7/12 huangmj First release

until [ "$yn" == "yes" -o "$yn" == "YES" ]
do
	read -p "Please input yes/YES to stop this program:" yn
done
echo "OK! you input the correct answer."
```
### sh12-3.sh Calculate "1+2+3+...+100"
```bash
#!/bin/bash
# Program:
#	Using "while", Calculate "1+2+3+...+100"

# History:
#	2017/7/12 huangmj First release

# 1. 初始化数据为0
sum=0
# 1,2,3...,100
i=0

# 2. 执行循环累加
while [ "$i" != "100" ]
do
	i=$(($i+1))
	sum=$(($sum+$i))
done

# 3. 输出结果
echo "The result of '1+2+3+...+100'is ===> $sum"
```
### sh12-4.sh 
```bash
#!/bin/bash
# Program:
#	Using "for"

# History:
#	2017/7/12 huangmj First release

for animal in dog cat elephant
do 
	echo "There are ${animal} s..."
done
```
### sh12-5.sh
```bash
#!/bin/bash
# Program:
#	Use id,finger command to check system account's information

# History:
#	2017/7/12 huangmj First release

# 获取帐号名称
users=$(cut -d ':' -f 1 /etc/passwd)

for user_name in $users 
do 
	id $user_name
	finger $user_name
done
```
### sh12-6.sh
```bash
#!/bin/bash
# Program:
#	Use ping command to check the network's PC state

# History:
#	2017/7/12 huangmj First release

network="192.168.1"
# seq 为 sequence(连续）
for site_num in $(seq 1 100)
do
	ping -c 1 -w 1 ${network}.${site_num} &> /dev/null && result=0 || result=1
	if [ "$result" == 0 ];then
		echo "Server ${network}.${site_num} is OK UP."
	else
		echo "Server ${network}.${site_num} is Down!"
	fi
done
```
### sh12-7.sh
```bash
#!/bin/bash
# Program:
#	User input dir name,program find the permission of files

# History:
#	2017/7/12 huangmj First release

read -p "Please input a directory: " dir

# 1. 验证用户输入
if [ "$dir" == "" -o ! -d "$dir" ];then
	echo "The $dir is NOT  exit in your system!"
	exit 1
fi

# 2. 开始执行
# 列出该目录下的所有文件名
file_list=$(ls $dir)
for file_name in $file_list
do
	perm=""
	test -r "$dir/$file_name" && perm="$perm Readable"
	test -w "$dir/$file_name" && perm="$perm Writeable"
	test -x "$dir/$file_name" && perm="$perm Executable"
	echo "The file $dir/$file_name's permission is $perm"
done
```