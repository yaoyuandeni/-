#企业级LNMP环境应用实践
标签（空格分隔）： LNMP

---

[TOC]

##一，LNMP应用环境
###1.1LNMP介绍
>大约在2010年以前，互联网公司最常用的经典web服务环境组合就是LAMP（即linux，Apache，MySQL，PHP），近几年随着nginx web 服务的逐渐流行，又出现了新的web服务环境组合--LNMP或LEMP，其中LNMP为linux，nginx，MySQL，PHP等首字母的缩写，而LEMP中的E则表示nginx，它取自nginx名字的发音（engine x）。现在，LNMP已经逐渐成为国内大中型互联网公司网站的主流组合环境，因此，我们必须熟练掌握LNMP环境的搭建，优化及维护方法。

###1.2 LNMP组合工作流程
>在深入学习LNMP组合之前，有必要先来了解一下LNMP环境组合的基本原理，也就是他们之间是怎么互相调度的？
在LNMP组合工作时，首先是用户通过浏览器输入域名请求nginx web服务，如果请求时静态资源，则由nginx解析返回给用户；如果是动态请求（.php结尾），那么nginx就会把它通过FastCGI接口（生产常用方法）发送给PHP引擎服务（FastCGI进程php-fpm）进行解析，如果这个动态请求要读取数据库数据，那么PHP就会继续向后请求Mysql数据库，以读取需要的数据，并最终通过nginx服务把获取的数据返回给用户，这就是LNMP环境的基本请求顺序流程。这个请求流程是企业使用LNMP环境的常用流程。
![image_1e81p4h5215h2jt8kruqgev7s9.png-281.7kB][1]

##二，LNMP之Mysql数据库
###2.1 Mysql数据库介绍
>mysql是互联网领域里非常重要的，深受广大用户欢迎的一款开源关系型数据库软件，由瑞典Mysql AB公司开发与维护。2006年，Mysql AB公司被SUN公司收购，2008年，SUN公司又被传统数据库领域大佬甲骨文（Oracle）公司收购。因此，Mysql数据库软件目前属于Oracle公司，但仍是开源的，Oracle公司收购Mysql的战略意图显而易见，其自身的Oracle数据库继续服务于传统大型企业，而利用收购的Mysql抢占互联网领域数据库份额，完成其战略布局。
Mysql是一种关系型数据库管理软件，关系型数据库的特点是将数据库存在不同的二维表中，并且将这些表放入不同的数据库中，而不是把所有的数据统一放在一个大仓库里，这样的设计增加了Mysql的读取速度，灵活性和可管理性也得到了很大提高，访问及管理Mysql数据库的最常用标准化语言为SQL结构化查询语言。

###2.2 为什么选择Mysql数据库
>目前，绝大多数使用linux操作系统的互联网企业都使用Mysql作为后端的数据库，从大型的BAT门户，到电商门户平台，分类门户平台等无一例外。那么，Mysql数据库到底有哪些优势和特点，让大家毫不犹豫的选择它呢？

**原因可能有以下几点：**
1.性能卓越，服务稳定，很少出现异常宕机。
2.开放源代码且无版权制约，自主性强，使用成本低。
3.历史悠久，社区及用户非常活跃，遇到问题，可以很快的获取到帮助。
4.软件体积小，安装使用简单，并且易于维护，安装及维护成本低。
5.支持多种操作系统，提供多种API接口，支持多种开发语言，特别是对流行的PHP语言无缝支持。
6.品牌口碑效应，使得企业无需考虑就直接用之。

###2.3 安装Mysql数据库
####2.3.1 安装预览
>Mysql有几种不同的产品线，且每种产品线又有很多不同的版本，这里选择当前企业使用最广的社区版Mysql5.5系列作为LNMP的组合环境数据库平台。
Mysql的安装方法也有很多，常见的安装方法如下表所示：

|序号|Mysql安装方式|特点说明|
|-|-|-|
|1|yum/rpm包安装|特点是简单，速度快，但是没法定制安装，入门新手常用这种方式|
|2|二进制安装|解压软件，简单配置后就可以使用，不用安装，速度较快，专业DBA喜欢这种方式。软件名如：mysql-5.5.32-linux2.6-x86_64.tar.gz|
|3|源码编译安装|特点是可以定制安装，但是安装时间长，例如：字符集安装路径，等。软件名如：mysql-5.5.32.tar.gz|
|4|源码软件结合yum/rpm安装|把源码软件制作成符合要求的rpm,放到yum仓库里，然后通过yum来安装。结合了上面1和3的优点，即安装快速，可任意定制参数，但是安装者也需要具备更深能力。本书结尾有rpm定制包的内容介绍|

**备注：**安装Mysql的注意事项如下：
（1）建议和之前介绍的Nginx服务安装在同一台机器上。
（2）重视操作过程的报错输出，有错误要解决掉再继续，不能忽略编译中的错误。

####2.3.2 安装步骤介绍
>本节采用Mysql二进制安装包进行安装演示

（1）创建mysql用户的账号
```
[root@localhost ~]# groupadd mysql
[root@localhost ~]# useradd -s /sbin/nologin -g mysql -M mysql
[root@localhost ~]# tail -1 /etc/passwd
mysql:x:501:501::/home/mysql:/sbin/nologin
[root@localhost ~]# id mysql
uid=501(mysql) gid=501(mysql) groups=501(mysql)
```
(2)获取MySQL二进制软件包

![image_1e81p7cu9tnr8k89pc1pj7urlm.png-166.6kB][2]

(3) 采用二进制方式安装MySQL
```
[root@localhost ~]# tar xf mysql-5.5.32-linux2.6-x86_64.tar.gz -C /usr/local/
[root@localhost ~]# cd /usr/local/
[root@localhost local]# mv mysql-5.5.32-linux2.6-x86_64 mysql-5.5.32
[root@localhost local]# ln -s mysql-5.5.32 mysql
[root@localhost local]# ls
bin  games    lib    libexec  mysql-5.5.32  nginx-1.10.2  share
etc  include  lib64  mysql    nginx         sbin          src
[root@localhost local]# cd /usr/local/mysql
[root@localhost mysql]# ls
bin      data  include         lib  mysql-test  scripts  sql-bench
COPYING  docs  INSTALL-BINARY  man  README      share    support-files

#提示：
二进制安装包，仅需要解压就可以了，不需要执行cmake/configure,make,make install等过程
```
- [x] :当安装LNMP一体化环境时，MySQL数据库要装在Nginx所在的机器上。如果MySQL和Nginx不在一台机器上，那么，Nginx服务器上的MySQL数据库软件包只要解压移动到/usr/local/目录，改名为mysql就可以了，不需要进行后面的初始化配置。
- [x] :在非一体的LNMP环境（Nginx和MySQL不在一台机器上），编译PHP环境时，也是需要MySQL数据库环境的，但是高版本的PHP，例如5.3版本以上，内置了PHP需要的MySQL程序，因此，对于此类版本就不需要在Nginx服务器上安装MySQL软件了，只需要在编译PHP时指定相关参数即可。这个PHP的编译参数为--with-mysql=mysqld,表示PHP程序在编译时会调用内置的MySQL的库。

（4）初始化MySQL配置文件my.cnf
**命令如下：**
```
[root@localhost ~]# cd /usr/local/mysql
[root@localhost mysql]# ls -l support-files/*.cnf
-rw-r--r--. 1 7161 wheel  4691 Jun 19  2013 support-files/my-huge.cnf
-rw-r--r--. 1 7161 wheel 19759 Jun 19  2013 support-files/my-innodb-heavy-4G.cnf
-rw-r--r--. 1 7161 wheel  4665 Jun 19  2013 support-files/my-large.cnf
-rw-r--r--. 1 7161 wheel  4676 Jun 19  2013 support-files/my-medium.cnf
-rw-r--r--. 1 7161 wheel  2840 Jun 19  2013 support-files/my-small.cnf
[root@localhost mysql]# /bin/cp support-files/my-small.cnf /etc/my.cnf
```
**提示：**
>- support-files下有my.cnf的各种配置样例。
- 使用cp全路径/bin/cp，可实现拷贝而不出现替换提示，即如果有重名文件会直接覆盖
- 本例为测试安装环境，因此选择参数配置小的my-small.cnf配置模版，如果是生产环境可以根据硬件选择更高级的配置文件，上述配置文件模版对硬件的要求从低到高依次为：

```
my-medium.cnf (最低)
my-small.cnf
my-large.cnf
my-huge.cnf
my-innodb-heavy-4G.cnf（最高）
```
（5）初始化MySQL数据库文件
**初始化命令如下：**
```
[root@localhost ~]# mkdir -p /usr/local/mysql/data #建立MySQL数据文件目录
[root@localhost ~]# chown -R mysql.mysql /usr/local/mysql #授权mysql用户管理MySQL的安装目录
[root@localhost ~]# yum -y install libaio #光盘源安装依赖包，否则下一步的编译会报错
[root@localhost ~]# /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
#初始化MySQL数据库文件，会有很多信息提示，如果没有ERROR级别的错误，会有两个OK的字样，表示初始化成功，否则就要解决初始化的问题

初始化内容如下：
Installing MySQL system tables...
OK
Filling help tables...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/local/mysql/bin/mysqladmin -u root password 'new-password'
/usr/local/mysql/bin/mysqladmin -u root -h localhost password 'new-password'

Alternatively you can run:
/usr/local/mysql/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr/local/mysql ; /usr/local/mysql/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/local/mysql/mysql-test ; perl mysql-test-run.pl

Please report any problems with the /usr/local/mysql/scripts/mysqlbug script!
```
**以上的命令主要作用是生成如下数据库文件**
```
[root@localhost ~]# tree /usr/local/mysql/data/
/usr/local/mysql/data/
├── mysql
│   ├── columns_priv.frm
│   ├── columns_priv.MYD
│   ├── columns_priv.MYI
│   ├── db.frm
│   ├── db.MYD
│   ├── db.MYI
│   ├── event.frm
│   ├── event.MYD
│   ├── event.MYI
│   ├── func.frm
│   ├── func.MYD
│   ├── func.MYI
│   ├── general_log.CSM
│   ├── general_log.CSV
│   ├── general_log.frm
│   ├── help_category.frm
│   ├── help_category.MYD
│   ├── help_category.MYI
│   ├── help_keyword.frm

...以下省略若干...
```
>这些MySQL数据文件是MySQL正确运行所必需的基本数据库文件，其功能是对MySQL权限，状态等进行管理。

####2.3.3 初始化故障排错集锦
**错误示例1:**
```
usr/local/mysql/bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared ob

#错误原因是没有libaio函数库的支持。需要
yum -y install libaio
```
**错误示例2:**
```
WARNING:The host'mysql'could not be looked up with resolveip

#需要修改主机名解析，使其和uname -n一样，修改后的结果如下：
[root@localhost ~] # grep `uname -n` /etc/hosts
```
**错误示例3:**
```
ERROR:1004Can't create file '/tmp/#sql300e_1_o.frm'(errno:13)

#原因是/tmp目录的权限有问题。
解决办法为处理/tmp目录，如下：

[root@localhost ~]# ls -ld /tmp
drwxrwxrwt. 3 root root 4096 Jul 14 07:56 /tmp
[root@localhost ~]# chmod -R 1777 /tmp/
```
此故障必须解除，否则，后面会出现登陆不了数据库等问题。

###2.4 配置并启动MySQL数据库
（1）设置MySQL启动脚本，命令如下：
```
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
#拷贝MySQL启动脚本到MySQL的命令路径
[root@localhost mysql]# chmod +x /etc/init.d/mysqld 
#使脚本可执行
```
（2）MySQL二进制默认安装路径是/usr/local/mysql，启动脚本里是/usr/local/mysql。如果安装路径不同，那么脚本里路径等都需要替换

（3）启动MySQL数据库，命令如下：
```
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL.. SUCCESS! 
```
>以上是启动数据库的规范方法之一，但还可以用如下方式启动，
/usr/local/mysql/bin/mysqld_safe --user=mysql &
这个命令结尾的“&”符号，作用是在后台执行MySQL服务，命令执行完还需要按下回车才能进入命令行状态。

（4）检查MySQL数据库是否启动，命令如下：
```
[root@localhost mysql]# netstat -antup | grep mysql
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      1702/mysqld  
```
>如果发现3306端口没起来，请tail -100 /usr/local/mysql/data/主机名.err查看日志信息，看是否有报错信息，然后根据相关错误提示进行调试。经常查看服务运行日志是个很好的习惯，也是高手的习惯。

（5）查看MySQL数据库启动结果日志，命令如下：
```
[root@localhost mysql]# tail -10 /usr/local/mysql/data/localhost.err 
InnoDB: Creating foreign key constraint system tables
InnoDB: Foreign key constraint system tables created
170714  8:33:47  InnoDB: Waiting for the background threads to start
170714  8:33:48 InnoDB: 5.5.32 started; log sequence number 0
170714  8:33:48 [Note] Server hostname (bind-address): '0.0.0.0'; port: 3306
170714  8:33:48 [Note]   - '0.0.0.0' resolves to '0.0.0.0';
170714  8:33:48 [Note] Server socket created on IP: '0.0.0.0'.
170714  8:33:49 [Note] Event Scheduler: Loaded 0 events
170714  8:33:49 [Note] /usr/local/mysql/bin/mysqld: ready for connections.
Version: '5.5.32'  socket: '/tmp/mysql.sock'  port: 3306  MySQL Community Server (GPL)
```
（6）设置MySQL开机自启动，命令如下：
```
[root@localhost mysql]# chkconfig --add mysqld
[root@localhost mysql]# chkconfig mysqld on
[root@localhost mysql]# chkconfig --list mysqld
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```
**提示**：也可以将启动命令/etc/init.d/mysqld start 放到/etc/rc.local里面
（7）配置mysql命令的全局使用路径，命令如下：
```
[root@localhost mysql]# ln -s /usr/local/mysql/bin/* /usr/local/bin/
[root@localhost mysql]# which mysqladmin
/usr/local/bin/mysqladmin
```
（8）登陆MySQL测试，命令如下：
```
[root@localhost mysql]# mysql   #直接输入命令即可登陆
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.32 MySQL Community Server (GPL)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;    #查看当前所有的数据库
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

mysql> select user();   #查看当前的登陆用户
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

mysql> quit
Bye
```
>提示：
MySQL安装完成以后，默认情况下，root账户是无密码的，这个必须要设置。

###2.5 MySQL安全配置
（1）为MySQL的root用户设置密码，命令如下：
```
[root@localhost mysql]# mysqladmin -u root password '123123' #设置密码
[root@localhost mysql]# mysql   #无法直接登陆了
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
[root@localhost mysql]# mysql -uroot -p  #新的登陆方式
Enter password:                 #输入设置的密码
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.5.32 MySQL Community Server (GPL)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```
（2）清理无用的MySQL用户及库，命令如下：
```
mysql> select user,host from mysql.user;
+------+-----------+
| user | host      |
+------+-----------+
| root | 127.0.0.1 |
| root | ::1       |
|      | localhost |
| root | localhost |
+------+-----------+
4 rows in set (0.00 sec)

mysql> drop user "root"@"::1";
Query OK, 0 rows affected (0.00 sec)

mysql> drop user ""@"localhost";
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------+-----------+
| user | host      |
+------+-----------+
| root | 127.0.0.1 |
| root | localhost |
+------+-----------+
2 rows in set (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

##三，Fastcgi介绍
###3.1 什么是CGI
> **·** CGI的全称为“通用网关接口”（Common Gateway Interface），为HTTP服务器与其他机器上的程序服务通信交流的一种工具，CGI程序需运行在网络服务器上。
**·** 传统的CGI主要缺点是性能较差，因为每次HTTP服务器遇到动态程序是都需要重新启动解析器来执行解析，之后结果才会被返回给HTTP服务器，这在处理高并发访问时几乎是不可用的，因此就诞生了FastCGI.另外，传统的CGI接口方式安全性也很差，故而现在已经很少被使用了。

###3.2 什么是FastCGI
>FastCGI是一个可伸缩的，高速的在HTTP服务器和动态脚本语言间通信的接口（在linux下，FastCGI接口即为socket，这个socket可以是文件socket，也可以是IP socket），主要优点是把动态语言和HTTP服务器分离出来。多数流行的HTTP服务器都支持FastCGI，包括Apache，nginx，和Lighttpd等。
同时，fastcgi也被许多脚本语言所支持，例如当前比较流行的脚本语言PHP。fastcgi接口采用的是C/S架构，它可以将HTTP服务器和脚本解析服务器分开，同时还能在脚本解析服务器上启动一个或多个脚本来解析守护进程。当HTTP服务器遇到动态程序时，可以将其直接交给FastCGI进程来执行，然后将得到的结果返回给浏览器，这种方式可以让HTTP服务器专一的处理静态请求，或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。

**FastCGI的重要特点如下：**

* HTTP服务器和动态脚本语言间通信的接口或工具
* 可把动态语言解析和HTTP服务器分离开
* nginx，Apache，Lighttpd，以及多数动态语言都支持fastCGI
* fastCGI接口方式采用C/S架构，分为客户端（HTTP服务器）和服务器端（动态语言解析服务器）
* PHP动态语言服务器端可以启动多个FastCGI的守护进程（例如php-fpm(fcgi process mangement)）
* HTTP服务器通过（例如nginx fastcgi_pass）FastCGI客户端和动态语言FastCGI服务器端通信（例如PHP-fpm）

###3.3 Nginx FastCGI的
>Nginx不支持对外部动态程序的直接调用或者解析，所有的外部程序（包括PHP）必须通过FastCGI接口来调用。FastCGI接口在linux下是socket，为了调用CGI程序，还需要一个FastCGI的wrapper（可以理解为用于启动另一个程序的程序），这个wrapper绑定在某个固定的socket上，如端口或文件socket。当nginx将CGI请求发送给这个socket的时候，通过FastCGI接口，wrapper接受到请求，然后派生一个新的线程，这个线程调用解释器或外部程序处理脚本来读取返回的数据；接着，wrapper再将返回的数据通过FastCGI接口，沿着固定的socket传递给Nginx，最后，Nginx将返回的数据发送给客户端，这就是Nginx+FastCGI的整个运作过程。

![image_1e81ppc439vu3fmjllkst1s8s13.png-461.2kB][3]

FastCGI的主要优点是把动态语言和HTTP服务器分离开来，使Nginx专门处理静态请求及向后转发的动态请求，而PHP/PHP-FPM服务器则专门解析php动态请求。

###3.4 LNMP之PHP（FastCGI方式）服务的安装和准备
####3.4.1 检查Nginx及MySQL的安装情况
（1）检查确认Nginx及MySQL的安装路径，命令如下：
```
[root@localhost ~]# ls -ld /usr/local/nginx
lrwxrwxrwx. 1 root root 24 Jul  9 14:31 /usr/local/nginx -> /usr/local/nginx-1.10.2/
[root@localhost ~]# ls -ld /usr/local/mysql
lrwxrwxrwx. 1 mysql mysql 12 Jul 14 07:13 /usr/local/mysql -> mysql-5.5.32
```
（2）检查端口及启动情况，命令如下：
```
[root@localhost ~]# netstat -antup | grep -E "80|3306"
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      1193/nginx          
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      1702/mysqld 
```
（3）测试访问Nginx及MySQL是否OK，命令如下：
```
[root@localhost ~]# wget 127.0.0.1    #测试Nginx
--2017-07-14 09:54:12--  http://127.0.0.1/
Connecting to 127.0.0.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 624 [text/html]
Saving to: “index.html”

100%[=========================================================================================>] 624         --.-K/s   in 0s      

2017-07-14 09:54:12 (2.12 MB/s) - “index.html” saved [624/624]

[root@localhost ~]# mysql -uroot -p    #测试MySQL
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.5.32 MySQL Community Server (GPL)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> quit
Bye

```
**如果访问结果和上述一致，就表明Nginx及MySQL的安装一切正常**

####3.4.2 检查安装PHP所需的lib库
PHP程序在开发及运行时会调用一些诸如zlib，gd等函数库，因此需要确认lib库是否已经安装，执行过程如下：
```
[root@localhost ~]# rpm -qa zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel
zlib-devel-1.2.3-29.el6.x86_64
[root@localhost ~]# rpm -qa freetype-devel libpng-devel gd libcurl-devel libxslt-devel
```
**提示：**

- [x] :每个lib一般都会存在对应的以“*-devel”命名的包，安装lib对应的-devel包后，对应的lib包就会自动安装好，例如安装gd-devel时就会安装gd。
- [x] ：这些lib库不是必须安装的，但是目前的企业环境下一般都需要安装。否则，PHP程序运行时会出现问题，例如验证码无法显示等。

**执行下面命令安装相关的lib软件包**
```
[root@localhost ~]# yum -y install zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel
[root@localhost ~]# yum -y install freetype-devel libpng-devel gd libcurl-devel libxslt-devel
```

**安装后的结果如下：**
```
[root@localhost ~]# rpm -qa zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel
zlib-devel-1.2.3-29.el6.x86_64
libxml2-devel-2.7.6-14.el6.x86_64
libjpeg-turbo-devel-1.2.1-1.el6.x86_64
#这里仅缺少libiconv-devel包
[root@localhost ~]# rpm -qa freetype-devel libpng-devel gd libcurl-devel libxslt-devel
freetype-devel-2.3.11-14.el6_3.1.x86_64
libpng-devel-1.2.49-1.el6_2.x86_64
libcurl-devel-7.19.7-37.el6_4.x86_64
libxslt-devel-1.1.26-2.el6_3.1.x86_64
gd-2.0.35-11.el6.x86_64
```
>从以上结果看出，仅有libiconv-devel这个包没有安装，因为默认的yum源没有此包，后面会编译安装。

####3.4.3 安装yum无法安装的libiconv库
```
[root@localhost ~]# wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
[root@localhost ~]# ls
anaconda-ks.cfg  install.log         libiconv-1.14.tar.gz                 nginx-1.10.2.tar.gz
index.html       install.log.syslog  mysql-5.5.32-linux2.6-x86_64.tar.gz
[root@localhost ~]# tar xf libiconv-1.14.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/libiconv-1.14/
[root@localhost libiconv-1.14]# ./configure --prefix=/usr/local/libiconv && make && make install
```
####3.4.4 安装libmcrypt库
```
推荐使用简单的在线yum的方式安装：wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
编译安装过程略
[root@localhost yum.repos.d]# yum -y install libmcrypt-devel
```
####3.4.5 安装mhash加密扩展库
```
推荐使用简单的在线yum的方式安装：wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
编译安装过程略
[root@localhost yum.repos.d]# yum -y install mhash
```
####3.4.6 安装mcrvpt加密扩展库
```
推荐使用简单的在线yum的方式安装：wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
编译安装过程略
[root@localhost yum.repos.d]# yum -y install mcrypt
```
###3.5 开始安装PHP（FastCGI方式）服务
####3.5.1 获取PHP软件包
```
[root@localhost ~]# wget http://cn2.php.net/get/php-5.3.28.tar.gz/from/this/mirror
```
####3.5.2 解压配置PHP
```
[root@localhost ~]# tar xf php-5.3.28.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/php-5.3.28/
[root@localhost php-5.3.28]# ./configure \
> --prefix=/usr/local/php5.3.28 \
> --with-mysql=/usr/local/mysql \
> --with-iconv-dir=/usr/local/libiconv \
> --with-freetype-dir \
> --with-jpeg-dir \
> --with-png-dir \
> --with-zlib \
> --with-libxml-dir=/usr \
> --enable-xml \
> --disable-rpath \
> --enable-safe-mode \
> --enable-bcmath \
> --enable-shmop \
> --enable-sysvsem \
> --enable-inline-optimization \
> --with-curl \
> --with-curlwrappers \
> --enable-mbregex \
> --enable-fpm \
> --enable-mbstring \
> --with-mcrypt \
> --with-gd \
> --enable-gd-native-ttf \
> --with-openssl \
> --with-mhash \
> --enable-pcntl \
> --enable-sockets \
> --with-xmlrpc \
> --enable-zip \
> --enable-soap \
> --enable-short-tags \
> --enable-zend-multibyte \
> --enable-static \
> --with-xsl \
> --with-fpm-user=www \
> --with-fpm-group=www \
> --enable-ftp

#特别强调：上述每行结尾的换行符反斜线（\）之后不能再有任何字符包括空格
```
[预配置代码文件-百度网盘提取码：vxk4](https://pan.baidu.com/s/11jqEvTYUt1w-Y7GuZc4UbA)

**执行上述命令后，最后的正确输出提示为下图**

![image_1e821c8fj1sjho6ftd41hc7c6c1g.png-77.4kB][4]

**对于上面命令，部分参数说明如下：**

- [x] :--prefix=/usr/local/php5.2.28
表示指定PHP的安装路径为/usr/local/php5.3.28
- [x] :--with-mysql=/usr/local/mysql
表示需要指定MySQL的安装路径，安装PHP需要的MySQL相关内容。当然，如果没有MySQL软件包，也可以不单独安装，这样的情况可使用--with-mysql=mysqlnd替代--with-mysql=/usr/local/mysql，因为PHP软件里已经自带了连接MySQL的客户端工具。
- [x] :--with-fpm-user=www
nginx表示指定PHP-FPM进程管理的用户为www，此处最好和Nginx服务用户统一
- [x] : --with-fpm-group=www
表示指定PHP-FPM进程管理的组为www，此处最好与Nginx服务用户组统一。
- [x] :--enable-fpm
表示激活PHP-FPM方式服务，即以FastCGIF方式运行PHP服务。

####3.5.3 编译PHP
正确执行前文配置PHP软件的./configure系列命令后，就可以编译PHP软件了，具体操作过程如下：
```
[root@localhost php-5.3.28]# ln -s /usr/local/mysql/lib/libmysqlclient.so.18
libmysqlclient.so.18      libmysqlclient.so.18.0.0  
[root@localhost php-5.3.28]# ln -s /usr/local/mysql/lib/libmysqlclient.so.18 /usr/lib64/
[root@localhost php-5.3.28]# touch ext/phar/phar.phar
[root@localhost php-5.3.28]# make

#make最后的正确提示
Build complete.
Don't forget to run 'make test'.
```

####3.5.4 安装PHP生成文件到系统
```
[root@localhost php-5.3.28]# make install
```

####3.5.5 配置PHP引擎配置文件php.ini
（1）设置软链接以方便访问，命令如下：
```
[root@localhost ~]# ln -s /usr/local/php5.3.28/ /usr/local/php
[root@localhost ~]# ls -l /usr/local/php
lrwxrwxrwx. 1 root root 21 Jul 14 13:06 /usr/local/php -> /usr/local/php5.3.28/
```
（2）查看PHP配置默认模版文件，命令如下：
```
[root@localhost ~]# cd /usr/src/php-5.3.28/
[root@localhost php-5.3.28]# ls php.ini*
php.ini-development  php.ini-production
```
>请注意以上两文件的异同之处，可通过diff或vimdiff命令比较，如下图所示：
![image_1e821hssh18jfqpe1jjj12qk3n41t.png-809kB][5]

**从对比结果可以看出，开发环境更多的是开启日志，调试信息，而生产环境都是关闭状态**

（3）拷贝PHP配置文件到PHP默认目录，并更改文件名称为php.ini，命令如下：
```
[root@localhost php-5.3.28]# cp php.ini-production /usr/local/php/lib/php.ini
[root@localhost php-5.3.28]# ls -l /usr/local/php/lib/php.ini 
-rw-r--r--. 1 root root 69627 Jul 14 13:25 /usr/local/php/lib/php.ini
```

####3.5.6 配置PHP（FastCGI方式）的配置文件php-fpm.conf
```
[root@localhost php-5.3.28]# cp php.ini-production /usr/local/php/lib/php.ini
[root@localhost php-5.3.28]# ls -l /usr/local/php/lib/php.ini 
-rw-r--r--. 1 root root 69627 Jul 14 13:25 /usr/local/php/lib/php.ini
[root@localhost php-5.3.28]# cd /usr/local/php/etc/
[root@localhost etc]# ls
pear.conf  php-fpm.conf.default
[root@localhost etc]# cp php-fpm.conf.default php-fpm.conf
```
>关于php-fpm.conf，暂时可用默认的配置，先把服务搭好，以后再进行优化。

####3.5.7 启动PHP服务（FastCGI方式）
（1）启动PHP服务php-fpm，命令如下：
```
[root@localhost etc]# /usr/local/php/sbin/php-fpm
```

（2）检查PHP服务php-fpm的进程及启动端口的情况，命令如下：
```
[root@localhost etc]# ps -ef | grep php-fpm
root     126611      1  0 13:36 ?        00:00:00 php-fpm: master process (/usr/local/php5.3.28/etc/php-fpm.conf)
nginx    126612 126611  0 13:36 ?        00:00:00 php-fpm: pool www          
nginx    126613 126611  0 13:36 ?        00:00:00 php-fpm: pool www          
root     126619 126548  0 13:39 pts/1    00:00:00 grep php-fpm
[root@localhost etc]# lsof -i:9000  #默认9000端口提供服务
COMMAND    PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
php-fpm 126611  root    7u  IPv4 136041      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 126612 nginx    0u  IPv4 136041      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 126613 nginx    0u  IPv4 136041      0t0  TCP localhost:cslistener (LISTEN)
```

###3.6 配置Nginx支持PHP程序请求访问
####3.6.1 修改Nginx配置文件
（1）查看nginx当前的配置，命令如下：
```
[root@localhost etc]# cd /usr/local/nginx/conf/
[root@localhost conf]# cp nginx.conf nginx.conf.02
[root@localhost conf]# cat nginx.conf
worker_processes  1;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include extra/www.conf;
    include extra/mail.conf;
    include extra/status.conf;
    include extra/blog.conf;
    
}
```
（2）PHP解析，这里以blog为例讲解，内容如下：
```
[root@localhost conf]# cat extra/blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.html index.htm;
        }
    }

```
**最终blog虚拟机的完整配置如下：**
```
[root@localhost conf]# cat extra/blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.html index.htm;
        }
		location ~ .*\.(php|php5)?$ {
			root	/var/www/html/blogcom;
			fastcgi_pass	127.0.0.1:9000;
			fastcgi_index	index.php;
			include		fastcgi.conf;
		}
    }

```
####3.6.2 检查并启动Nginx
可通过如下命令检查Nginx配置文件的语法：
```
[root@localhost conf]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx-1.10.2//conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx-1.10.2//conf/nginx.conf test is successful
[root@localhost conf]# /usr/local/nginx/sbin/nginx -s reload
```
**此步在生产环境很关键，如不提前检查语法，重启后发现语法错误会导致Nginx无法提供服务，，给用户访问体验带来不好的影响。**

####3.6.3 测试LNMP环境生效情况
（1）测试PHP解析请求是否OK
1）进入指定的默认站点目录后，编辑index.php,添加如下内容：
```
[root@localhost conf]# cd /var/www/html/blogcom/
[root@localhost blogcom]# echo "<?php phpinfo(); ?>" >test_info.php
[root@localhost blogcom]# cat test_info.php 
<?php phpinfo(); ?>
```
**以上代码为显示PHP配置信息的简单PHP文件代码**
>注意：
对于初学者来说，以上内容最好手工录入而不要拷贝，否则可能会导致意外结果。

2）调整Windows下的host解析（192.168.0.121为当前的机器IP），命令如下：
```
192.168.0.121 www.yunjisuan.com mail.yunjisuan.com yunjisuan.com blog.yunjisuan.com
```
3）打开浏览器，输入http://blog.yunjisuan.com/test_info.php 即可打开如下图所示界面：

![image_1e821qu5k1lc14u4k7l1j5l1k3o2a.png-174.3kB][6]

**出现上述界面，表示Nginx配合PHP解析已经正常。**


（2）针对Nginx请求访问PHP，然后对PHP连接MySQL的情况进行测试
编辑test_mysql.php,加入如下内容：
```
[root@localhost blogcom]# cat test_mysql.php 
<?php
	//$link_id=mysql_connect('主机名','用户','密码');
	$link_id=mysql_connect('localhost','root','123123');
	if($link_id){
		echo "mysql successful by Mr.chen !";
	}else{
		echo mysql_error();
	}
?>
```
**测试结果如下：**

![image_1e821stnn1n671j61h0pk8j6n52n.png-101.8kB][7]

**至此，LNMP的组合已基本搭建完毕。当然，我们还没有做相关优化，因此，我们需要将虚拟机保存好。留待以后之用**

##四， 部署一个blog程序服务
###4.1 开源博客程序WordPress介绍
>WordPress 是一套利用PHP语言和MySQL数据库开发的开源免费的blog（博客，网站）程序，用户可以在支持PHP环境和MySQL数据库的服务器上建立blog站点。它的功能非常强大，拥有众多插件，易于扩充功能。其安装和使用也都非常方便。目前WordPress已经成为搭建blog平台的主流，很多发布平台都是根据WordPress二次开发的，如果你也想像他们一样拥有自己的blog，可购买网上的域名及空间，然后搭建LNMP环境，部署WordPress程序后就可以轻松成就自己的梦想了。

**注意：**
>WordPress是单用户个人博客，与blog.51cto.com的多用户博客是有区别的。

###4.2 WordPress 博客程序的搭建准备
**（1）MySQL数据库配置准备**
1）登陆MySQL数据库，操作如下：
```
[root@localhost ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.5.32 MySQL Community Server (GPL)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```
2）创建一个专用的数据库WordPress，用于存放blog数据，操作如下：
```
mysql> create database wordpress;   #创建一个数据库，名字为wordpress
Query OK, 1 row affected (0.00 sec)

mysql> show databases like 'wordpress';  #查看
+----------------------+
| Database (wordpress) |
+----------------------+
| wordpress            |
+----------------------+
1 row in set (0.00 sec)

mysql> 
```
3）创建一个专用的WordPress blog管理用户，命令如下：
```
mysql> grant all on wordpress.* to wordpress@'localhost' identified by '123123';                    #localhost为客户端地址
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;            #刷新权限，使得创建用户生效
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for wordpress@'localhost';   #查看用户对应权限
+------------------------------------------------------------------------------------------------------------------+
| Grants for wordpress@localhost                                                                                   |
+------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'wordpress'@'localhost' IDENTIFIED BY PASSWORD '*E56A114692FE0DE073F9A1DD68A00EEB9703F3F1' |
| GRANT ALL PRIVILEGES ON `wordpress`.* TO 'wordpress'@'localhost'                                                 |
+------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> select user,host from mysql.user;        #查看数据库里创建的wordpress用户
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| root      | 127.0.0.1 |
| root      | localhost |
| wordpress | localhost |   #只允许本机通过wordpress用户访问数据库
+-----------+-----------+
3 rows in set (0.00 sec)

mysql> quit
Bye

```
**（2）Nginx及PHP环境配置准备**
1）选择之前配置好的支持LNMP的blog域名对应的虚拟主机，命令如下：
```
[root@localhost extra]# cat blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.php index.html index.htm;  #补充一个首页文件index.php
        }
		location ~ .*\.(php|php5)?$ {
			root	/var/www/html/blogcom;
			fastcgi_pass	127.0.0.1:9000;
			fastcgi_index	index.php;
			include		fastcgi.conf;
		}
    }

[root@localhost extra]# /usr/local/nginx/sbin/nginx -s reload
```
2）获取WordPress博客程序，并放置到blog域名对应虚拟主机的站点目录下，即/var/www/html/blogcom,操作命令如下：
```
[root@localhost blogcom]# ls   #浏览www.wordpress.org下载博客程序
index.html  test_info.php  test_mysql.php  wordpress-4.7.4-zh_CN.tar.gz
[root@localhost blogcom]# tar xf wordpress-4.7.4-zh_CN.tar.gz #解压
[root@localhost blogcom]# ls
index.html  test_info.php  test_mysql.php  wordpress  wordpress-4.7.4-zh_CN.tar.gz
[root@localhost blogcom]# rm -f index.html test_info.php  test_mysql.php #删除无用文件
[root@localhost blogcom]# ls
wordpress  wordpress-4.7.4-zh_CN.tar.gz
[root@localhost blogcom]# mv wordpress/* .  #把目录里的内容移动到blogcom根目录下
[root@localhost blogcom]# /bin/mv wordpress-4.7.4-zh_CN.tar.gz /root/ #移走源程序
[root@localhost blogcom]# ls -l  #完整的blog程序内容
total 192
-rw-r--r--.  1 nobody 65534   418 Sep 24  2013 index.php
-rw-r--r--.  1 nobody 65534 19935 Jan  2  2017 license.txt
-rw-r--r--.  1 nobody 65534  6956 Apr 23 09:24 readme.html
drwxr-xr-x.  2 nobody 65534  4096 Jul 14 16:04 wordpress
-rw-r--r--.  1 nobody 65534  5447 Sep 27  2016 wp-activate.php
drwxr-xr-x.  9 nobody 65534  4096 Apr 23 09:24 wp-admin
-rw-r--r--.  1 nobody 65534   364 Dec 19  2015 wp-blog-header.php
-rw-r--r--.  1 nobody 65534  1627 Aug 29  2016 wp-comments-post.php
-rw-r--r--.  1 nobody 65534  2930 Apr 23 09:24 wp-config-sample.php
drwxr-xr-x.  5 nobody 65534  4096 Apr 23 09:24 wp-content
-rw-r--r--.  1 nobody 65534  3286 May 24  2015 wp-cron.php
drwxr-xr-x. 18 nobody 65534 12288 Apr 23 09:24 wp-includes
-rw-r--r--.  1 nobody 65534  2422 Nov 20  2016 wp-links-opml.php
-rw-r--r--.  1 nobody 65534  3301 Oct 24  2016 wp-load.php
-rw-r--r--.  1 nobody 65534 33939 Nov 20  2016 wp-login.php
-rw-r--r--.  1 nobody 65534  8048 Jan 11  2017 wp-mail.php
-rw-r--r--.  1 nobody 65534 16255 Apr  6 14:23 wp-settings.php
-rw-r--r--.  1 nobody 65534 29896 Oct 19  2016 wp-signup.php
-rw-r--r--.  1 nobody 65534  4513 Oct 14  2016 wp-trackback.php
-rw-r--r--.  1 nobody 65534  3065 Aug 31  2016 xmlrpc.php
root@localhost blogcom]# chown -R www.www ../blogcom/ #授权用户访问
[root@localhost blogcom]# ls -l  #最终博客目录和权限
total 192
-rw-r--r--.  1 www www   418 Sep 24  2013 index.php
-rw-r--r--.  1 www www 19935 Jan  2  2017 license.txt
-rw-r--r--.  1 www www  6956 Apr 23 09:24 readme.html
drwxr-xr-x.  2 www www  4096 Jul 14 16:04 wordpress
-rw-r--r--.  1 www www  5447 Sep 27  2016 wp-activate.php
drwxr-xr-x.  9 www www  4096 Apr 23 09:24 wp-admin
-rw-r--r--.  1 www www   364 Dec 19  2015 wp-blog-header.php
-rw-r--r--.  1 www www  1627 Aug 29  2016 wp-comments-post.php
-rw-r--r--.  1 www www  2930 Apr 23 09:24 wp-config-sample.php
drwxr-xr-x.  5 www www  4096 Apr 23 09:24 wp-content
-rw-r--r--.  1 www www  3286 May 24  2015 wp-cron.php
drwxr-xr-x. 18 www www 12288 Apr 23 09:24 wp-includes
-rw-r--r--.  1 www www  2422 Nov 20  2016 wp-links-opml.php
-rw-r--r--.  1 www www  3301 Oct 24  2016 wp-load.php
-rw-r--r--.  1 www www 33939 Nov 20  2016 wp-login.php
-rw-r--r--.  1 www www  8048 Jan 11  2017 wp-mail.php
-rw-r--r--.  1 www www 16255 Apr  6 14:23 wp-settings.php
-rw-r--r--.  1 www www 29896 Oct 19  2016 wp-signup.php
-rw-r--r--.  1 www www  4513 Oct 14  2016 wp-trackback.php
-rw-r--r--.  1 www www  3065 Aug 31  2016 xmlrpc.php
```

###4.3 开始安装blog博客程序
很多开源程序都支持浏览器傻瓜式的界面安装，此处也用这种方法。
1）打开浏览器输入blog.yunjisuan.com（提前做好hosts或DNS解析），回车后，出现下图：

![image_1e8223q14ht7v731jq01bur1aot34.png-108.7kB][8]

2）仔细阅读页面的文字信息后，单击“现在就开始”按钮继续，然后在出现的页面表单上填写相应的内容，如下图所示：

![image_1e8224fak15sk1gb11hr0pn01sc741.png-129.6kB][9]

3）在页面表单里填好内容后，单击结尾的“提交”按钮继续，得到下图：

![image_1e8224vkvuotg0t172lv181el04e.png-41.8kB][10]

4）出现上图就表示可以安装了，单击“进行安装”按钮继续，进入下图：

![image_1e8225lsq5lsrb519d71tepr9u4r.png-161.8kB][11]

5）根据界面提示设置blog站点的信息后，单击“安装WordPress”按钮继续。
出现下图所示的信息就表明已经成功安装了WordPress博客。

![image_1e8226bjfkm7rpf1dnn1395f5a58.png-46kB][12]

###4.4 博客的简单使用
（1）后台登录，如下图：

![image_1e82276blv781a0g1vta1itm14k95l.png-47.4kB][13]

![image_1e8227jvj1hkdksk1nl59ii83t62.png-476.3kB][14]

###4.5 实现WordPress博客程序URL静态化
>实现此功能时，首先要在WordPress后台依次单击设置--->固定链接--->自定义结构，然后输入下面的代码，并保存更改。

```
/archives/%post_id%.html

#说明：%post_id%是数据库对应博文内容的唯一ID，例如423
```

![image_1e822913lf1t1vso1eigd5hfag6f.png-400.7kB][15]

**接着，在Nginx配置文件的server容器中添加下面的代码：**
```
[root@localhost extra]# cat blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
	root	/var/www/html/blogcom;
        location / {
            	index  index.php index.html index.htm;
		if (-f $request_filename/index.html){
			rewrite (.*) $1/index.html break;
		}
		if (-f $request_filename/index.php){
			rewrite (.*) $1/index.php;
		}
		if (!-f $request_filename){
			rewrite (.*) /index.php;
		}
        }
	location ~ .*\.(php|php5)?$ {
		fastcgi_pass	127.0.0.1:9000;
		fastcgi_index	index.php;
		include		fastcgi.conf;
	}
    }
```
**最后检查语法并重新加载Nginx服务，操作如下：**
```
[root@localhost extra]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx-1.10.2//conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx-1.10.2//conf/nginx.conf test is successful
[root@localhost extra]# /usr/local/nginx/sbin/nginx -s reload
```

**现在可以通过浏览器访问了，如下图所示：**

![image_1e822bq5a190q60n1k8720g4326s.png-860kB][16]

##五， 本章重点回顾
1，LNMP的组合中各组件工作调度逻辑关系。
2，Nginx与PHP通过FastCGI模式通信的原理。
3，LNMP环境的企业级搭建。
4，WordPress博客程序的安装搭建与URL静态化



  [1]: http://static.zybuluo.com/yao-yuan-ge/x00511d6tc3q2rzctovcmqsk/image_1e81p4h5215h2jt8kruqgev7s9.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/qcvmben80pf9ofl2hu5y6o9s/image_1e81p7cu9tnr8k89pc1pj7urlm.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/uw75hkdr19atd7vlcgv1ns86/image_1e81ppc439vu3fmjllkst1s8s13.png
  [4]: http://static.zybuluo.com/yao-yuan-ge/gvi8sr8xfmj7b1qqief6pwf3/image_1e821c8fj1sjho6ftd41hc7c6c1g.png
  [5]: http://static.zybuluo.com/yao-yuan-ge/7ns2fm4n6h3wn6rg888b3vcw/image_1e821hssh18jfqpe1jjj12qk3n41t.png
  [6]: http://static.zybuluo.com/yao-yuan-ge/b38k0cwrjggq7d04yyydfbec/image_1e821qu5k1lc14u4k7l1j5l1k3o2a.png
  [7]: http://static.zybuluo.com/yao-yuan-ge/fvel3ct7oe073dsr60bths6h/image_1e821stnn1n671j61h0pk8j6n52n.png
  [8]: http://static.zybuluo.com/yao-yuan-ge/vnw1wp0w4alplps2t8vt0v3h/image_1e8223q14ht7v731jq01bur1aot34.png
  [9]: http://static.zybuluo.com/yao-yuan-ge/iehq5vg4vp39vruhndtnozl2/image_1e8224fak15sk1gb11hr0pn01sc741.png
  [10]: http://static.zybuluo.com/yao-yuan-ge/pc5qw6txlq111ajmljd1c5gk/image_1e8224vkvuotg0t172lv181el04e.png
  [11]: http://static.zybuluo.com/yao-yuan-ge/j5p6viezqstwew33fw80bvp3/image_1e8225lsq5lsrb519d71tepr9u4r.png
  [12]: http://static.zybuluo.com/yao-yuan-ge/3kxmg7q3ztt2m8hp6eauo58k/image_1e8226bjfkm7rpf1dnn1395f5a58.png
  [13]: http://static.zybuluo.com/yao-yuan-ge/w6it9hsasnn51k8tld3vj4a5/image_1e82276blv781a0g1vta1itm14k95l.png
  [14]: http://static.zybuluo.com/yao-yuan-ge/i01mjosu5uz5tpy8wyryzrjb/image_1e8227jvj1hkdksk1nl59ii83t62.png
  [15]: http://static.zybuluo.com/yao-yuan-ge/jjezxho5o50h119e9uzhlmkm/image_1e822913lf1t1vso1eigd5hfag6f.png
  [16]: http://static.zybuluo.com/yao-yuan-ge/jmuyklqfh7bnbkwbserppzzl/image_1e822bq5a190q60n1k8720g4326s.png