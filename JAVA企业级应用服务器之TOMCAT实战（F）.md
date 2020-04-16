# JAVA企业级应用服务器之TOMCAT实战（F）

标签（空格分隔）： Tomcat

---Mr Wang

[TOC]

##第一章 Tomcat 简介

>**·** Tomcat是Apache软件基金会（Apache Software Foundation）的Jakarta项目中的一个核心项目，由Apache,Sun和其他一些公司及个人共同开发而成。
**·** Tomcat服务器是一个免费的开放源代码的Web应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP程序的首选。
**·** Tomcat和Nginx，Apache（httpd），lighttpd等Web服务器一样，具有处理html页面的功能，另外他还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态html的能力不如Nginx/Apache服务器。

**对比php软件，区别？**
目前Tomcat最新版本为9.0.Java容器还有resin,weblogic等。

##第二章 Tomcat安装
###2.1 软件准备
JDK下载:http://www.oracke.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
Tomcat下载:http://tomcat.apache.org/

###2.2 部署Java环境jdk
```
#首次将jdk-8u60-linux-x64.tar.gz拉进服务器
#jdk的解压安装
[root@localhost ~]# tar xf jdk-8u60-linux-x64.tar.gz -C /usr/local/
[root@localhost ~]# ln -s /usr/local/jdk1.8.0_60/ /usr/local/jdk

#配置java环境变量
[root@localhost ~]# sed -i.ori '$a export JAVA_HOME=/usr/local/jdk
\nexport PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH\nexport CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
[root@localhost ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

#让java环境变量立刻生效
[root@localhost ~]# source /etc/profile

#检查java环境安装情况
[root@localhost ~]# which java
/usr/local/jdk/bin/java
[root@localhost ~]# java -version #出现以下信息表示部署成功
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```

>**·** 关于上面那个sed命令的说明：
sed -i.ori:-i表示对文件本身操作，.ori表示修改的同时备份源文件
a:表示文件内容的最后一行，a表示在下面进行数据插入
\n:表示插入数据时换行

###2.3 安装Tomcat
```
#解压安装Tomcat
[root@localhost ~]# tar xf apache-tomcat-8.0.27.tar.gz -C /usr/local/
[root@localhost ~]# ln -s /usr/local/apache-tomcat-8.0.27/ /usr/local/tomcat

#配置Tomcat环境变量
[root@localhost ~]# echo 'export TOMCAT_HOME=/usr/local/tomcat' >>/etc/profile
[root@localhost ~]# source /etc/profile

#对jdk及Tomcat安装目录递归授权root
[root@localhost ~]# chown -R root.root /usr/local/jdk/ /usr/local/tomcat/

#检查环境变量配置情况
[root@localhost ~]# tail -4 /etc/profile
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
export TOMCAT_HOME=/usr/local/tomcat
```

####2.3.1部署java环境jdk及安装tomcat整体脚本

[脚本链接-提取码：4wyu](https://pan.baidu.com/s/1dFd_xCAcW7Yw0ERsXusTJA)

```
[root@localhost ~]# cat /server/scripts/tomcat.sh 
#!/bin/bash
#Mr Wang
#Create install tomcat script

a=/root/jdk-8u60-linux-x64.tar.gz
b=/root/apache-tomcat-8.0.27.tar.gz
[ -f "$a" ] || ( echo "没有jdk源码包" && exit )
cd && tar xf jdk-8u60-linux-x64.tar.gz -C /usr/local/
ln -s /usr/local/jdk1.8.0_60/ /usr/local/jdk
sed -i.ori '$a export JAVA_HOME=/usr/local/jdk\nexport PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH\nexport CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
source /etc/profile
which java | grep /usr/local/jdk/bin/java
if [ $? -ne 0 ];then
	echo "java环境jdk部署失败" && exit
fi
[ -f "$b" ] || ( echo "没有tomcat源码包" && exit )
cd && tar xf apache-tomcat-8.0.27.tar.gz -C /usr/local/
ln -s /usr/local/apache-tomcat-8.0.27/ /usr/local/tomcat
echo 'export TOMCAT_HOME=/usr/local/tomcat' >>/etc/profile
source /etc/profile
chown -R root.root /usr/local/jdk/ /usr/local/tomcat/
/usr/local/tomcat/bin/startup.sh

```



###2.4 Tomcat目录介绍
```
[root@localhost ~]# cd /usr/local/tomcat/
[root@localhost tomcat]# ls
bin   lib      logs    RELEASE-NOTES  temp     work
conf  LICENSE  NOTICE  RUNNING.txt    webapps
[root@localhost tomcat]# tree -L 1   #tree命令用本地yum装
.
├── bin             #用于启动，关闭Tomcat或者其他功能的脚本（.bat文件和.sh文件）
├── conf            #用于配置Tomcat的XML及DTD文件
├── lib             #存放web应用能访问的JAR包
├── LICENSE     
├── logs            #Catalina和其他Web应用程序的日志文件
├── NOTICE
├── RELEASE-NOTES
├── RUNNING.txt
├── temp            #临时文件
├── webapps         #Web应用程序根目录
└── work            #用以产生有JSP编译出的Servlet的.java和.class文件

7 directories, 4 files

[root@localhost tomcat]# cd webapps/
[root@localhost webapps]# ll
total 20
drwxr-xr-x 14 root root 4096 Feb 14 04:34 docs      #tomcat帮助文档
drwxr-xr-x  6 root root 4096 Feb 14 04:34 examples     #web应用实例
drwxr-xr-x  5 root root 4096 Feb 14 04:34 host-manager        #管理
drwxr-xr-x  5 root root 4096 Feb 14 04:34 manager     #管理
drwxr-xr-x  3 root root 4096 Feb 14 04:34 ROOT      #默认网站根目录
```

###2.5 启动Tomcat
>**启动程序:** `/usr/local/tomcat/bin/startup.sh`
**关闭程序:** `/usr/local/tomcat/bin/shutdown.sh`
```
[root@localhost webapps]# /usr/local/tomcat/bin/startup.sh #程序启动
Using CATALINA_BASE:   /usr/local/tomcat #检查环境变量CATALINA_BASE
Using CATALINA_HOME:   /usr/local/tomcat #检查环境变量CATALINA_HOME
Using CATALINA_TMPDIR: /usr/local/tomcat/temp #检查环境变量CATALINA_TMPDIR
Using JRE_HOME:        /usr/local/jdk #检查环境变量JRE_HOME
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.

[root@localhost webapps]# netstat -antup | grep java
tcp        0      0 :::8080                     :::*                        LISTEN      1112/java           
tcp        0      0 ::ffff:127.0.0.1:8005       :::*                        LISTEN      1112/java           
tcp        0      0 :::8009                     :::*                        LISTEN      1112/java           

```

###2.6 访问网站
>网址：192.168.200.160:8080(访问时请注意关闭iptables)

![](https://ae01.alicdn.com/kf/He86b9c3a0d244f65a3492937b1f42db2a.png)

```
查看Tomcat的日志
[root@localhost ~]# cd /usr/local/tomcat/logs/
[root@localhost logs]# ls
catalina.2020-02-14.log      localhost.2020-02-14.log
catalina.out                 localhost_access_log.2020-02-14.txt
host-manager.2020-02-14.log  manager.2020-02-14.log

[root@localhost logs]# cat catalina.out 
14-Feb-2020 04:47:38.968 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version:        Apache Tomcat/8.0.27
14-Feb-2020 04:47:38.970 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Sep 28 2015 08:17:25 UTC
14-Feb-2020 04:47:38.970 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server number:         8.0.27.0
14-Feb-2020 04:47:38.970 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
14-Feb-2020 04:47:38.970 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            2.6.32-431.el6.x86_64
14-Feb-2020 04:47:38.970 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
14-Feb-2020 04:47:38.971 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/local/jdk1.8.0_60/jre

```

##第三章 Tomcat的配置文件
###3.1 Tomcat配置文件
```
[root@localhost logs]# cd /usr/local/tomcat/conf
[root@localhost conf]# ll
total 216
drwxr-xr-x 3 root root   4096 Feb 14 04:47 Catalina
-rw------- 1 root root  12374 Sep 28  2015 catalina.policy
-rw------- 1 root root   7106 Sep 28  2015 catalina.properties
-rw------- 1 root root   1577 Sep 28  2015 context.xml
-rw------- 1 root root   3387 Sep 28  2015 logging.properties
-rw------- 1 root root   6458 Sep 28  2015 server.xml  #主配置文件
-rw------- 1 root root   1744 Sep 28  2015 tomcat-users.xml    #Tomcat管理用户配置文件
-rw------- 1 root root   1846 Sep 28  2015 tomcat-users.xsd
-rw------- 1 root root 167302 Sep 28  2015 web.xml

```

###3.2 Tomcat 管理
>**测试功能，生产环境不要用：**
Tomcat管理功能用于对Tomcat自身以及部署在Tomcat上的应用进行管理的Web应用。在默认情况下是处于禁用状态的。如果需要开启这个功能，就需要配置管理用户，及配置前面说过的tomcat-users.xml
```
#配置/usr/local/tomcat/conf/tomcat-users.xml
#在38行的下一行加入如下三行代码
[root@localhost ~]# tail -4 /usr/local/tomcat/conf/tomcat-users.xml
<role rolename="manager-gui"/> #加入此行
<role rolename="admin-gui"/>  #加入此行
<user username="tomcat" password="tomcat" roles="manager-gui,admin-gui"/>  #加入此行
</tomcat-users> #打开配置文件找到这一行

#重启tomcat服务
[root@localhost ~]# /usr/local/tomcat/bin/shutdown.sh 
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
[root@localhost ~]# /usr/local/tomcat/bin/startup.sh 
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
```
**在浏览器里输入http://192.168.200.160:8080/manager/status进行Tomcat管理页面**
>登录验证信息：账号：tomcat 密码：tomcat
![](https://ae01.alicdn.com/kf/Hd49aa2ad4da34afb9eedbe7c569d5312T.png)

###3.3 Tomcat 主配置文件server.xml详解
####3.3.1 server.xml组件类别
>**·** 顶级组件：位于整个配置的顶层，如server；
**·** 容器类组件：可以包含其他组件的组件，如service,engine,host,context
**·** 连接器组件：连接用户请求至tomcat，如connector
**·** 被嵌套类组件：位于一个容器当中，不能包含其他组件，如Valve,logger.
```
<server>
    <service>
    <connector />
    <engine>
    <hsot>
    <context></context>
    </host>
    <host>
    <context></context>
    </host>
    </engine>
    </service>
</server>
```
####3.3.2 组件详解
>**·** engine:核心容器组件，catalina引擎，负责通过connector接受用户请求，并处理请求，将请求转至对应的虚拟主机host。
**·** host:类似于httpd中的虚拟主机，一般而言支持基于FQDN的虚拟主机。
**·** context:定义一个应用程序，是一个最内层的容器类组件（不能再嵌套）。配置context的主要目的指定对应对的webapp的根目录，类似于httpd的alias,其还能为webapp指定额外的属性，如部署方式等。
**·** connector:接受用户请求，类似于httpd的listen配置监听端口。
**·** service（服务）:将connector关联至engine，因此一个service内部可以有多个connector，但只能有一个引擎engine。service内部有两个connector，一个engine，因此，一般情况下一个server内部只有一个service，一个service内部只有一个engine，但一个service内不可以有多个connector。
**·** server:表示一个运行于JVM的tomcat实例。
**·** Valve：阀门，拦截请求并在将其转至对应的webapp前进行某种处理操作，可以用于任何容器中，比如记录日志（access log valve），基于IP做访问控制（remote address filer valve）。
**·** logger:日志记录器，用于记录组件内部的状态信息，可以用于除context外的任何容器中。
**·** realm:可以用于任意容器类的组件中，关联一个个用户认证库，实现认证和授权，可以关联的认证库有两种:UserDatabaseRealm,MemoryRealm和JDBCRealm。
**·** UserDatabaseRealm:使用JNDI自定义的用户认证库。
**·** MemoryRealm:认证信息定义在tomcat-users.xml中。
**·** JDBCRealm:认证信息定义在数据库中，并通过JDBC连接至数据库中查找认证用户。

####3.3.3配置文件注释
```
<?xml version='1.0' encoding='utf-8'?>
<!--
<Server>元素代表整个容器，是Tomcat实例的顶层元素，由org.apache.catalina.Server接口来定义，它包含一个<Server>元素，并且他不能作为任何元素的字元素。
    port指定Tomcat监听shutdown命令端口，终止服务器运行时，必须在Tomcat服务器所在的机器上发出shutdown命令，该属性是必须的.
    shutdown指定终止Tomcat服务器时，发给Tomcat服务器的shutdown监听端口的字符串，该属性必须设置。
 -->
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.security.SecurityListener" />
  -->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <!--service服务组件-->
  <Service name="Catalina">
  <!--
  connector:接受用户请求，类似于httpd的listen配置监听端口
  port:指定服务器端要创建的端口号，并在这个端口监听来自客户端的请求。
  address:指定连接监听的地址，默认为所有地址（即0.0.0.0）
  protocol:连接器使用的协议，支持HTTP和AJP。AJP（Apache Jserv Protocol）专用于Tomcat与Apache建立通信的，在httpd反向代理用户请求至Tomcat是使用（可见nginx反向代理时不可用AJP协议）。
  minProcessors服务器启动时创建的处理请求的线程数
  maxProcessors最大可以创建的处理请求的线程数
  enableLookups如果为true，则可以通过调用request.getRemoteHost()进行DNS查询来得到远程客户端的实际主机名，若为false则不进行DNS查询，而是返回其ip地址
  redirectPort指定服务器正在处理http请求时收到了一个SSL传输请求后重定向的端口号
  acceptCount指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理
  connectionTimeout指定超时的时间数（以毫秒为单位）
    -->
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <!--engine,核心容器组件，catalina引擎，负责通过connector接受用户请求，并处理请求，将请求转至对应的虚拟主机host
        defaultHost指定缺省的处理请求的主机名，它至少与其中的一个host元素的name属性值是一样的
    -->
    <Engine name="Catalina" defaultHost="localhost">
      <!--Realm表示存放用户名，密码及role的数据库-->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <!--
      host表示一个虚拟主机
        name指定主机名
        appBase应用程序基本目录，即存放应用程序的目录，一般为appBase="webapps",相对于CATALINA_HOME而言的，也可以写绝对路径。
        uppackWARs如果为true，则tomcat会自动将WAR文件解压，否则不解压，直接从WAR文件中运行应用程序
        autoDeploy:在tomcat启动时，是否自动部署。
        xmlValidation:是否启动xml的校验功能，一般xmlValidation="false"
        xmlNamespaceAware:检测名称空间，一般xmlNamespaceAware="false"
      -->
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
      <!--
      Context表示一个web应用程序，通常为WAR文件
        docBase应用程序的路径或者是WAR文件存放的路径，也可以使用相对路径，起初路径为此Context所属Host中appBase定义的路径
        path表示此web应用程序的url的前缀这样请求的url为http://localhost:8080/path/****
        reloadable这个属性非常重要，如果为true，则tomcat会自动检测应用程序的/WEB-INF/lib和/WEB-INF/classes目录的变化，自动装载新的应用程序，可以在不重启tomcat的情况下改变应用程序
        -->
      <Context path="" docBase="" debug=""/>
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>

```

##第四章 Web站点部署
>上线的代码有两种方式，第一种方式是直接将程序目录放在webapps目录下面，这种方式都已经明白了。第二种方式就是使用开发工具将程序打包成war包，然后上传到webapps目录下面。下面我们来看一下这种方式。

###4.1 使用war包部署web站点
```
#部署war包
[root@localhost ~]# ll memtest.war  #先将此包拷贝到服务器
-rw-r--r-- 1 root root 643 Feb 14 11:20 memtest.war
[root@localhost ~]# cp memtest.war /usr/local/tomcat/webapps/
[root@localhost ~]# ls /usr/local/tomcat/webapps/
docs  examples  host-manager  manager  memtest.war  ROOT

#重启tomcat服务
[root@localhost ~]# /usr/local/tomcat/bin/shutdown.sh 
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
[root@localhost ~]# /usr/local/tomcat/bin/startup.sh 
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.

#来看war包的解压缩情况
[root@localhost ~]# ls /usr/local/tomcat/webapps/
docs  examples  host-manager  manager  memtest  memtest.war  ROOT   #此时war包已经被解压出来了
```
**用户浏览器访问：**http:192.168.200.160:8080/memtest/meminfo.jsp**如下：**
![](https://ae01.alicdn.com/kf/He2d885711d064f6c90c7bc4bb9405159t.png)

###4.2 自定义默认网站目录
上面访问的网址为：http://192.168.200.160:8080/memtest/meminfo.jsp
现在我想访问的格式为：http://192.168.200.160:8080/meminfo.jsp

**方法一：**
>将meminfo.jsp或其他程序放在tomcat/webapps/ROOT目录下即可。因为默认网站根目录为tomcat/webapps/ROOT

**方法二：**
```
[root@tomcat ~]# vim /usr/local/tomcat/conf/server.xml
    <Host name="localhost" appBase="webapps"
        unpackWARs="true" autoDeploy="true">
      <Context path="" docBase="/usr/local/tomcat/webapps/memtest" debug="0" reloadable="false" crossContext="true"/>  #在虚拟机这里添加一行代码限定web站点的根目录路径
[root@tomcat ~]# /usr/local/tomcat/bin/shutdown.sh
[root@tomcat ~]# /usr/local/tomcat/bin/startup.sh
```

##第五章 Tomcat多实例及集群架构
###5.1 Tomcat 多实例
####5.1.1 复制Tomcat目录
```
[root@localhost ~]# cd /usr/local/
[root@localhost ~]# cp -a apache-tomcat-8.0.27 tomcat8_1
[root@localhost ~]# cp -a apache-tomcat-8.0.27 tomcat8_2
```

####5.1.2 修改多实例配置文件
```
#创建多实例的网页根目录
[root@localhost ~]# mkdir -p /data/www/www/ROOT

#将网页程序拷贝到多实例目录ROOT下
[root@localhost ~]# cp /usr/local/tomcat/webapps/memtest/meminfo.jsp /data/www/www/ROOT/

#修改多实例配置文件的以下三行
[root@localhost local]# cat -n /usr/local/tomcat/conf/server.xml | sed -n '22p;69p;123p'
    22 <Server port="8005" shutdown="SHUTDOWN">   #管理端口及停止命令
    69      <Connector port="8080" protocol="HTTP/1.1"   #对外提供服务的端口
    123       <Host name="localhost" appBase="webapps"    #网站域名及网页根目录路径
```
```
#修改第一个多实例配置文件
[root@localhost local]# sed -i '22s#8005#8011#;69s#8080#8081#;123s#appBase=".*"#appBase="/data/www/www"#' /usr/local/tomcat8_1/conf/server.xml
[root@localhost local]# sed -n '22p;69p;123p' /usr/local/tomcat8_1/conf/server.xml
<Server port="8011" shutdown="SHUTDOWN">
    <Connector port="8081" protocol="HTTP/1.1"
      <Host name="localhost"  appBase="/data/www/www/"

#修改第二个多实例配置文件
[root@localhost local]# sed -i '22s#8005#8012#;69s#8080#8082#;123s#appBase=".*"#appBase="/data/www/www"#' /usr/local/tomcat8_1/conf/server.xml
[root@localhost local]# sed -n '22p;69p;123p' /usr/local/tomcat8_2/conf/server.xml
<Server port="8012" shutdown="SHUTDOWN">
    <Connector port="8082" protocol="HTTP/1.1"
      <Host name="localhost"  appBase="/data/www/www"
```

####5.1.3 启动多实例
```
#启动多实例
[root@localhost local]# /usr/local/tomcat8_1/bin/startup.sh 
Using CATALINA_BASE:   /usr/local/tomcat8_1
Using CATALINA_HOME:   /usr/local/tomcat8_1
Using CATALINA_TMPDIR: /usr/local/tomcat8_1/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat8_1/bin/bootstrap.jar:/usr/local/tomcat8_1/bin/tomcat-juli.jar
Tomcat started.
[root@localhost local]# /usr/local/tomcat8_2/bin/startup.sh 
Using CATALINA_BASE:   /usr/local/tomcat8_2
Using CATALINA_HOME:   /usr/local/tomcat8_2
Using CATALINA_TMPDIR: /usr/local/tomcat8_2/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat8_2/bin/bootstrap.jar:/usr/local/tomcat8_2/bin/tomcat-juli.jar
Tomcat started.
查看多实例进程启动情况
[root@localhost local]# netstat -antup | grep java
tcp        0      0 ::ffff:127.0.0.1:8011       :::*                        LISTEN      3247/java           
tcp        0      0 ::ffff:127.0.0.1:8012       :::*                        LISTEN      3067/java           
tcp        0      0 :::8080                     :::*                        LISTEN      6194/java           
tcp        0      0 :::8081                     :::*                        LISTEN      3247/java           
tcp        0      0 :::8082                     :::*                        LISTEN      3067/java           
tcp        0      0 ::ffff:127.0.0.1:8005       :::*                        LISTEN      6194/java           
tcp        0      0 :::8009                     :::*                        LISTEN      6194/java           
```
浏览器可以分别访问http://192.168.200.160:8081/meminfo.jsp和http://192.168.200.160:8082/meminfo.jsp

###5.2 Tomcat集群
**使用nginx+Tomcat反向代理集群**
####5.2.1 安装nginx(此处将nginx安装在了本地，也可专门用一台服务器做负载均衡)
```
>先将源码包nginx-1.10.2.tar.gz拉进虚拟机
（1）搭建本地YUM仓库，然后装依赖包
yum -y installpcre-developenssl-devel
(2)解包
tar xf nginx-1.10.2.tar.gz -C /usr/src/
(3)创建程序用户
useradd -s /sbin/nologin -M nginx
（4）预配置
cd /usr/src/nginx-1.10.2
./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
(5)编译
make
（6）安装
make install
（7）做个软连接
ln -s /usr/local/nginx/sbin/* /sbin/
```
####5.2.2修改nginx配置文件如下
```
#创建配置文件模板
[root@local ~]# cd /usr/local/nginx/conf
[root@local ~]# egrep -v "#|^$" nginx.conf.default >nginx.conf
#修改配置文件内容如下：
[root@localhost conf]# cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream web_pools {
	server 127.0.0.1:8080;
	server 127.0.0.1:8081;
}
    server {
	listen	80;
	server_name www.yunjisuan.com;
	location / {
	    root html;
	    index index.jsp index.html;
	    proxy_pass http://web_pools;
        }
    }
}
#检测语法并启动nginx
[root@localhost conf]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@localhost conf]# nginx
```
>然后自行测试负载均衡效果

###5.3 使用Tomcat安装Jpress
>Jpress,一个wordpress的java代替版本，使用JFinal开发，需要maven支持
```
[root@localhost ~]# tar xf apache-maven-3.3.9-bin.tar.gz -C /usr/local/
[root@localhost local]# ln -s apache-maven-3.3.9/ maven
[root@localhost ROOT]# tail -2 /etc/profile
export MAVEN_HOME=/usr/local/maven
export PATH="$MAVEN_HOME/bin:$PATH"
[root@localhost local]# source /etc/profile
[root@localhost local]# mvn -version #出现这个表示成功
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_60, vendor: Oracle Corporation
Java home: /usr/local/jdk1.8.0_60/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-431.el6.x86_64", arch: "amd64", family: "unix"

```
**将jpress-web-newest.war包放到tomcat网站根目录下**
```
#将war包放到网站根目录下
[root@localhost ~]# mv jpress-web-newest.war /data/www/www/ROOT/

#解压war包
[root@localhost ~]# which jar
/usr/local/jdk/bin/jar
[root@localhost ~]# cd /data/www/www/ROOT/
[root@localhost ROOT]# jar xf jpress-web-newest.war 
[root@localhost ROOT]# ls
jpress-web-newest.war  meminfo.jsp  robots.txt  templates
logs                   META-INF     static      WEB-INF
```

**用户访问：http://192.168.200.160/install进入jpress安装向导**
![](https://ae01.alicdn.com/kf/Hbc6cf1a4f3e94f2883262ff819a95dee4.png)
>**特别提示：**
**·** 虽然，Tomcat已经打开了自动解压缩war包的功能，但是细心的人会发现我并没有重启tomcat服务，因此，war包并没有被自动解压缩，故，我们需要通过jar命令进行解压缩，命令的参数和tar是一样的。
**·** 接下来我们就需要为jpress安装数据库了，之后的所有流程和阅读材料和LNMP的章节完全一样，因此，我这里就不继续操作了。

##第六章 Tomcat安全优化和性能优化
###6.1 安全优化
>最重要的优化为如下4项，但并不止这四种
**·** 降权启动
**·** telnet管理端口保护
**·** ajp连接端口保护
**·** 禁用管理端

**具体操作如下：**
（1）降权启动（同nginx优化部分的监牢模式）

>降权的原则就是利用普通用户来启动Tomcat
1）将tomcat程序目录拷贝到普通用户家目录下
2）修改家目录下程序的配置文件（启动端口，检测端口等），并重新指定网页根目录路径。
3）递归授权拷贝后的Tomcat程序的属主属组为普通用户。
4）用su命令切换为普通用户，启动Tomcat进程
5）此时Tomcat进程的权限为普通用户权限
6）如果利用/etc/rc.local文件配置普通用户程序的开机启动，那么需要利用su -c临时切换身份启动，具体可参考linux基础教案里的用户管理部分

（2）telnet管理端口保护
```
[root@localhost ~]# sed -n '22p' /usr/local/tomcat/conf/server.xml
<Server port="8005" shutdown="SHUTDOWN">   #表示通过8005端口来接受SHUTDOWN，用来停止Tomcat进程，默认的方式是非常危险的。需要进行修改。
[root@localhost ~]# netstat -antup | grep java
tcp        0      0 ::ffff:127.0.0.1:8011       :::*                        LISTEN      7157/java      #本地8011端口接收SHUTDOWN命令     
tcp        0      0 ::ffff:127.0.0.1:8012       :::*                        LISTEN      3067/java      #本地8012端口接收SHUTDOWN命令     
tcp        0      0 :::8080                     :::*                        LISTEN      6930/java           
tcp        0      0 :::8081                     :::*                        LISTEN      7157/java           
tcp        0      0 :::8082                     :::*                        LISTEN      3067/java           
tcp        0      0 ::ffff:127.0.0.1:8005       :::*                        LISTEN      6930/java     #本地8005端口接收SHUTDOWN命令     
tcp        0      0 :::8009                     :::*                        LISTEN      6930/java           
```
>Tomcat默认通过8005端口来接受SHUTDOWN这个字符串来关闭Tomcat进程，但这是非常危险的，因此需要修改端口号来防护，否则，通过telnet命令即可强行关闭Tomcat进程
```
#利用telnet来关闭Tomcat进程
[root@localhost ~]# telnet 127.0.0.1 8005   #通过telnet连接本地8005端口
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
SHUTDOWN   #发送SHUTDOWN字符串
Connection closed by foreign host.
[root@localhost ~]# netstat -antup | grep java  #可以发现8005和8080端口的Tomcat进程没有了
tcp        0      0 ::ffff:127.0.0.1:8011       :::*                        LISTEN      7157/java           
tcp        0      0 ::ffff:127.0.0.1:8012       :::*                        LISTEN      3067/java           
tcp        0      0 :::8081                     :::*                        LISTEN      7157/java           
tcp        0      0 :::8082                     :::*                        LISTEN      3067/java           

```
（3）ajp连接端口
 ```
 [root@localhost ~]# sed -n '91p' /usr/local/tomcat/conf/server.xml
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />  #这是AJP协议打开的端口，我们并不需要开启这个端口，因此注释掉本行
 ```
 (4)禁用管理端
 >Tomcat默认在安装完成后的网页目录里有很多多余的目录，删除所有不需要的目录，并清空ROOT网页默认根目录下的所有的东西，规避可能的代码漏洞
 ```
 [root@localhost ~]# cd /usr/local/tomcat/webapps/
[root@localhost webapps]# ls
docs      host-manager  memtest      ROOT
examples  manager       memtest.war   #有很多多余的东西，只留下ROOT目录，其他都删掉或者mv移走
[root@localhost webapps]# ls ROOT/   #很多多余的东西，因此清空本目录，或者都移走
asf-logo.png       bg-upper.png       tomcat.gif
asf-logo-wide.gif  build.xml          tomcat.png
bg-button.png      favicon.ico        tomcat-power.gif
bg-middle.png      index.jsp          tomcat.svg
bg-nav-item.png    RELEASE-NOTES.txt  WEB-INF
bg-nav.png         tomcat.css
 ```
 
###6.2 性能优化
####6.2.1 屏蔽DNS查询 `enableLookups="false"`
 >DNS查询非常消耗时间，如果开启会影响Tomcat性能，因此关闭。
 ```
 #默认没有，需添加配置文件如下代码段，在Connector标签位置。表示禁止DNS查询
 
 ```
 
####6.2.2 jvm调优
 >Tomcat最吃内存，只要内存足够，这只猫就跑的很快。
 如果系统资源有限，那就需要进行调优，提高资源使用率。
 ```
 #优化catalina.sh初始化脚本。在catalina.sh初始化脚本中添加以下代码：
 #catalina.sh的路径为:/usr/local/tomcat/bin/catalina.sh
 #此行优化代码需要加在脚本的最开始，声明位置，不要放在后边
 JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms1024m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=512m -XX:PermSize=512m -XX:MaxPermSize=512m"
 
 #代码说明：
 server:一定要作为第一个参数，在多个CPU时性能佳
 -Xms:初始堆内存Heap大小，使用的最小内存，cup性能高时此值应设的大一些
 -Xmx:初始堆内存heap最大值，使用的最大内存
 上面两个值是分配JVM的最小和最大内存，取决于硬件物理内存的大小，建议均设为物理内存的一半。
 -XX:PermSize:设定内存的永久保存区域
 -XX:MaxPermSize:设定最大内存的永久保存区域
 -XX:MaxNewSize:
 -Xss 15120 这使得JBoss每增加一个线程（thread）就会立即消耗15M内存，而最佳值应该是128K，默认值好像是512K，
 +XX：AggressiveHeap 会使得 Xms没有意义，这个参数让jvm忽略Xmx参数，疯狂的吃完一个G物理内存，再吃尽一个G的swap。
 -Xss：每个线程的stack大小
 -verbose:gc 显示垃圾收集信息
 -Xloggc:gc.log 指定垃圾收集日志文件
 -Xmn:young generation的heap大小，一般设置为Xmx的3、4分之一
 -XX:+UseParNewGC:缩短 minor收集的时间
 -XX:+UseConcMarKSweepGC:缩短major收集的时间
 ```
 >JVM的调优比价复杂，对于初学的人来说掌握这些就足够了，如果想要更详细的理解JVM如何调优，那么请参考网友文章：http://www.cnblogs.com/xingzc/p/5756119.html
 
 
##附录1：企业案例：linux下java/http进程高解决案例
>生产环境下某台Tomcat7服务器，在刚发布的时候一切都很正常，在运行一段时间后就出现CPU占用很高的问题，基本上是负载一天比一天高，，诸如此类问题，请排查!

**问题分析：**

>（1）程序属于CPU密集型，和开发沟通过，排除此类情况
（2）程序代码有问题，出现死循环，可能性极大

**问题解决：**
>（1）开发那边无法排查代码某个模块有问题，从日志上也无法分析得出
(2)我们可以尝试通过jstack命令来精确定位出现错误的代码段，从而拿给开发排查

（1）首次查找进程高的PID号（先找到是哪个PID号的进程导致的）
`top -H`
（2）查看这个进程所有系统调用（再找到是哪个PID号的进程导致的）
strace -p 进程的PID
(3)如果是web应用，可以继续打印该线程的堆栈信息（找出有问题的代码块）
`printf "%\n" 线程的PID`-->#将有问题的线程的PID号转换成16进制格式
`jstack 进程的PID | grep 线程PID号的十六进制格式 -A 30`#过滤出有问题的线程的堆栈信息，找出问题代码块

**实际操作演示：**
```
[root@localhost ~]# pgrep -l java
3067 java
7157 java
[root@localhost ~]# strace -p 7157
Process 7157 attached - interrupt to quit
futex(0x7f35609ca9d0, FUTEX_WAIT, 7158, NULL) = ? ERESTARTSYS (To be restarted)
--- SIGQUIT (Quit) @ 0 (0) ---
futex(0x7f355fdbe580, FUTEX_WAKE_PRIVATE, 1) = 1
rt_sigreturn(0x7f355fdbe580)            = 202
futex(0x7f35609ca9d0, FUTEX_WAIT, 7158, NULL
[root@localhost ~]# printf "%x\n" 7158
1bf6
[root@localhost ~]# jstack 7157 | grep 1bf6 -A 30
"main" #1 prio=5 os_prio=0 tid=0x00007f3558008800 nid=0x1bf6 runnable [0x00007f35609c8000]
   java.lang.Thread.State: RUNNABLE
	at java.net.PlainSocketImpl.socketAccept(Native Method)
	at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:409)
	at java.net.ServerSocket.implAccept(ServerSocket.java:545)
	at java.net.ServerSocket.accept(ServerSocket.java:513)
	at org.apache.catalina.core.StandardServer.await(StandardServer.java:446)
	at org.apache.catalina.startup.Catalina.await(Catalina.java:713)
	at org.apache.catalina.startup.Catalina.start(Catalina.java:659)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.catalina.startup.Bootstrap.start(Bootstrap.java:351)
	at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:485)

"VM Thread" os_prio=0 tid=0x00007f355806d000 nid=0x1bf7 runnable 

"VM Periodic Task Thread" os_prio=0 tid=0x00007f35580b8800 nid=0x1bfe waiting on condition 

JNI global references: 335
```

##附录2：jstack命令（Java stack Trace）
（1）**介绍**
>**·** jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项"-J-d64",windows的jstack使用方式只支持以下的这种方式：
jstack [-l] pid
**·** 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack 的信息，从而可以轻松的知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息，如果现在运行的java程序呈现hung的状态，jstack是非常有用的。
**（2）命令格式**
jstack[option]pid
jstack[option]executable core
jstack[option][server-id@]remote-hostname-or-IP
**（3）常用的参数说明**
1）options：
executable Java executable from which the core dump was produced.
(可能是产生core dump的java可执行程序)
core将被打印信息的特core dump文件
remote-hostname-or-IP 远程debug服务的主机名或iP
server-id 唯一id，假如一台主机上多个远程debug服务

2）基本参数：
-F：当jstack [-l] pid 没有响应的时候强制打印栈信息
-l：长列表，打印关于锁的附加信息，例如属于java。util.concurrent的ownable synchronizers列表
-m:打印java和native c/c++框架的所有栈xinx
-h|-help:打印帮助信息
pid:需要被打印配置信息的java进程id，可以用jps查询
