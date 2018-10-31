# 5 Linux
## 5.1 Shell 编程
### 5.1.1 基本命令
#### 5.1.1.1 chmod

```shell
# 为文件 file 添加可执行权限
chmod +x file
```
#### 5.1.1.2 特殊字符

```shell
# $# 表示传递到脚本的参数个数
# $* 以一个单字符显示所有传递到脚本的参数
# $$ 脚本当前运行进程的 ID 号
# $@ 与 $* 相似，不被" "包含时，$*和$@都以$1 $2… $n 的形式组成参数列表。被" "包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n" 的形式组成一个整串；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式组成一个参数列表。
# $? 显示最后命令的退出状态
#!/bin/bash
echo $0
echo $1
echo ${10}
echo 个数是：$#
echo 所有的参数是:$*
echo 这个进程执行了我: $$
echo 所有的参数是:$@
echo 上一条命令的执行状态：$?
```
执行结果 :

1

10

个数是：10

所有的参数是:1 2 3 4 5 6 7 8 9 10

这个进程执行了我: 3364

所有的参数是:1 2 3 4 5 6 7 8 9 10

上一条命令的执行状态：0

#### 5.1.1.3 运算符

```shell
# (()) 和 [] 进行算术运算
((count++))
[count++]
a=$((count++))
b=$[count++]

# = 和 == 的区别，当等号两边是纯数字时，二者的使用方法相同，当等号两边包含正则表达式时，则必须使用 [[]]，并且 * 必须写在等号的右边
#!/bin/bash
a=123ab*
b=123abc
if [[$b == $a]]
then echo a 和 b 相等
elif echo a 和 b 不相等
fi
```
#### 5.1.1.4 流程控制

```shell
# if else
#!/bin/bash
echo 请输入月份
read month
if [$month -ge 3 -a $month -le 5]
then echo 春天
if [$month -ge 6 -a $month -le 8]
then echo 夏天
if [$month -ge 9 -a $month -le 11]
then echo 秋天
if [$month -ge 12 -a $month -le 2]
then echo 冬天

# for
#!/bin/bash
echo 方法一
for item in 1 2 3
do
echo $item
done

echo 方法二
for item in 1 2 3;do;echo $item;done

echo 方法三
for ((i=1; i<= 10; i++))
do
echo $i
done

echo 方法四
for item in {2..9};do;echo $item;done

# while
i=1
while(($i <= 10))
do
echo $i
done

# case
echo 请输入你的分数
read score
score=(($score / 10))
case score in
10)
echo 满分
;;
9)
echo 优秀
;;
8)
echo 良好+
;;
7)
echo 良好-
;;
6)
echo 及格
;;
*)
echo 不及格
;;
esac
```
#### 5.1.1.5 ``

```shell
# `` 可以引用其所包含的值并将其赋值给一个变量
num=`cat /usr/local/file/test/times.txt`
```
#### 5.1.1.6 -e

```shell
# -e 可以判断一个文件是否存在
if [ -e /usr/local/file/test/times.txt ]
then
num=`cat /usr/local/file/test/times.txt`
fi
```
#### 5.1.1.7 替换

```shell
# 替换 old 第一次出现所在行的第一个 old
:s/old/new

# 替换 old 第一次出现所在行的所有 old
:/s/old/new/g

# 替换整篇文章中的 old
:%s/old/new/g
```

## 5.2 shell 脚本

Shell 脚本编写流程 :

1、#!/bin/bash

2、设置环境变量

3、设置一些主类、目录等常量

4、Shell 主程序，结合流程控制（if .... else）去分别执行 Shell 命令

### 5.2.1 在集群中执行相同命令

```shell
#!/bin/bash
num=21
command=$*
tmp=${command% *}
if [ x$tmp = x ]
then
echo 请输入要执行的命令
else
  while((num <= 23))
  do
  	echo ------------------------------------- hadoop$num -------------------------------------
  	echo
    ssh root@hadoop$num "source /etc/profile;$command"
    ((num++))
    echo
  done
fi
```
### 5.2.2 在集群中复制文件

```shell
#!/bin/bash
num=22
source=$1
dest=$2
if [ x$source = x ]
  # 源文件为空
then
echo 请输入源文件
else
  # 源文件不为空，判断源文件是否为绝对路径
  isRoot=`echo $source | grep "/"`
  if [ x$isRoot = x -o x$source = x ]
    # 源文件不是绝对路径，将源文件加上当前目录
  then
    source=`pwd`/$source
  else
    # 源文件是绝对路径，直接使用源文件
    source=$1
  fi
  if [ x$dest != x ]
  # 目标文件不为空，直接使用目标文件
  then
    dest=$dest
  else
  # 目标文件为空，使用源文件，即将源文件复制到目的主机相同的路径
  dest=${source%/*}
  fi
  while(($num <= 23))
  do
    echo ------------------------------------- hadoop$num -------------------------------------
    echo
    scp -r $source root@hadoop$num:$dest
    ((num++))
    echo
  done
fi
```
### 5.2.3 工作流调度
#### 5.2.3.1 把 Flume 的输出数据，移动到 MapReduce 的输入目录

```shell
export JAVA_HOME=/export/server/jdk1.8.0_65
export PATH=${JAVA_HOME}/bin:$PATH
export HADOOP_HOME=/export/server/hadoop-2.7.4
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH


log_flume_dir=/weblog/flume-collection
log_pre_input=/data/weblog/preprocess/input

preday=`date -d '-1 day' +%Y-%m-%d`
syear=`date +%Y`
smonth=`date +%m`
sday=`date +%d`


files=`hadoop fs -ls $log_flume_dir | grep $preday | wc -l`
if [ $files  -gt 0 ]; then
	hadoop fs -mv ${log_flume_dir}/$preday ${log_pre_input}
fi

问题：
	a、flume收集到的数据的目录应该会有好多个（以天为单位），这些目录用来存放flume收集到的数据
	b、mr程序读取目录中的数据，会不会处理目录的子目录的数据（测试了：不会，而且有子目录会报错，意味着，必须把当天收集到的数据，放在同一个目录中）
```
#### 5.2.3.2 调用 MapReduce

**预处理 MapReduce :**

```shell
export JAVA_HOME=/export/server/jdk1.8.0_65
export PATH=${JAVA_HOME}/bin:$PATH
export HADOOP_HOME=/export/server/hadoop-2.7.4
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH

log_pre_input=/data/weblog/preprocess/input
log_pre_output=/data/weblog/preprocess/output

preday=`date -d '-1 day' +%Y-%m-%d`
syear=`date +%Y`
smonth=`date +%m`
sday=`date +%d`

files=`hadoop fs -ls ${log_pre_input} | grep $preday | wc -l`
if [ $files -gt 0 ]; then
	hadoop fs -mkdir -p $log_pre_output
	hadoop jar pre.jar $log_pre_input/$preday $log_pre_output/$preday
fi
```
**pageviews 预处理 :**

```shell
export JAVA_HOME=/export/server/jdk1.8.0_65
export PATH=${JAVA_HOME}/bin:$PATH
export HADOOP_HOME=/export/server/hadoop-2.7.4
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH

log_pageviews_input=/data/weblog/preprocess/output
log_pageviews_output=/data/weblog/preprocess/pageviews

preday=`date -d '-1 day' +%Y-%m-%d`
syear=`date +%Y`
smonth=`date +%m`
sday=`date +%d`

files=`hadoop fs -ls ${log_pageviews_input} | grep $preday | wc -l`
if [ $files -gt 0 ]; then
	hadoop fs -mkdir -p $log_pageviews_output
	hadoop jar pageviews.jar $log_pageviews_input/$preday $log_pageviews_output/$preday
fi
```
**visit 预处理 :**

```shell
export JAVA_HOME=/export/server/jdk1.8.0_65
export PATH=${JAVA_HOME}/bin:$PATH
export HADOOP_HOME=/export/server/hadoop-2.7.4
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH

log_visit_input=/data/weblog/preprocess/pageviews
log_visit_output=/data/weblog/preprocess/visit

preday=`date -d '-1 day' +%Y-%m-%d`
syear=`date +%Y`
smonth=`date +%m`
sday=`date +%d`

files=`hadoop fs -ls ${log_visit_input} | grep $preday | wc -l`
if [ $files -gt 0 ]; then
	hadoop fs -mkdir -p $log_visit_output
	hadoop jar visit.jar $log_visit_input/$preday $log_visit_output/$preday
fi
```
#### 5.2.3.3 导入 Hive

```shell
export JAVA_HOME=/export/server/jdk1.8.0_65
export PATH=${JAVA_HOME}/bin:$PATH
export HADOOP_HOME=/export/server/hadoop-2.7.4
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH
export HIVE_HOME=/export/server/hive

preday=`date -d '-1 day' +%Y-%m-%d`
syear=`date +%Y`
smonth=`date +%m`
sday=`date +%d`

log_pre_output=/data/weblog/preprocess/output/$preday
log_pageviews_output=/data/weblog/preprocess/pageviews/$preday
log_visit_output=/data/weblog/preprocess/visit/$preday

load_pre_sql="load data inpath '$log_pre_output' into table ods_weblog_origin partition(datestr='$preday')"
load_pageviews_sql="load data inpath '$log_pageviews_output' into table ods_weblog_pageviews partition(datestr='$preday')"
load_visit_sql="load data inpath '$log_visit_output' into table ods_weblog_visit partition(datestr='$preday')"


$HIVE_HOME/bin/hive -e "${load_pre_sql};${load_pageviews_sql};${load_visit_sql};"
```
### 5.2.4 远程运行其它机器上的脚本

~~~txt
其中 host 是远程机器的主机名，command 是要在远程机器上运行的脚本（包括完整路径）
ssh root@host command
~~~

### 5.2.5 读取文件内容

~~~txt
其中 file 为文件名
cat file while read line
do
echo $line
done
~~~

 

### 5.2.6 Java 程序打包成 jar 包在 Linux 上运行的方法

**本地运行**

```txt
//    其中 java -jar 运行的是已指定主类的 jar 包，java -cp 运行的是未指定主类的 jar 包，jar1、jar2 是 Java 程序打包成的 jar 包以及引用的依赖包，main-class 是 Java 程序打包成 jar 包的主类的全类名（包括包名和类名），args 是要传递的参数
java -cp jar1:jar2 main-class args
```
**yarn 上运行**

```txt
//    其中 jar1 是 Java 程序打包成的 jar 包，main-class 是 Java 程序打包成 jar 包的主类的全类名（包括包名和类名），args 是要传递的参数，jar2、jar3 是 Java 程序打包成的 jar 包引用的依赖包
yarn jar jar1 main-class args -libjars jar2:jar3
```
## 5.3 Linux 命令
### 5.3.1 > 和 >> 的区别

```shell
# > 表示覆盖原文件的内容，>> 表示向原文件中追加内容
echo 用户日志信息 > /usr/local/file/test/access.log
echo 用户日志信息 >> /usr/local/file/test/access.log
```
### 5.3.2 rpm

```shell
# 查询所安装的软件
rpm -qa | grep java

# 卸载所安装的软件
rpm -e 软件名

# 安装软件
rpm -ivh 软件名
```
### 5.3.3 截取字符串

```shell
# 以 / 为分隔符去除 / 及右边所有字符，保留左边所有字符
${var%/*}

# 以 / 为分隔符去除 / 及左边所有字符，保留右边所有字符
${var##*/}
```
### 5.3.4 判断一个字符串中是否含有指定字符

```shell
# 判断一个字符串中是否含有 /
isRoot=`echo $source | grep /`
```
### 5.3.5 读取一个文本中指定行的数据

```shell
# 读取一个文本中第二行的数据
sed -n -2p file
```
### 5.3.6 tail

~~~txt
tail -f filename 说明：监视filename文件的尾部内容（默认10行，相当于增加参数 -n 10），刷新显示在屏幕上。退出，按下CTRL+C。
tail -n 20 filename 说明：显示filename最后20行。
tail -r -n 10 filename 说明：逆序显示filename最后10行
~~~
```shell
```

### 5.3.7 awk

~~~txt
awk 的基本选项：
-F 指定分隔符，-F后面加指定的分隔符
-f 指定脚本，-f后面加脚本文件，从该脚本中读取awk命令，代替输入

输出a,b,c，$1 $2 $3 分别对应输出的a，b，c
echo a b c | awk'{print $1,$2,$3}'

指定分隔符是" : ",  输出passwd文本的第一列，即系统的用户名
awk -F ":" '{print $1}' /etc/passwd

用-f 选项调用一个awk脚本来对sample.txt做操作，awk脚本是test.sh
awk -f test.sh sample.txt
~~~

### 5.3.8 java

~~~txt
在 linux 上运行打包成 jar 的 java 程序，其中 mainClassReference 是主类的引用名，包括包名和类名，agrs 是主方法中通过 args 接收的参数
java -cp jarName.jar mainClassReference args
~~~

### 5.3.9 dirname

~~~txt
获取文件的路径，其中 file 为文件名
dirname file
~~~

### 5.3.10 截取字符串

~~~txt
1. # 截取，删除左边字符，保留右边字符
${str#*reg}

2. % 截取，删除右边字符，保留左边字符
${str%reg*}
~~~

### 5.3.11 tar

```txt
//    压缩文件
tar -zcvf 打包压缩后的文件名 要打包的文件

//    解压文件
tar -zxvf 压缩文件 -C 指定解压的位置
```
### 5.3.12 网卡操作

```txt
//    查看网卡信息
ip addr

//    关闭网卡
ifdown 网卡名

//    启动网卡
ifup 网卡名
```
### 5.3.13 mount

```shell
# 把 iso9660 类型的移动设备 /dev/cdrom 以只读的方式挂载到 /mnt/cdrom 目录，注意，/mnt/cdrom 目录需要提前创建好
mount -t iso9660  -o ro  /dev/cdrom   /mnt/cdrom

# 卸载设备
umount /dev/cdrom
```
### 5.3.14 crontab

```shell
# 安装 crontab
yum -y install crontabs

# 启动服务
service crond start

# 停止服务
service crond stop

# 重启服务
service crond restart

# 用指定的文件替代目前的 crontab, -u user 表示用来设定某个用户的 crontab 服务
crontab file [-u user]

# 列出用户目前的 crontab 服务
crontab -l [-u user]

# 编辑用户目前的 crontab 服务
crontab -e [-u user]

# 分时日月周、命令
# 分 : 1 - 59, 每分钟用 * 或 */1 表示
# 时 : 0 - 23
# 日 : 1 - 31
# 月 : 1 - 12
# 周 : 0 - 6, 0 表示星期日
*   *    *   *   *   command

# 每月 1、10、22 日的 4 : 45 重启 apache
45 4 1,10,22 * * service httpd restart

# 每天 18 : 00 至 23 : 00 之间每隔 30 分钟重启 apache
0,30 18-23 * * * service httpd restart

# 晚上 11 : 00 到早上 7 : 00 每隔 1 小时重启 apache
0 23-7/1 * * * service httpd restart
```
## 5.4 Linux 中文本编辑快捷键
### 5.4.1 shift + g

```shell
# 到达文本末尾
shift + g
```
### 5.4.2 o

```shell
# 跳到下一行
在命令行模式输入 o
```
### 5.4.3 date

```shell
# Y : 年份 m : 月份 d : 一个月的第几天 H : 小时 M : 分钟 S : 秒
# 2018-10-18 23:46:17
date "+%Y-%m-%d %H:%M:%S"

# 在当前日期加 1 天
date -d'+1 day' "+%Y-%m-%d %H:%M:%S"
```