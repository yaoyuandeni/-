# 自动化运维专题（九）：项目案例之Pipeline流水线及分布式流水线发布PHP项目

标签（空格分隔）： Jenkins自动化运维专题

---

[TOC]

##四，Jenkins的Pipeline流水线
|主机名|IP地址|备注|
|-|-|-|
|Git|192.168.200.192|Git服务器|
|Jenkins|192.168.200.193|Jenkins服务器|

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

###4.1 Pipeline流水线介绍
百度查

###4.2 创建一个基于Pipeline流水线的项目
![image_1e736skmsrqk1rg1s61eno1bpq9.png-129.7kB][1]

###4.3 添加项目Git参数化构建
![image_1e736thhggr81fi6foc1mfe40am.png-40.1kB][2]

![image_1e736tq2i1la67t6vje1dj71b8a13.png-24.1kB][3]

###4.4 Pipeline脚本语法架构介绍
```
#Pipeline脚本语法架构
node （'slave节点名'） {          #被操控的节点服务器
    def 变量    #def可以进行变量声明
    stage（'阶段名A'）{     #流水线阶段一
        执行步骤A
        执行步骤B
        执行步骤C
    }
    stage（'阶段名B'）{     #流水线阶段二
        执行步骤A
        执行步骤B
        执行步骤C
    }
    stage（'阶段名C'）{     #流水线阶段三
        执行步骤A
        执行步骤B
        执行步骤C
    }
}
```

![image_1e736uspj16001bj51vd4in01cg71g.png-64.3kB][4]

```
#流水线模板脚本
node {
   def mvnHome
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/jglick/simple-maven-project-with-tests.git'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'M3'
   }
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
   }
}
```

###4.5 利用Pipeline Syntax，编写Pipeline Script并构建
（1）进入Pipeline Syntax

![image_1e736vvkpfep1s5228d1cu9vq91t.png-43.7kB][5]

（2）通过脚本代码生成器，生成Pipeline脚本代码

![image_1e7370so6182h2sunfu8631hgj2a.png-109.7kB][6]

![image_1e73715robm8j1g1qbv8s7rhv2n.png-135.5kB][7]

（3）将生成的代码复制到流水线脚本相应步骤的stage函数里

![image_1e7371lt3sjt1jst12b3kt81vs34.png-62.9kB][8]

（4）开始构建Pipeline项目

![image_1e73725fm1kta1c76prt1lj11g163h.png-61.6kB][9]

![image_1e7372bm81rng1caa1sflr3qens3u.png-89.7kB][10]

###4.6 从远程仓库下载Pipeline Script，并构建
（1）在Git服务器上创建一个存放Pipeline脚本的仓库
```
[root@Git ~]# su - git
上一次登录：六 10月  6 23:17:51 CST 2018pts/0 上
[git@Git ~]$ cd /home/git/repos/
[git@Git repos]$ pwd
/home/git/repos
[git@Git repos]$ ls
app.git
[git@Git repos]$ mkdir jenkinsfile  #创建存放Pipeline脚本的仓库
[git@Git repos]$ cd jenkinsfile/
[git@Git jenkinsfile]$ git --bare init  #初始化仓库
初始化空的 Git 版本库于 /home/git/repos/jenkinsfile/
```
（2）在jenkins服务器上，往远程仓库提交一个Pipeline脚本。
```
[root@Jenkins ~]# mkdir /test
[root@Jenkins ~]# cd /test
[root@Jenkins test]# git clone git@192.168.200.192:/home/git/repos/jenkinsfile
正克隆到 'jenkinsfile'...
warning: 您似乎克隆了一个空版本库。
[root@Jenkins test]# ls
jenkinsfile
[root@Jenkins test]# cd jenkinsfile/
[root@Jenkins jenkinsfile]# mkdir itemA
[root@Jenkins jenkinsfile]# vim itemA/jenkinsfile
[root@Jenkins jenkinsfile]# cat itemA/jenkinsfile
node {
   //def mvnHome
   stage('checkout') { // for display purposes
        checkout([$class: 'GitSCM', branches: [[name: '${branch}']], 
        doGenerateSubmoduleConfigurations: false, extensions: [], 
        submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'd0721eb3-07e1-49f2-bb30-2fae94220fad', 
        url: 'git@192.168.200.192:/home/git/repos/app.git']]])
   }
   stage('maven Build') {
        echo "maven build ..."
   }
   stage('deploy') {
       echo "deploy..."
   }
   stage('test') {
        echo "test..."
   }
}

#将脚本推送到远程仓库的master分支
[root@Jenkins jenkinsfile]# git add *
[root@Jenkins jenkinsfile]# git commit -m "第一次提交"
[master（根提交） 52cbf43] 第一次提交
 1 file changed, 18 insertions(+)
 create mode 100644 itemA/jenkinsfile
[root@Jenkins jenkinsfile]# git push -u origin master
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 573 bytes | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/jenkinsfile
 * [new branch]      master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```

（3）利用远程仓库里的Pipeline脚本，进行流水线的构建

![image_1e73750cg11j117bifj21hbc1cvu4b.png-75.8kB][11]

![image_1e7375f7n14cp1cla1bs71jcn1k9b4o.png-57.6kB][12]

![image_1e7375pgdecs11fs12h1118932c55.png-85.3kB][13]

##五，项目案例一：Jenkins+Pipeline+Git+PHP博客项目流水线自动发布
|主机名|IP地址|备注|
|-|-|-|
|Git|192.168.200.192|Git服务器|
|Jenkins|192.168.200.193|Jenkins服务器|
|Web|192.168.200.194|Web服务器|

```
#所有服务器进行如下操作
[root@Git ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
[root@Git ~]# uname -r
3.10.0-862.3.3.el7.x86_64
[root@Git ~]# systemctl stop firewalld
[root@Git ~]# systemctl disable firewalld
[root@Git ~]# systemctl stop NetworkManager
[root@Git ~]# systemctl disable NetworkManager
```

![image_1e737a7675e31n8h1rtn13f01fl05i.png-141.8kB][14]

###5.1 创建一个Pipeline流水线项目并进行参数化构建

![image_1e737ausodt41ocqof81fu1het5v.png-107.4kB][15]

![image_1e737b727k14jqq4961m1972l6c.png-81.3kB][16]

>由于我们仍旧打算将pipeline脚本放在远程Git仓库里，因此我们需要从远程Git仓库拉取Pipeline脚本，所以，参数化构建不支持Git的参数化。我们只能使用字符结构的参数化构建。

![image_1e737bnjh1lk11h6aj1t19ipat56p.png-68.3kB][17]

###5.2 下载用于自动化发布的PHP源码wordpress源码包，并上传远程git仓库
```
#在Git服务器上创建用于存放源码的Git仓库
[root@Git ~]# hostname -I
192.168.200.192 
[root@Git ~]# cd /home/git/repos/
[root@Git repos]# ls
app.git  jenkinsfile
[root@Git repos]# mkdir wordpress
[root@Git repos]# cd wordpress/
[root@Git wordpress]# git --bare init
初始化空的 Git 版本库于 /home/git/repos/wordpress/
[root@Git wordpress]# cd ..
[root@Git repos]# ll
总用量 0
drwxrwxr-x 7 git  git  119 9月  25 22:19 app.git
drwxrwxr-x 7 git  git  119 10月  6 23:18 jenkinsfile
drwxr-xr-x 7 root root 119 10月 14 19:38 wordpress
[root@Git repos]# chown -R git.git wordpress

#在jenkins服务器上，克隆创建好的远程Git仓库
[root@Jenkins ~]# mkdir /php-app
[root@Jenkins ~]# cd /php-app
[root@Jenkins php-app]# git clone git@192.168.200.192:/home/git/repos/wordpress
正克隆到 'wordpress'...
warning: 您似乎克隆了一个空版本库。
[root@Jenkins php-app]# ls
wordpress
[root@Jenkins php-app]# cd wordpress/
[root@Jenkins wordpress]# ls

#下载wordpres源代码
[root@Jenkins wordpress]# wget https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz
--2018-10-14 19:41:40--  https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz
正在解析主机 cn.wordpress.org (cn.wordpress.org)... 198.143.164.252
正在连接 cn.wordpress.org (cn.wordpress.org)|198.143.164.252|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：9082696 (8.7M) [application/octet-stream]
正在保存至: “wordpress-4.9.4-zh_CN.tar.gz”

100%[======================================================>] 9,082,696   2.20MB/s 用时 3.9s   

2018-10-14 19:41:45 (2.20 MB/s) - 已保存 “wordpress-4.9.4-zh_CN.tar.gz” [9082696/9082696])

[root@Jenkins wordpress]# ls
wordpress-4.9.4-zh_CN.tar.gz
[root@Jenkins wordpress]# tar xf wordpress-4.9.4-zh_CN.tar.gz 
[root@Jenkins wordpress]# ls
wordpress  wordpress-4.9.4-zh_CN.tar.gz
[root@Jenkins wordpress]# mv wordpress-4.9.4-zh_CN.tar.gz /tmp/
[root@Jenkins wordpress]# ls
wordpress
[root@Jenkins wordpress]# mv wordpress/* .
[root@Jenkins wordpress]# ls
index.php    wp-activate.php       wp-config-sample.php  wp-links-opml.php  wp-settings.php
license.txt  wp-admin              wp-content            wp-load.php        wp-signup.php
readme.html  wp-blog-header.php    wp-cron.php           wp-login.php       wp-trackback.php
wordpress    wp-comments-post.php  wp-includes           wp-mail.php        xmlrpc.php
[root@Jenkins wordpress]# rm -rf wordpress/



#在jenkins上提交代码到远程Git仓库
[root@Jenkins wordpress]# git add *
[root@Jenkins wordpress]# git commit -m "第一次提交"
[root@Jenkins wordpress]# git push -u origin master
Counting objects: 1661, done.
Compressing objects: 100% (1635/1635), done.
Writing objects: 100% (1661/1661), 8.86 MiB | 11.34 MiB/s, done.
Total 1661 (delta 172), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/wordpress
 * [new branch]      master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```

###5.3 设置分布式构建的slave管理节点
>我们计划利用分布式构建的方式，启动pipeline的流水线项目发布 
slave管理节点就设置为需要用于发布项目的Web服务器

####5.3.1 添加用于连接slave代理节点的SSH证书
![image_1e737dn6d1t5mcaq4vvnp5iuj76.png-41.5kB][18]

####5.3.2 添加并设置slave管理从节点

![image_1e737ee3v172p20f1erl1kna5kb7j.png-46.9kB][19]

![image_1e737enrm1rts17co1jth5enpc580.png-124.2kB][20]

![image_1e737f00s2j2mrv3bd1b01u4d8d.png-42.3kB][21]

####5.3.3 slave从节点安装java环境，并启动jenkins的slave管理节点
```
#解压安装jdk
[root@Web ~]# tar xf jdk-8u171-linux-x64.tar.gz -C /usr/local
[root@Web ~]# cd /usr/local
[root@Web local]# mv jdk1.8.0_171 jdk
[root@Web local]# /usr/local/jdk/bin/java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

#配置java环境
[root@Web ~]# sed -i.org '$a export JAVA_HOME=/usr/local/jdk/' /etc/profile
[root@Web ~]# sed -i.org '$a export PATH=$PATH:$JAVA_HOME/bin' /etc/profile
[root@Web ~]# sed -i.org '$a export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
[root@Web ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/local/jdk/
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
[root@Web ~]# source /etc/profile
[root@Web ~]# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

![image_1e737g0367k0cd5uq9grk1le28q.png-48.5kB][22]

###5.4 Web服务器安装LNMP环境，并手动拉取代码模拟访问
```
[root@Web ~]# yum -y install epel-release
[root@Web ~]# yum -y install nginx php-fpm php-mysql
[root@Web ~]# cd /etc/nginx/
[root@Web nginx]# ls
conf.d                fastcgi_params          mime.types          scgi_params           win-utf
default.d             fastcgi_params.default  mime.types.default  scgi_params.default
fastcgi.conf          koi-utf                 nginx.conf          uwsgi_params
fastcgi.conf.default  koi-win                 nginx.conf.default  uwsgi_params.default
[root@Web nginx]# cp nginx.conf{,.bak}
[root@Web nginx]# egrep -v "#|^$" nginx.conf.bak > nginx.conf
[root@Web nginx]# cat nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;           #include了一个配置文件目录
    server {
        listen       80 default_server;     #默认的server配置（如果用IP访问就进入这个server）
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;     #默认的网页目录
        include /etc/nginx/default.d/*.conf;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}

#由于默认的配置文件include了/etc/nginx/conf.d/*.conf因此我们增加一个server配置文件即可
[root@Web nginx]# cat /etc/nginx/conf.d/wp.conf 
server {
    listen 80;
    server_name www.yunjisuan.com;
    root    /usr/share/nginx/html/www.yunjisuan.com;
    location / {
        index index.php index.html;
    }
    location ~\.php {
        fastcgi_index index.php;
        fastcgi_pass 127.0.0.1:9000;
        include fastcgi.conf;
    }
}


#创建网页目录
[root@Web ~]# cd /usr/share/nginx/html/
[root@Web html]# ls
404.html  50x.html  index.html  nginx-logo.png  poweredby.png
[root@Web html]# mkdir www.yunjisuan.com
[root@Web html]# cd www.yunjisuan.com
[root@Web www.yunjisuan.com]# ls

#克隆Git仓库代码到本地网页目录
[root@Web www.yunjisuan.com]# yum -y install git
[root@Web www.yunjisuan.com]# git clone git@192.168.200.192:/home/git/repos/wordpress
正克隆到 'wordpress'...
The authenticity of host '192.168.200.192 (192.168.200.192)' cant be established.
ECDSA key fingerprint is SHA256:DbY5ZLFytaIrrM0hUUSYj12DHprd/boGy3Kim6rMrJA.
ECDSA key fingerprint is MD5:59:39:e3:1a:6e:f8:66:4e:0d:de:08:80:cc:89:f4:20.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.200.192' (ECDSA) to the list of known hosts.
git@192.168.200.192's password: 
Permission denied, please try again.
git@192.168.200.192's password: 
remote: Counting objects: 1663, done.
remote: Compressing objects: 100% (1465/1465), done.
remote: Total 1663 (delta 172), reused 1661 (delta 172)
接收对象中: 100% (1663/1663), 8.87 MiB | 0 bytes/s, done.
处理 delta 中: 100% (172/172), done.
[root@Web www.yunjisuan.com]# ls
wordpress
[root@Web www.yunjisuan.com]# mv wordpress/* .
mv：是否覆盖"./wordpress"？ y
mv: 无法将"wordpress/wordpress" 移动至"./wordpress": 目录非空
[root@Web www.yunjisuan.com]# ls
index.php    wp-activate.php       wp-config-sample.php  wp-links-opml.php  wp-settings.php
license.txt  wp-admin              wp-content            wp-load.php        wp-signup.php
readme.html  wp-blog-header.php    wp-cron.php           wp-login.php       wp-trackback.php
wordpress    wp-comments-post.php  wp-includes           wp-mail.php        xmlrpc.php
[root@Web www.yunjisuan.com]# rm -rf wordpress/

#将网页目录权限授权给apache程序用户
[root@Web www.yunjisuan.com]# cd ..
[root@Web html]# ls
404.html  50x.html  index.html  nginx-logo.png  poweredby.png  www.yunjisuan.com
[root@Web ~]# id apache
uid=48(apache) gid=48(apache) 组=48(apache)
[root@Web ~]# chown -R apache.apache /usr/share/nginx/html/www.yunjisuan.com

#启动nginx服务和php-fpm服务
[root@Web ~]# systemctl start nginx
[root@Web ~]# systemctl start php-fpm
[root@Web ~]# ss -antup | egrep "80|9000"
tcp    LISTEN     0      128    127.0.0.1:9000                  *:*                   users:(("php-fpm",pid=16444,fd=0),("php-fpm",pid=16443,fd=0),("php-fpm",pid=16442,fd=0),("php-fpm",pid=16441,fd=0),("php-fpm",pid=16440,fd=0),("php-fpm",pid=16439,fd=6))
tcp    LISTEN     0      128       *:80                    *:*                   users:(("nginx",pid=16432,fd=6),("nginx",pid=16431,fd=6))
tcp    LISTEN     0      50        *:8081                  *:*                   users:(("java",pid=12313,fd=735))
tcp    ESTAB      0      52     192.168.200.194:22                 192.168.200.1:50480               users:(("sshd",pid=16336,fd=3))
tcp    LISTEN     0      128      :::80                   :::*                   users:(("nginx",pid=16432,fd=7),("nginx",pid=16431,fd=7))
```

**做好宿主机的域名映射后，浏览器访问测试**

![image_1e737hu7ds1j1cajk8tpo1k9f97.png-74.9kB][23]

###5.5 在远程Git仓库中创建一个用于构建的Pipeline脚本
```
#在jenkins服务器上进行如下操作
[root@Jenkins ~]# rm -rf /test
[root@Jenkins ~]# mkdir /test
[root@Jenkins ~]# cd /test
[root@Jenkins test]# git clone git@192.168.200.192:/home/git/repos/jenkinsfile
正克隆到 'jenkinsfile'...
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 0), reused 0 (delta 0)
接收对象中: 100% (4/4), done.
[root@Jenkins test]# ls
jenkinsfile
[root@Jenkins test]# cd jenkinsfile/
[root@Jenkins jenkinsfile]# ls
itemA
[root@Jenkins jenkinsfile]# ls itemA/
jenkinsfile

#通过流水线脚本生成器生成如下脚本内容
[root@Jenkins jenkinsfile]# vim itemA/jenkinsfile-php-wp
[root@Jenkins jenkinsfile]# cat itemA/jenkinsfile-php-wp
node ("PHP-slave1-192.168.200.194") {
   stage('git checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '${branch}']], 
        doGenerateSubmoduleConfigurations: false, extensions: [], 
        submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'd0721eb3-07e1-49f2-bb30-2fae94220fad', 
        url: 'git@192.168.200.192:/home/git/repos/wordpress']]])
   }
   stage('code copy') {
        sh '''rm -rf ${WORKSPACE}/.git
		[ -d /data/backup ] || mkdir -p /data/backup
		mv /usr/share/nginx/html/www.yunjisuan.com /data/backup/www.yunjisuan.com-$(date +%F_%T)
        cp -rf ${WORKSPACE} /usr/share/nginx/html/www.yunjisuan.com'''
   }
   stage('test') {
        sh 'curl http://www.yunjisuan.com/status.html'
   }
}

#推送到Git远程仓库
[root@Jenkins jenkinsfile]# git add *
[root@Jenkins jenkinsfile]# git commit -m "xxxx"
[master 4c9ff24] xxxx
 1 file changed, 17 insertions(+)
 create mode 100644 itemA/jenkinsfile-php-wp
[root@Jenkins jenkinsfile]# git push -u origin master
Counting objects: 6, done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 713 bytes | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/jenkinsfile
   52cbf43..4c9ff24  master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```

**通过流水线脚本生成器生成的阶段代码示例：**
![image_1e737j6md1ukbch81ggt1alf1b869k.png-77.5kB][24]

![image_1e737ji8bg3ejba15tjv54141ka1.png-112.3kB][25]

![image_1e737jta2ic31fqa10g4s651f6pae.png-89.7kB][26]

###5.6 在PHP项目代码里增加Pipeline验证用的测试页面
```
#在项目代码里加入一个健康检查测试页面，并推送到远程Git仓库
#在Web服务器上进行如下操作
[root@Web test]# git clone git@192.168.200.192:/home/git/repos/wordpress
正克隆到 'wordpress'...
git@192.168.200.192s password: 
remote: Counting objects: 1663, done.
remote: Compressing objects: 100% (1465/1465), done.
remote: Total 1663 (delta 172), reused 1661 (delta 172)
接收对象中: 100% (1663/1663), 8.87 MiB | 0 bytes/s, done.
处理 delta 中: 100% (172/172), done.
[root@Web test]# ls
wordpress
[root@Web test]# cd wordpress/
[root@Web wordpress]# ls
index.php    wordpress        wp-blog-header.php    wp-content   wp-links-opml.php  wp-mail.php      wp-trackback.php
license.txt  wp-activate.php  wp-comments-post.php  wp-cron.php  wp-load.php        wp-settings.php  xmlrpc.php
readme.html  wp-admin         wp-config-sample.php  wp-includes  wp-login.php       wp-signup.php
[root@Web wordpress]# echo "OK-version V2.0" >> status.html

#将测试用页面提交到远程Git仓库
[root@Web wordpress]# git add *
[root@Web wordpress]# git commit -m "version V2.0"
[master a0248d6] version V2.0
 1 file changed, 1 insertion(+)
 create mode 100644 status.html
[root@Web wordpress]# git push -u origin master
git@192.168.200.192s password: 
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 288 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To git@192.168.200.192:/home/git/repos/wordpress
   9396b08..a0248d6  master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。

#在web服务器做域名映射（因为要进行curl验证）
[root@Web wordpress]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.200.194 www.yunjisuan.com
```

###5.7 浏览器访问jenkins进行PHP项目流水线发布构建

![image_1e737l4pusdr105p1671oaqsfrar.png-59.4kB][27]

![image_1e737leml1h3h1a4q2l41nq81qiab8.png-119.1kB][28]


  [1]: http://static.zybuluo.com/yao-yuan-ge/4pv31ssa3gogp4ee8bok471q/image_1e736skmsrqk1rg1s61eno1bpq9.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/o6asdxb7p2ycoe6abrth2769/image_1e736thhggr81fi6foc1mfe40am.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/fyz7edunt3nq549huvgabwfg/image_1e736tq2i1la67t6vje1dj71b8a13.png
  [4]: http://static.zybuluo.com/yao-yuan-ge/n9k9eqhfrdoersp3svry3w0y/image_1e736uspj16001bj51vd4in01cg71g.png
  [5]: http://static.zybuluo.com/yao-yuan-ge/7an1tjsz3rtcnz0q8n9uoomi/image_1e736vvkpfep1s5228d1cu9vq91t.png
  [6]: http://static.zybuluo.com/yao-yuan-ge/w8zm4q6uo09hvd2r1iu2p9gb/image_1e7370so6182h2sunfu8631hgj2a.png
  [7]: http://static.zybuluo.com/yao-yuan-ge/bdlw8rnxc6iqp4v4fllewkqx/image_1e73715robm8j1g1qbv8s7rhv2n.png
  [8]: http://static.zybuluo.com/yao-yuan-ge/7gewsph0v2db2j40ptsiz10o/image_1e7371lt3sjt1jst12b3kt81vs34.png
  [9]: http://static.zybuluo.com/yao-yuan-ge/sxv6v9rwhyg3ro62301vzd79/image_1e73725fm1kta1c76prt1lj11g163h.png
  [10]: http://static.zybuluo.com/yao-yuan-ge/suu6uuml7yb0znh02zdl304u/image_1e7372bm81rng1caa1sflr3qens3u.png
  [11]: http://static.zybuluo.com/yao-yuan-ge/fr108w671kavrw1d8zxau5xs/image_1e73750cg11j117bifj21hbc1cvu4b.png
  [12]: http://static.zybuluo.com/yao-yuan-ge/gvl48r45nk797ig6b06cuwry/image_1e7375f7n14cp1cla1bs71jcn1k9b4o.png
  [13]: http://static.zybuluo.com/yao-yuan-ge/d673x8o0sclbnk3237uv4q0c/image_1e7375pgdecs11fs12h1118932c55.png
  [14]: http://static.zybuluo.com/yao-yuan-ge/cgv2xcmsxaq3jzidrxznkw6h/image_1e737a7675e31n8h1rtn13f01fl05i.png
  [15]: http://static.zybuluo.com/yao-yuan-ge/pp7wmjuny1al2spipvnqrdey/image_1e737ausodt41ocqof81fu1het5v.png
  [16]: http://static.zybuluo.com/yao-yuan-ge/7oii2n4j8ci2m5yiif2sk0tn/image_1e737b727k14jqq4961m1972l6c.png
  [17]: http://static.zybuluo.com/yao-yuan-ge/lfmqss0tx9knrst5gww440su/image_1e737bnjh1lk11h6aj1t19ipat56p.png
  [18]: http://static.zybuluo.com/yao-yuan-ge/0y9v1vqmgsv0n2ew5f0icc6v/image_1e737dn6d1t5mcaq4vvnp5iuj76.png
  [19]: http://static.zybuluo.com/yao-yuan-ge/px9nv6nlacfjkoyiturzfnvy/image_1e737ee3v172p20f1erl1kna5kb7j.png
  [20]: http://static.zybuluo.com/yao-yuan-ge/40ljmivwdx51xw0gv8cmi0ne/image_1e737enrm1rts17co1jth5enpc580.png
  [21]: http://static.zybuluo.com/yao-yuan-ge/9jbo8g6cetxrhoweynijdr1v/image_1e737f00s2j2mrv3bd1b01u4d8d.png
  [22]: http://static.zybuluo.com/yao-yuan-ge/elwlyfw5czsq0m90dzfs5a35/image_1e737g0367k0cd5uq9grk1le28q.png
  [23]: http://static.zybuluo.com/yao-yuan-ge/ojx1p1014mqenxyvgkbatqjh/image_1e737hu7ds1j1cajk8tpo1k9f97.png
  [24]: http://static.zybuluo.com/yao-yuan-ge/e2wrplamhlq8mbfqbqe3rbx0/image_1e737j6md1ukbch81ggt1alf1b869k.png
  [25]: http://static.zybuluo.com/yao-yuan-ge/x6b540lk7hd2u3xngqzu2y2o/image_1e737ji8bg3ejba15tjv54141ka1.png
  [26]: http://static.zybuluo.com/yao-yuan-ge/pfhr0mpxz7q7z0zmh2918xog/image_1e737jta2ic31fqa10g4s651f6pae.png
  [27]: http://static.zybuluo.com/yao-yuan-ge/ze2b2fvitn7jr938z0hi3h80/image_1e737l4pusdr105p1671oaqsfrar.png
  [28]: http://static.zybuluo.com/yao-yuan-ge/dzi59h274nwglvdfkmgtyf45/image_1e737leml1h3h1a4q2l41nq81qiab8.png