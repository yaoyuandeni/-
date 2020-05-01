# 自动化运维专题（八）：Jenkins企业应用进阶详解

标签（空格分隔）： Jenkins自动化运维专题

---

[TOC]

##一，CI/CD，DevOps介绍
- [x] 持续集成（Continuous Integration，CI）： 
代码合并，构建，部署，测试都在一起，不断地执行这个过程，并对结果反馈
- [x] 持续交付（Continuous Delivery，CD）： 
部署到生产环境，给用户使用
- [x] 持续部署（Continuous Deployment,CD）: 
部署到生产环境

![image_1e75upa6f17d88ck47o8uct609.png-45.2kB][1]

##二，部署Git版远程仓库
###2.1 系统环境要求
|主机名	|IP地址|	备注|
|-|-|-|
|Git	|192.168.200.192|	Git服务器|
|Jenkins|	192.168.200.193	|Jenkins服务器|

```
[root@Git ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
[root@Git ~]# uname -r
3.10.0-862.3.3.el7.x86_64
[root@Git ~]# systemctl stop firewalld
[root@Git ~]# systemctl disable firewalld
[root@Git ~]# systemctl stop NetworkManager
[root@Git ~]# systemctl disable NetworkManager
```

###2.2 部署Git版远程仓库
```
#在Git服务器上进行如下操作
#安装Git
[root@Git ~]# yum -y install git
#创建Git账户
[root@Git ~]# useradd git
[root@Git ~]# passwd
更改用户 root 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
#创建Git远程仓库
[git@Git ~]$ mkdir repos    #创建Git仓库目录
[git@Git ~]$ cd repos
[git@Git repos]$ mkdir app.git      #创建app的项目目录
[git@Git repos]$ pwd
/home/git/repos
[git@Git repos]$ cd app.git/        
[git@Git app.git]$ pwd
/home/git/repos/app.git
[git@Git app.git]$ git --bare init  #--bare创建一个裸仓库（只用做远程推送仓库不支持本地git命令）
初始化空的 Git 版本库于 /home/git/repos/app.git/
[git@Git app.git]$ ls
branches  config  description  HEAD  hooks  info  objects  refs
说明：
使用”git init –bare”方法创建一个所谓的裸仓库，之所以叫裸仓库是因为这个仓库只保存git历史提交的版本信息，而不允许用户在上面进行各种git操作，如果你硬要操作的话，只会得到下面的错误（”This operation must be run in a work tree”）
[git@Git app.git]$ git status
fatal: This operation must be run in a work tree
```

###2.3 在Jenkins服务器进行git代码远程推送测试
```
#在Jenkins服务器上进行如下操作
#安装Git
[root@Jenkins ~]# yum -y install git
#创建一个目录，尝试git clone远程Git服务器仓库的代码
[root@Jenkins test]# git clone git@192.168.200.192:/home/git/repos/app.git
正克隆到 'app'...
git@192.168.200.192s password:     #输入远程服务器git用户的密码
warning: 您似乎克隆了一个空版本库。
[root@Jenkins test]# ls
app
[root@Jenkins test]# ls app/
#进行代码提交测试
[root@Jenkins app]# touch test  #创建一个文件
[root@Jenkins app]# echo "welcome to yunjisuan" >> test 
[root@Jenkins app]# cat test 
welcome to yunjisuan
[root@Jenkins app]# git add *   #将文件添加到本地暂存区
[root@Jenkins app]# git commit -m '测试提交'    #将代码提交到本地git仓库
*** Please tell me who you are.
Run
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
to set your accounts default identity.
Omit --global to set the identity only in this repository.
fatal: unable to auto-detect email address (got 'root@Jenkins.(none)')
#配置git全局配置
[root@Jenkins app]# git config --global user.email "215379068@qq.com"
[root@Jenkins app]# git config --global user.name "Mr.chen"
#进行代码提交测试
root@Jenkins app]# git commit -m '测试提交'
[master（根提交） 0976c56] 测试提交
 1 file changed, 1 insertion(+)
 create mode 100644 test
[root@Jenkins app]# git remote add origin git@192.168.200.192:/home/git/repos/app.git #添加远程仓库地址
fatal: 远程 origin 已经存在。
[root@Jenkins app]# git remote -v   #发现已经被添加过了（之前clone时自动添加）
origin  git@192.168.200.192:/home/git/repos/app.git (fetch)
origin  git@192.168.200.192:/home/git/repos/app.git (push)
[root@Jenkins app]# git push -u origin master   #将代码推送到远程仓库的master分支
git@192.168.200.192s password: 
Counting objects: 3, done.
Writing objects: 100% (3/3), 235 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/app.git
 * [new branch]      master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
#查看分支情况
[root@Jenkins app]# git branch -a
* master    #本地当前所处分支
  remotes/origin/master #远程仓库已有分支
```

###2.4 在Jenkins服务器进行SSH免密钥操作
```
[root@Jenkins ~]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:rriMXrCDTHugjKCLaWDc5HZJmd69g5f7qw8lfjG/Bds root@Jenkins
The keys randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|      o          |
|   . +           |
|oo= o o S. + .   |
|X+o* + o..o + +  |
|B++.o   ooo. o E |
|.+.= . o =o   o  |
|=.o +.. .o*+..   |
+----[SHA256]-----+
#进行公钥分发
[root@Jenkins ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub git@192.168.200.192
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
git@192.168.200.192s password: 
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'git@192.168.200.192'"
and check to make sure that only the key(s) you wanted were added.
#测试免密钥的git推送测试
[root@Jenkins ~]# cd /test/app/
[root@Jenkins app]# ls
test
[root@Jenkins app]# echo "testtesttest" >> test 
[root@Jenkins app]# tail -1 test
testtesttest
[root@Jenkins app]# git add *
[root@Jenkins app]# git commit -m "免密钥推送测试"
[master 86a072e] 免密钥推送测试
 1 file changed, 1 insertion(+)
[root@Jenkins app]# git push -u origin master
Counting objects: 5, done.
Writing objects: 100% (3/3), 286 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/app.git
   0976c56..86a072e  master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```

##三，Jenkins的企业应用管理
jenkins官网：https://jenkins.io/ 
redhat版jenkins官方页面：https://pkg.jenkins.io/redhat-stable/

###3.1 Jenkins的安装与基础配置
- [x] 安装Jenkins的三种方法 
利用Yum源安装
下载jenkins的rpm包安装
jenkins的war包安装
>由于在之前已经介绍过war包安装jenkins的方式了，在这里我们只用Yum来安装Jenkins 
jenkins的rpm包也已经提供给同学们了。 
yum -y localinstall jenkins-2.138.1-1.1.noarch.rpm

####3.1.1 利用Yum方式安装最新版本jenkins
```
#下载Jenkins的yum源文件
[root@Jenkins app]# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
#导入jenkins的rpm证书
[root@Jenkins app]# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
#利用Yum安装最新版本jenkins
[root@Jenkins ~]# yum -y install jenkins
#查看jenkins安装路径
[root@Jenkins ~]# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins      #jenkins配置文件
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins
```

####3.1.2 安装和配置jdk环境
>由于jenkins是java开发的所以需要jdk支持
```
#解压安装jdk
[root@Jenkins ~]# tar xf jdk-8u171-linux-x64.tar.gz -C /usr/local
[root@Jenkins ~]# cd /usr/local
[root@Jenkins local]# mv jdk1.8.0_171 jdk
[root@Jenkins local]# /usr/local/jdk/bin/java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
#配置java环境
[root@Jenkins ~]# sed -i.org '$a export JAVA_HOME=/usr/local/jdk/' /etc/profile
[root@Jenkins ~]# sed -i.org '$a export PATH=$PATH:$JAVA_HOME/bin' /etc/profile
[root@Jenkins ~]# sed -i.org '$a export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
[root@Jenkins ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/local/jdk/
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
[root@Jenkins ~]# source /etc/profile
[root@Jenkins ~]# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

####3.1.3 安装和配置maven环境
```
#解压安装maven
[root@Jenkins ~]# ll apache-maven-3.5.0-bin.tar.gz 
-rw-r--r-- 1 root root 8534562 9月  26 21:48 apache-maven-3.5.0-bin.tar.gz
[root@Jenkins ~]# tar xf apache-maven-3.5.0-bin.tar.gz -C /usr/local/
[root@Jenkins ~]# cd /usr/local/
[root@Jenkins local]# mv apache-maven-3.5.0 maven
#配置maven环境变量
[root@Jenkins ~]# sed -i '$a MAVEN_HOME=/usr/local/maven' /etc/profile
[root@Jenkins ~]# sed -i '$a export PATH=${MAVEN_HOME}/bin:$PATH' /etc/profile
[root@Jenkins ~]# tail -2 /etc/profile
MAVEN_HOME=/usr/local/maven
export PATH=${MAVEN_HOME}/bin:$PATH
[root@Jenkins ~]# source /etc/profile
[root@Jenkins ~]# mvn -v
Apache Maven 3.5.0 (ff8f5e7444045639af65f6095c62210b5713f426; 2017-04-04T03:39:06+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_171, vendor: Oracle Corporation
Java home: /usr/local/jdk/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-862.3.3.el7.x86_64", arch: "amd64", family: "unix"
```

####3.1.4 启动jenkins
```
#启动jenkins，报错
[root@Jenkins ~]# systemctl start jenkins
Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.
#查看原因
[root@Jenkins ~]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: failed (Result: exit-code) since 二 2018-09-25 23:52:59 CST; 4min 35s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 12728 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=1/FAILURE)
9月 25 23:52:59 Jenkins systemd[1]: Starting LSB: Jenkins Automation Server...
9月 25 23:52:59 Jenkins runuser[12733]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
9月 25 23:52:59 Jenkins jenkins[12728]: Starting Jenkins bash: /usr/bin/java: 没有那个文件或目录    #原来是找不到java命令
9月 25 23:52:59 Jenkins jenkins[12728]: [失败]
9月 25 23:52:59 Jenkins systemd[1]: jenkins.service: control process exited, code=exited status=1
9月 25 23:52:59 Jenkins systemd[1]: Failed to start LSB: Jenkins Automation Server.
9月 25 23:52:59 Jenkins systemd[1]: Unit jenkins.service entered failed state.
9月 25 23:52:59 Jenkins systemd[1]: jenkins.service failed.
#做一个java命令的软连接
[root@Jenkins ~]# ln -s /usr/local/jdk/bin/java /usr/bin/
#再次启动jenkins
[root@Jenkins ~]# systemctl start jenkins
[root@Jenkins ~]# systemctl status jenkins  #正常启动
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since 二 2018-09-25 23:58:32 CST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 12773 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/jenkins.service
           └─12792 /usr/bin/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/li...
9月 25 23:58:32 Jenkins systemd[1]: Starting LSB: Jenkins Automation Server...
9月 25 23:58:32 Jenkins runuser[12778]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
9月 25 23:58:32 Jenkins jenkins[12773]: Starting Jenkins [  确定  ]
9月 25 23:58:32 Jenkins systemd[1]: Started LSB: Jenkins Automation Server.
#查看jenkins监听端口8080
[root@Jenkins ~]# netstat -antup | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      12792/java          
#添加开机自启动
[root@Jenkins ~]# systemctl enable jenkins
```

![image_1e75v4gaa19i61fr3v28ism1poim.png-121.4kB][2]

####3.1.5 初始化jenkins
```
#查看jenkins解锁密码，并复制到jenkins的web界面，解锁jenkins
[root@Jenkins ~]# cat /var/lib/jenkins/secrets/initialAdminPassword
1627eb7b16c14e86b942fe9bc3ef6802
```
![image_1e75v5do117rjnmj17471345c5413.png-180.9kB][3]

>使用默认插件配置安装即可

![image_1e75v5vctmenq9jfl130v1g4c1g.png-230.9kB][4]

![image_1e75v67ttnd212lpipv1ut91ctu1t.png-65.5kB][5]

####3.1.6 常用的系统模块介绍

![image_1e75v711b10vl1mpv1mec1o361khd2a.png-127.4kB][6]

####3.1.7 系统管理--->全局工具配置介绍

![image_1e75v7mlb1noo1f9j1gdo1g8s7e12n.png-79.9kB][7]

![image_1e75v80b21uc6bq4aut16j4bn34.png-90.4kB][8]

![image_1e75v8cd51clb1g0p14f510pv1hpk3h.png-90.4kB][9]

- [x] Maven配置: 
配置maven的settings.xml配置文件的位置路径（不修改为默认路径）。
- [x] 配置JDK: 
配置java命令的执行路径；JDK是java语言的软件开发工具包。
- [x] 配置Git: 
配置Git的命令执行路径；Git是分布式版本控制软件。
- [x] 配置Gradle: 
配置Gradle的执行路径；Gradle是面向java的自动化构建开源工具，同maven。
- [x] 配置Ant: 
配置Ant的执行路径；Ant是面向java的自动化构建开源工具，同maven。
- [x] 配置Maven： 
配置maven的命令执行路径；maven是面向java的自动化构建开源工具。
- [x] 配置Docker: 
配置Docker的命令执行路径；Docker是最近大热的容器级虚拟化产品。

####3.1.8 全局工具配置----> 配置JDK

![image_1e75va69d1ho71mr03f1hgd19tj3u.png-63.7kB][10]

####3.1.9 全局工具配置----> 配置Git

![image_1e75vam8j17p41dercf1r941ipg4b.png-30.2kB][11]

####3.1.10 全局工具配置----> 配置maven
>- Apache Maven是一种创新的软件项目管理工具，提供了一个项目对象模型（POM）文件的新概念来管理项目的构建，相关性和文档。最强大的功能就是能够自动下载项目依赖的库文件。
- 在开发中，为了保证编译通过，开发会到处去寻找项目依赖的jar包（类似rpm安装软件时需要的一堆依赖包）
- 因此，就要用到Maven（Ant和gradle也是干这个的）
- Maven其实就类似Linux的Yum仓库，可以自动帮我们下载（公网源）和安装java项目所依赖的支持包。

![image_1e75vcel5cic11ntf4csabg724o.png-33.3kB][12]

###3.2 用户权限管理
>在一个成熟的企业应用环境中，jenkins是需要通过权限来控制角色功能使用的 
开发人员利用jenkins====>生产环境项目代码版本发布（A/B测试等） 
测试人员利用jenkins====>测试环境自动化部署 
运维人员利用jenkins====>生产环境项目代码版本回滚

####3.2.1 安装插件Role-based Authorization Strategy
![image_1e75vdgah1gm4nu31bau14p91ptt55.png-92.7kB][13]

![image_1e75vdopr1jcq1pupt1s16ch1gi75i.png-52.9kB][14]

####3.2.2 全局安全配置--->授权策略--->Role-Based Strategy

![image_1e75ve9ca11sj1ue0f1qd241e5v.png-90.5kB][15]

![image_1e75vehc116r7cg51n7bo5150c6c.png-89.7kB][16]

####3.2.3 注册两个用户（开发和测试）

![image_1e75vf8dk12d919931b78bsc1pdp6p.png-49.8kB][17]

![image_1e75vffjkds1d271ru6vv2hs176.png-39.3kB][18]

![image_1e75vg1hd33a13k517kg1a8k14547j.png-35.9kB][19]

>由于开启了Role-Based Strategy，此时用户没有任何权限

![image_1e75vgis7nm3jhv1sp3pnt1u0780.png-22.2kB][20]

####3.2.4 系统管理--->Manage and Assign Roles

![image_1e75vh3141brq1o4h16o1p54g8t8d.png-29kB][21]

- [x] Manage Roles:权限管理
- [x] Assign Roles:授权管理

**（1）进入权限管理**

![image_1e75viv4k5l013qi1rlmdk1hb48q.png-60.4kB][22]

- [x] Golbal roles:全局权限管理 
jenkins的整体权限分配，至少要开读的权限
- [x] Project roles:项目权限管理 
正则匹配具体的项目，分配管理权限

![image_1e75vjvjb127a1v3unhsapv1q7u97.png-67kB][23]

**（2）进入授权管理**

- [x] Global roles:全局权限授权
- [x] Item roles:项目权限授权

![image_1e75vlqc616p27ivcc0jarb5o9k.png-82.1kB][24]

**（3）创建两个项目分别以A-和B-开头**

![image_1e75vmh6d14s717ie1i4711ft1tv0a1.png-58.2kB][25]

**（4）登陆用户user1和user2进行权限登陆测试**

![image_1e75vn2b111m83o3qiu1cnistdae.png-47.7kB][26]

![image_1e75vna7b10tf1e1bjp2vq6c27ar.png-34.1kB][27]

###3.3 参数化构建
####3.3.1 什么是参数化构建？
>参数化构建就是在执行自动构建之前可以对构建过程手动传入外部参数，从而改变构建的过程。

**（1）配置一个构建脚本，然后执行**
![image_1e7760p371um018tk6m21e359pg9.png-45.2kB][28]

![image_1e776143sgr4lij8l2has1voem.png-31kB][29]

![image_1e7761csq1um7vos1kbj1mmf7nv13.png-88.2kB][30]

![image_1e7761kt1cu71ivtab21j1r1rlr1g.png-31.8kB][31]

![image_1e7761u261vup17ro1oj01tscamt1t.png-46kB][32]

![image_1e77624ulgv015dq12o612e4dn52a.png-42.3kB][33]

![image_1e7762bra2aj1kogmet12rknv2n.png-36.7kB][34]

**（2）添加参数化构建功能**

![image_1e77630uqp1215ho1tsrdir109o34.png-67.5kB][35]

![image_1e7763a4q1g98mec1m9pit0uk23h.png-59.2kB][36]

![image_1e7763im31td15us1b0r18kn17593u.png-64.8kB][37]

**（3）执行参数构建**

![image_1e77644jjtbmdta11qt19q5aah4b.png-62.1kB][38]

![image_1e7764cl919pi2i7k218lf22g4o.png-54.1kB][39]

![image_1e7764ik1l4q1js11j911adh1jna55.png-41.9kB][40]

>当然，我们在构建的时候也可以修改参数的默认值

![image_1e77654ti1irp1ne7cv911sg1bh75i.png-47.1kB][41]

![image_1e7765d49csjv6p2begst1vgd5v.png-40.8kB][42]

####3.3.2 安装插件Extended Choice Parameter
- [x] Extended Choice Parameter 
作用就是在参数化构建时可以出现一个下拉框让用户直接选择多个值
![image_1e7766i7vo6g11e5sq8ndt57c6c.png-70.4kB][43]

![image_1e7766sq71njfobn1a2jpn71lfu6p.png-62.3kB][44]

![image_1e77675b6178c35v1nq7ng3t5n76.png-44.7kB][45]

![image_1e7767cusbue33u1nqbtt81ngb7j.png-62.3kB][46]

![image_1e7767jhutq5motutju39159680.png-37.7kB][47]

![image_1e7767q101fcgg5b1ft1rno1cq88d.png-53kB][48]

![image_1e77685eq1egv16q91okhsbj1dhd8q.png-37kB][49]

>我们还可以在一个文件里创建参数，然后让插件引用这个参数

```
#创建一个参数定义文件
[root@Jenkins ~]# vim /opt/jenkins.property
[root@Jenkins ~]# cat /opt/jenkins.property
yunjisuan=test01,test02,test06
```

![image_1e77691552c38uu802llqa5q97.png-73.4kB][50]

![image_1e77696vr13k51tbune71u8bp7k9k.png-35.8kB][51]

###3.4 Git参数化构建插件
>Git Parameter插件可以直接获取Git仓库的branch，tag等信息
####3.4.1 安装插件Git Parameter

![image_1e776a3mj1sohouf1fuh1ba21ucea1.png-37.1kB][52]

####3.4.2 添加远程Git仓库的密钥管理

![image_1e776ajmn1e7u2m51ccbjlf83oae.png-50.3kB][53]

![image_1e776areq129as4onbq19b53sbar.png-30.1kB][54]

>由于我们之前用jenkins的root账户已经做过免密钥连接git了 
因此，我们创建SSH的密钥管理

![image_1e776bc52ba73m1ab31nkp11ovb8.png-68.1kB][55]

####3.4.3 进行Git参数化构建
（1）配置Git Parameter插件

![image_1e776bqva1onp1smu1522icrar4bl.png-54.6kB][56]

![image_1e776c5u0165dsj1p9aerk1iahc2.png-35.8kB][57]

（2）配置Git远程仓库

![image_1e776cihl18g13h914rg89s1m2ncf.png-47.8kB][58]

![image_1e776cqf8ld41munlvh1r081ugecs.png-46.5kB][59]

>最后保存退出

（3）进行Git参数化构建

![image_1e776dmg8sv7mle1al65lka2bd9.png-48.5kB][60]

![image_1e776dvqj79g1o6q12111gg9gdudm.png-41.5kB][61]

（4）给Git远程仓库增加分支
```
#在jenkins上操作
[root@Jenkins ~]# mkdir -p /test
[root@Jenkins ~]# cd /test
[root@Jenkins test]# git init
初始化空的 Git 版本库于 /test/.git/
[root@Jenkins test]# git clone git@192.168.200.192:/home/git/repos/app.git
正克隆到 'app'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 0 (delta 0)
接收对象中: 100% (6/6), done.
[root@Jenkins test]# ls
app
[root@Jenkins test]# cd app
[root@Jenkins app]# git branch
* master
[root@Jenkins app]# git branch dev
[root@Jenkins app]# git checkout dev
切换到分支 'dev'
[root@Jenkins app]# ls
test
[root@Jenkins app]# git add *
[root@Jenkins app]# git commit -m "开发分支提交"
# 位于分支 dev
无文件要提交，干净的工作区
[root@Jenkins app]# git push -u origin dev
Total 0 (delta 0), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/app.git
 * [new branch]      dev -> dev
分支 dev 设置为跟踪来自 origin 的远程分支 dev。
```

（5）进行jenkins项目代码拉取测试

![image_1e776fnu61ci4mofqrttm3tqre3.png-64kB][62]

![image_1e776fug81i5bc7f1nik1kvrerieg.png-37.1kB][63]

###3.5 Master-Slave架构

![image_1e776gitk1iq21bt818gqefd1njeet.png-88.2kB][64]

|主机名|	IP地址|	备注|
|-|-|-|
|Git|	192.168.200.192	|Git服务器（Jenkins的slave节点）|
|Jenkins	|192.168.200.193|	Jenkins服务器|

>jenkins的分布式构建操作，可以通过slave代理节点来执行项目任务；

####3.5.1 添加一个用于连接slave代理节点的SSH证书

![image_1e776iv2nbftoa5njjpv14ugfa.png-38.3kB][65]

####3.5.2 系统管理--->节点管理

![image_1e776jdrj19ap1dsr1ek41dd8tvdfn.png-43.7kB][66]

![image_1e776jnbs1i9o4sdavi4qt1j0cg4.png-45.5kB][67]

![image_1e776jteg7pl1tj11clgsel17otgh.png-102.8kB][68]

>**特别提示：** 
如果没有设置工具位置--->jdk 
那么jenkins会默认去/var/lib/jenkins/jdk下找java命令 
如果找不到代理就会连接不上

####3.5.3 slave节点安装java环境
```
#解压安装jdk
[root@Git ~]# tar xf jdk-8u171-linux-x64.tar.gz -C /usr/local
[root@Git ~]# cd /usr/local
[root@Git local]# mv jdk1.8.0_171 jdk
[root@Git local]# /usr/local/jdk/bin/java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
#配置java环境
[root@Git ~]# sed -i.org '$a export JAVA_HOME=/usr/local/jdk/' /etc/profile
[root@Git ~]# sed -i.org '$a export PATH=$PATH:$JAVA_HOME/bin' /etc/profile
[root@Git ~]# sed -i.org '$a export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
[root@Git ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/local/jdk/
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
[root@Git ~]# source /etc/profile
[root@Git ~]# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

####3.5.4 启动slave节点代理程序
![image_1e776lmn2c3jusunpon0i15fdgu.png-44.8kB][69]

![image_1e776m14219bm1b8n1lr61puv7c2hb.png-49kB][70]

![image_1e776md8v2041crg1f2j5blqgmho.png-32.6kB][71]

####3.5.5 查看slave节点启动日志

![image_1e776mvabanoupn1rk3du1ndgi5.png-74kB][72]

####3.5.6 强制让项目A-WebA运行在slave1节点上，并运行项目

![image_1e776ne5ggeubt7df11ee97suii.png-57.7kB][73]

![image_1e776njen1nio1slk17hi2j3bkiv.png-50.2kB][74]

![image_1e776o284nti46kjbc1k89n99jc.png-69.8kB][75]

####3.5.7 查看A-WebA的构建日志

![image_1e776opr51m491rcn1on212jo1l38js.png-54.3kB][76]

```
[root@Git ~]# cd /var/lib/jenkins/workspace/
[root@Git workspace]# ls
A-webA  A-webA@tmp
[root@Git workspace]# cat A-webA/test 
welcome to yunjisuan
testtesttest
devdevdev
[root@Git workspace]# hostname -I
192.168.200.192 
```

  [1]: http://static.zybuluo.com/yao-yuan-ge/4amf5a8dehok7yhgfmwhpfk2/image_1e75upa6f17d88ck47o8uct609.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/nmhd2rnn9idtscm27mvgyvc1/image_1e75v4gaa19i61fr3v28ism1poim.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/c5oy0at2zomt5cp8drapogm9/image_1e75v5do117rjnmj17471345c5413.png
  [4]: http://static.zybuluo.com/yao-yuan-ge/6az9kik1f1uv02ea4b6shvoi/image_1e75v5vctmenq9jfl130v1g4c1g.png
  [5]: http://static.zybuluo.com/yao-yuan-ge/oqpfwl1hvz6bq4jrzac8jk0y/image_1e75v67ttnd212lpipv1ut91ctu1t.png
  [6]: http://static.zybuluo.com/yao-yuan-ge/1t75py41vl9y1ga210om9k9u/image_1e75v711b10vl1mpv1mec1o361khd2a.png
  [7]: http://static.zybuluo.com/yao-yuan-ge/kyei02k9cskyc6q5etkguy78/image_1e75v7mlb1noo1f9j1gdo1g8s7e12n.png
  [8]: http://static.zybuluo.com/yao-yuan-ge/sqvsw7sv0v4dcn5f04d16xez/image_1e75v80b21uc6bq4aut16j4bn34.png
  [9]: http://static.zybuluo.com/yao-yuan-ge/qjp2fv7yld0i7atbf4bs1jpa/image_1e75v8cd51clb1g0p14f510pv1hpk3h.png
  [10]: http://static.zybuluo.com/yao-yuan-ge/pkpaw82gtcmq9gke1yzfbdqw/image_1e75va69d1ho71mr03f1hgd19tj3u.png
  [11]: http://static.zybuluo.com/yao-yuan-ge/pz2id456a1lc8aimqgh81ub7/image_1e75vam8j17p41dercf1r941ipg4b.png
  [12]: http://static.zybuluo.com/yao-yuan-ge/7pcq5zkxzrqw1ptsh43sgglo/image_1e75vcel5cic11ntf4csabg724o.png
  [13]: http://static.zybuluo.com/yao-yuan-ge/0solhc2vwe1vuxd27cqqagv6/image_1e75vdgah1gm4nu31bau14p91ptt55.png
  [14]: http://static.zybuluo.com/yao-yuan-ge/is2poewyfop60qe2lhtase6h/image_1e75vdopr1jcq1pupt1s16ch1gi75i.png
  [15]: http://static.zybuluo.com/yao-yuan-ge/f85m4vv2u8yqqc1hpgrrt5gr/image_1e75ve9ca11sj1ue0f1qd241e5v.png
  [16]: http://static.zybuluo.com/yao-yuan-ge/w74hk5l1l8iu7mwcpvi8e1t0/image_1e75vehc116r7cg51n7bo5150c6c.png
  [17]: http://static.zybuluo.com/yao-yuan-ge/mvzao5cumbtvd1ycf37uw7ku/image_1e75vf8dk12d919931b78bsc1pdp6p.png
  [18]: http://static.zybuluo.com/yao-yuan-ge/ampft31pkwmyb2ll56i72a33/image_1e75vffjkds1d271ru6vv2hs176.png
  [19]: http://static.zybuluo.com/yao-yuan-ge/sljfsp9gc74zxs1xkcqfmeju/image_1e75vg1hd33a13k517kg1a8k14547j.png
  [20]: http://static.zybuluo.com/yao-yuan-ge/xrs3suzh07dy15djxhwwb2m3/image_1e75vgis7nm3jhv1sp3pnt1u0780.png
  [21]: http://static.zybuluo.com/yao-yuan-ge/wdhxs42l6o5azl0ufltaein6/image_1e75vh3141brq1o4h16o1p54g8t8d.png
  [22]: http://static.zybuluo.com/yao-yuan-ge/d5pgmb6yndp20eb5u8jwh2fj/image_1e75viv4k5l013qi1rlmdk1hb48q.png
  [23]: http://static.zybuluo.com/yao-yuan-ge/e72o3s0hmrpnsh6aaaskrtko/image_1e75vjvjb127a1v3unhsapv1q7u97.png
  [24]: http://static.zybuluo.com/yao-yuan-ge/moaqwr2ecl6vxqt8wx8vf1q6/image_1e75vlqc616p27ivcc0jarb5o9k.png
  [25]: http://static.zybuluo.com/yao-yuan-ge/v4mmeywyvhci42h40mrciu8z/image_1e75vmh6d14s717ie1i4711ft1tv0a1.png
  [26]: http://static.zybuluo.com/yao-yuan-ge/5cbkzntjyhmmlt1xz5o2olg5/image_1e75vn2b111m83o3qiu1cnistdae.png
  [27]: http://static.zybuluo.com/yao-yuan-ge/eq8cn4y1zrr413caxpp9x3dv/image_1e75vna7b10tf1e1bjp2vq6c27ar.png
  [28]: http://static.zybuluo.com/yao-yuan-ge/ws2iagutbrxwgo2xa4w1i7a5/image_1e7760p371um018tk6m21e359pg9.png
  [29]: http://static.zybuluo.com/yao-yuan-ge/53191vzloyi45i150cwjs82w/image_1e776143sgr4lij8l2has1voem.png
  [30]: http://static.zybuluo.com/yao-yuan-ge/ptrf6im94fq2qkj6uqa8dktz/image_1e7761csq1um7vos1kbj1mmf7nv13.png
  [31]: http://static.zybuluo.com/yao-yuan-ge/7jnq9ut1d61yf2wrz2hrj76r/image_1e7761kt1cu71ivtab21j1r1rlr1g.png
  [32]: http://static.zybuluo.com/yao-yuan-ge/mdhl4cr06w14alnk08qyyk50/image_1e7761u261vup17ro1oj01tscamt1t.png
  [33]: http://static.zybuluo.com/yao-yuan-ge/5wczepus5lbn5wkss13avqii/image_1e77624ulgv015dq12o612e4dn52a.png
  [34]: http://static.zybuluo.com/yao-yuan-ge/5lgnzg9hr60j4l4wkwtqeobm/image_1e7762bra2aj1kogmet12rknv2n.png
  [35]: http://static.zybuluo.com/yao-yuan-ge/i0py1xkknlq88sqbcjfq7nn8/image_1e77630uqp1215ho1tsrdir109o34.png
  [36]: http://static.zybuluo.com/yao-yuan-ge/mbbl8k2wro6cv8aq3p4mmt74/image_1e7763a4q1g98mec1m9pit0uk23h.png
  [37]: http://static.zybuluo.com/yao-yuan-ge/jqe3ebfkob5j660wz178wiae/image_1e7763im31td15us1b0r18kn17593u.png
  [38]: http://static.zybuluo.com/yao-yuan-ge/2lkht765u9ef6dhbc7bhknox/image_1e77644jjtbmdta11qt19q5aah4b.png
  [39]: http://static.zybuluo.com/yao-yuan-ge/zjg43cx5d9qvdhay2n64gktk/image_1e7764cl919pi2i7k218lf22g4o.png
  [40]: http://static.zybuluo.com/yao-yuan-ge/e85qmksfztwppf0wwknepkud/image_1e7764ik1l4q1js11j911adh1jna55.png
  [41]: http://static.zybuluo.com/yao-yuan-ge/flz3f1857yh9au73uswkpvhx/image_1e77654ti1irp1ne7cv911sg1bh75i.png
  [42]: http://static.zybuluo.com/yao-yuan-ge/s2ajn14o31e8c9wf5wiisziv/image_1e7765d49csjv6p2begst1vgd5v.png
  [43]: http://static.zybuluo.com/yao-yuan-ge/dggvmehuadlf2cjugxuiqdcm/image_1e7766i7vo6g11e5sq8ndt57c6c.png
  [44]: http://static.zybuluo.com/yao-yuan-ge/ktfcvqk3qmo18rvoh9vk31a3/image_1e7766sq71njfobn1a2jpn71lfu6p.png
  [45]: http://static.zybuluo.com/yao-yuan-ge/7pvqu4lm0swfd30pfdus5qyv/image_1e77675b6178c35v1nq7ng3t5n76.png
  [46]: http://static.zybuluo.com/yao-yuan-ge/hnbi6c8q1iuizv3zfo0qmwxc/image_1e7767cusbue33u1nqbtt81ngb7j.png
  [47]: http://static.zybuluo.com/yao-yuan-ge/g1716vqomt1unx2cbb443hmk/image_1e7767jhutq5motutju39159680.png
  [48]: http://static.zybuluo.com/yao-yuan-ge/r5mo7wkrrjdbnf3v18md5szu/image_1e7767q101fcgg5b1ft1rno1cq88d.png
  [49]: http://static.zybuluo.com/yao-yuan-ge/ce6bvt9y2o74krcvidp3urz0/image_1e77685eq1egv16q91okhsbj1dhd8q.png
  [50]: http://static.zybuluo.com/yao-yuan-ge/ppp5r1cavwunw2a57r7c384v/image_1e77691552c38uu802llqa5q97.png
  [51]: http://static.zybuluo.com/yao-yuan-ge/vp2u7v2jdl2pd8smumygdqjr/image_1e77696vr13k51tbune71u8bp7k9k.png
  [52]: http://static.zybuluo.com/yao-yuan-ge/55r35q9dslrnqt00sh249taf/image_1e776a3mj1sohouf1fuh1ba21ucea1.png
  [53]: http://static.zybuluo.com/yao-yuan-ge/v4y7158hkvt9k2v1k99wkh7e/image_1e776ajmn1e7u2m51ccbjlf83oae.png
  [54]: http://static.zybuluo.com/yao-yuan-ge/t52vyqcbepp2zev9rzw5v8cf/image_1e776areq129as4onbq19b53sbar.png
  [55]: http://static.zybuluo.com/yao-yuan-ge/rhn9nw9xddjl3n7d4uvbcgys/image_1e776bc52ba73m1ab31nkp11ovb8.png
  [56]: http://static.zybuluo.com/yao-yuan-ge/xbqwpa67mgxxrm15q0uyl0i4/image_1e776bqva1onp1smu1522icrar4bl.png
  [57]: http://static.zybuluo.com/yao-yuan-ge/c2m78i3265km0eszv0dm9eos/image_1e776c5u0165dsj1p9aerk1iahc2.png
  [58]: http://static.zybuluo.com/yao-yuan-ge/4adfeq3cm5mmvqh8095ldnko/image_1e776cihl18g13h914rg89s1m2ncf.png
  [59]: http://static.zybuluo.com/yao-yuan-ge/yg5s8srw9bvon9yomjpuig4z/image_1e776cqf8ld41munlvh1r081ugecs.png
  [60]: http://static.zybuluo.com/yao-yuan-ge/tyh9va8qqjfdx973j8pulcqx/image_1e776dmg8sv7mle1al65lka2bd9.png
  [61]: http://static.zybuluo.com/yao-yuan-ge/kjfx0hldo05nf8izkm59z5hf/image_1e776dvqj79g1o6q12111gg9gdudm.png
  [62]: http://static.zybuluo.com/yao-yuan-ge/6u04107bfd886eolr7l9rle9/image_1e776fnu61ci4mofqrttm3tqre3.png
  [63]: http://static.zybuluo.com/yao-yuan-ge/l8jdufves51albans78136qa/image_1e776fug81i5bc7f1nik1kvrerieg.png
  [64]: http://static.zybuluo.com/yao-yuan-ge/3em7mbd3ahild9h9irwo6ww7/image_1e776gitk1iq21bt818gqefd1njeet.png
  [65]: http://static.zybuluo.com/yao-yuan-ge/ll7agr3ean1x5gubj5a87k40/image_1e776iv2nbftoa5njjpv14ugfa.png
  [66]: http://static.zybuluo.com/yao-yuan-ge/g00e1khtvw4sj27kdcfx8cif/image_1e776jdrj19ap1dsr1ek41dd8tvdfn.png
  [67]: http://static.zybuluo.com/yao-yuan-ge/8kpfdy3ec4xjrbeuvgfjzwy1/image_1e776jnbs1i9o4sdavi4qt1j0cg4.png
  [68]: http://static.zybuluo.com/yao-yuan-ge/dvcwjwnca28sez69yxwi6nqf/image_1e776jteg7pl1tj11clgsel17otgh.png
  [69]: http://static.zybuluo.com/yao-yuan-ge/wr9m4lpfpkkpyn28wunu7n6q/image_1e776lmn2c3jusunpon0i15fdgu.png
  [70]: http://static.zybuluo.com/yao-yuan-ge/duwlrsg5uhkbknglzbpmotvy/image_1e776m14219bm1b8n1lr61puv7c2hb.png
  [71]: http://static.zybuluo.com/yao-yuan-ge/tcdetkx7wvxvq5tmitn8k751/image_1e776md8v2041crg1f2j5blqgmho.png
  [72]: http://static.zybuluo.com/yao-yuan-ge/titrubjchvfctodwogvmneuz/image_1e776mvabanoupn1rk3du1ndgi5.png
  [73]: http://static.zybuluo.com/yao-yuan-ge/4k88e9kbpszv7b2gia56fzmi/image_1e776ne5ggeubt7df11ee97suii.png
  [74]: http://static.zybuluo.com/yao-yuan-ge/jz66xfkbg3gplb5uhlyix6sv/image_1e776njen1nio1slk17hi2j3bkiv.png
  [75]: http://static.zybuluo.com/yao-yuan-ge/7kfjamtfzse2gkp60vgfuz4m/image_1e776o284nti46kjbc1k89n99jc.png
  [76]: http://static.zybuluo.com/yao-yuan-ge/ud6iauw0ouodt392euxmf5h8/image_1e776opr51m491rcn1on212jo1l38js.png