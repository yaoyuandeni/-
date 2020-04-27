# 企业实战之监控升级可视化zabbix4.0 系列 (二)

标签（空格分隔）： zabbix4.0系列

---Mr.Wang

[TOC]

https://www.zybuluo.com/commandersu/note/1651152
##实战环境
|被监控的主机名|IP地址|运行的项目|
|-|-|-|
|www_001|192.168.200.128|nginx|
|www_002|192.168.200.131|nginx|

##添加流程
1.添加主机组
2.添加主机
3.添加监控项
4.创建图形以及触发器 
>关联触发器以及图形必须有监控项才能创建，想要有监控项必须有主机，想要有主机必须有主机组。

##一、创建一个可被监控的主机
###1.1添加主机组

![image_1e3h9tgp61v7qm811ig61kbt1bto9.png-100kB][1]

![image_1e3h9u5931agg1lu01admcft1lr7m.png-153.7kB][2]

![image_1e3h9ujjh1o95bjinnd194s1gsi13.png-74.8kB][3]

![image_1e3h9vcmb135eqqgqc21vr119911g.png-158.3kB][4]

![image_1e3h9vvsn1k3l3plg10kff182l1t.png-72.2kB][5]

###1.2添加主机

![image_1e3ha1hm9ci21l0h6ceju912uk2a.png-102.3kB][6]

![image_1e3ha23p31hgq11881liu1t7s1bdf2n.png-108.8kB][7]

![image_1e3ha2kjl13g415296muq1g1r0334.png-108.3kB][8]

![image_1e3ha373g1qqo19clqpjbrokod3h.png-93.5kB][9]

![image_1e3ha4091mtd1r639o213vu192d3u.png-108.1kB][10]

![image_1e3ha4grr12upug014jo9sd1uqf4b.png-107kB][11]

##二、添加监控项
* 监控原理 
zabbix需要定义一个key这个可以去获取一些关键值，我们在通过这个这些值就可以判断我们需要关联那些触发器，触发器后面关联动作，就可以实现告警

###2.1添加一个简单的监控项
```
#我们需要添加一个最简单的监控项ping，看下被监控的服务器是否存活和丢包检测，我们需要在zabbix-server端去下载一个叫做fping的工具，这个工具在epel源。
#如果没有epel源请运行下面的命令
[root@zabbix ~]# yum -y install epel-release
如果有就直接运行命令
[root@zabbix ~]# yum -y isntall fping
[root@zabbix ~]# which fping
/usr/sbin/fping
```

###2.2开始添加一个监控项在zabbixweb端

* 把被监控的128这台机器开起来 

![image_1e3ha9n36v3qd3fiiu1v9u7s74o.png-154kB][12]

![image_1e3haakuqm9u1lgquh6od8jho55.png-140.1kB][13]

![image_1e3hab49j1r221nh01e9c73i15vd5i.png-194.9kB][14]

![image_1e3habk3e5u41om4l9f1ckk14bl5v.png-139.4kB][15]

![image_1e3hac2l51501p461viba2fcf86c.png-160.4kB][16]

![image_1e3haceor1l1nno51cnmgunq0b6p.png-125.5kB][17]

![image_1e3hacu8c15h4rde9401j9d10av76.png-246.1kB][18]

![image_1e3hadi7s1ohteqp53s1hcr15la83.png-119.7kB][19]

![image_1e3hae293lk7per1ff79dk1pg48g.png-153.3kB][20]

![image_1e3haehcgf2244bbiefghnga8t.png-151.9kB][21]

![image_1e3haf78s70rva7135blfh1ui09a.png-137.3kB][22]

![image_1e3hafv2t2md1fotlerg8s1ed69n.png-154.6kB][23]

![image_1e3hagglkapk1lmv1811ldp15sba4.png-124kB][24]

![image_1e3hah1r71dql11921cobmu7k9iah.png-132.5kB][25]

###2.3如果监控项错了，我们才能让等待时间变短

![image_1e3hallv4uk31c1u5m67cho4iau.png-155.3kB][26]

![image_1e3ham7fo1t1jg331vto129jslibb.png-119.6kB][27]

###2.4使用变量定义监控项

![image_1e3han5m65hk1rni945k721ncmbo.png-149.6kB][28]

![image_1e3hanhqn1van3nm1tr6v2u1bn2c5.png-147.2kB][29]

![image_1e3hant3h1hqq1k2h9gk3f7t1aci.png-125.3kB][30]

![image_1e3haobql1mdm1htk12da83blepcv.png-229.2kB][31]

![image_1e3haon491jdr2vc1o011k12fl9dc.png-117.5kB][32]

###2.5我们同理在克隆一个监控80端口

![image_1e3haps3a1sho1pbv1nt21q9c1f6rdp.png-137.8kB][33]

![image_1e3haq85gue21lkd9k216ag4b6e6.png-141.8kB][34]

![image_1e3haqgn5kv11llgtlibfpfgkej.png-160.4kB][35]

![image_1e3haqqq51s50rff7qjhf91i5sf0.png-157kB][36]

##附录

>附录1 
多种监控方式添加 
simple check，被监控的服务器无需安装客户端，如ping，端口检测之类的 
zabbix agent，被动式监控服务器 
zabbix agent(active)，主动式监控服务器 
snmp check，使用snmp协议去获取监控信息 
zabbix trapper，主动式监控 
External check，zabbix server上可编写监控脚本 
Jmx agent，监控java进程

>附录2 
值的类型 
无符号整型，值是正数，无负数。例如监控端口是否存活，存活返回1，不存活返回0 
浮点型，值可以为负数、小数。例如监控cpu的idle值 
character，字符串，最大255个字节 
Text，字符串，无限制

>附录3 
application应用集 
多个类型相同的监控项目可以定义一个应用集 
icmp存活，icmp丢包我们都可以把它归为icmp应用集

##三、添加监控图形和触发器
###3.1最简单的查看图形

![image_1e3hb1o5a1h03oom1jb01o7fvsvfd.png-134.1kB][37]

![image_1e3hb24v029ash5smp1jl423ufq.png-122.9kB][38]

###3.2几个监控项合成一个图形

![image_1e3hb2tu1cldd691u5s17dqb68g7.png-142.1kB][39]

![image_1e3hb37ds4nekom1f59nbosv5gk.png-139.4kB][40]

###3.3自己创建图形

![image_1e3hb3qdnghmt0m1ttajb8fa1h1.png-127.1kB][41]

![image_1e3hb43vm5uq1vuve6eiann82he.png-112kB][42]

![image_1e3hb4dt11bvs1csq1fo3n91flqhr.png-156.9kB][43]

![image_1e3hb4mrc1i5q10ta135i12p310dvi8.png-123.6kB][44]

![image_1e3hb4utdfm2bgj3re1gfgfucil.png-106.6kB][45]

###3.4查看我们自己创建的图形

![image_1e3hb5qht13r01q44rkm3mq70pj2.png-152.7kB][46]

![image_1e3hb659lj6q1jht19u75d1hjvjf.png-124.8kB][47]

![image_1e3hb6ebs127p79egrjtbhrosjs.png-135kB][48]

##四、触发器
>监控项的分类

* 未分类
* 信息
* 警告
* 一般严重
* 严重
* 灾难

###4.1添加一个触发器

![image_1e3hb9nt4i8u1ffp1ioq1mus7mmk9.png-142.9kB][49]

![image_1e3hba1991v2cgp31tn01b953hvkm.png-125.7kB][50]

![image_1e3hba9t3bhok2mn0pvmn1ah5l3.png-139.5kB][51]

![image_1e3hbap71ve41a04r2h1no61n6dlg.png-148kB][52]

![image_1e3hbb3evg4f1lt41m57gs33v5lt.png-164.6kB][53]

![image_1e3hbbotob2j1j9c10s619jjog6mq.png-176.2kB][54]

![image_1e3hbcaka9jo1he2livj9q8lfn7.png-125.3kB][55]

![image_1e3hbclbl1ie1166oelv18gucrrnk.png-115.1kB][56]

![image_1e3hbd842196j1c1rjp01s1r1dsbo1.png-153.1kB][57]

![image_1e3hbdtcd17pfn7ebgf1vg2vloe.png-64.6kB][58]

![image_1e3hbed44ppf1chb1cqb1f9ttfmor.png-138.8kB][59]

![image_1e3hben9u1d85s9v56v47q1b99p8.png-32.8kB][60]

![image_1e3hbevp2405rs31g5i15p8nl1pl.png-133.4kB][61]

###4.2nodata触发器

![image_1e3hbh2kn130iuee1k8c10on7ejq2.png-133kB][62]

![image_1e3hbhg8n2np1kdi3ve9flkd8qf.png-120.3kB][63]

![image_1e3hbi2veot97831ap28o4169qqs.png-146.5kB][64]

![image_1e3hbid771cub1sgo3hk157jtiir9.png-109.2kB][65]

![image_1e3hbjnjja9g1q74tacrlp141grm.png-134.2kB][66]

![image_1e3hbk2fp12tcjtns8e1h2v1g2ps3.png-140.9kB][67]

![image_1e3hbkc6411v9i3k1vpr9tm6jcsg.png-105.5kB][68]

![image_1e3hbku5u1f0fpfs1rg41g1j3ujst.png-147.3kB][69]

###4.3大家照着上面两部将触发器和监控项改回去

![image_1e3hblp5n1ph5c231548lh5im3ta.png-142.1kB][70]

##五、模板
>模板的重要 
手动添加监控比较麻烦，得监控项 -> 图形 -> 触发器。例如有100台服务器需要检查81端口 
修改监控比较麻烦。例如100台服务器81改成82 
模板分为两类，一种是自带的，一种是自定义的，今天我们讲自定义的

###5.1系统自带模板，后续监控linux会讲,
![image_1e3hbnpbbsrqjnd1lsq72ghdgtn.png-172.4kB][71]

###5.2自定义模板
####5.2.1我们先把之前定义的监控项触发器，和图形，应用集都删掉
![image_1e3hbpau9179e1air11m6evh4ogu4.png-133.9kB][72]

![image_1e3hbpn9g1ied4mn6f7k3hvdduh.png-103kB][73]

![image_1e3hbqegrq7r1msg6pf1h4p17d6uu.png-140.9kB][74]

>如果有剩下的依次删除，我这里监控项删除了，相关联的图形和触发都没了

####5.2.2创建模板

![image_1e3hbridrdekl4s4en12t9fc6vb.png-187kB][75]

![image_1e3hbruhm1h1n1g2s1c3p9mglovo.png-127.3kB][76]

![image_1e3hbs6jkb7q1q0o178s1c2f1jcp105.png-120.6kB][77]

![image_1e3hbse9p1lqj18kii37snsp1j10i.png-111.6kB][78]

![image_1e3hbsnfa1iro16up1it541kpiv10v.png-123.2kB][79]

![image_1e3hbsv8n1f3j1molmqbfnedr211c.png-135.5kB][80]

####5.2.3主机添加使用模板

![image_1e3hbu91b1fgugds1molec9qp411p.png-130.6kB][81]

![image_1e3hbvi69mj117h2lrp1lruql2126.png-126kB][82]

![image_1e3hbvrcq1qig2akekehg21tr512j.png-122.6kB][83]

![image_1e3hc04ubb8mt0giq61ojf6l130.png-146.4kB][84]

![image_1e3hc0f4k8jc10vbg6vh4a11sj13d.png-116.8kB][85]

![image_1e3hc0per96tij0fr41abe1slj13q.png-150.8kB][86]

![image_1e3hc11o9liv17om1fd51hl939o147.png-137.5kB][87]

####5.2.4修改模板所有使用模板的主机会一起改变，不管有多少台

![image_1e3hc1ogq1p2nkhc15dqi47f9i14k.png-121.5kB][88]

![image_1e3hc297q1efk1g2f1dc7pao1je0151.png-138.7kB][89]

![image_1e3hc3e2k1lb21h871jdlnj8ea815e.png-134.1kB][90]

![image_1e3hc3pma3tag6ftq15e211hb15r.png-115.7kB][91]

![image_1e3hc48cbpddvohmq71hbmv1u16o.png-149.6kB][92]

![image_1e3hc52os7ld12l7hhi1ut0e3s175.png-135.7kB][93]

![image_1e3hc5bns1nnh1e4n7l5ute1sv17i.png-135.9kB][94]

>模板的好处在于我们能只修改模板就可以把使用他的所有主机都更改了。不用我们自己一步一步的去点

##六，作业
将模板完善，定义触发器，以及图形



  [1]: http://static.zybuluo.com/yao-yuan-ge/ofi3pgdbk7l8ncqao50onkci/image_1e3h9tgp61v7qm811ig61kbt1bto9.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/ma21tfbcs965b3q4mnvq3xyv/image_1e3h9u5931agg1lu01admcft1lr7m.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/kwik5ymd8pyzpu3kkgb4szx0/image_1e3h9ujjh1o95bjinnd194s1gsi13.png
  [4]: http://static.zybuluo.com/yao-yuan-ge/mc5fk3di983msmyb7xozlu1q/image_1e3h9vcmb135eqqgqc21vr119911g.png
  [5]: http://static.zybuluo.com/yao-yuan-ge/psq45wq2tiyndwc86iqnu5a1/image_1e3h9vvsn1k3l3plg10kff182l1t.png
  [6]: http://static.zybuluo.com/yao-yuan-ge/7eu2vbvbe1zimt0fg475qekh/image_1e3ha1hm9ci21l0h6ceju912uk2a.png
  [7]: http://static.zybuluo.com/yao-yuan-ge/ddc0de6rbbvpazipx8v708os/image_1e3ha23p31hgq11881liu1t7s1bdf2n.png
  [8]: http://static.zybuluo.com/yao-yuan-ge/fti5nlcl9x7rq3h7cv4ebnvp/image_1e3ha2kjl13g415296muq1g1r0334.png
  [9]: http://static.zybuluo.com/yao-yuan-ge/vy09qk369htqzer9xgexc6mr/image_1e3ha373g1qqo19clqpjbrokod3h.png
  [10]: http://static.zybuluo.com/yao-yuan-ge/xylp08n6ek79mn8ama3wbwuq/image_1e3ha4091mtd1r639o213vu192d3u.png
  [11]: http://static.zybuluo.com/yao-yuan-ge/xl3jq7hyqisugtzpnhx21eki/image_1e3ha4grr12upug014jo9sd1uqf4b.png
  [12]: http://static.zybuluo.com/yao-yuan-ge/hywlu31rvnmb0vvj5crhwwfa/image_1e3ha9n36v3qd3fiiu1v9u7s74o.png
  [13]: http://static.zybuluo.com/yao-yuan-ge/edg6b8hgz5686w4fld8dcu0x/image_1e3haakuqm9u1lgquh6od8jho55.png
  [14]: http://static.zybuluo.com/yao-yuan-ge/5cx2m7u48kkf3r0hm52b743h/image_1e3hab49j1r221nh01e9c73i15vd5i.png
  [15]: http://static.zybuluo.com/yao-yuan-ge/yxyttv0rky0f87x5b9zakhbk/image_1e3habk3e5u41om4l9f1ckk14bl5v.png
  [16]: http://static.zybuluo.com/yao-yuan-ge/6ll34tce9fgy036es0x6e0h0/image_1e3hac2l51501p461viba2fcf86c.png
  [17]: http://static.zybuluo.com/yao-yuan-ge/4wov908wipa0zy539hu2bcu8/image_1e3haceor1l1nno51cnmgunq0b6p.png
  [18]: http://static.zybuluo.com/yao-yuan-ge/lsy51xemxdoav26jkhzowgo4/image_1e3hacu8c15h4rde9401j9d10av76.png
  [19]: http://static.zybuluo.com/yao-yuan-ge/e6q7esk4spl7xtp2sjng2ypj/image_1e3hadi7s1ohteqp53s1hcr15la83.png
  [20]: http://static.zybuluo.com/yao-yuan-ge/68vwiwi2owpi2ted9k6ewl01/image_1e3hae293lk7per1ff79dk1pg48g.png
  [21]: http://static.zybuluo.com/yao-yuan-ge/2d24zedrffau9veb5l5w3w6r/image_1e3haehcgf2244bbiefghnga8t.png
  [22]: http://static.zybuluo.com/yao-yuan-ge/bi3d9ant51jmunys6i2rrn43/image_1e3haf78s70rva7135blfh1ui09a.png
  [23]: http://static.zybuluo.com/yao-yuan-ge/rv0am7joxhxzlzt2phsp266l/image_1e3hafv2t2md1fotlerg8s1ed69n.png
  [24]: http://static.zybuluo.com/yao-yuan-ge/z5wv6096t3ffnj23yk1le45m/image_1e3hagglkapk1lmv1811ldp15sba4.png
  [25]: http://static.zybuluo.com/yao-yuan-ge/rourrqomayuowfakx0v0wxoh/image_1e3hah1r71dql11921cobmu7k9iah.png
  [26]: http://static.zybuluo.com/yao-yuan-ge/0vve7wn1hqa24wi55s17tafv/image_1e3hallv4uk31c1u5m67cho4iau.png
  [27]: http://static.zybuluo.com/yao-yuan-ge/nih4hft8nnan3c303r02koct/image_1e3ham7fo1t1jg331vto129jslibb.png
  [28]: http://static.zybuluo.com/yao-yuan-ge/et2tr1vou1bhxb0fun19qr0j/image_1e3han5m65hk1rni945k721ncmbo.png
  [29]: http://static.zybuluo.com/yao-yuan-ge/8vi82rnz5n9w1s78k69r4j4k/image_1e3hanhqn1van3nm1tr6v2u1bn2c5.png
  [30]: http://static.zybuluo.com/yao-yuan-ge/qqi6wysbfvo0xlixxgy13t74/image_1e3hant3h1hqq1k2h9gk3f7t1aci.png
  [31]: http://static.zybuluo.com/yao-yuan-ge/2bk04qj1ta11h0u98asek129/image_1e3haobql1mdm1htk12da83blepcv.png
  [32]: http://static.zybuluo.com/yao-yuan-ge/pi0was3gpjvzjfxbk1r1dxxj/image_1e3haon491jdr2vc1o011k12fl9dc.png
  [33]: http://static.zybuluo.com/yao-yuan-ge/dc2yp44l7z9ov5qi4ocb5ujm/image_1e3haps3a1sho1pbv1nt21q9c1f6rdp.png
  [34]: http://static.zybuluo.com/yao-yuan-ge/9cpehn2l11t67ea974is8npm/image_1e3haq85gue21lkd9k216ag4b6e6.png
  [35]: http://static.zybuluo.com/yao-yuan-ge/swni5mkbnkuk7ma45lvwa11i/image_1e3haqgn5kv11llgtlibfpfgkej.png
  [36]: http://static.zybuluo.com/yao-yuan-ge/2vfjjzv2mlkhwg0jysdewf1l/image_1e3haqqq51s50rff7qjhf91i5sf0.png
  [37]: http://static.zybuluo.com/yao-yuan-ge/ge5piojnpa3e4qd28iyxkp0h/image_1e3hb1o5a1h03oom1jb01o7fvsvfd.png
  [38]: http://static.zybuluo.com/yao-yuan-ge/exym96p5rv2w2d5epanh6znm/image_1e3hb24v029ash5smp1jl423ufq.png
  [39]: http://static.zybuluo.com/yao-yuan-ge/ybqslcw85f0j1wq1mnzwe0n6/image_1e3hb2tu1cldd691u5s17dqb68g7.png
  [40]: http://static.zybuluo.com/yao-yuan-ge/sc4x47v88ozkkf08h45wjxpf/image_1e3hb37ds4nekom1f59nbosv5gk.png
  [41]: http://static.zybuluo.com/yao-yuan-ge/wuszbp7cpq10em7xv54lomzc/image_1e3hb3qdnghmt0m1ttajb8fa1h1.png
  [42]: http://static.zybuluo.com/yao-yuan-ge/xgpvt8ksetwlm6uuz6a7ntf8/image_1e3hb43vm5uq1vuve6eiann82he.png
  [43]: http://static.zybuluo.com/yao-yuan-ge/a3b0s9n5tbhdydr6ul61paue/image_1e3hb4dt11bvs1csq1fo3n91flqhr.png
  [44]: http://static.zybuluo.com/yao-yuan-ge/ietkq95svyvsdryiahif1yi3/image_1e3hb4mrc1i5q10ta135i12p310dvi8.png
  [45]: http://static.zybuluo.com/yao-yuan-ge/dhf1qcj72mjucug86r2s15qt/image_1e3hb4utdfm2bgj3re1gfgfucil.png
  [46]: http://static.zybuluo.com/yao-yuan-ge/onyx9kqasekbfwlymxzmxfxl/image_1e3hb5qht13r01q44rkm3mq70pj2.png
  [47]: http://static.zybuluo.com/yao-yuan-ge/8yc0urt1cnul2k8cdbsbxtz0/image_1e3hb659lj6q1jht19u75d1hjvjf.png
  [48]: http://static.zybuluo.com/yao-yuan-ge/qakb6ipd57p5scw8on470zcm/image_1e3hb6ebs127p79egrjtbhrosjs.png
  [49]: http://static.zybuluo.com/yao-yuan-ge/o1zf8ye4rycsmxz6kdahttoc/image_1e3hb9nt4i8u1ffp1ioq1mus7mmk9.png
  [50]: http://static.zybuluo.com/yao-yuan-ge/txbhge1enho7nvlf0yooc82y/image_1e3hba1991v2cgp31tn01b953hvkm.png
  [51]: http://static.zybuluo.com/yao-yuan-ge/jaeeao7aca2poj4tah0vcv99/image_1e3hba9t3bhok2mn0pvmn1ah5l3.png
  [52]: http://static.zybuluo.com/yao-yuan-ge/x65jqzxx4hwridvstqn4u9ie/image_1e3hbap71ve41a04r2h1no61n6dlg.png
  [53]: http://static.zybuluo.com/yao-yuan-ge/w6lcxasxqt8xdx99eeupu60p/image_1e3hbb3evg4f1lt41m57gs33v5lt.png
  [54]: http://static.zybuluo.com/yao-yuan-ge/lt2j7itfdd77rzmqd4cgbqge/image_1e3hbbotob2j1j9c10s619jjog6mq.png
  [55]: http://static.zybuluo.com/yao-yuan-ge/d493k33d7ox02xydh077yk5i/image_1e3hbcaka9jo1he2livj9q8lfn7.png
  [56]: http://static.zybuluo.com/yao-yuan-ge/klt5e9bggxvusrnwvcq76igm/image_1e3hbclbl1ie1166oelv18gucrrnk.png
  [57]: http://static.zybuluo.com/yao-yuan-ge/7klm4zcx1pofmqrxtuwbnnpp/image_1e3hbd842196j1c1rjp01s1r1dsbo1.png
  [58]: http://static.zybuluo.com/yao-yuan-ge/xo7a97f5ogkrafapyamrw3m1/image_1e3hbdtcd17pfn7ebgf1vg2vloe.png
  [59]: http://static.zybuluo.com/yao-yuan-ge/6ebc0qj8v2vjachvhallnf9c/image_1e3hbed44ppf1chb1cqb1f9ttfmor.png
  [60]: http://static.zybuluo.com/yao-yuan-ge/6aaef32or23ze9dd89it8rqv/image_1e3hben9u1d85s9v56v47q1b99p8.png
  [61]: http://static.zybuluo.com/yao-yuan-ge/2cwd7bybp1qonxwh89tt4mhg/image_1e3hbevp2405rs31g5i15p8nl1pl.png
  [62]: http://static.zybuluo.com/yao-yuan-ge/w9mey3wf2dg6pg32yx6nb33z/image_1e3hbh2kn130iuee1k8c10on7ejq2.png
  [63]: http://static.zybuluo.com/yao-yuan-ge/t3dis5ng3olagwrgteni2ds1/image_1e3hbhg8n2np1kdi3ve9flkd8qf.png
  [64]: http://static.zybuluo.com/yao-yuan-ge/5f4oyzzqcww6nnpw07f5333b/image_1e3hbi2veot97831ap28o4169qqs.png
  [65]: http://static.zybuluo.com/yao-yuan-ge/r07lzlz1mt3xvv8x9nimuz3i/image_1e3hbid771cub1sgo3hk157jtiir9.png
  [66]: http://static.zybuluo.com/yao-yuan-ge/oga54chp44xlz4zlgd20qtp8/image_1e3hbjnjja9g1q74tacrlp141grm.png
  [67]: http://static.zybuluo.com/yao-yuan-ge/n149alvuql5y1jumj5ubgui9/image_1e3hbk2fp12tcjtns8e1h2v1g2ps3.png
  [68]: http://static.zybuluo.com/yao-yuan-ge/y90zom6a12tlgmcgmu42dpub/image_1e3hbkc6411v9i3k1vpr9tm6jcsg.png
  [69]: http://static.zybuluo.com/yao-yuan-ge/90mxt7kraeyqhes0t7w9gb7k/image_1e3hbku5u1f0fpfs1rg41g1j3ujst.png
  [70]: http://static.zybuluo.com/yao-yuan-ge/rfl1kuxqcv655hx0sxasvc0v/image_1e3hblp5n1ph5c231548lh5im3ta.png
  [71]: http://static.zybuluo.com/yao-yuan-ge/l16z07a4b45wa5u94h549ohz/image_1e3hbnpbbsrqjnd1lsq72ghdgtn.png
  [72]: http://static.zybuluo.com/yao-yuan-ge/1kobm30v48ms3owohu67n7l5/image_1e3hbpau9179e1air11m6evh4ogu4.png
  [73]: http://static.zybuluo.com/yao-yuan-ge/0kbmvaxno82slw29x13e3tgv/image_1e3hbpn9g1ied4mn6f7k3hvdduh.png
  [74]: http://static.zybuluo.com/yao-yuan-ge/w5g0dna83sxoboso27wrqwb2/image_1e3hbqegrq7r1msg6pf1h4p17d6uu.png
  [75]: http://static.zybuluo.com/yao-yuan-ge/i1wout9b73uaq3ezcvdlec18/image_1e3hbridrdekl4s4en12t9fc6vb.png
  [76]: http://static.zybuluo.com/yao-yuan-ge/t5x4aafcarjf6ndrvbsezyub/image_1e3hbruhm1h1n1g2s1c3p9mglovo.png
  [77]: http://static.zybuluo.com/yao-yuan-ge/w5sz78heqhuztkl84ntj5anl/image_1e3hbs6jkb7q1q0o178s1c2f1jcp105.png
  [78]: http://static.zybuluo.com/yao-yuan-ge/6ksg96bfd1ogypdkwhdayuwx/image_1e3hbse9p1lqj18kii37snsp1j10i.png
  [79]: http://static.zybuluo.com/yao-yuan-ge/u3l09aima1bphx8i7sge9bqk/image_1e3hbsnfa1iro16up1it541kpiv10v.png
  [80]: http://static.zybuluo.com/yao-yuan-ge/9pq2f1jyysar09yr1ronsdps/image_1e3hbsv8n1f3j1molmqbfnedr211c.png
  [81]: http://static.zybuluo.com/yao-yuan-ge/p4681hmzbrqmh33runhmrcoy/image_1e3hbu91b1fgugds1molec9qp411p.png
  [82]: http://static.zybuluo.com/yao-yuan-ge/twfqa9mbc2zp3zjz9iy1ki4x/image_1e3hbvi69mj117h2lrp1lruql2126.png
  [83]: http://static.zybuluo.com/yao-yuan-ge/8f47de2lhprm9y71ea8yn39r/image_1e3hbvrcq1qig2akekehg21tr512j.png
  [84]: http://static.zybuluo.com/yao-yuan-ge/zq3i5g3rktssfjl8k3vprx2i/image_1e3hc04ubb8mt0giq61ojf6l130.png
  [85]: http://static.zybuluo.com/yao-yuan-ge/s9q2r6mjnk0gn7zp44wnhxj9/image_1e3hc0f4k8jc10vbg6vh4a11sj13d.png
  [86]: http://static.zybuluo.com/yao-yuan-ge/50otp20bp2dxr89d3ok7gzg5/image_1e3hc0per96tij0fr41abe1slj13q.png
  [87]: http://static.zybuluo.com/yao-yuan-ge/eox6jyqpqsk9jeri899omix9/image_1e3hc11o9liv17om1fd51hl939o147.png
  [88]: http://static.zybuluo.com/yao-yuan-ge/h3bs77kv7ajehzplsfn6sz8w/image_1e3hc1ogq1p2nkhc15dqi47f9i14k.png
  [89]: http://static.zybuluo.com/yao-yuan-ge/xikubuz6p4aovl0u9mlels0n/image_1e3hc297q1efk1g2f1dc7pao1je0151.png
  [90]: http://static.zybuluo.com/yao-yuan-ge/nmf99nux45rcx324r09h4hz9/image_1e3hc3e2k1lb21h871jdlnj8ea815e.png
  [91]: http://static.zybuluo.com/yao-yuan-ge/cy5hkzlgncpq0joniyk45w8a/image_1e3hc3pma3tag6ftq15e211hb15r.png
  [92]: http://static.zybuluo.com/yao-yuan-ge/g8g6pdea9mjhqhry6xwf71l0/image_1e3hc48cbpddvohmq71hbmv1u16o.png
  [93]: http://static.zybuluo.com/yao-yuan-ge/a95ml6lresz2v478maiwms07/image_1e3hc52os7ld12l7hhi1ut0e3s175.png
  [94]: http://static.zybuluo.com/yao-yuan-ge/k3y9ertw185s70jipk5zirbp/image_1e3hc5bns1nnh1e4n7l5ute1sv17i.png