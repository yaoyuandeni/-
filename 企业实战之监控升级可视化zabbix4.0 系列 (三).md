# 企业实战之监控升级可视化zabbix4.0 系列 (三)

标签（空格分隔）： zabbix4.0系列

---Mr.Wang

[TOC]

https://www.zybuluo.com/commandersu/note/1670447
##前言
**下面讲用户管理及告警实战**

##一、实验环境
###zabbix-server端
|操作系统|主机名|IP|内存|
|-|-|-|
|centos7.5|zabbix|192.168.200.173|1G|
###zabbix-agent端
|操作系统|被监控的主机名|IP地址|运行的项目|内存|
|-|-|-|
|centos6.5|www_001|192.168.200.128|nginx|1G|
|centos7.5|www_002|192.168.200.131|nginx|1G|

##二，用户管理
###zabbix创建用户
###2.1创建用户组 dev
>设置用户组权限，权限必须在用户组设置

![image_1e488hsj81lts13d8jvoks9s6u9.png-109.5kB][1]

![image_1e488i94n1rmpq3va5mnnj140bm.png-110.1kB][2]

![image_1e488ij5f1mfueh43rh164d7ml13.png-106.4kB][3]

![image_1e488itek8dea6fsl39di1vg51g.png-111.4kB][4]

###2.2创建用户dev_suluo，属于用户组dev

![image_1e488km4817eb13po1bp15r41b761t.png-120.6kB][5]

![image_1e488l2r01da11v8l1tag1l3d17m02a.png-111.3kB][6]

![image_1e488lc681auq1iec1sqsdrqtb02n.png-125.7kB][7]

![image_1e488m0do3vd4iq10oea8ga7e34.png-110.3kB][8]

###2.3使用新建的用户登录Zabbix

![image_1e488n09o10te1bgrlej27uq613h.png-91.9kB][9]

![image_1e488nc6f1m4n1af683a6291d5n3u.png-139.2kB][10]

![image_1e488npdf1pqh1l39109e15l1a964b.png-115.1kB][11]

![image_1e488oa9a1o4d148pr459oh13994o.png-140.5kB][12]

![image_1e488oivd1ovd44bn3f5hv1ojv55.png-95.3kB][13]

![image_1e488oupkac8dcl1jri1916quk5i.png-104.2kB][14]

##三、邮件告警
###3.1实验环境准备
```
#在zabbix服务器先把映射文件发送到我们需要监控的服务器上
[root@zabbix ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.200.173 zabbix
192.168.200.128 www_001
192.168.200.131 www_002
[root@zabbix ~]# scp /etc/hosts 192.168.200.128:/etc/
The authenticity of host '192.168.200.128 (192.168.200.128)' can't be established.
RSA key fingerprint is SHA256:LWinuYuB8lkixPIA7KfaMNL5vA+hQ3LG7e6Hv2U/6r0.
RSA key fingerprint is MD5:60:1e:00:37:85:8b:a1:74:68:b3:35:3b:eb:20:33:3a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.200.128' (RSA) to the list of known hosts.
root@192.168.200.128's password: 
hosts                                                                    100%  229   218.6KB/s   00:00 
[root@zabbix ~]# scp /etc/hosts 192.168.200.131:/etc/
The authenticity of host '192.168.200.131 (192.168.200.131)' can't be established.
ECDSA key fingerprint is SHA256:46jNFg96R/vgl81faS4UuFgiAaeYj2mC+OtQtVfR1I8.
ECDSA key fingerprint is MD5:32:70:a3:6c:1f:50:33:7f:b6:9a:e4:c5:4b:ea:d9:99.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.200.131' (ECDSA) to the list of known hosts.
root@192.168.200.131's password: 
hosts                                                                    100%  229    11.3KB/s   00:00
```

```
#在www_001这台服务器上修改主机名
[root@192 ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@192 ~]# hostname -I
192.168.200.128 
[root@192 ~]# vim /etc/sysconfig/network
[root@192 ~]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=www_001
[root@192 ~]# hostname www_001
[root@192 ~]# exit
[root@www_001 ~]# 
#在www_002这台服务器上修改主机名
[root@192 ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
[root@192 ~]# hostname -I
192.168.200.131 
[root@192 ~]# vim /etc/hostname 
[root@192 ~]# cat /etc/hostname 
www_002
[root@192 ~]# reboot
[root@www_002 ~]# 
```

###3.2在agent端www_001服务器建立被监控的nginx服务
```
[root@www_001 ~]# ls 
anaconda-ks.cfg  install.log  install.log.syslog  nginx-1.16.1.tar.gz
[root@www_001 ~]# mount /dev/sr0 /media/cdrom/
mount: block device /dev/sr0 is write-protected, mounting read-only
[root@www_001 ~]# yum install -y wget gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel  
[root@www_001 ~]# tar xf nginx-1.16.1.tar.gz -C /usr/src/
[root@www_001 ~]# cd /usr/src/nginx-1.16.1/
[root@www_001 nginx-1.16.1]# ./configure --prefix=/usr/local/nginx && make && make install
[root@www_001 nginx-1.16.1]# ln -s /usr/local/nginx/sbin/* /usr/bin/
[root@www_001 nginx-1.16.1]# cd /usr/local/nginx/conf/
[root@www_001 conf]# egrep -v "#|^$" nginx.conf.default > nginx.conf
[root@www_001 conf]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@www_001 conf]# nginx 
[root@www_001 conf]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4006,6),("nginx",4007,6))
```

###3.3在agent端www_002服务器建立被监控的nginx服务
```
[root@www_002 ~]# ls 
anaconda-ks.cfg  nginx-1.16.1.tar.gz
[root@www_002 ~]# yum install -y wget gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel
[root@www_002 ~]# tar xf nginx-1.16.1.tar.gz -C /usr/src/
[root@www_002 ~]# cd /usr/src/nginx-1.16.1/
[root@www_002 nginx-1.16.1]# ./configure --prefix=/usr/local/nginx && make && make install
[root@www_002 nginx-1.16.1]# ln -s /usr/local/nginx/sbin/* /usr/bin/
[root@www_002 nginx-1.16.1]# cd /usr/local/nginx/conf/
[root@www_002 conf]# egrep -v "#|^$" nginx.conf.default > nginx.conf
[root@www_002 conf]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@www_002 conf]# nginx 
[root@www_002 conf]# ss -antup | grep 80
tcp    LISTEN     0      128       *:80                    *:*                   users:(("nginx",pid=3867,fd=6),("nginx",pid=3866,fd=6))
#使用systemctl管理起来
[root@www_002 conf]# cat /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx 
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
[Install]
WantedBy=multi-user.target
#如果这里有问题请重启下服务器
[root@www_002 ~]# systemctl stop nginx
[root@www_002 ~]# ss -antup
Netid State      Recv-Q Send-Q      Local Address:Port                     Peer Address:Port              
udp   UNCONN     0      0                       *:68                                  *:*                   users:(("dhclient",pid=832,fd=6))
tcp   LISTEN     0      128                     *:22                                  *:*                   users:(("sshd",pid=889,fd=3))
tcp   LISTEN     0      100             127.0.0.1:25                                  *:*                   users:(("master",pid=1043,fd=13))
tcp   ESTAB      0      52        192.168.200.131:22                      192.168.200.1:52460               users:(("sshd",pid=969,fd=3))
tcp   LISTEN     0      128                    :::22                                 :::*                   users:(("sshd",pid=889,fd=4))
tcp   LISTEN     0      100                   ::1:25                                 :::*                   users:(("master",pid=1043,fd=14))
[root@www_002 ~]# ss -antup | grep 80
tcp    LISTEN     0      128       *:80                    *:*                   users:(("nginx",pid=1201,fd=6),("nginx",pid=1200,fd=6))
```

##四、zabbix告警原理
>触发器触发后，可以配置对应的动作 
在动作里可以发邮件、微信、钉钉、短信等

###4.1查看告警脚本的放置位置
```
#在zabbixserver端
[root@zabbix ~]# cat /usr/local/zabbix/etc/zabbix_server.conf
LogFile=/usr/local/zabbix/zabbix_server.log
DBHost=127.0.0.1
DBName=zabbix
DBUser=zabbix
DBPassword=zabbixpwd
DBPort=3306
Timeout=30
AlertScriptsPath=/usr/local/zabbix/alertscripts   #邮件告警和微信告警的脚本放置位置
ExternalScripts=/usr/local/zabbix/externalscripts
LogSlowQueries=3000
[root@zabbix ~]# ll /usr/local/zabbix/alertscripts #发现没有需要我们自己去创建一个
ls: cannot access /usr/local/zabbix/alertscripts: No such file or directory
[root@zabbix ~]# mkdir -p /usr/local/zabbix/alertscripts
[root@zabbix ~]# cd /usr/local/zabbix/alertscripts/
#这里特别注意zabbix_server配置文件里不能出现中文，如果有中文在做邮件报警的时候回报错，请删掉并且重启zabbix_server。如果还是不行，就直接重启服务器。然后把各个服务重启一遍
命令：
[root@zabbix ~]#cp /usr/src/zabbix-4.0.3/misc/init.d/fedora/core/zabbix_server /etc/init.d/
[root@zabbix ~]# /etc/init.d/zabbix_server restart
```
>附录1：邮件告警注意事项 
服务器往外的25端口一般被云厂商禁止 
所以建议使用465端口发送邮件 
qq邮箱、163邮箱需要配置开启smtp

###4.2生产邮箱授权码

![image_1e4893l4v12dd1m9o4iu1l171luf5v.png-164.5kB][15]

![image_1e48947c11l6f108lrsi1vsu46c.png-212.5kB][16]

![image_1e4894u3s1qr01jcu58ju5sjkk79.png-195.8kB][17]

![image_1e4895coh66418vs1as5115h169t7m.png-217.6kB][18]

>附录2：qq授权码

**·** [x]pjtryeiuazqdcbbe

###4.3创建邮件告警脚本，并测试
```
[root@zabbix alertscripts]# cat /usr/local/zabbix/alertscripts/zabbix_sendmail.py
#!/usr/bin/python
# -*- coding: utf-8 -*-             
from email.mime.text import MIMEText   #python 加载的包
from email.header import Header     #python 加载的包
from smtplib import SMTP_SSL        #python 加载的包
import sys
smtpaddr = 'smtp.qq.com'      #表示使用qq邮箱，如果你要使用163邮箱，就写成smtp.163.com
myemail='493115250@qq.com'     #你的邮箱地址
password='pjtryeiuazqdcbbe'    #这里写你的授权码，每次都是不一样的，不要照抄
recvmail=sys.argv[1]        #对应邮件的发送的主题
subject=sys.argv[2]         #对应邮件的内容
content=sys.argv[3]         #对应邮件的发给谁
msg = MIMEText("""%s"""%(content), "plain", "utf-8")  #（content 里面是内容）
msg['Subject'] = Header(subject, 'utf-8').encode()   #（这里是主题）
msg['From'] = myemail       #调用上边的谁发的
msg['To'] =  recvmail       #调用上边的参数发给谁
try:
  smtp = SMTP_SSL( smtpaddr )  #使用的协议，写着几行的时候一定要前面空格，要不报错
  smtp.login(myemail, password) #调用上边的账号和密码
  smtp.sendmail(myemail, recvmail.split(','), msg.as_string()) #myemail是我的邮箱，recvmailsplit(',')是发送给谁，多个人用逗号隔开，一定要英文的，msg.as_string是发送的内容
  smtp.quit()
  print("success") #成功就发成功
except Exception as e:
  print("fail: "+str(e))  #失败就发失败
#注意复制粘贴以后吧注释去掉

#验证脚本是否成功
[root@zabbix alertscripts]# chmod +x zabbix_sendmail.py 
[root@zabbix alertscripts]# /usr/local/zabbix/alertscripts/zabbix_sendmail.py 1653605943@qq.com 'zabbix disk' 'content: disk > 90%'
success
注意格式：
/usr/local/zabbix/alertscripts/zabbix_sendmail.py 邮箱地址 '邮件名字' '邮件内容'
```

###4.4查看邮箱是否收到邮件

![image_1e489ap4511h526pi15pm213d383.png-163.4kB][19]

###4.5报警媒介添加邮件告警

![image_1e489c20dmhcij51bhkr9s1jh28g.png-111.8kB][20]

![image_1e489cdoi1j94iqb55ase2ebp8t.png-123.4kB][21]

![image_1e489cmvtfgoo0m197m11kh19df9a.png-108.1kB][22]

>附录3:三个参数
* 发送给谁{ALERT.SENDTO}
* 发送的主题{ALERT.SUBJECT}
* 发送内容{ALERT.MESSAGE}

###4.6用户设置报警媒介

![image_1e489sl2a6dh17mrscp1kbhihs9n.png-108kB][23]

![image_1e489t0mi8er16a4v481n4p23ea4.png-135.3kB][24]

![image_1e489teu7u0ulqr13151d2r1jglb1.png-109.9kB][25]

###4.7修改模板的监控项

![image_1e489u4bb1g0f1f3vcul1vef1vd1be.png-93.5kB][26]

![image_1e489uep71qpg15omtkm8g5766br.png-120.7kB][27]

![image_1e489un642am1ir01r1u1t851pp3c8.png-129.3kB][28]

![image_1e489v09b1ka51karloq1gm419krcl.png-129.1kB][29]

###4.8定义模板触发器

![image_1e489vn101735d8ucfb1gsmoktd2.png-93.5kB][30]

![image_1e489vv3t1kk6g2sjv01qj21nk2df.png-118.1kB][31]

![image_1e48a083f1j811mq8pu4qkbb5kds.png-131.7kB][32]

![image_1e48a0h3ie75117vdd61n2p4o0e9.png-107.7kB][33]

![image_1e48a0p791soq1nou171ijabad0em.png-110.3kB][34]

###4.9触发器关联动作

![image_1e48a2fmksdt151417m2r0t13h2f3.png-95.3kB][35]

![image_1e48a2ngrqae16lerha1v4p1h3pfg.png-103.2kB][36]

![image_1e48a32sd195o8qjkr915mlqn5ft.png-96.6kB][37]

![image_1e48a3bbq151u3425552n1t8fga.png-350.7kB][38]

![image_1e48a3jd5p5lji21i0g1cmkj4ggn.png-136.8kB][39]

![image_1e48a3ru4ao2esan916kf178hh4.png-139.2kB][40]

![image_1e48a442i1ste1gd91tgl1tlrqgchh.png-120.5kB][41]

![image_1e48a4aa657qs1g186i1qs6ii0hu.png-144.3kB][42]

![image_1e48a4i5ec80hem14re1hn741vib.png-103.7kB][43]

![image_1e48a4rnkfh56n1qhka1b8dhio.png-97kB][44]

###4.10验证发送邮件

![image_1e48a5h0no271pt41gb28um7p7j5.png-119.8kB][45]

```
我们让www_001服务器的nginx挂掉
[root@www_001 conf]# nginx -s stop
```

![image_1e48a6mvf1ilpdvaeii12kt1kvfji.png-170.8kB][46]

![image_1e48a79kl1je91kvc5jjkdospsjv.png-136kB][47]

###4.11将001服务器回复我们看一下
```
[root@www_001 ~]# nginx 
[root@www_001 ~]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4395,6),("nginx",4396,6))
```
![image_1e48a8e5110d91rum1v0c7ch17t5kc.png-127.2kB][48]

![image_1e48a8qvf10061ho71m57fngb99kp.png-185.1kB][49]

###4.12告警内容自定义以及告警抑制
>一般来说我们自定义主题，里边带上主机名，以及主机ip就可以了，这样我们只看主题就可以很明确的看出是哪个服务器出问题了

* 主机名变量：{HOST.NAME1}
* 主机IP变量：{HOST.IP1}

![image_1e48ab1uc1ibv1e5i11iogod1i7pl6.png-90.6kB][50]

![image_1e48abdqbojk82bvag1ga2e6llj.png-109.9kB][51]

![image_1e48abm581ao11qo51jsu1s7p84am0.png-110.7kB][52]

![image_1e48abununs7dmc1d2tgrn1e60md.png-91.2kB][53]

###4.13发送邮件来验证自定义告警主题内容
```
让www_001nginx挂掉
[root@www_001 ~]# nginx -s stop
[root@www_001 ~]# ss -antup | grep 80
```

![image_1e48ad77n1ui419vp1ji51t29r4bmq.png-145.5kB][54]

![image_1e48adhgd8gt1ltcto936126qn7.png-166kB][55]

###4.14将www_001主机的nginx回复我们看下邮件
```
[root@www_001 ~]# nginx 
[root@www_001 ~]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4454,6),("nginx",4455,6))
```

![image_1e48aeoei10m9in0141qbvjd0onk.png-117kB][56]

![image_1e48af3c517ks79kdj97p76tqo1.png-163kB][57]

###4.15告警抑制
>为什么要用告警抑制 
像我们监控80端口这样的监控项，有的时候会因为网络波动获取不到数据，这个时候如果直接报警，我们设定通知100个人，那么100个人会收到邮件，下次网络恢复了，会在发邮件，这样的话，运维的垃圾邮件太多。所以我们要使用告警抑制，如果连续2分钟或者5分钟检查不到80端口，这个时候我们在发送邮件

![image_1e48ag1l6ell1j3toia1c0d158eoe.png-89.1kB][58]

![image_1e48aga9915c3hk0t12hg0uj0or.png-103.9kB][59]

![image_1e48agij6r5f1c2ecrah0d1mdqp8.png-137.4kB][60]

![image_1e48agqhicln6mb4pe1m921p2fpl.png-111.5kB][61]

![image_1e48ah2tj8oa1k95ivnptb1a6dq2.png-122.5kB][62]

![image_1e48ahce31dh11b3i1fc41v4m1o5qf.png-104.1kB][63]

###4.16验证告警抑制

**模拟网络波动**
```
[root@www_001 ~]# nginx -s stop
[root@www_001 ~]# ss -antup | grep 80
```

![image_1e48aivsjnio1f0nv23c7n1dihqs.png-132kB][64]

```
[root@www_001 ~]# nginx 
[root@www_001 ~]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4505,6),("nginx",4506,6))
```

![image_1e48ajv9m1ig8uf1722162i1po1r9.png-121.4kB][65]

![image_1e48ak7sj1h8t1kqrbu21is12nvrm.png-165kB][66]

###4.17我们来测试抑制两分钟后，再发送邮件
```
[root@www_001 ~]# nginx -s stop
[root@www_001 ~]# ss -antup | grep 80
```

![image_1e48alatc9lo401c8k1mbci2ms3.png-133.7kB][67]

![image_1e48alj2vlg21cln1r2aer47fqsg.png-149.1kB][68]

![image_1e48alrffolj14dj1rjun571eqvst.png-158.6kB][69]

**我们恢复nginx**
```
[root@www_001 ~]# nginx 
[root@www_001 ~]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4548,6),("nginx",4549,6))
```

![image_1e48anbn6lt9gaua7gi8s7abta.png-125.5kB][70]

![image_1e48ao0qv1ovgp0ljmqsg1bt8tn.png-228.3kB][71]

>附录4：
在线上一般来说数据库等压力比较大，我们会一分钟探测一次，连续3次没有连接才会发邮件，也就是说我们需要在次数哪里设置成4次。

##五、微信告警
###5.1注册企业微信
```
企业微信注册网址: https://work.weixin.qq.com/
```

![image_1e48arohq13v8kna1h121ruo16piu4.png-700.8kB][72]

**自己按着步骤去注册一个** 

![image_1e48askj91e46tg21egd1pjq1ng8uh.png-116.8kB][73]

![image_1e48assmm18e60a7p1g4rbuvuu.png-120.3kB][74]

**不用下载企业微信就可以接受消息操作**

![image_1e48atneo1vub1chvnev106s182qvb.png-123.9kB][75]

![image_1e48au1b9u9h1dj01851e2i1poavo.png-128.7kB][76]

**获取zabbix给微信发消息的账号**

![image_1e48aurm6im05fj1t7q1qv21c7e105.png-120.7kB][77]

![image_1e48av5n3c2k1ovd4cf1p161dsl10i.png-135.1kB][78]

**创建应用，并且选择给谁发微信** 

![image_1e48avrbfdbp6qh1sc31phqkeb10v.png-103kB][79]

![image_1e48b03k21b9k70fm0p1e3l17nf11c.png-107.2kB][80]

![image_1e48b0e8tqt8fghrnbgjqkau11p.png-109.6kB][81]

![image_1e48b0mhm19dtjkr1751q6d11sq126.png-120kB][82]

**记录我们重要的三个参数**

![image_1e48b1jvf1md4naf1reh1df6155f12j.png-140.4kB][83]

![image_1e48b1smm128beb8g344m3ss0130.png-127.3kB][84]

###5.2编写发送给微信的脚本
>附录5：
**·** agentid = '1000002' #这里是你的应用ID
**·** corpid = 'wweb7edbef7979a552' #这里是你的企业ID
**·** corpsecret = 'XZdSEpQJxL3qEojydpGd3CymHx3te4kryZzkw5e8kYM' #这里是你的应用ID的密码
**·** touser=sys.argv1 #many user: 'zhangsan|wangwu' 这个是发送给谁，我们这里使用参数，如果是多用户可以使用 | 去分割
**·** content=sys.argv2 #content 发送的内容 
下边的脚本可以直接粘贴使用，但是需要改下应用ID，企业ID,应用密码，后边的都不用改，其余的直接复制，脚本里不可以出现中文。

```
[root@zabbix ~]# cat /usr/local/zabbix/alertscripts/zabbix_wx.py
#!/usr/bin/python
# -*- coding: utf-8 -*-
import json
import sys
import urllib,urllib2
agentid = '1000002'
corpid = 'wweb7edbef7979a552'
corpsecret = 'XZdSEpQJxL3qEojydpGd3CymHx3te4kryZzkw5e8kYM'
#get tocken
gettoken_url = 'https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=' + corpid + '&corpsecret=' + corpsecret
token_file = urllib2.urlopen(gettoken_url)
token_data = token_file.read().decode('utf-8')
token_json = json.loads(token_data)
my_token = token_json['access_token']
#send wechart 
touser=sys.argv[1]  #many user: 'zhangsan|wangwu' 
content=sys.argv[2] #content  
post_content = {
        "touser":touser,
        "agentid":agentid,
        "msgtype": "text",
        "text":{
                "content":content,
        }
}
json_content = json.dumps(post_content)
url = 'https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=' + my_token
response = urllib2.urlopen(url,json_content)
print(response.read().decode('utf-8'))
[root@zabbix ~]# chmod +x /usr/local/zabbix/alertscripts/zabbix_wx.py
[root@zabbix ~]# /usr/local/zabbix/alertscripts/zabbix_wx.py 'Suluo' 'disk >90'
{"errcode":0,"errmsg":"ok","invaliduser":""}
#'Suluo' 是企业微信里的成员账号，后边的是''里边是发送的内容
```

![image_1e48b7hqi1edoq962uq1k4p1gbe13d.png-164.8kB][85]

###5.3在web界面设置媒介
![image_1e48h89qasap4547d7r2l68s13q.png-111.9kB][86]

![image_1e48h8kod11u7ip413a1134q1b87147.png-103.9kB][87]

![image_1e48h8u1s16at1lmd1ilk1kmu11p814k.png-111kB][88]

![image_1e48h984o1r3idpve0v19k41ukl151.png-105.1kB][89]

###5.4定义给那个用户

![image_1e48h9urfmj11bnafb8i9tdor15e.png-101.9kB][90]

![image_1e48ha7aen2tf2kgvf16qajna15r.png-104.8kB][91]

![image_1e48hag8e1k9e1jiq1hl51u0o19r3168.png-132.7kB][92]

![image_1e48haoqg1m921klc40i3di14n716l.png-105.3kB][93]

![image_1e48hb2q4essv9m1q071f214ni172.png-142.1kB][94]

###5.5定义动作

![image_1e48hbkek1ir2u141tepo5pq9017f.png-95.7kB][95]

![image_1e48hbscn1o6712cj1krfmcb1df517s.png-97.3kB][96]

![image_1e48hc4c5tit158c1eh612akstr189.png-93.6kB][97]

![image_1e48hcbtal717e2cp41md7112118m.png-124.8kB][98]

![image_1e48hcl7dr2g1rl81o391apj1gl193.png-125.8kB][99]

![image_1e48hcu3p1v6hbue90iu733bm19g.png-115.1kB][100]

![image_1e48hd5jnh6g1di1rq9fvgdtu19t.png-113.3kB][101]

![image_1e48hdf2l2me5361amcttiibo1aa.png-113.8kB][102]

###5.6测试微信发送告警
```
[root@www_001 ~]# nginx -s stop
[root@www_001 ~]# ss -antup | grep 80
```

![image_1e48hejeq8r22i42nkrtb1g3s1an.png-160.6kB][103]

![image_1e48hf4d9s1jepn16jvsq91s2c1b4.png-180.7kB][104]

```
[root@www_001 ~]# nginx 
[root@www_001 ~]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4713,6),("nginx",4714,6))
```

![image_1e48hg7oj1dos1aj31ganklf1n6g1bh.png-130.4kB][105]

![image_1e48hgilb1ef6e7cc4duhv1jjd1bu.png-243.5kB][106]

##六、钉钉告警
###6.1注册一个企业钉钉
>注册地址 
https://oa.dingtalk.com/ 
根据提示自己注册一个 

![image_1e48hi2m9k851dahqs52bmsi81cb.png-170.2kB][107]

![image_1e48hic5e1uvgcbe1n81143kj8j1co.png-144.3kB][108]

![image_1e48hik4o14ua1i6g163m1vfi1eiq1d5.png-155.8kB][109]

![image_1e48hispb1lao9o212ce1udu1bv91di.png-125.6kB][110]

![image_1e48hj4h74a8nd369p11i1v6t1dv.png-125.4kB][111]

![image_1e48hjeqkv5g2rar85m91j2e1ec.png-155.3kB][112]

![image_1e48hjn5d6d01qa214umigv1gjb1ep.png-337.1kB][113]

![image_1e48hjursgjh18b42dd18ls17ev1f6.png-119.8kB][114]

![image_1e48hk6j41a77e7l19dueqegsa1fj.png-149kB][115]

![image_1e48hkgep1b951ot8e201qh11tgk1g0.png-180.9kB][116]

###6.2编写发送给钉钉的脚本
```
[root@zabbix ~]# cat /usr/local/zabbix/alertscripts/zabbix_dd.py
#!/usr/bin/python
# -*- coding: utf-8 -*-
#curl 'https://oapi.dingtalk.com/gettoken?corpid=xxx&corpsecret=xxx'
import json,urllib2,sys
appkey = 'ding6wxsawxeygipfztg'    #这里发现跟微信的脚本的用法一样，同样的需要这三个重要的信息
appsecret = 'Xm3N2C4L05omoBogh8_qKhj5f4IB0jtH6ely0Yw1oQbKJPUjl0nyMrSi234BSk8x'
agentid = '583990691' 
touser = sys.argv[1]
content = sys.argv[2]
tockenurl = 'https://oapi.dingtalk.com/gettoken?corpid=' + appkey + "&corpsecret=" + appsecret
tockenresponse = urllib2.urlopen(tockenurl)
tockenresult = json.loads(tockenresponse.read().decode('utf-8'))
tocken =  tockenresult['access_token']
sendurl = 'https://oapi.dingtalk.com/message/send?access_token=' + tocken
headers = {
    'Content-Type':'application/json'
}
main_content = {
    "touser": touser,
    "toparty": "",
    "agentid": agentid,
    "msgtype": "text",
    "text": {
        "content": content
    }
}
main_content = json.dumps(main_content)
req = urllib2.Request(sendurl,headers=headers)
response = urllib2.urlopen(req, main_content.encode('utf8'))
print(response.read().decode('utf-8')
##注意脚本里不要出现中文，还有这个重要的信息有可能会变，大家看好自己的在去改下。
```

![image_1e48hmbfr1pn41o8i9tf98ucef1gd.png-163.7kB][117]

###6.3测试钉钉脚本
```
[root@zabbix ~]# chmod +x /usr/local/zabbix/alertscripts/zabbix_dd.py
[root@zabbix ~]# /usr/local/zabbix/alertscripts/zabbix_dd.py manager6580 'disk > 90'
{"errcode":0,"errmsg":"ok","messageId":"578997677db63d4bb6df100dc61c1c0b","invalidparty":"","invaliduser":""}
```
###6.4钉钉接受到了消息

![image_1e48hohoc29rkk9dem1f58lbd1ha.png-22kB][118]

###6.5在web界面定义媒介

![image_1e48hpht8up8erl1sklk6f15de1hn.png-121.4kB][119]

![image_1e48hpqhv4qq1spc1k571molo6r1i4.png-105.4kB][120]

![image_1e48hq3fu1vh11a6i10kj1gg9g1n1ih.png-112.7kB][121]

![image_1e48hqc411sh61htq1s49jhu1l6h1iu.png-124.3kB][122]

###6.6定义给那个用户

![image_1e48hqura11kf67v174n16e2i3g1jb.png-117.7kB][123]

![image_1e48hr9vf6381e467f4i814ec1jo.png-110.1kB][124]

![image_1e48hripq5r816lf17gftbd1d5v1k5.png-130.7kB][125]

![image_1e48hrrgj14ge1kt1oualo51ncp1ki.png-108kB][126]

![image_1e48hs37g15fgnqn1ql1dum1gk01kv.png-117.8kB][127]

###6.7定义动作

![image_1e48hsm2t1iuem8c1gpq1lra1tf1lc.png-116.1kB][128]

![image_1e48hsti118kh1vcu188auk01jqd1lp.png-100kB][129]

![image_1e48ht6m61j3ivcmclj1nbm1g8j1m6.png-107.9kB][130]

![image_1e48htfqj3kr1o274nk6kc1dau1mj.png-121.5kB][131]

![image_1e48htpi31hlptea3ro15foh6h1n0.png-124.6kB][132]

![image_1e48hu23a1s941i6o1pck16q6oda1nd.png-114.8kB][133]

![image_1e48hub0ll6l1dp3186d17n5loh1nq.png-125.7kB][134]

###6.8测试钉钉
```
[root@www_001 ~]# nginx -s stop
[root@www_001 ~]# ss -antup | grep 80
```

![image_1e48hvjlcfe5u2122h1e5vh61o7.png-142.2kB][135]

![image_1e48hvufa1lll7gefs3qnte1f1ok.png-28.7kB][136]

**恢复www_001**
```
[root@www_001 ~]# nginx 
[root@www_001 ~]# ss -antup | grep 80
tcp    LISTEN     0      128                    *:80                    *:*      users:(("nginx",4849,6),("nginx",4850,6))
```

![image_1e48i1htg47v7ms1dd219k61afq1p1.png-125.8kB][137]

![image_1e48i1srn1mu1nqeib71sdgl1u1pe.png-39.3kB][138]


  [1]: http://static.zybuluo.com/yao-yuan-ge/eubja6t7jfgxxfsypqll59wk/image_1e488hsj81lts13d8jvoks9s6u9.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/h83p7d1i57pcyypxheo6c6x5/image_1e488i94n1rmpq3va5mnnj140bm.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/a6sw9odbop15iu8qvlqi18fs/image_1e488ij5f1mfueh43rh164d7ml13.png
  [4]: http://static.zybuluo.com/yao-yuan-ge/e5xv8mzygzs791gwx3i8mwjy/image_1e488itek8dea6fsl39di1vg51g.png
  [5]: http://static.zybuluo.com/yao-yuan-ge/gfvueqsknig5frz7pcqaw24k/image_1e488km4817eb13po1bp15r41b761t.png
  [6]: http://static.zybuluo.com/yao-yuan-ge/hn42d32l6tlwu95cdnpzyl60/image_1e488l2r01da11v8l1tag1l3d17m02a.png
  [7]: http://static.zybuluo.com/yao-yuan-ge/copcy8z24qlt7136xngb25aa/image_1e488lc681auq1iec1sqsdrqtb02n.png
  [8]: http://static.zybuluo.com/yao-yuan-ge/s88d1rcnn49h1vfufha4pngd/image_1e488m0do3vd4iq10oea8ga7e34.png
  [9]: http://static.zybuluo.com/yao-yuan-ge/8zd9qiiojs8g0u21gjpwfc7s/image_1e488n09o10te1bgrlej27uq613h.png
  [10]: http://static.zybuluo.com/yao-yuan-ge/slk829pnf37iy5qwdibpgzez/image_1e488nc6f1m4n1af683a6291d5n3u.png
  [11]: http://static.zybuluo.com/yao-yuan-ge/f8a3edy3ec4huvx8j81rghsa/image_1e488npdf1pqh1l39109e15l1a964b.png
  [12]: http://static.zybuluo.com/yao-yuan-ge/4mkxtlzqovaht5tsjnpw0xhk/image_1e488oa9a1o4d148pr459oh13994o.png
  [13]: http://static.zybuluo.com/yao-yuan-ge/q04zmz9cpi14vh1lzrvlqqoy/image_1e488oivd1ovd44bn3f5hv1ojv55.png
  [14]: http://static.zybuluo.com/yao-yuan-ge/ldhmfjbpuids7gcz37umflrl/image_1e488oupkac8dcl1jri1916quk5i.png
  [15]: http://static.zybuluo.com/yao-yuan-ge/n7rqxhh04i14poko7wo10j87/image_1e4893l4v12dd1m9o4iu1l171luf5v.png
  [16]: http://static.zybuluo.com/yao-yuan-ge/zwbnk4w8ow3m68s5h3hfxnyz/image_1e48947c11l6f108lrsi1vsu46c.png
  [17]: http://static.zybuluo.com/yao-yuan-ge/p4l73wacldrn05vhm5cmb6pk/image_1e4894u3s1qr01jcu58ju5sjkk79.png
  [18]: http://static.zybuluo.com/yao-yuan-ge/z1ttw51rnyycdvxu5pn36eaq/image_1e4895coh66418vs1as5115h169t7m.png
  [19]: http://static.zybuluo.com/yao-yuan-ge/t8rpzqu8mp90a8m9qwq007fj/image_1e489ap4511h526pi15pm213d383.png
  [20]: http://static.zybuluo.com/yao-yuan-ge/ffmgvtc14rm9k5vtsto00on2/image_1e489c20dmhcij51bhkr9s1jh28g.png
  [21]: http://static.zybuluo.com/yao-yuan-ge/ck2b2w0q527iy6vvvmepfuqh/image_1e489cdoi1j94iqb55ase2ebp8t.png
  [22]: http://static.zybuluo.com/yao-yuan-ge/n745kq66a9imgv1wk4lo1spa/image_1e489cmvtfgoo0m197m11kh19df9a.png
  [23]: http://static.zybuluo.com/yao-yuan-ge/2051ft20pnh8a4gmlqgnohru/image_1e489sl2a6dh17mrscp1kbhihs9n.png
  [24]: http://static.zybuluo.com/yao-yuan-ge/bf94513ik5hwvrp09c06s0gu/image_1e489t0mi8er16a4v481n4p23ea4.png
  [25]: http://static.zybuluo.com/yao-yuan-ge/6029ix95q49315qtm0zt8q2s/image_1e489teu7u0ulqr13151d2r1jglb1.png
  [26]: http://static.zybuluo.com/yao-yuan-ge/r1txtxdd3u5tdjq22a511ipf/image_1e489u4bb1g0f1f3vcul1vef1vd1be.png
  [27]: http://static.zybuluo.com/yao-yuan-ge/edtandlnyw4pgthhl98p655e/image_1e489uep71qpg15omtkm8g5766br.png
  [28]: http://static.zybuluo.com/yao-yuan-ge/op7ci5htjt9wv2wcf6zv5hkj/image_1e489un642am1ir01r1u1t851pp3c8.png
  [29]: http://static.zybuluo.com/yao-yuan-ge/0ln2sabwfsak7v8hnanqdd0j/image_1e489v09b1ka51karloq1gm419krcl.png
  [30]: http://static.zybuluo.com/yao-yuan-ge/rpqebjrgssmh98foitjht90t/image_1e489vn101735d8ucfb1gsmoktd2.png
  [31]: http://static.zybuluo.com/yao-yuan-ge/7bgdn284sbuwz0283rlc8pbm/image_1e489vv3t1kk6g2sjv01qj21nk2df.png
  [32]: http://static.zybuluo.com/yao-yuan-ge/tunk33ncalj0qjwlwdpvjyfu/image_1e48a083f1j811mq8pu4qkbb5kds.png
  [33]: http://static.zybuluo.com/yao-yuan-ge/3fnoqn5zs8xtc280d6blqfid/image_1e48a0h3ie75117vdd61n2p4o0e9.png
  [34]: http://static.zybuluo.com/yao-yuan-ge/yf8l6at9oqdd770sk4d2yu6b/image_1e48a0p791soq1nou171ijabad0em.png
  [35]: http://static.zybuluo.com/yao-yuan-ge/132khnku3txtgjq46cmb2kn2/image_1e48a2fmksdt151417m2r0t13h2f3.png
  [36]: http://static.zybuluo.com/yao-yuan-ge/rdh3z2hxw2oqk0hgn37ukko2/image_1e48a2ngrqae16lerha1v4p1h3pfg.png
  [37]: http://static.zybuluo.com/yao-yuan-ge/1qtxumquqfunh49sj2jjrdv0/image_1e48a32sd195o8qjkr915mlqn5ft.png
  [38]: http://static.zybuluo.com/yao-yuan-ge/z9eidfdbvapka36bzjmfsaz7/image_1e48a3bbq151u3425552n1t8fga.png
  [39]: http://static.zybuluo.com/yao-yuan-ge/6tle39u65rultlicjcxgvf6c/image_1e48a3jd5p5lji21i0g1cmkj4ggn.png
  [40]: http://static.zybuluo.com/yao-yuan-ge/nplk41udi5emf2pgjmepkn6g/image_1e48a3ru4ao2esan916kf178hh4.png
  [41]: http://static.zybuluo.com/yao-yuan-ge/ci0r10qtnk8sxrefcsu9xiyu/image_1e48a442i1ste1gd91tgl1tlrqgchh.png
  [42]: http://static.zybuluo.com/yao-yuan-ge/rvyb7wlgjuoedcigej0mebix/image_1e48a4aa657qs1g186i1qs6ii0hu.png
  [43]: http://static.zybuluo.com/yao-yuan-ge/mnf79l68i6tahdttv9s0pr0b/image_1e48a4i5ec80hem14re1hn741vib.png
  [44]: http://static.zybuluo.com/yao-yuan-ge/3uz2nwghaji1i072crgyzqha/image_1e48a4rnkfh56n1qhka1b8dhio.png
  [45]: http://static.zybuluo.com/yao-yuan-ge/nkf14od9axwl4tn25bka8nwg/image_1e48a5h0no271pt41gb28um7p7j5.png
  [46]: http://static.zybuluo.com/yao-yuan-ge/v04f0v6h4z4mhfo6c21m6wbs/image_1e48a6mvf1ilpdvaeii12kt1kvfji.png
  [47]: http://static.zybuluo.com/yao-yuan-ge/lu5aftly9cgsabceylahzeeg/image_1e48a79kl1je91kvc5jjkdospsjv.png
  [48]: http://static.zybuluo.com/yao-yuan-ge/e69rzubgrssr223zmss2glqi/image_1e48a8e5110d91rum1v0c7ch17t5kc.png
  [49]: http://static.zybuluo.com/yao-yuan-ge/6m70so9g6g0vc73if9trmyha/image_1e48a8qvf10061ho71m57fngb99kp.png
  [50]: http://static.zybuluo.com/yao-yuan-ge/6xsv4oqczepofd6e4gz6otc2/image_1e48ab1uc1ibv1e5i11iogod1i7pl6.png
  [51]: http://static.zybuluo.com/yao-yuan-ge/5nydad1sz32i1m6g0i5562v5/image_1e48abdqbojk82bvag1ga2e6llj.png
  [52]: http://static.zybuluo.com/yao-yuan-ge/age1oump0gyg92m3lun85jhf/image_1e48abm581ao11qo51jsu1s7p84am0.png
  [53]: http://static.zybuluo.com/yao-yuan-ge/98zrhn7n870ph8m278trlu2t/image_1e48abununs7dmc1d2tgrn1e60md.png
  [54]: http://static.zybuluo.com/yao-yuan-ge/8r5j4jm2g70oqs7w0l9dulq1/image_1e48ad77n1ui419vp1ji51t29r4bmq.png
  [55]: http://static.zybuluo.com/yao-yuan-ge/bh8p2fguto2hxc3qs2cwky0l/image_1e48adhgd8gt1ltcto936126qn7.png
  [56]: http://static.zybuluo.com/yao-yuan-ge/cg1is8k2u6wc2uh4jme8a4l4/image_1e48aeoei10m9in0141qbvjd0onk.png
  [57]: http://static.zybuluo.com/yao-yuan-ge/l259d9laxebm37z0u830yhqx/image_1e48af3c517ks79kdj97p76tqo1.png
  [58]: http://static.zybuluo.com/yao-yuan-ge/diydidsvjgm65r4wearxbtib/image_1e48ag1l6ell1j3toia1c0d158eoe.png
  [59]: http://static.zybuluo.com/yao-yuan-ge/6eey8yk19znme7hct42azslc/image_1e48aga9915c3hk0t12hg0uj0or.png
  [60]: http://static.zybuluo.com/yao-yuan-ge/zkuxmmhdqygvu0l2wuqlpya4/image_1e48agij6r5f1c2ecrah0d1mdqp8.png
  [61]: http://static.zybuluo.com/yao-yuan-ge/a1ggivvnqzu3zdm2q1kju5en/image_1e48agqhicln6mb4pe1m921p2fpl.png
  [62]: http://static.zybuluo.com/yao-yuan-ge/a3vqvxcgl0wksv5kl9t0mqkt/image_1e48ah2tj8oa1k95ivnptb1a6dq2.png
  [63]: http://static.zybuluo.com/yao-yuan-ge/nibmi7t1vt2yli200b8p2gin/image_1e48ahce31dh11b3i1fc41v4m1o5qf.png
  [64]: http://static.zybuluo.com/yao-yuan-ge/r84ts98m8ovlq457n4e9k8o6/image_1e48aivsjnio1f0nv23c7n1dihqs.png
  [65]: http://static.zybuluo.com/yao-yuan-ge/033ehtko5btmv08bzbmt5c05/image_1e48ajv9m1ig8uf1722162i1po1r9.png
  [66]: http://static.zybuluo.com/yao-yuan-ge/bhre9z1gk7w647d7rxvyq3w2/image_1e48ak7sj1h8t1kqrbu21is12nvrm.png
  [67]: http://static.zybuluo.com/yao-yuan-ge/ni54551w0dffoft881zk11sr/image_1e48alatc9lo401c8k1mbci2ms3.png
  [68]: http://static.zybuluo.com/yao-yuan-ge/o9p353dodzakwlhj2d1e6dd9/image_1e48alj2vlg21cln1r2aer47fqsg.png
  [69]: http://static.zybuluo.com/yao-yuan-ge/wg1nx94gdw7wtw5rj7hbm54e/image_1e48alrffolj14dj1rjun571eqvst.png
  [70]: http://static.zybuluo.com/yao-yuan-ge/bkt0qpd1fdbaf1mxzs9jwte9/image_1e48anbn6lt9gaua7gi8s7abta.png
  [71]: http://static.zybuluo.com/yao-yuan-ge/tpzu1phs60yc8kdu3xg2un4t/image_1e48ao0qv1ovgp0ljmqsg1bt8tn.png
  [72]: http://static.zybuluo.com/yao-yuan-ge/hfz6kwcf9mwy9hom0yxlf4lj/image_1e48arohq13v8kna1h121ruo16piu4.png
  [73]: http://static.zybuluo.com/yao-yuan-ge/tedghdd96bwszhkybwmphfyt/image_1e48askj91e46tg21egd1pjq1ng8uh.png
  [74]: http://static.zybuluo.com/yao-yuan-ge/80rlz61sbyscfucxswtunqq7/image_1e48assmm18e60a7p1g4rbuvuu.png
  [75]: http://static.zybuluo.com/yao-yuan-ge/pwbz3egb0823xwhltoc7o28y/image_1e48atneo1vub1chvnev106s182qvb.png
  [76]: http://static.zybuluo.com/yao-yuan-ge/yplc2k40d0wncv219ez111td/image_1e48au1b9u9h1dj01851e2i1poavo.png
  [77]: http://static.zybuluo.com/yao-yuan-ge/azdauqpqvhnioff6w2roies5/image_1e48aurm6im05fj1t7q1qv21c7e105.png
  [78]: http://static.zybuluo.com/yao-yuan-ge/3xm906jx2fulad7hmbmkbi2f/image_1e48av5n3c2k1ovd4cf1p161dsl10i.png
  [79]: http://static.zybuluo.com/yao-yuan-ge/byuwi5xovn2w0rkr2x1gpyrh/image_1e48avrbfdbp6qh1sc31phqkeb10v.png
  [80]: http://static.zybuluo.com/yao-yuan-ge/mhno4119zt0l2m4mmglir78j/image_1e48b03k21b9k70fm0p1e3l17nf11c.png
  [81]: http://static.zybuluo.com/yao-yuan-ge/06lj2s9wrb2qas55xq896r8x/image_1e48b0e8tqt8fghrnbgjqkau11p.png
  [82]: http://static.zybuluo.com/yao-yuan-ge/z5ckc6ihqo7ojmr61qco5xti/image_1e48b0mhm19dtjkr1751q6d11sq126.png
  [83]: http://static.zybuluo.com/yao-yuan-ge/kyv7towzntqwuxtzdv5ryv39/image_1e48b1jvf1md4naf1reh1df6155f12j.png
  [84]: http://static.zybuluo.com/yao-yuan-ge/9o2oa5394cghvwpldu85s2pn/image_1e48b1smm128beb8g344m3ss0130.png
  [85]: http://static.zybuluo.com/yao-yuan-ge/h3n0o3tyhka8fm4o4oezl0r2/image_1e48b7hqi1edoq962uq1k4p1gbe13d.png
  [86]: http://static.zybuluo.com/yao-yuan-ge/klvpkd93ynhrirbvdclb133k/image_1e48h89qasap4547d7r2l68s13q.png
  [87]: http://static.zybuluo.com/yao-yuan-ge/qjdfp9og26h3bd51if1u8nme/image_1e48h8kod11u7ip413a1134q1b87147.png
  [88]: http://static.zybuluo.com/yao-yuan-ge/ui4anrbi5cv5olnrq4ozm3yq/image_1e48h8u1s16at1lmd1ilk1kmu11p814k.png
  [89]: http://static.zybuluo.com/yao-yuan-ge/8clvhi4tu2z40h6flhyjzqy2/image_1e48h984o1r3idpve0v19k41ukl151.png
  [90]: http://static.zybuluo.com/yao-yuan-ge/6gl3uie6vdhcbmf6yisetywh/image_1e48h9urfmj11bnafb8i9tdor15e.png
  [91]: http://static.zybuluo.com/yao-yuan-ge/2u1vg2i2jygcby8w3doo0en6/image_1e48ha7aen2tf2kgvf16qajna15r.png
  [92]: http://static.zybuluo.com/yao-yuan-ge/v9fyuufkza7a2tabkgw69a03/image_1e48hag8e1k9e1jiq1hl51u0o19r3168.png
  [93]: http://static.zybuluo.com/yao-yuan-ge/fy3z3cw3p6hsx6yjbbyrsct2/image_1e48haoqg1m921klc40i3di14n716l.png
  [94]: http://static.zybuluo.com/yao-yuan-ge/sr8tpsrmuuqk50387bc7lg8z/image_1e48hb2q4essv9m1q071f214ni172.png
  [95]: http://static.zybuluo.com/yao-yuan-ge/2zfamd1qtf9iyrhlu6it1x98/image_1e48hbkek1ir2u141tepo5pq9017f.png
  [96]: http://static.zybuluo.com/yao-yuan-ge/6cj294wgi78d4zmr7w32pqrq/image_1e48hbscn1o6712cj1krfmcb1df517s.png
  [97]: http://static.zybuluo.com/yao-yuan-ge/4i3rngr8t4mgnrsazuu3hc8o/image_1e48hc4c5tit158c1eh612akstr189.png
  [98]: http://static.zybuluo.com/yao-yuan-ge/catuobiuouqrvfs9ochgj7g8/image_1e48hcbtal717e2cp41md7112118m.png
  [99]: http://static.zybuluo.com/yao-yuan-ge/rtau09ilo0k2iej7f9vfsxg8/image_1e48hcl7dr2g1rl81o391apj1gl193.png
  [100]: http://static.zybuluo.com/yao-yuan-ge/i0wv7cccn4k7nud4l2mh2i3n/image_1e48hcu3p1v6hbue90iu733bm19g.png
  [101]: http://static.zybuluo.com/yao-yuan-ge/e5mbvddick1ea653mbo148me/image_1e48hd5jnh6g1di1rq9fvgdtu19t.png
  [102]: http://static.zybuluo.com/yao-yuan-ge/486vrhl148fxij278tbeei1l/image_1e48hdf2l2me5361amcttiibo1aa.png
  [103]: http://static.zybuluo.com/yao-yuan-ge/109zlcrjcf1pxdeou6atxpim/image_1e48hejeq8r22i42nkrtb1g3s1an.png
  [104]: http://static.zybuluo.com/yao-yuan-ge/zbrgrmzn44n4neqwa0uwmwhd/image_1e48hf4d9s1jepn16jvsq91s2c1b4.png
  [105]: http://static.zybuluo.com/yao-yuan-ge/qsj7a7dsu42sw1hp8ybpwixm/image_1e48hg7oj1dos1aj31ganklf1n6g1bh.png
  [106]: http://static.zybuluo.com/yao-yuan-ge/h98dlh34ab36aelwlbzlxmqp/image_1e48hgilb1ef6e7cc4duhv1jjd1bu.png
  [107]: http://static.zybuluo.com/yao-yuan-ge/90ert3tkd8ff2yqy96ecrz5z/image_1e48hi2m9k851dahqs52bmsi81cb.png
  [108]: http://static.zybuluo.com/yao-yuan-ge/uli9qzc6jd7yi7zlai4w05zi/image_1e48hic5e1uvgcbe1n81143kj8j1co.png
  [109]: http://static.zybuluo.com/yao-yuan-ge/tdbvw1r67b3riuqnb6oh6pf1/image_1e48hik4o14ua1i6g163m1vfi1eiq1d5.png
  [110]: http://static.zybuluo.com/yao-yuan-ge/tbq32j0uledllpw197l1pvlj/image_1e48hispb1lao9o212ce1udu1bv91di.png
  [111]: http://static.zybuluo.com/yao-yuan-ge/2v45xun5riua26pcjownhfux/image_1e48hj4h74a8nd369p11i1v6t1dv.png
  [112]: http://static.zybuluo.com/yao-yuan-ge/8rd84pejpcd1lsiljzplb4dk/image_1e48hjeqkv5g2rar85m91j2e1ec.png
  [113]: http://static.zybuluo.com/yao-yuan-ge/9lbqxxepklp3rttdvssjj99j/image_1e48hjn5d6d01qa214umigv1gjb1ep.png
  [114]: http://static.zybuluo.com/yao-yuan-ge/1a0sgaat71stpmskixco2u3z/image_1e48hjursgjh18b42dd18ls17ev1f6.png
  [115]: http://static.zybuluo.com/yao-yuan-ge/3uryxhbvu204vh5rpu0dlnhn/image_1e48hk6j41a77e7l19dueqegsa1fj.png
  [116]: http://static.zybuluo.com/yao-yuan-ge/s8ean3t79k7zwruledhnr1jf/image_1e48hkgep1b951ot8e201qh11tgk1g0.png
  [117]: http://static.zybuluo.com/yao-yuan-ge/a7l6s32ydejiw4uy95eh4eeg/image_1e48hmbfr1pn41o8i9tf98ucef1gd.png
  [118]: http://static.zybuluo.com/yao-yuan-ge/kky2lehy341m7bkcsplk7fqm/image_1e48hohoc29rkk9dem1f58lbd1ha.png
  [119]: http://static.zybuluo.com/yao-yuan-ge/ycq84ot0om6lzopd8dof5sva/image_1e48hpht8up8erl1sklk6f15de1hn.png
  [120]: http://static.zybuluo.com/yao-yuan-ge/z9ybpb2hjj7wj2pgwnpw4bqq/image_1e48hpqhv4qq1spc1k571molo6r1i4.png
  [121]: http://static.zybuluo.com/yao-yuan-ge/4jdoa1h50tsgea32s6bnwwzj/image_1e48hq3fu1vh11a6i10kj1gg9g1n1ih.png
  [122]: http://static.zybuluo.com/yao-yuan-ge/gsilgg70fph3tqkesk3qy4ls/image_1e48hqc411sh61htq1s49jhu1l6h1iu.png
  [123]: http://static.zybuluo.com/yao-yuan-ge/zaj2q3gno4xxh7c46kolgw7t/image_1e48hqura11kf67v174n16e2i3g1jb.png
  [124]: http://static.zybuluo.com/yao-yuan-ge/366atelkvf6n9ixv482d0dwr/image_1e48hr9vf6381e467f4i814ec1jo.png
  [125]: http://static.zybuluo.com/yao-yuan-ge/iquz9el3zoq14i6xyefy3dut/image_1e48hripq5r816lf17gftbd1d5v1k5.png
  [126]: http://static.zybuluo.com/yao-yuan-ge/hbu7n94s2ty61ntm8dvo7xbc/image_1e48hrrgj14ge1kt1oualo51ncp1ki.png
  [127]: http://static.zybuluo.com/yao-yuan-ge/6zyd0acef8u118x0hajj6us0/image_1e48hs37g15fgnqn1ql1dum1gk01kv.png
  [128]: http://static.zybuluo.com/yao-yuan-ge/ifsty7ww9b5ut19zuy46thjn/image_1e48hsm2t1iuem8c1gpq1lra1tf1lc.png
  [129]: http://static.zybuluo.com/yao-yuan-ge/um2d0j0qf5lj1uu3k3pvxtsa/image_1e48hsti118kh1vcu188auk01jqd1lp.png
  [130]: http://static.zybuluo.com/yao-yuan-ge/ljbpovc57xp5l0upo26h0u7w/image_1e48ht6m61j3ivcmclj1nbm1g8j1m6.png
  [131]: http://static.zybuluo.com/yao-yuan-ge/4osgn7fgsnue7ju4po656si3/image_1e48htfqj3kr1o274nk6kc1dau1mj.png
  [132]: http://static.zybuluo.com/yao-yuan-ge/byik18x7uagtwt2ikga4t4ia/image_1e48htpi31hlptea3ro15foh6h1n0.png
  [133]: http://static.zybuluo.com/yao-yuan-ge/apz0kfmgk859wc0sq4c7p7vv/image_1e48hu23a1s941i6o1pck16q6oda1nd.png
  [134]: http://static.zybuluo.com/yao-yuan-ge/tlpr25ksahx28zkdks3x2nzl/image_1e48hub0ll6l1dp3186d17n5loh1nq.png
  [135]: http://static.zybuluo.com/yao-yuan-ge/7qwiw5z6l3zls24ft7yr6y4f/image_1e48hvjlcfe5u2122h1e5vh61o7.png
  [136]: http://static.zybuluo.com/yao-yuan-ge/4mw2mai8bnbftu3p7xepcpiu/image_1e48hvufa1lll7gefs3qnte1f1ok.png
  [137]: http://static.zybuluo.com/yao-yuan-ge/fcx88hrmwtiiyzzbku5j2vp6/image_1e48i1htg47v7ms1dd219k61afq1p1.png
  [138]: http://static.zybuluo.com/yao-yuan-ge/tsjvhpqa547h9rheqndre26g/image_1e48i1srn1mu1nqeib71sdgl1u1pe.png