# 企业实战之监控升级可视化zabbix4.0 系列 （四）

标签（空格分隔）： zabbix4.0系列

---

[TOC]

https://www.zybuluo.com/commandersu/note/1670449

##添加agent客户端，linux系统及windows系统
##一、实验环境
**zabbix-server端**
|操作系统|主机名|ip|内存|
|-|-|-|
|centos7.5|zabbix|192.168.200.173|1G|
**zabbix-agent端**
|操作系统|被监控的主机名|ip地址|运行的项目|内存|
|-|-|-|
|centos6.5|www_001|192.168.200.128|nginx|1G|
|centos7.5|www_002|192.168.200.131|nginx|1G|
|windows10|suluo|192.168.200.149|--|2G|

##二、添加监控项的方法
>添加监控项目有多种方式

* simple check，被监控的服务器无需安装客户端，如ping，端口检测之类的
* zabbix agent，被动式监控服务器
* zabbix agent(active)，主动式监控服务器
* snmp check，使用snmp协议去获取监控信息
* zabbix trapper，主动式监控
* External check，zabbix server上可编写监控脚本
* Jmx agent，监控java进程

###2.1下载安装agent客户端
>agent客户端 
客户端监控、简单监控比较 
能获取到更多的监控信息，例如cpu、内存等 
zabbix客户端内置了很多key，方便我们监控基本硬件信息 
zabbix客户端能够自定义监控，方便我们监控部署的应用
```
#我们在www_001这台主机上安装agent端
[root@www_001 ~]# yum install -y gcc gcc-c++ make pcre-devel
[root@www_001 ~]# useradd -M -s /sbin/nologin zabbix
#[root@www_001 ~]# wget https://nchc.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/4.0.3/zabbix-4.0.3.tar.gz --no-check-certificate #网上下载安装包，或者直接拉进去
[root@www_001 ~]# ls 
anaconda-ks.cfg  install.log  install.log.syslog  nginx-1.16.1.tar.gz  zabbix-4.0.3.tar.gz
#你会发现跟zabbixserver是一样的包，这个包里包含了agent端，和server端
[root@www_001 ~]# tar xf zabbix-4.0.3.tar.gz -C /usr/src/
[root@www_001 ~]# cd /usr/src/zabbix-4.0.3/
[root@www_001 zabbix-4.0.3]# ./configure --prefix=/usr/local/zabbix --enable-agent

***********************************************************
*            Now run 'make install'                       *
*                                                         *
*            Thank you for using Zabbix!                  *
*              <http://www.zabbix.com>                    *
***********************************************************
#出现这个就成功了
[root@www_001 zabbix-4.0.3]# make && make install
[root@www_001 zabbix-4.0.3]# echo "$?"
0
[root@www_001 zabbix-4.0.3]# chown zabbix:zabbix -R /usr/local/zabbix/
[root@www_001 zabbix-4.0.3]# tail -1 /etc/profile
export PATH=$PATH:/usr/local/zabbix/sbin/:/usr/local/zabbix/bin/  #这里大家也可以做软链接，本意是让linux找到命令
[root@www_001 zabbix-4.0.3]# source /etc/profile
[root@www_001 zabbix-4.0.3]# zabbix_agentd -V
zabbix_agentd (daemon) (Zabbix) 4.0.3
Revision 87993 20 December 2018, compilation time: Feb 28 2020 19:29:14

Copyright (C) 2018 Zabbix SIA
License GPLv2+: GNU GPL version 2 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it according to
the license. There is NO WARRANTY, to the extent permitted by law.

[root@www_001 zabbix-4.0.3]# cat /usr/local/zabbix/etc/zabbix_agentd.conf #将原有的清空写成下面的样子
PidFile=/usr/local/zabbix/zabbix_agentd.pid
LogFile=/usr/local/zabbix/zabbix_agentd.log
Hostname=www_001  #建议最好和web界面的主机名，已经本地主机名三个名字一样，
Server=192.168.200.173 #这个就是类似白名单谁可以过来获取信息，这里写server端的ip就可以
ServerActive=192.168.200.173 #同上，也是白名单，
UnsafeUserParameters=1
Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/*.conf #插入其他的配置文件



[root@www_001 zabbix-4.0.3]# zabbix_agentd
[root@www_001 zabbix-4.0.3]# ss -antup | grep 10050
tcp    LISTEN     0      128                    *:10050                 *:*      users:(("zabbix_agentd",12714,4),("zabbix_agentd",12715,4),("zabbix_agentd",12716,4),("zabbix_agentd",12717,4),("zabbix_agentd",12718,4),("zabbix_agentd",12719,4))

[root@www_001 zabbix-4.0.3]# ps -ef | grep zabbix
zabbix    12714      1  0 19:43 ?        00:00:00 zabbix_agentd
zabbix    12715  12714  0 19:43 ?        00:00:00 zabbix_agentd: collector [idle 1 sec]
zabbix    12716  12714  0 19:43 ?        00:00:00 zabbix_agentd: listener #1 [waiting for connection]
zabbix    12717  12714  0 19:43 ?        00:00:00 zabbix_agentd: listener #2 [waiting for connection]
zabbix    12718  12714  0 19:43 ?        00:00:00 zabbix_agentd: listener #3 [waiting for connection]
zabbix    12719  12714  0 19:43 ?        00:00:00 zabbix_agentd: active checks #1 [idle 1 sec]
root      12723   4884  0 19:44 pts/0    00:00:00 grep zabbix
[root@www_001 zabbix-4.0.3]# tail -f /usr/local/zabbix/zabbix_agentd.log 
 12714:20200228:194342.301 IPv6 support:           NO
 12714:20200228:194342.301 TLS support:            NO
 12714:20200228:194342.301 **************************
 12714:20200228:194342.301 using configuration file: /usr/local/zabbix/etc/zabbix_agentd.conf
 12714:20200228:194342.302 agent #0 started [main process]
 12717:20200228:194342.303 agent #3 started [listener #2]
 12718:20200228:194342.304 agent #4 started [listener #3]
 12716:20200228:194342.304 agent #2 started [listener #1]
 12719:20200228:194342.305 agent #5 started [active checks #1]
 12715:20200228:194342.321 agent #1 started [collector]
```

###2.2zabbix-get
>zabbix提供一个zabbix_get工具，可以跟zabbix agent通讯获取监控信息 
使用方式：zabbix_get -s 指定获取数据的agent端 -k 关键的键 
zabbix agent查看所有可监控项目，在agent端输入这个命令就可以查看zabbix_agentd -p
```
#在server端进行操作
[root@zabbix ~]# zabbix_get -s 192.168.200.128 -k system.hostname
www_001
[root@zabbix ~]# zabbix_get -s 192.168.200.128 -k system.uname
Linux www_001 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64
```
###2.3监控cpu的负载
>cpu负载的键(也可以通过zabbix_agentd -p来过滤查看)

* key: system.cpu.load[all,avg1] Float型
* key: system.cpu.load[all,avg5] Float型
* key: system.cpu.load[all,avg15] Float型

![image_1e4vdjq5o1geica84i3e2p14a49.png-112.1kB][1]

![image_1e4vdk6gi6f0q8csi5mnq14mrm.png-103.1kB][2]

![image_1e4vdkdi7vea1feeisde6d1kac13.png-94.4kB][3]

![image_1e4vdkkava47vcg19bk1sj2pge1g.png-85.8kB][4]

![image_1e4vdkqib3hm93h14jm10p7srq1t.png-91.5kB][5]

![image_1e4vdl13nu3gs481sfv6nk1leq2a.png-105.8kB][6]

![image_1e4vdl7l11ng81nrn4l5nj166f2n.png-104.4kB][7]

![image_1e4vdldqr1k3uljt148hmc612fa34.png-176.4kB][8]

![image_1e4vdljr91po01eutpk34fp1qpi3h.png-122.9kB][9]

![image_1e4vdlrd71q8j1hd9p5sbcn1v4o3u.png-87.3kB][10]

![image_1e4vdm1de1dnr1c0219d1ocqedt4b.png-110.1kB][11]

![image_1e4vdm8dgnf41dnfl7kqpcb274o.png-121.9kB][12]

![image_1e4vdmh84abfisa10ovb48tk255.png-103.6kB][13]

![image_1e4vdmoi43qn114jbb1s4lak25i.png-119.5kB][14]

![image_1e4vdmv1vs5q1bsm1mv4ilk13em5v.png-135.9kB][15]

![image_1e4vdn4tedgq162h1rgt1bmb1nlr6c.png-123.3kB][16]

![image_1e4vdnb5idso1bd310u27sf1jdh6p.png-115.1kB][17]

![image_1e4vdnheu167top91utk1g8c1hmk76.png-95.5kB][18]

![image_1e4vdnnvq13pm1j8j1ffujsfvdm7j.png-108.1kB][19]

![image_1e4vdnuii134r2rpald19rttv080.png-90.6kB][20]

![image_1e4vdo4g8lse12ck1t0r1cf0dum8d.png-107.6kB][21]

![image_1e4vdoapiv6o1f821p9k1q2m1njf8q.png-85.7kB][22]

![image_1e4vdoghv1fcr9qr1vq3kgb127j97.png-116kB][23]

![image_1e4vdonv8u4n97mhn116pvlu9k.png-88.1kB][24]

![image_1e4vdovgdl0v183j1g7r3c1qrva1.png-90.7kB][25]

![image_1e4vdp6pbt0c14tg1uvq43r1v1cae.png-119.6kB][26]

![image_1e4vdpcfm1oe31avij0nsct6miar.png-118.7kB][27]

![image_1e4vdpivh1ptg1ugl1e9gfer1a8cb8.png-128.3kB][28]

![image_1e4vdpql35fl1521vvo1tao1pbhbl.png-110.9kB][29]

###2.4监控cpu使用和空闲
>system.cpu.util[,iowait,] Float型 cpu读写磁盘等待 
system.cpu.util[,system,] Float型 系统使用cpu 
system.cpu.util[,user,] Float型 用户使用cpu 
system.cpu.util[,idle,] Float型 cpu空闲

  [1]: http://static.zybuluo.com/yao-yuan-ge/10dvypdpq05f667qsudxdnor/image_1e4vdjq5o1geica84i3e2p14a49.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/xi6bm1rco0bippbgbbarejmc/image_1e4vdk6gi6f0q8csi5mnq14mrm.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/fvcqs2hcqdtixc0chbj6sf25/image_1e4vdkdi7vea1feeisde6d1kac13.png
  [4]: http://static.zybuluo.com/yao-yuan-ge/7fe0g14m4xc0rnu2zuh29nch/image_1e4vdkkava47vcg19bk1sj2pge1g.png
  [5]: http://static.zybuluo.com/yao-yuan-ge/1w9n7he9r0vrwxsswiseixe9/image_1e4vdkqib3hm93h14jm10p7srq1t.png
  [6]: http://static.zybuluo.com/yao-yuan-ge/psysod9528oej4vgowyovxh1/image_1e4vdl13nu3gs481sfv6nk1leq2a.png
  [7]: http://static.zybuluo.com/yao-yuan-ge/jx1gcllgt9hrtazsw59cadp8/image_1e4vdl7l11ng81nrn4l5nj166f2n.png
  [8]: http://static.zybuluo.com/yao-yuan-ge/n3rwslc54uy4ltyhfnvjt9k2/image_1e4vdldqr1k3uljt148hmc612fa34.png
  [9]: http://static.zybuluo.com/yao-yuan-ge/s19u8zhn3jij3eeyxizg8ght/image_1e4vdljr91po01eutpk34fp1qpi3h.png
  [10]: http://static.zybuluo.com/yao-yuan-ge/2psgodyazcp3fgou799owl0b/image_1e4vdlrd71q8j1hd9p5sbcn1v4o3u.png
  [11]: http://static.zybuluo.com/yao-yuan-ge/afqjmlv8hn0owi1uz7zusm88/image_1e4vdm1de1dnr1c0219d1ocqedt4b.png
  [12]: http://static.zybuluo.com/yao-yuan-ge/hqy613u8tr4rpje7r3rfs0x4/image_1e4vdm8dgnf41dnfl7kqpcb274o.png
  [13]: http://static.zybuluo.com/yao-yuan-ge/le2zmcth68hb3rc7q8le3pos/image_1e4vdmh84abfisa10ovb48tk255.png
  [14]: http://static.zybuluo.com/yao-yuan-ge/4qceqxfw07vzedazikvau2he/image_1e4vdmoi43qn114jbb1s4lak25i.png
  [15]: http://static.zybuluo.com/yao-yuan-ge/jmu6lmdcgv0pwirjtcuv03j2/image_1e4vdmv1vs5q1bsm1mv4ilk13em5v.png
  [16]: http://static.zybuluo.com/yao-yuan-ge/4svtrlz30n9ycp0ct69c1njx/image_1e4vdn4tedgq162h1rgt1bmb1nlr6c.png
  [17]: http://static.zybuluo.com/yao-yuan-ge/ha5sz8ock82dwc2y3k7dtlk1/image_1e4vdnb5idso1bd310u27sf1jdh6p.png
  [18]: http://static.zybuluo.com/yao-yuan-ge/lvsozgcltqh3hvzi0gpmtxxo/image_1e4vdnheu167top91utk1g8c1hmk76.png
  [19]: http://static.zybuluo.com/yao-yuan-ge/tlfgw8gjyo1gg5kluue7g05v/image_1e4vdnnvq13pm1j8j1ffujsfvdm7j.png
  [20]: http://static.zybuluo.com/yao-yuan-ge/q4bdjlpvba0ddkicg9ixdmn8/image_1e4vdnuii134r2rpald19rttv080.png
  [21]: http://static.zybuluo.com/yao-yuan-ge/yzc0v5cd7f61uv82fxhzrvpr/image_1e4vdo4g8lse12ck1t0r1cf0dum8d.png
  [22]: http://static.zybuluo.com/yao-yuan-ge/p6t0omods9mlldgsuzguk5dt/image_1e4vdoapiv6o1f821p9k1q2m1njf8q.png
  [23]: http://static.zybuluo.com/yao-yuan-ge/yfbngkgg0pml9el72hnjzsp4/image_1e4vdoghv1fcr9qr1vq3kgb127j97.png
  [24]: http://static.zybuluo.com/yao-yuan-ge/gtkgvfq6cuh98ew0kzc4ccq2/image_1e4vdonv8u4n97mhn116pvlu9k.png
  [25]: http://static.zybuluo.com/yao-yuan-ge/ai3rp3krx6hyov3dnzcqy1mx/image_1e4vdovgdl0v183j1g7r3c1qrva1.png
  [26]: http://static.zybuluo.com/yao-yuan-ge/28gzb3iclppb1181h0mmq5pl/image_1e4vdp6pbt0c14tg1uvq43r1v1cae.png
  [27]: http://static.zybuluo.com/yao-yuan-ge/haojzpnq472zht61e9lwyjvy/image_1e4vdpcfm1oe31avij0nsct6miar.png
  [28]: http://static.zybuluo.com/yao-yuan-ge/kc6i6aqbln9d60glypn4of9j/image_1e4vdpivh1ptg1ugl1e9gfer1a8cb8.png
  [29]: http://static.zybuluo.com/yao-yuan-ge/ophkgaahfsvpo7ba9via34vw/image_1e4vdpql35fl1521vvo1tao1pbhbl.png