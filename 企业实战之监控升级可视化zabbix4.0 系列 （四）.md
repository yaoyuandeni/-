# 企业实战之监控升级可视化zabbix4.0 系列 （四）

标签（空格分隔）： zabbix4.0系列

---

[TOC]

https://www.zybuluo.com/commandersu/note/1670449
##添加agent客户端，linux系统及windows系统
##一、实验环境
###zabbix-server端
|操作系统|主机名|ip|内存|
|-|-|-|
|centos7.5|zabbix|192.168.200.173|1G|
###zabbix-agent端
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

![image_1e6tu4odu13k314ok61r1cugs3o9.png-115.1kB][30]

![image_1e6tu58851bal2tq5cs1t211idm.png-134.9kB][31]

![image_1e6tu5rut1sa99qt16inr541c7d13.png-106.1kB][32]

![image_1e6tu6j4a1269eeg8rg1qg71i9f1g.png-201.5kB][33]

![image_1e6tu70th1unhbahai6mkbeu91t.png-138.2kB][34]

![image_1e6tu7dqg1l96ttm1da5a8318p92a.png-140kB][35]

![image_1e6tu7ps7111m1rr417u41hk1ug2n.png-134.6kB][36]

同理我们在克隆几个出来我们监控cpu的状态 

![image_1e6tu9gm0903189u137p6u81hlo34.png-107.5kB][37]

![image_1e6tua1ed1n1p11gcp1cdu92fd3h.png-144.7kB][38]

![image_1e6tuaeii1rjb7di1c0ja631hr3u.png-109.6kB][39]

![image_1e6tuannj12i61sis1dvvkld1nl4b.png-136.5kB][40]

![image_1e6tub0a51i6515iprf81kocmui4o.png-148kB][41]

![image_1e6tub9eu9732c07p51i7ghgv55.png-114.3kB][42]

![image_1e6tubjl81i6kvbtk7mc1q1aph5i.png-128.8kB][43]

![image_1e6tuca5jkmo1baqtf3bt91k8c5v.png-156.9kB][44]

![image_1e6tuco781omv1ofs11p03fc1a4k6c.png-110kB][45]

![image_1e6tud0lmq4idodkirl1u10tp6p.png-127.3kB][46]

![image_1e6tud8g016tu1orf117oc92139g76.png-161.6kB][47]

![image_1e6tudjca181f1ra612ki1vjgo5d7j.png-148.6kB][48]

###2.5监控剩余内存(buffers\cached实际上也是剩余的)
vm.memory.size[available]
![image_1e6tuet1rbiu1gegd1118bvsqn83.png-114.8kB][49]

![image_1e6tuf9n6dqj12fn9u81rf41oam8g.png-147.4kB][50]

![image_1e6tufksh18j5u6t12d31lq91ot88t.png-112.7kB][51]

![image_1e6tug2tj1l7toenkt15ic1f8s9a.png-135.4kB][52]

![image_1e6tugegh8bn7451a2rg7igid9n.png-153.6kB][53]

![image_1e6tugotu12us18d3sni1f3j1kbna4.png-147.3kB][54]

###2.6监控磁盘
>vfs.fs.size[/,pfree] Float型 监控磁盘容量剩余百分比 
vfs.fs.inode[/,pfree] Float型 监控磁盘iNode的使用率剩余的百分比 

![image_1e6tui7umu2l10t1ksg13rfdf5ah.png-114.8kB][55]

![image_1e6tuikaergj10t6mga1443ibtau.png-147.1kB][56]

![image_1e6tuj30uhj51vd61eov9op757bn.png-109.2kB][57]

![image_1e6tuja4k1qtr1pignd8rcd1eiqc7.png-145.8kB][58]

![image_1e6tujj4r38vgp2a1o1ak810bvck.png-159.9kB][59]

![image_1e6tuk8isenn4av108buc017ned1.png-130.6kB][60]

![image_1e6tukgct1ojgti2ms1p97qhbde.png-106.3kB][61]

![image_1e6tukpf138u1sq810no17vcorfdr.png-156.6kB][62]

![image_1e6tul1vj14vp76a1gs215e61uqae8.png-110.9kB][63]

![image_1e6tul9olcb41ljp14g110usfp0el.png-137.7kB][64]

![image_1e6tulie916mg1qkc109d3utojvf2.png-127.5kB][65]

###2.7监控流量
>net.if.in[eth0] 入流量 整形(每秒速率) 默认返回字节数，需要*8 
net.if.out[eth0] 出流量 整形(每秒速率) 默认返回字节数，需要*8 
我们zabbix获取的流量是总的，我们想要每秒多少流量是需要算的 
流量计算公式：（本次获取的流量总数-上次获取的流量总数）/时间间隔（ps：我们一般是30秒）*8

![image_1e6tumj0ljjl1te2av1l3e18giff.png-136kB][66]

![image_1e6tums7g1h651an1a6f163215u3fs.png-113.5kB][67]

![image_1e6tun5oc1l1fp51vcd1olivbkg9.png-167.1kB][68]

![image_1e6tunf3a1s1r1j2v379uin19asgm.png-109.9kB][69]

![image_1e6tunp1chs181l16ughf11dsch3.png-149.1kB][70]

![image_1e6tunvvj18dpmdeia217o716cmhg.png-98.5kB][71]

![image_1e6tuo95u1cgor1mt1j1vde1up8ht.png-116.4kB][72]

![image_1e6tuoha21so1l67805188uttjia.png-111.1kB][73]

![image_1e6tuoqaio7h13l91otm2f1qkin.png-157.5kB][74]

![image_1e6tup7c7124dk191j9e6hv1ec4jj.png-112.7kB][75]

![image_1e6tupq1h16g1vub1cnp12ea1q3mk0.png-131.4kB][76]

![image_1e6tuq1eq1fb21ajm1v9rpld1cg1kd.png-107.1kB][77]

![image_1e6tuq9r48slasu1utd1h628o7kq.png-110.3kB][78]

![image_1e6tuqgt71d2h9805ss1lplgthl7.png-153kB][79]

![image_1e6tuqr85moo3ev272187c1ti6lk.png-131.8kB][80]

附录>如果不知道key的参数是做什么的，我们可以去官网查看 
官网地址：https://www.zabbix.com/ 
查询key怎么用的地址：https://www.zabbix.com/documentation/4.0/manual

![image_1e6turv58t361sd01u9ii33s8pm1.png-209.2kB][81]

![image_1e6tus95sn6r1ple4lt1ulc2btme.png-179.8kB][82]

##三、使用模板监控

![image_1e6tussnhiek1fra15i81fg61eiqmr.png-146.9kB][83]

![image_1e6tut7kfcc413c21bf915ocnnvn8.png-124.7kB][84]

![image_1e6tutg0n7ug1k0310jn66a1lhnl.png-149.2kB][85]

![image_1e6tutq1q46nim9ord1oen1rtro2.png-113.8kB][86]

![image_1e6tuu1jcbjv1m6ktig1sg62ceof.png-148kB][87]

![image_1e6tuu9egad57iq1ch016461982os.png-122.9kB][88]

![image_1e6tuuhoq1lfp134b1po53pu1g1ip9.png-114kB][89]

![image_1e6tuuqlu1p2t1qsok4g118u1grrpm.png-147.4kB][90]

![image_1e6tuv4nc1bu723215jnuetem8q3.png-145.7kB][91]

![image_1e6tuvd91vlc163p1f6m62k180bqg.png-152.2kB][92]

![image_1e6tuvlb21p066om1phf40jmbtqt.png-142.9kB][93]

![image_1e6tuvubkatg2rh1n4i1v2eg4vra.png-136kB][94]

![image_1e6tv054f1po7dnl1mmkl54slcrn.png-161.3kB][95]

![image_1e6tv0cjilf3kt1kpt129on7ms4.png-172.2kB][96]

![image_1e6tv0ip1aj01f6a1b71i8c1mlosh.png-130.6kB][97]

![image_1e6tv0r4g1sp51v4b3oc153151dsu.png-120.1kB][98]

![image_1e6tv18kkjoa1hl3i1mequsatr.png-203.1kB][99]

![image_1e6tv1gp1gpb1u0cdgg1not8e3u8.png-160.9kB][100]

![image_1e6tv1pb1ger158q11dn1rgkbuful.png-212.2kB][101]

![image_1e6tv203q1rc4vi113dkfgjisjv2.png-197.4kB][102]

![image_1e6tv2f7a1na31mvp7ib1glv1s36vf.png-151.3kB][103]

![image_1e6tv3gbi1j2t14451bqvv4kbunvs.png-156kB][104]

![image_1e6tv3tmj1b3oj1i167e9d7i22109.png-124.7kB][105]

![image_1e6tv464jdmumla1014391iv10m.png-163.3kB][106]

![image_1e6tv4c681uclj6k11hf1ne21g5j113.png-179.7kB][107]

![image_1e6tv4i7niur18bjb5m9ku7ku11g.png-122.9kB][108]

![image_1e6tv4r0r10uph841ovtvgvc3p11t.png-149.5kB][109]

![image_1e6tv52pljge15lf1t8ok1ip7912a.png-174.5kB][110]

![image_1e6tv5ajve3v8a31b2kktcaqh12n.png-103.2kB][111]

![image_1e6tv5gceak7pat1t9174111s8134.png-135.9kB][112]

![image_1e6tv5m701h9s11jd3c71nnti6s13h.png-115.4kB][113]

![image_1e6tv5s9817h01dbaobqs2k17am13u.png-144.4kB][114]

![image_1e6tv64956qpfq3122g10u7bo214b.png-196.2kB][115]

![image_1e6tv70v21lki17k81jv5p3hhu014o.png-135.9kB][116]

![image_1e6tv7a8usq7j6sjhv1rhmup7155.png-112.7kB][117]

##四、自定义key监控项
>自定义key 
无参数自定义key 
有参数自定义key 
使用自定义Shell脚本监控内存，一般脚本只需要输出数字即可

![image_1e6tv8dlit341kp71ipk26tt6u15i.png-146kB][118]

![image_1e6tv8jqu1iid10fn16eat4d1j7515v.png-108.2kB][119]

![image_1e6tv8pkgr7ean41ovf177bd0t16c.png-133.7kB][120]

###4.1在agent端修改配置文件
```
#我们自己写一个无参数的获取可用内存的key，在001这台主机上读取内存，我们需要输出的值只是个数字
[root@www_001 ~]# free  -m | grep 'Mem:' |awk '{print $NF}'
311
#将这条命令写到脚本里
[root@www_001 ~]# cat /tmp/memavailable.sh
#!/bin/bash
free  -m | grep 'Mem:' |awk '{print $NF}'

[root@www_001 ~]# sh /tmp/memavailable.sh
311
[root@www_001 ~]# cat /usr/local/zabbix/etc/zabbix_agentd.conf
PidFile=/usr/local/zabbix/zabbix_agentd.pid
LogFile=/usr/local/zabbix/zabbix_agentd.log
Hostname=www_001
Server=192.168.200.173
ServerActive=192.168.200.173
UnsafeUserParameters=1
Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/*.conf  #看下agent端的配置文件，如果要定义key需要修改配置文件，我们如果都在主配置文件里写会特别的大，修改起来还麻烦，所以我们引用别的配置文件，类似nginx的配置文件标准化

[root@www_001 ~]# cat /usr/local/zabbix/etc/zabbix_agentd.conf.d/mem.conf 
UserParameter=mem.available,sh /tmp/memavailable.sh   
#这个是我们的自定义key的配置文件
#UserParameter -->表示自定义key
#mem.available--->逗号前边的是我们的key的名字，这个名字跟我们在web页面上写的时候要一样，不然报错
#sh /tmp/memavailable.sh--->逗号后边是我们需要执行的命令，这个命令的执行结果一定是个值，这个值就我们key返回的值

#重启服务
[root@www_001 ~]# pkill zabbix_agentd
[root@www_001 ~]# ps -ef | grep zabbix
root      15813  15723  0 12:45 pts/0    00:00:00 grep zabbix
[root@www_001 ~]# zabbix_agentd 
[root@www_001 ~]# ps -ef | grep zabbix
zabbix    15816      1  0 12:45 ?        00:00:00 zabbix_agentd
zabbix    15817  15816  0 12:45 ?        00:00:00 zabbix_agentd: collector [idle 1 sec]
zabbix    15818  15816  0 12:45 ?        00:00:00 zabbix_agentd: listener #1 [waiting for connection]
zabbix    15819  15816  0 12:45 ?        00:00:00 zabbix_agentd: listener #2 [waiting for connection]
zabbix    15820  15816  0 12:45 ?        00:00:00 zabbix_agentd: listener #3 [waiting for connection]
zabbix    15821  15816  0 12:45 ?        00:00:00 zabbix_agentd: active checks #1 [idle 1 sec]
root      15823  15723  0 12:45 pts/0    00:00:00 grep zabbix
```

###4.2在server端测试获取值
```
[root@zabbix ~]# zabbix_get -s 192.168.200.128 -k mem.available
311
#-s 指定去哪里获取
#-k  我的key 是什么名字
```

###4.3在网页上添加

![image_1e6tvamo2rijoum1jlrm1unfh16p.png-128.9kB][121]

![image_1e6tvatfp9vf152p7pqrtssqv176.png-100.2kB][122]

![image_1e6tvb5fm1j144ai5b66kopj17j.png-117.4kB][123]

![image_1e6tvbk31en81j4c10m919oh1mdn180.png-111.5kB][124]

![image_1e6tvbrvb11uj9l22nq7gcukq18d.png-121kB][125]

![image_1e6tvc3fq1o2n16nd17cd1cd24b218q.png-134.4kB][126]

![image_1e6tvcaacfok14kqsfk1lpc1ole197.png-127.6kB][127]

![image_1e6tvcho8eq213cl1qm01iv1oa319k.png-132.5kB][128]

![image_1e6tvcpvq1gqvqo1dcfp6apd1a1.png-118.1kB][129]

![image_1e6tvd0ge1rhd1ucm1ij71bapk2r1ae.png-135.6kB][130]

![image_1e6tvd81e162h3us8jgs3s139e1ar.png-116.1kB][131]

##五、可以传参的自定义key
###5.1在agent端编写脚本
```
[root@www_001 ~]# cat /tmp/mem.sh
#!/bin/bash
case "$1" in 
  "available") free  -m | grep 'Mem:' |awk '{print $NF}';;
  "total") free  -m | grep 'Mem:' |awk '{print $2}';;
  "used") free  -m | grep 'Mem:' |awk '{print $3}';;
  *) echo "not supported";;
esac
[root@www_001 ~]# sh /tmp/mem.sh total
980
[root@www_001 ~]# sh /tmp/mem.sh used
408
#修改配置文件
[root@www_001 ~]# cat /usr/local/zabbix/etc/zabbix_agentd.conf.d/mem.conf
UserParameter=mem.available,sh /tmp/memavailable.sh
UserParameter=mem.check[*],sh /tmp/mem.sh $1  #新添加的一行
#重启服务
[root@www_001 ~]# pkill zabbix_agentd
[root@www_001 ~]# ps -ef | grep zabbix
root      15991  15723  0 13:18 pts/0    00:00:00 grep zabbix
[root@www_001 ~]# zabbix_agentd 
[root@www_001 ~]# ps -ef | grep zabbix
zabbix    15994      1  0 13:18 ?        00:00:00 zabbix_agentd
zabbix    15995  15994  0 13:18 ?        00:00:00 zabbix_agentd: collector [idle 1 sec]
zabbix    15996  15994  0 13:18 ?        00:00:00 zabbix_agentd: listener #1 [waiting for connection]
zabbix    15997  15994  0 13:18 ?        00:00:00 zabbix_agentd: listener #2 [waiting for connection]
zabbix    15998  15994  0 13:18 ?        00:00:00 zabbix_agentd: listener #3 [waiting for connection]
zabbix    15999  15994  0 13:18 ?        00:00:00 zabbix_agentd: active checks #1 [idle 1 sec]
root      16001  15723  0 13:18 pts/0    00:00:00 grep zabbix
[root@www_001 ~]# 
```

###5.2在server端验证
```
[root@zabbix ~]# zabbix_get -s 192.168.200.128 -k mem.check[used]
408
[root@zabbix ~]# zabbix_get -s 192.168.200.128 -k mem.check[total]
980
```

###5.3在web界面添加监控项

![image_1e6tvfprc1vo71jc6101a3856b91b8.png-115.1kB][132]

![image_1e6tvg3506tvisbvqr1tif1mb81bl.png-123.8kB][133]

![image_1e6tvg9uf1t0u119is8j1r6eugc1c2.png-105.3kB][134]

![image_1e6tvgfio1o1k2pu1ijr611i8p1cf.png-124.7kB][135]

![image_1e6tvglkoldt1lvl9qam9m9031cs.png-128.6kB][136]

![image_1e6tvgtekf93o691ost74415qm1d9.png-116.7kB][137]

##六、Windows系统添加zabbix
###6.1我们创建一个Windows系统

![image_1e6tvhmtl10h617c5tr1grihuj1dm.png-75.5kB][138]

![image_1e6tvhtm81hbk1oaj3aps9eb91e3.png-34.4kB][139]

![image_1e6tvi58c1sf51u2g6qq187sohr1eg.png-472.3kB][140]

![image_1e6tvic2qbi418v1egc107u52j1et.png-202.2kB][141]

![image_1e6tvij4b1l6a1efr1hj31ih1n9f1fa.png-124.6kB][142]

![image_1e6tvipqt1lru4t11c3mok71dns1fn.png-208.6kB][143]

![image_1e6tvivn11edl19k214b1art1dl1g4.png-259.4kB][144]

![image_1e6tvj5vj17i274aamm1sud8ic1gh.png-433.5kB][145]

![image_1e6tvjdhnhf010r49kudjoghf1gu.png-93.5kB][146]

![image_1e6tvjjjojhu1808jr4187u1vb81hb.png-35.5kB][147]

![image_1e6tvjpgt1jts1jjlcuq73po221ho.png-69.4kB][148]

![image_1e6tvjvca5uktsv1pmv1597hlk1i5.png-214.8kB][149]

![image_1e6tvk57chljn081jn246rm851ii.png-221.7kB][150]

![image_1e6tvkb9m1f172rec88h4kgs21iv.png-129.7kB][151]

![image_1e6tvkhg9iiehc42p955u1o5l1jc.png-155.9kB][152]

![image_1e6tvknd3aao2m4gso17i5e5c1jp.png-461kB][153]

![image_1e6tvkt3h101p1k1pu1i12m417tm1k6.png-381.5kB][154]

![image_1e6tvl7qaa5g1kbg1rtuttg1jsk1kj.png-91.7kB][155]

![image_1e6tvlecpgcmp00k3o1hc0ech1l0.png-167.2kB][156]

![image_1e6tvlkm7q1arg11rc3h7q11ph1ld.png-115.4kB][157]

![image_1e6tvlsna1v88m8gc7bnrq1die1lq.png-29.1kB][158]

![image_1e6tvm3rg1sj6nc818ov1sbq14dp1m7.png-167.4kB][159]

![image_1e6tvma4m4h6101nr8d1kq98uc1mk.png-170kB][160]

![image_1e6tvmgcvt101opi1cer1ko8l6c1n1.png-142.9kB][161]

![image_1e6tvmmr21mk1s1h1oqr6ss1gsl1ne.png-124.2kB][162]

![image_1e6tvmtob13639h61rod15k1ucv1nr.png-333.5kB][163]

![image_1e6tvn5ol1ov0skau3b7rt1lmm1o8.png-216.4kB][164]

![image_1e6tvncem1mrj6sv155h160lfve1ol.png-151.3kB][165]

![image_1e6tvnjgt19beuunbf5u3n181f1p2.png-46.3kB][166]

![image_1e6tvnq07mg1b3r1lst17th11ed1pf.png-33.8kB][167]

![image_1e6tvo03g51g1pcj1n5m7ua8c51ps.png-51.6kB][168]

![image_1e6tvo5o514159c61qps11gp1uv21q9.png-23.1kB][169]

![image_1e6tvob9u39p10j7v2jgc91s01qm.png-17.6kB][170]

![image_1e6tvoinp1d8tb3e18s51suc1v8h1r3.png-121.6kB][171]

![image_1e6tvoohl134h9fj1polmf4dsa1rg.png-98.7kB][172]

###6.2zabbix监控Windows
```
[root@zabbix ~]# zabbix_get -s 192.168.200.149 -k system.uname
Windows DESKTOP-MRVB912 10.0.10586 Microsoft Windows 10 企业版 x64
[root@zabbix ~]# zabbix_get -s 192.168.200.149 -k vm.memory.size[free]
1142566912
[root@zabbix ~]# zabbix_get -s 192.168.200.149 -k vfs.fs.size[C:,pfree]
82.884299
[root@zabbix ~]# zabbix_get -s 192.168.200.149 -k vfs.fs.size[C:,free]
52903399424
```

![image_1e6tvplc79fgbb71dlhe7p1gq01rt.png-94.8kB][173]

![image_1e6tvpsccdr21fu1104o1ghp5f1sa.png-115.3kB][174]

![image_1e6tvq1j29i71pod9kb1kaaumk1sn.png-111.2kB][175]

![image_1e6tvq6n01b9q17mb1r711g6b16j51t4.png-130.3kB][176]

![image_1e6tvqf771nkv1b78fqt1ls71net1th.png-142kB][177]


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
  [30]: http://static.zybuluo.com/yao-yuan-ge/la08kz590parx719tod3lqtm/image_1e6tu4odu13k314ok61r1cugs3o9.png
  [31]: http://static.zybuluo.com/yao-yuan-ge/cdtpcjutfxa0hilrkuaq3epx/image_1e6tu58851bal2tq5cs1t211idm.png
  [32]: http://static.zybuluo.com/yao-yuan-ge/tsjsl6hjornuwl7vor9df665/image_1e6tu5rut1sa99qt16inr541c7d13.png
  [33]: http://static.zybuluo.com/yao-yuan-ge/epkxs0o0rdj9tmetsacmqlth/image_1e6tu6j4a1269eeg8rg1qg71i9f1g.png
  [34]: http://static.zybuluo.com/yao-yuan-ge/sgrbg0h0mgj6yw094qextl20/image_1e6tu70th1unhbahai6mkbeu91t.png
  [35]: http://static.zybuluo.com/yao-yuan-ge/f32fa6q8qfw2rdf96d0lumdn/image_1e6tu7dqg1l96ttm1da5a8318p92a.png
  [36]: http://static.zybuluo.com/yao-yuan-ge/fv6zxmmkvpjvqd01wizv2gk6/image_1e6tu7ps7111m1rr417u41hk1ug2n.png
  [37]: http://static.zybuluo.com/yao-yuan-ge/zzsiq33743e6seopu1cgifo4/image_1e6tu9gm0903189u137p6u81hlo34.png
  [38]: http://static.zybuluo.com/yao-yuan-ge/lcogykg9ypg0e7truxbu8uy0/image_1e6tua1ed1n1p11gcp1cdu92fd3h.png
  [39]: http://static.zybuluo.com/yao-yuan-ge/wjvsk1p8v1kiyvuufd25el05/image_1e6tuaeii1rjb7di1c0ja631hr3u.png
  [40]: http://static.zybuluo.com/yao-yuan-ge/3p5xsv3rksqzip36v85710me/image_1e6tuannj12i61sis1dvvkld1nl4b.png
  [41]: http://static.zybuluo.com/yao-yuan-ge/s4c7m8ufpnay6df71t5pns5l/image_1e6tub0a51i6515iprf81kocmui4o.png
  [42]: http://static.zybuluo.com/yao-yuan-ge/gu7y19di7wxgvlz8fe6er4qe/image_1e6tub9eu9732c07p51i7ghgv55.png
  [43]: http://static.zybuluo.com/yao-yuan-ge/de3t1le6ta4e27jkgnm0nrkx/image_1e6tubjl81i6kvbtk7mc1q1aph5i.png
  [44]: http://static.zybuluo.com/yao-yuan-ge/2c168m9ybfpojuxjtzy34c3f/image_1e6tuca5jkmo1baqtf3bt91k8c5v.png
  [45]: http://static.zybuluo.com/yao-yuan-ge/zfvlaxzs6sbkvz72svy9wkln/image_1e6tuco781omv1ofs11p03fc1a4k6c.png
  [46]: http://static.zybuluo.com/yao-yuan-ge/csfxve337rdi2fg5tnkhcys6/image_1e6tud0lmq4idodkirl1u10tp6p.png
  [47]: http://static.zybuluo.com/yao-yuan-ge/2q65b4w5dj1wqmavuvg46o96/image_1e6tud8g016tu1orf117oc92139g76.png
  [48]: http://static.zybuluo.com/yao-yuan-ge/4xxamby9pbetr10pt2g0rnic/image_1e6tudjca181f1ra612ki1vjgo5d7j.png
  [49]: http://static.zybuluo.com/yao-yuan-ge/om0tzqxhrlwimr6606ul620o/image_1e6tuet1rbiu1gegd1118bvsqn83.png
  [50]: http://static.zybuluo.com/yao-yuan-ge/t6rua5z5paaq2emw0zyahp0d/image_1e6tuf9n6dqj12fn9u81rf41oam8g.png
  [51]: http://static.zybuluo.com/yao-yuan-ge/scbrmvku85uz68hzpvbggpxm/image_1e6tufksh18j5u6t12d31lq91ot88t.png
  [52]: http://static.zybuluo.com/yao-yuan-ge/39uloz1mm992m2krwxs7nsh9/image_1e6tug2tj1l7toenkt15ic1f8s9a.png
  [53]: http://static.zybuluo.com/yao-yuan-ge/akjan1svpi4i8hskrvmgm6ep/image_1e6tugegh8bn7451a2rg7igid9n.png
  [54]: http://static.zybuluo.com/yao-yuan-ge/dpou8xcrv8qz5pgwgbykt5ta/image_1e6tugotu12us18d3sni1f3j1kbna4.png
  [55]: http://static.zybuluo.com/yao-yuan-ge/8vvnu5vt6uf01l44r1q3a3m6/image_1e6tui7umu2l10t1ksg13rfdf5ah.png
  [56]: http://static.zybuluo.com/yao-yuan-ge/mguhhc2h7k6k4wixekwj99ss/image_1e6tuikaergj10t6mga1443ibtau.png
  [57]: http://static.zybuluo.com/yao-yuan-ge/mae6zfi3wd2p45xszlev4e9t/image_1e6tuj30uhj51vd61eov9op757bn.png
  [58]: http://static.zybuluo.com/yao-yuan-ge/t23tze1l2a5p1646zzc73hsn/image_1e6tuja4k1qtr1pignd8rcd1eiqc7.png
  [59]: http://static.zybuluo.com/yao-yuan-ge/9wqoj0h7mf6xaj6t3rjbtuxu/image_1e6tujj4r38vgp2a1o1ak810bvck.png
  [60]: http://static.zybuluo.com/yao-yuan-ge/niw56ze1e1j078l41omt09qz/image_1e6tuk8isenn4av108buc017ned1.png
  [61]: http://static.zybuluo.com/yao-yuan-ge/urh6brfz42f4bcwnkvrjg5qp/image_1e6tukgct1ojgti2ms1p97qhbde.png
  [62]: http://static.zybuluo.com/yao-yuan-ge/tfsdp2mmr9d8u3n8ceyx3qj3/image_1e6tukpf138u1sq810no17vcorfdr.png
  [63]: http://static.zybuluo.com/yao-yuan-ge/mf1thx8n8q9sgbc4b5yvyg7n/image_1e6tul1vj14vp76a1gs215e61uqae8.png
  [64]: http://static.zybuluo.com/yao-yuan-ge/vqy8f7og1hspc1avde7hk9fj/image_1e6tul9olcb41ljp14g110usfp0el.png
  [65]: http://static.zybuluo.com/yao-yuan-ge/9mh4d776ba9aa86rsfxvpxe1/image_1e6tulie916mg1qkc109d3utojvf2.png
  [66]: http://static.zybuluo.com/yao-yuan-ge/w7b6s6ogfy27abc9b8xqc5pe/image_1e6tumj0ljjl1te2av1l3e18giff.png
  [67]: http://static.zybuluo.com/yao-yuan-ge/4mxa2upgzcjegex7sqmrdejp/image_1e6tums7g1h651an1a6f163215u3fs.png
  [68]: http://static.zybuluo.com/yao-yuan-ge/gezy3l3yizxic21m2x4rbnj4/image_1e6tun5oc1l1fp51vcd1olivbkg9.png
  [69]: http://static.zybuluo.com/yao-yuan-ge/nr3xt6w777e618jaccgsibon/image_1e6tunf3a1s1r1j2v379uin19asgm.png
  [70]: http://static.zybuluo.com/yao-yuan-ge/agdaoegyslv573h6mo064c1l/image_1e6tunp1chs181l16ughf11dsch3.png
  [71]: http://static.zybuluo.com/yao-yuan-ge/nm19gi8qphnbcmgky72msahw/image_1e6tunvvj18dpmdeia217o716cmhg.png
  [72]: http://static.zybuluo.com/yao-yuan-ge/n56apr3908snavpwjncxewjq/image_1e6tuo95u1cgor1mt1j1vde1up8ht.png
  [73]: http://static.zybuluo.com/yao-yuan-ge/bpwghc52dsx8d45givdqgxec/image_1e6tuoha21so1l67805188uttjia.png
  [74]: http://static.zybuluo.com/yao-yuan-ge/44yhxcbwo87oq6uqf2urvguw/image_1e6tuoqaio7h13l91otm2f1qkin.png
  [75]: http://static.zybuluo.com/yao-yuan-ge/88v9eo101ylbjub3v81bsjyx/image_1e6tup7c7124dk191j9e6hv1ec4jj.png
  [76]: http://static.zybuluo.com/yao-yuan-ge/w9k7ll1phz17owrqjdyrev2m/image_1e6tupq1h16g1vub1cnp12ea1q3mk0.png
  [77]: http://static.zybuluo.com/yao-yuan-ge/m7d8t6s2yqtaxq55lzt7mbtr/image_1e6tuq1eq1fb21ajm1v9rpld1cg1kd.png
  [78]: http://static.zybuluo.com/yao-yuan-ge/lrou7jcbpk6dgzqokd02z394/image_1e6tuq9r48slasu1utd1h628o7kq.png
  [79]: http://static.zybuluo.com/yao-yuan-ge/sy3a0vt5v13tu4bsyh29x9vz/image_1e6tuqgt71d2h9805ss1lplgthl7.png
  [80]: http://static.zybuluo.com/yao-yuan-ge/fnsfm1buwy5ntg1eormvv7hd/image_1e6tuqr85moo3ev272187c1ti6lk.png
  [81]: http://static.zybuluo.com/yao-yuan-ge/9ef09cmnxvocei2zuwna8e07/image_1e6turv58t361sd01u9ii33s8pm1.png
  [82]: http://static.zybuluo.com/yao-yuan-ge/ut4kqvw8iezxnlki5b08srec/image_1e6tus95sn6r1ple4lt1ulc2btme.png
  [83]: http://static.zybuluo.com/yao-yuan-ge/f7gi5gqcphfbnomtpm012y59/image_1e6tussnhiek1fra15i81fg61eiqmr.png
  [84]: http://static.zybuluo.com/yao-yuan-ge/w3scpfnrqlisn241x821ykhk/image_1e6tut7kfcc413c21bf915ocnnvn8.png
  [85]: http://static.zybuluo.com/yao-yuan-ge/mvv4uek4b890h6giy23qua5y/image_1e6tutg0n7ug1k0310jn66a1lhnl.png
  [86]: http://static.zybuluo.com/yao-yuan-ge/izopp32gfngligfwsi30ddnt/image_1e6tutq1q46nim9ord1oen1rtro2.png
  [87]: http://static.zybuluo.com/yao-yuan-ge/x9703nalcsnw1ipn66im9z73/image_1e6tuu1jcbjv1m6ktig1sg62ceof.png
  [88]: http://static.zybuluo.com/yao-yuan-ge/sp8olfjne7tncpzik6hn7bb0/image_1e6tuu9egad57iq1ch016461982os.png
  [89]: http://static.zybuluo.com/yao-yuan-ge/h32x60c13w0f5d4sxpjyszv8/image_1e6tuuhoq1lfp134b1po53pu1g1ip9.png
  [90]: http://static.zybuluo.com/yao-yuan-ge/4y960byckr7t75z1u29ozam6/image_1e6tuuqlu1p2t1qsok4g118u1grrpm.png
  [91]: http://static.zybuluo.com/yao-yuan-ge/cievfgkesnh6b4d9j614yzeg/image_1e6tuv4nc1bu723215jnuetem8q3.png
  [92]: http://static.zybuluo.com/yao-yuan-ge/krax1r6my898k3yo5wp86nvv/image_1e6tuvd91vlc163p1f6m62k180bqg.png
  [93]: http://static.zybuluo.com/yao-yuan-ge/jas0sn9mftktncq3rg56x25i/image_1e6tuvlb21p066om1phf40jmbtqt.png
  [94]: http://static.zybuluo.com/yao-yuan-ge/kfklz22wkpsudgfv2fu3yqn2/image_1e6tuvubkatg2rh1n4i1v2eg4vra.png
  [95]: http://static.zybuluo.com/yao-yuan-ge/e05ihsr24eq4nk402phdqxn4/image_1e6tv054f1po7dnl1mmkl54slcrn.png
  [96]: http://static.zybuluo.com/yao-yuan-ge/jzdnbvdlopuluoq8y2lt9r7t/image_1e6tv0cjilf3kt1kpt129on7ms4.png
  [97]: http://static.zybuluo.com/yao-yuan-ge/3kj0teu1ozhtmikvv81w8n9t/image_1e6tv0ip1aj01f6a1b71i8c1mlosh.png
  [98]: http://static.zybuluo.com/yao-yuan-ge/ew5hjbjzexhm94rhhx4e2wz3/image_1e6tv0r4g1sp51v4b3oc153151dsu.png
  [99]: http://static.zybuluo.com/yao-yuan-ge/9r9gfgvd6uqlft4cm0ybgmnf/image_1e6tv18kkjoa1hl3i1mequsatr.png
  [100]: http://static.zybuluo.com/yao-yuan-ge/n5zu9wjbg2ct2aq02d4gajr7/image_1e6tv1gp1gpb1u0cdgg1not8e3u8.png
  [101]: http://static.zybuluo.com/yao-yuan-ge/xvz2dobmpj7biuckjqqnpuer/image_1e6tv1pb1ger158q11dn1rgkbuful.png
  [102]: http://static.zybuluo.com/yao-yuan-ge/zcnntaofbl5o7w6xe0i60dgn/image_1e6tv203q1rc4vi113dkfgjisjv2.png
  [103]: http://static.zybuluo.com/yao-yuan-ge/uoa5h0r0xhpraegsinbywzzs/image_1e6tv2f7a1na31mvp7ib1glv1s36vf.png
  [104]: http://static.zybuluo.com/yao-yuan-ge/cjdknp0o8u8oagpj3xfeanjc/image_1e6tv3gbi1j2t14451bqvv4kbunvs.png
  [105]: http://static.zybuluo.com/yao-yuan-ge/5vqir51p1yomjifw0vvor7wt/image_1e6tv3tmj1b3oj1i167e9d7i22109.png
  [106]: http://static.zybuluo.com/yao-yuan-ge/r67n96qwmklcyjtplnoq37qi/image_1e6tv464jdmumla1014391iv10m.png
  [107]: http://static.zybuluo.com/yao-yuan-ge/tt01hpfs9xdc4if5i3cvan3l/image_1e6tv4c681uclj6k11hf1ne21g5j113.png
  [108]: http://static.zybuluo.com/yao-yuan-ge/m3jl5kxoaom8grnmul662ysj/image_1e6tv4i7niur18bjb5m9ku7ku11g.png
  [109]: http://static.zybuluo.com/yao-yuan-ge/jrmq56bkg7vmh340nn9y644w/image_1e6tv4r0r10uph841ovtvgvc3p11t.png
  [110]: http://static.zybuluo.com/yao-yuan-ge/fxqh93rjocfa9nvtpckgbqnp/image_1e6tv52pljge15lf1t8ok1ip7912a.png
  [111]: http://static.zybuluo.com/yao-yuan-ge/z2olkdldz3i44jxk80fu4wtj/image_1e6tv5ajve3v8a31b2kktcaqh12n.png
  [112]: http://static.zybuluo.com/yao-yuan-ge/t5l3hawlf7o8be8s5u3yppb6/image_1e6tv5gceak7pat1t9174111s8134.png
  [113]: http://static.zybuluo.com/yao-yuan-ge/9e1qrlzecfpwqp5v3edary9w/image_1e6tv5m701h9s11jd3c71nnti6s13h.png
  [114]: http://static.zybuluo.com/yao-yuan-ge/og2hn45h08d4j604g8als0cm/image_1e6tv5s9817h01dbaobqs2k17am13u.png
  [115]: http://static.zybuluo.com/yao-yuan-ge/qt6w6zujgflks5o0p3xc66xv/image_1e6tv64956qpfq3122g10u7bo214b.png
  [116]: http://static.zybuluo.com/yao-yuan-ge/t3jnx2tn7j0b90zhlh1gi2mq/image_1e6tv70v21lki17k81jv5p3hhu014o.png
  [117]: http://static.zybuluo.com/yao-yuan-ge/e9gpg0tw4l8g2vw50decccvk/image_1e6tv7a8usq7j6sjhv1rhmup7155.png
  [118]: http://static.zybuluo.com/yao-yuan-ge/tbgn1zg2bi4for7c4w2exm3w/image_1e6tv8dlit341kp71ipk26tt6u15i.png
  [119]: http://static.zybuluo.com/yao-yuan-ge/cefj3srop4kuvlxnj52pfhrc/image_1e6tv8jqu1iid10fn16eat4d1j7515v.png
  [120]: http://static.zybuluo.com/yao-yuan-ge/dkplq8q3m187vojpwa31mnox/image_1e6tv8pkgr7ean41ovf177bd0t16c.png
  [121]: http://static.zybuluo.com/yao-yuan-ge/9x2hklb8uahgkgu973ftvwu8/image_1e6tvamo2rijoum1jlrm1unfh16p.png
  [122]: http://static.zybuluo.com/yao-yuan-ge/oy9j7xpj0umrqnhk8c6y9ipt/image_1e6tvatfp9vf152p7pqrtssqv176.png
  [123]: http://static.zybuluo.com/yao-yuan-ge/9y1ezwk1cqqz34kqzceewt9n/image_1e6tvb5fm1j144ai5b66kopj17j.png
  [124]: http://static.zybuluo.com/yao-yuan-ge/qqq17mw60f4bnwwog85uxol0/image_1e6tvbk31en81j4c10m919oh1mdn180.png
  [125]: http://static.zybuluo.com/yao-yuan-ge/qxh35uc4me7g5zdkkpz6gsp9/image_1e6tvbrvb11uj9l22nq7gcukq18d.png
  [126]: http://static.zybuluo.com/yao-yuan-ge/1pa8zurxyt2lh6wdb2als7r8/image_1e6tvc3fq1o2n16nd17cd1cd24b218q.png
  [127]: http://static.zybuluo.com/yao-yuan-ge/2st5eniha8ec3m38gluwdo0j/image_1e6tvcaacfok14kqsfk1lpc1ole197.png
  [128]: http://static.zybuluo.com/yao-yuan-ge/0vqz3vs1c1905w2cqw7dksll/image_1e6tvcho8eq213cl1qm01iv1oa319k.png
  [129]: http://static.zybuluo.com/yao-yuan-ge/9forego69660dvs2tjurs3or/image_1e6tvcpvq1gqvqo1dcfp6apd1a1.png
  [130]: http://static.zybuluo.com/yao-yuan-ge/5t6oyuk1f0p96kj9q3e50nb2/image_1e6tvd0ge1rhd1ucm1ij71bapk2r1ae.png
  [131]: http://static.zybuluo.com/yao-yuan-ge/7z8v48ew4u6ft4bafhbcsb3s/image_1e6tvd81e162h3us8jgs3s139e1ar.png
  [132]: http://static.zybuluo.com/yao-yuan-ge/y88l9tjjoehx10kv5bo2l5so/image_1e6tvfprc1vo71jc6101a3856b91b8.png
  [133]: http://static.zybuluo.com/yao-yuan-ge/93342x9iadopdrr79w86lk45/image_1e6tvg3506tvisbvqr1tif1mb81bl.png
  [134]: http://static.zybuluo.com/yao-yuan-ge/lfdvzacv629uj520mnheyska/image_1e6tvg9uf1t0u119is8j1r6eugc1c2.png
  [135]: http://static.zybuluo.com/yao-yuan-ge/do7l1ohi4ouv1rwq78nopw8r/image_1e6tvgfio1o1k2pu1ijr611i8p1cf.png
  [136]: http://static.zybuluo.com/yao-yuan-ge/p8eotuh1sqmbrxoy78m9r8kk/image_1e6tvglkoldt1lvl9qam9m9031cs.png
  [137]: http://static.zybuluo.com/yao-yuan-ge/59zj0xwqp0zxl7fobhhvptpa/image_1e6tvgtekf93o691ost74415qm1d9.png
  [138]: http://static.zybuluo.com/yao-yuan-ge/0pcemci5hrt74i4zv6eug1lb/image_1e6tvhmtl10h617c5tr1grihuj1dm.png
  [139]: http://static.zybuluo.com/yao-yuan-ge/sdhcmhlzdj09kka9kqq1h9kj/image_1e6tvhtm81hbk1oaj3aps9eb91e3.png
  [140]: http://static.zybuluo.com/yao-yuan-ge/qkpekm2u691odlcg1a8evru4/image_1e6tvi58c1sf51u2g6qq187sohr1eg.png
  [141]: http://static.zybuluo.com/yao-yuan-ge/r3i6yoeojykq8v7arycsiyvi/image_1e6tvic2qbi418v1egc107u52j1et.png
  [142]: http://static.zybuluo.com/yao-yuan-ge/ct035bg5x2b969gwgf0pcbcx/image_1e6tvij4b1l6a1efr1hj31ih1n9f1fa.png
  [143]: http://static.zybuluo.com/yao-yuan-ge/q1o9z7f1f9upn7hrmxqm51ak/image_1e6tvipqt1lru4t11c3mok71dns1fn.png
  [144]: http://static.zybuluo.com/yao-yuan-ge/gumrqy3hxcegvlxdy5vq2ego/image_1e6tvivn11edl19k214b1art1dl1g4.png
  [145]: http://static.zybuluo.com/yao-yuan-ge/7y0973sd3xv80b66qb9psmv0/image_1e6tvj5vj17i274aamm1sud8ic1gh.png
  [146]: http://static.zybuluo.com/yao-yuan-ge/6ynyt6zvwrhygjy5wxlstggu/image_1e6tvjdhnhf010r49kudjoghf1gu.png
  [147]: http://static.zybuluo.com/yao-yuan-ge/m5ugurjxt2apx68v0mgsl3xa/image_1e6tvjjjojhu1808jr4187u1vb81hb.png
  [148]: http://static.zybuluo.com/yao-yuan-ge/0az0midk7so9e8oavepc00j8/image_1e6tvjpgt1jts1jjlcuq73po221ho.png
  [149]: http://static.zybuluo.com/yao-yuan-ge/egw6grrmep3s323140horaxa/image_1e6tvjvca5uktsv1pmv1597hlk1i5.png
  [150]: http://static.zybuluo.com/yao-yuan-ge/h6vt14iarhb2n7y2d3f4neiy/image_1e6tvk57chljn081jn246rm851ii.png
  [151]: http://static.zybuluo.com/yao-yuan-ge/ycrlxnq4hp4tv0j0uh6ynoe1/image_1e6tvkb9m1f172rec88h4kgs21iv.png
  [152]: http://static.zybuluo.com/yao-yuan-ge/lk0ulbgyz040y87mmw499dlb/image_1e6tvkhg9iiehc42p955u1o5l1jc.png
  [153]: http://static.zybuluo.com/yao-yuan-ge/xvbnu40xio873t1dw9ksqzps/image_1e6tvknd3aao2m4gso17i5e5c1jp.png
  [154]: http://static.zybuluo.com/yao-yuan-ge/w7ic60u7yiz90l0rxbhomhsn/image_1e6tvkt3h101p1k1pu1i12m417tm1k6.png
  [155]: http://static.zybuluo.com/yao-yuan-ge/wjk6fk65ok4k4decahqge4gf/image_1e6tvl7qaa5g1kbg1rtuttg1jsk1kj.png
  [156]: http://static.zybuluo.com/yao-yuan-ge/u60jed0fpwis75mmyeskxkn3/image_1e6tvlecpgcmp00k3o1hc0ech1l0.png
  [157]: http://static.zybuluo.com/yao-yuan-ge/gfr3k5bjmj26cbjnui5918sj/image_1e6tvlkm7q1arg11rc3h7q11ph1ld.png
  [158]: http://static.zybuluo.com/yao-yuan-ge/cn551si7i0hfpm5zitx7qiz2/image_1e6tvlsna1v88m8gc7bnrq1die1lq.png
  [159]: http://static.zybuluo.com/yao-yuan-ge/zzr61063i0kv0ldzfued4em0/image_1e6tvm3rg1sj6nc818ov1sbq14dp1m7.png
  [160]: http://static.zybuluo.com/yao-yuan-ge/snutj0lcnon4dw0p5iz1qzyr/image_1e6tvma4m4h6101nr8d1kq98uc1mk.png
  [161]: http://static.zybuluo.com/yao-yuan-ge/d7701sk9ho8a3wf9jufvrzbb/image_1e6tvmgcvt101opi1cer1ko8l6c1n1.png
  [162]: http://static.zybuluo.com/yao-yuan-ge/izz5xzmj6nt0uqr71azi9s5l/image_1e6tvmmr21mk1s1h1oqr6ss1gsl1ne.png
  [163]: http://static.zybuluo.com/yao-yuan-ge/qnriln2szy9y9bfwmfvvi44m/image_1e6tvmtob13639h61rod15k1ucv1nr.png
  [164]: http://static.zybuluo.com/yao-yuan-ge/toryrswwhqomfxlnq6b3fdjo/image_1e6tvn5ol1ov0skau3b7rt1lmm1o8.png
  [165]: http://static.zybuluo.com/yao-yuan-ge/drcsiz5dwa8fs41opqr03bkm/image_1e6tvncem1mrj6sv155h160lfve1ol.png
  [166]: http://static.zybuluo.com/yao-yuan-ge/mnej53f9t3wq7dh7r8we5aqq/image_1e6tvnjgt19beuunbf5u3n181f1p2.png
  [167]: http://static.zybuluo.com/yao-yuan-ge/np6fzfc89xl9rjwy5ylu81of/image_1e6tvnq07mg1b3r1lst17th11ed1pf.png
  [168]: http://static.zybuluo.com/yao-yuan-ge/4gezkf09rxhqw9hf2y8rjcb2/image_1e6tvo03g51g1pcj1n5m7ua8c51ps.png
  [169]: http://static.zybuluo.com/yao-yuan-ge/9503dpqh0syu5wmhk26i5bkk/image_1e6tvo5o514159c61qps11gp1uv21q9.png
  [170]: http://static.zybuluo.com/yao-yuan-ge/htn1x4zm276as7ybqv66yrx7/image_1e6tvob9u39p10j7v2jgc91s01qm.png
  [171]: http://static.zybuluo.com/yao-yuan-ge/fsocimephs5e4om5hzto9dpv/image_1e6tvoinp1d8tb3e18s51suc1v8h1r3.png
  [172]: http://static.zybuluo.com/yao-yuan-ge/z8378bwy99op7gcvry0nbacl/image_1e6tvoohl134h9fj1polmf4dsa1rg.png
  [173]: http://static.zybuluo.com/yao-yuan-ge/w6mal5z1lk2vfpmtde3hgixi/image_1e6tvplc79fgbb71dlhe7p1gq01rt.png
  [174]: http://static.zybuluo.com/yao-yuan-ge/z98dk5zljs4w94opz31oqkhi/image_1e6tvpsccdr21fu1104o1ghp5f1sa.png
  [175]: http://static.zybuluo.com/yao-yuan-ge/noy7gphie89yogik5tt1e53g/image_1e6tvq1j29i71pod9kb1kaaumk1sn.png
  [176]: http://static.zybuluo.com/yao-yuan-ge/tcfhu6xu2vneixuleside8gx/image_1e6tvq6n01b9q17mb1r711g6b16j51t4.png
  [177]: http://static.zybuluo.com/yao-yuan-ge/qpv1x9sev53apf13ggxde9gv/image_1e6tvqf771nkv1b78fqt1ls71net1th.png