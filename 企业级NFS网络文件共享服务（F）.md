# 企业级NFS网络文件共享服务（F）

标签（空格分隔）： NFS

---Mr Wang

[TOC]

##第一章 NFS网络文件共享服务
###1.1 NFS介绍
（1）什么是NFS
> **·** NFS是Network File System的缩写，中文意思是网络文件系统。他的主要功能是通过网络（一般是局域网）让不同的主机系统之间可以共享文件或目录。NFS客户端（一般为应用服务器，例如web）可以通过挂载（mount）的方式将NFS服务器端共享的数据目录挂载到NFS客户端本地系统中（就是某一个挂载点下）。从客户端本地看，NFS服务器端共享的目录就好像是客户端自己的磁盘分区或者目录一样，而实际上却是远端的NFS服务器的目录。
**·** NFS网络文件系统很像Windows系统的网络共享，安全功能，网络驱动器映射，这也和Lliux系统里的samba服务类似。只不过一般情况下，Windows网络共享服务或samba服务用于办公局域网共享，而**互联网中小型网站集群架构后端常用NFS进行数据共享。**

（2）NFS在企业中的应用场景
>**·**  **在企业集群架构的工作场景中，NFS网络文件系统一般被用来存储共享视频，图片，附件等静态资源文件，**通常网站用户上传的文件都会放到NFS共享里，例如：BBS产品的图片，附件，头像（注意网站BBS程序不要放NFS共享里），然后前端所有的节点访问这些静态资源时都会读取NFS存储上的资源。NFS是当前互联网系统架构中最常用的数据存储服务之一，前面说过，小型网站公司应用频率更高，中大型公司或门户除了使用NFS外，还可能会使用更为复杂的分布式文件系统，比如Moosefs(mfs),GlusterFS,FastDFS等
![](https://ae01.alicdn.com/kf/Hb63d75faf69a42f2b899a026d70f6c099.png)

**·** 在企业生产集群架构中，NFS作为所有全端Web服务的共享存储，存储的内容一般包括网站用户上传的图片，附件，头像等，注意，网站的程序代码不要放NFS共享里，因为网站程序是开发运维人员统一发布的，不存在发布延迟问题，直接批量发布到web节点提供访问比共享到NFS里访问效率更高。

（3）企业生产集群为什么需要共享存储角色
>**·**这里通过图解给大家展示以下集群架构需要共享存储服务的理由,例如:A用户上传图片到Web1服务器,然后让B用户话问这张图
片,结果B用户访问的请求分发到了Web2,因为Web2上没有这张图片,这就导致它无法看到A用户上传的图片,如果此时有一个共享存储,A用户上传图片的请求无论是分发到Web1还是Web2上,最终都会存储到共享存储上,而在B用户访问图片时,无论请求分发到Web1还是Web2上,最终也都会去共享存储上找,这样就可以访问到需要的资源了。这个共享存储的位置可以通过开源软件和商业硬件实现,互联网中小型集群架构会用普通PC服务器配置NFS网络文件系统实现
**·**当及集群中没有NFS共享存储时,用户访问图片的情况如下图所示。
![](https://ae01.alicdn.com/kf/H0655173dd82c405595570cdaea253274U.png)

上图是企业生产集群没有NFS共享存储访问的示意图，下图是企业生产集群有NFS共享存储的情况。
![](https://ae01.alicdn.com/kf/Hac3c27f8479c41a1a7461c0ac22c1034T.png)

**·** 总小型企业一般不会买硬件存储，因为太贵，大公司如果业务发展很快的化，可能会临时买硬件存储顶一下网站压力，当网站并发继续加大时，硬件存储的扩展相对就会很费劲，且价格成几何数增长。例如：淘宝网就曾替换掉了很多硬件设备，比如，用LVS+Haproxy替换了netscaler负载均衡设备，用FastDFS,TFS配合PC服务器替换了netapp,emc等商业存储设备，去IOE正在成为互联网公司的主流。

###1.2NFS系统原理介绍
####1.2.1NFS系统挂载结构图解与介绍
下图是企业工作中的NFS服务器与客户端挂载情况结构图
![](https://ae01.alicdn.com/kf/H4ff7911c4db045e7a5c42505b5c2d7b45.png)
**·** 可以看到NFS服务器端/video共享目录挂载到了两台NFS客户端上，在客户端查看时，NFS服务器端的/video目录就相当于客户端本地的磁盘分区或目录，几乎感觉不到使用上的区别，根据NFS服务器端授予的NFS共享权限以及共享目录的本地系统权限，只要在指定的NFS客户端操作挂载/v/video或者/video的目录，就可以将数据轻松的存储到NFS服务器端上的/video目录中了。
客户端挂载NFS后，本地挂载基本信息显示如下：
```
[root@nfs-client tmp]# df -h 
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G 1015M   16G   7% /
tmpfs                         491M     0  491M   0% /dev/shm
/dev/sda1                     485M   33M  427M   8% /boot
/dev/sr0                      4.2G  4.2G     0 100% /media/cdrom
192.168.200.137:/data          18G 1016M   16G   7% /tmp
```
**·** 从挂载信息来看，和本地磁盘分区几乎没什么差别，只是文件系统对应列的开头是以IP地址开头的形式了。
**·** 经过前面的介绍，我们知道NFS系统是通过网络来进行数据传输的（所以叫网络文件系统）因此，NFS会使用一些端口来传输数据，那么，NFS到底使用哪些端口来进行数据传输呢？
**·** NFS在传输数据时使用的端口会随机选择，可能有人会纳闷，既然这样，NFS客户端是怎么知道NFS服务端使用的哪个端口呢？
**·** 答案就是通过RPC（中文意思就是远程过程调用，英文Remote P）rocedure Call简称RPC）协议/服务来实现，这个RPC服务的 应用在门户级的网站有很多，例如：百度等。

####1.2.2什么是RPC（Remote Procedure Call）
**·** 因为NFS支持的功能相当多，而不同的功能都会使用不同的程序来启动，每启动一个功能就会启用一些端口来传输数据，因此，NFS的功能所对应的端口无法固定，它会随机取用一些未被使用的端口来作为传输之用，其中CentOS5.x的随机端口都小于1024，而CentOS6.x的随机端口都是较大的。
**·** 因为端口不固定，这样一来就会造成NFS客户端与NFS服务端的通信障碍，因为NFS客户端必须知道NFS
服务端的数据传输端口才能进行通信，才能交互数据。
**·** 要解决上面的困扰，就需要通过远程过程调用RPC服务来帮忙了，NFS的RPC服务最主要的功能就是记录每个NFS功能所对应的端口号，并且在NFS客户端请求时将该端口和功能对应的信息传递给请求数据的NFS客户端，从而确保客户端可以连接到正确的NFS端口上去，达到实现数据传输交互数据的目的。这个RPC服务类似NFS服务端和NFS客户端之间的一个中介。

![TGZ`3`2CF$X4$XC~R`5E}8C.png-237.4kB][2]

**·** 就拿房屋中介打个比方：假设我们要找房子，这里的我们就相当于NFS客户端，中介介绍房子，中介就相当于RPC服务，房子所有者房东就相当于NFS服务，租房的人找房子，就要找中介，中介要预先存有房子主人房东的信息，才能将房源信息告诉租房的人。
**·** 那么RPC服务又是如何知道每个NFS的端口呢？
**·** 这是因为，当NFS服务端启动服务时会随机取用若干端口，并主动向RPC服务注册取用的相关端口及功能信息，如此一来，RPC服务就知道NFS每个端口对应的NFS功能了，然后RPC服务使用固定的111端口来监听NFS客户端提交的请求，并将正确的NFS端口信息回复给请求的NFS客户端，这样一来，NFS客户端就可以与NFS服务端进行数据传输了。
**·** 在启动NFS SERVER之前，首先要启动RPC服务（CentOS5.x下为portmap服务，CentOS6.x下为rpcbind服务，下同），否则NFS SERVER就无法向RPC服务注册了。另外，如果RPC服务重新启动，原来已经注册好的NFS端口数据就会丢失，因此，此时RPC服务管理的NFS程序也需要重新启动以重新向RPC注册。要特别注意的是，一般修改NFS配置文件后，是不需要重启NFS的，直接在命令执行/etc/init.d/nfs reload或exportfs -rv即可使修改的/etc/exports生效。
####1.2.3NFS的工作原理
 
 ![_P5BIUK(XN{YY@3]43VU6B2.png-198.2kB][2]
 ![](https://ae01.alicdn.com/kf/Haa35795983b54d5382f504eb5e47b9d42.png)

>当访问程序通过NFS客户端向NFS服务端存取文件时，其请求数据流程大致如下：
（1）首先用户访问网站程序，由程序在NFS客户端上发出存取NFS文件的请求，这时NFS客户端（即执行程序的服务器）的RPC服务（rpcbind服务）就会通过网络向NFS服务器端的RPC服务（rpcbind服务）的111端口发出NFS文件存取功能的询问请求。
（2）NFS服务端的RPC服务（rpcbind服务）找到对应的已注册的NFS端口后，通知NFS客户端的RPC服务（rpcbind服务）
（3）此时NFS客户端获取到正确的端口，并与NFS daemon联机存取数据。
（4）NFS客户端把数据存取成功后，返回给前端访问程序，告知给用户存取结果，作为网站用户，就完成了一次存取操作。
因为NFS的各项功能都需要向RPC服务（rpcbind服务）注册，所以只有RPC服务（rpcbind服务）才能获取到NFS服务的各项功能对应的端口号（port number），PID,NFS在主机所监听的IP等信息，而NFS客户端也只能通过向RPC服务（rpcbind服务）询问才能找到正确的端口。也就是说，NFS需要有RPC服务（rpcbind服务）的协助才能成功对外提供服务，从上面的描述，我们不难推断，无论是NFS客户端还是NFS服务端，当要使用NFS时，都需要首先启动RPC服务（rpcbind服务），NFS服务必须在RPC服务启动之后启动，客户端无需启动NFS服务，但需要启动RPC服务。

>**注意：**
NFS的RPC服务，在CentOS5.x下名为portmap，在CentOS6.x下名为rpcbind

###1.3NFS服务端部署环境准备
####1.3.1NFS服务部署服务器准备

|角色|IP|主机名|
|-|-|-|
|NFS服务端|192.168.200.137|nfs-server|
|NFS客户端|192.168.200.161|nfs-client|

####1.3.2修改主机名及标签
```
[root@localhost ~]# hostname nfs-server
[root@localhost ~]# sed -i 's#localhost#nfs-server#g' /etc/sysconfig/network
#然后exit 用Xshell退一下主机名就修改好了
```

####1.3.2修改映射文件
```
[root@nfs-server ~]# vim /etc/hosts
#加入映射，下面两行
192.168.200.137 nfs-server
192.168.200.161 nfs-client

#注nfs-client也要映射一下，做同样的操作
```

###1.4 NFS软件列表
**要部署NFS服务，需要安装下面的软件包**
（1）nfs-utils：NFS服务的主程序，包括rpc.nfsd,rpc.mountd这两个daemons和相关文档说明，以及执行命令文件等。
（2）rpcbind：CentOS6.X下面RPC的主程序，NFS可以视为一个RPC程序，在启动任何一个RPC程序之前，需要做好端口和功能的对应映射工作，这个映射工作就是由rpcbind服务来完成的。因此，在提供NFS服务之前必须先启动rpcbind服务才行。
>**注意：**
有关RPC协议知识这里大家不必细究，详细说明可见本章结尾命令部分。

####1.4.2 查看NFS软件包
可使用如下命令查看默认情况下CentOS6.X里NFS软件的安装情况。

`rpm -qa nfs-utils rpcbind`

当不知道软件名字的时候，可以用`rpm -qa | grep -E "nfs-|rpcbind"`来过滤包含在引号内的字符串。grep -E这里相当于egrep。CentOS6.8默认没有安装NFS软件包，可以使用`yum -y install nfs-utils rpcbind`命令来安装NFS软件。
```
[root@nfs-server ~]# yum -y install nfs-utils rpcbind
[root@nfs-server ~]# rpm -qa   nfs-utils rpcbind 
rpcbind-0.2.0-11.el6.x86_64
nfs-utils-1.2.3-39.el6.x86_64
```
如果出现rpcbind和nfs-utils开头的两个软件包，表示NFS服务端软件安装完毕。

####1.4.3 启动NFS相关服务
**查看某个命令属于已经安装的哪个rpm包**
```
[root@nfs-server ~]# which rpcinfo
/usr/sbin/rpcinfo
[root@nfs-server ~]# rpm -qf /usr/sbin/rpcbind
rpcbind-0.2.0-11.el6.x86_64
[root@nfs-server ~]# rpm -qf `which showmount`
nfs-utils-1.2.3-39.el6.x86_64
```

#####1.4.3.1 启动rpcbind
因为NFS极其辅助程序都是基于RPC(remote Procedure Call)协议的（使用的端口为111），所以首先要确保系统中运行了rpcbind服务。启动的实际操作如下：
```
[root@nfs-server ~]# /etc/init.d/rpcbind status
rpcbind is stopped
[root@nfs-server ~]# /etc/init.d/rpcbind start
Starting rpcbind:                                          [  OK  ]
[root@nfs-server ~]# rpcinfo -p localhost
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
[root@nfs-server ~]# rpcinfo
   program version netid     address                service    owner
    100000    4    tcp6      ::.0.111               portmapper superuser
    100000    3    tcp6      ::.0.111               portmapper superuser
    100000    4    udp6      ::.0.111               portmapper superuser
    100000    3    udp6      ::.0.111               portmapper superuser
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /var/run/rpcbind.sock  portmapper superuser
    100000    3    local     /var/run/rpcbind.sock  portmapper superuser

#提示：111端口为rpcbind服务对外提供服务的主端口
```

#####1.4.3.2 启动NFS服务
```
#启动NFS服务
[root@nfs-server ~]# /etc/init.d/nfs start
Starting NFS services:                                     [  OK  ]
Starting NFS mountd:                                       [  OK  ]
Starting NFS daemon:                                       [  OK  ]
Starting RPC idmapd:                                       [  OK  ]

#看NFS向rpc注册端口情况
[root@nfs-server ~]# rpcinfo
   program version netid     address                service    owner
    100000    4    tcp6      ::.0.111               portmapper superuser
    100000    3    tcp6      ::.0.111               portmapper superuser
    100000    4    udp6      ::.0.111               portmapper superuser
    100000    3    udp6      ::.0.111               portmapper superuser
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /var/run/rpcbind.sock  portmapper superuser
    100000    3    local     /var/run/rpcbind.sock  portmapper superuser
    100005    1    udp       0.0.0.0.218.51         mountd     superuser
    100005    1    tcp       0.0.0.0.196.196        mountd     superuser
    100005    1    udp6      ::.210.21              mountd     superuser
    100005    1    tcp6      ::.182.124             mountd     superuser
    100005    2    udp       0.0.0.0.160.74         mountd     superuser
    100005    2    tcp       0.0.0.0.225.30         mountd     superuser
    100005    2    udp6      ::.210.224             mountd     superuser
    100005    2    tcp6      ::.214.49              mountd     superuser
    100005    3    udp       0.0.0.0.159.115        mountd     superuser
    100005    3    tcp       0.0.0.0.211.140        mountd     superuser
    100005    3    udp6      ::.140.152             mountd     superuser
    100005    3    tcp6      ::.140.5               mountd     superuser
    100003    2    tcp       0.0.0.0.8.1            nfs        superuser
    100003    3    tcp       0.0.0.0.8.1            nfs        superuser
    100003    4    tcp       0.0.0.0.8.1            nfs        superuser
    100227    2    tcp       0.0.0.0.8.1            nfs_acl    superuser
    100227    3    tcp       0.0.0.0.8.1            nfs_acl    superuser
    100003    2    udp       0.0.0.0.8.1            nfs        superuser
    100003    3    udp       0.0.0.0.8.1            nfs        superuser
    100003    4    udp       0.0.0.0.8.1            nfs        superuser
    100227    2    udp       0.0.0.0.8.1            nfs_acl    superuser
    100227    3    udp       0.0.0.0.8.1            nfs_acl    superuser
    100003    2    tcp6      ::.8.1                 nfs        superuser
    100003    3    tcp6      ::.8.1                 nfs        superuser
    100003    4    tcp6      ::.8.1                 nfs        superuser
    100227    2    tcp6      ::.8.1                 nfs_acl    superuser
    100227    3    tcp6      ::.8.1                 nfs_acl    superuser
    100003    2    udp6      ::.8.1                 nfs        superuser
    100003    3    udp6      ::.8.1                 nfs        superuser
    100003    4    udp6      ::.8.1                 nfs        superuser
    100227    2    udp6      ::.8.1                 nfs_acl    superuser
    100227    3    udp6      ::.8.1                 nfs_acl    superuser
    100021    1    udp       0.0.0.0.138.135        nlockmgr   superuser
    100021    3    udp       0.0.0.0.138.135        nlockmgr   superuser
    100021    4    udp       0.0.0.0.138.135        nlockmgr   superuser
    100021    1    tcp       0.0.0.0.217.125        nlockmgr   superuser
    100021    3    tcp       0.0.0.0.217.125        nlockmgr   superuser
    100021    4    tcp       0.0.0.0.217.125        nlockmgr   superuser
    100021    1    udp6      ::.158.2               nlockmgr   superuser
    100021    3    udp6      ::.158.2               nlockmgr   superuser
    100021    4    udp6      ::.158.2               nlockmgr   superuser
    100021    1    tcp6      ::.220.231             nlockmgr   superuser
    100021    3    tcp6      ::.220.231             nlockmgr   superuser
    100021    4    tcp6      ::.220.231             nlockmgr   superuser

```
>**特别提示：**
如果不启动rpcbind服务直接启动nfs服务，nfs服务会启动失败。

##### 1.4.3.3 NFS服务常见进程详细说明
从上面NFS服务启动过程可以看出，运行NFS服务默认需要启动的服务或进程至少有：NFS quotas(rpc.rquotad),NFS daemon(nfsd),NFS mountd(rpc.mountd).可以通过执行如下命令查看启动NFS后，系统中运行的NFS相关进程：
```
[root@nfs-server ~]# ps -ef | egrep "nfs|rpc"
rpc        1143      1  0 18:44 ?        00:00:00 rpcbind
root       1174      2  0 18:47 ?        00:00:00 [rpciod/0]
root       1182      1  0 18:47 ?        00:00:00 rpc.mountd
root       1188      2  0 18:47 ?        00:00:00 [nfsd4]
root       1189      2  0 18:47 ?        00:00:00 [nfsd4_callbacks]
root       1190      2  0 18:47 ?        00:00:00 [nfsd]
root       1191      2  0 18:47 ?        00:00:00 [nfsd]
root       1192      2  0 18:47 ?        00:00:00 [nfsd]
root       1193      2  0 18:47 ?        00:00:00 [nfsd]
root       1194      2  0 18:47 ?        00:00:00 [nfsd]
root       1195      2  0 18:47 ?        00:00:00 [nfsd]
root       1196      2  0 18:47 ?        00:00:00 [nfsd]
root       1197      2  0 18:47 ?        00:00:00 [nfsd]
root       1220      1  0 18:47 ?        00:00:00 rpc.idmapd
root       1286      2  0 18:57 ?        00:00:00 [nfsiod]
root       1287      2  0 18:57 ?        00:00:00 [nfsv4.0-svc]
root       1617   1008  0 21:35 pts/0    00:00:00 egrep nfs|rpc

```
NFS服务的主要任务是共享文件系统数据，而文件系统数据的共享离不开权限问题，所以NFS服务器启动时最少需要两个不同的进程，一个是管理NFS客户端是否能够登入的rpc.nfsd主进程，另一个是用于管理NFS客户端是否能够取得对应权限的rpc.mountd进程。如果还需要管理磁盘配额，则NFS还要再加载rpc.rquotad进程。
|服务或进程名|用途说明|
|-|-|-|
|nfsd(rpc.nfsd)|rpc.nfsd主要功能是管理NFS客户端是否能够登入NFS服务端主机，其中还包括登入者的ID判断等|
|mountd(rpc.mountd)|rpc.mountd的主要功能则是管理NFS文件系统。当NFS客户端顺利通过rpc.nfsd登入NFS服务端主机时，在使用NFS服务器提供数据之前，它会去读NFS的配置文件/etc/exports来比对NFS客户端的权限，通过这一关之后，还会经过NFS服务端本地文件系统使用权限（就是owner，group，other权限）的认证程序。如果都通过了，NFS客户端就可以取得使用NFS服务器端文件的权限，注意，这个/etc/exports文件也是我们用来管理NFS共享目录的使用权限与安全设置的地方，特别强调，NFS本身设置的是网络共享权限，整个共享目录的权限自身系统权限有关|
|rpc.lockd(非必需)|可用来锁定文件，用于多客户端同时写入|

#####1.4.3.4 配置NFS服务端服务开机自启动
```
[root@nfs-server ~]# chkconfig rpcbind on
[root@nfs-server ~]# chkconfig nfs on
[root@nfs-server ~]# chkconfig --list rpcbind
rpcbind        	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@nfs-server ~]# chkconfig --list nfs
nfs            	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```
>**特别说明**：
在很多大企业里，大都是统一按照运维规范将服务的启动命令放到/etc/rc.local文件里，而不是用chkconfig管理的。把/etc/rc.local文件作为本机的重要服务档案文件，所有服务的开机自启动都必须放入/etc/rc.local。这样规范的好处是，一旦管理此服务器的人员离职，或者业务迁移都可以通过/etc/rc.local很容易的查看到服务器对应的相关服务，可以方便的运维管理，下面是把启动命令放入到/etc/rc.local文件中的配置信息，注意别忘了加上启动服务的注释。
```
[root@nfs-server ~]# vim /etc/rc.local
#在最后加上如下内容
#start up nfs service by Mr.Wang at 20200202
/etc/init.d/rpcbind start
/etc/init.d/nfs start
```

###1.5 实战配置NFS服务端
####1.5.1 NFS服务端配置文件路径
**NFS服务的默认配置文件路径为：/etc/exports，并且默认是空的**
```
[root@nfs-server ~]# ll /etc/exports
-rw-r--r-- 1 root root 32 Feb 15 18:50 /etc/exports
[root@nfs-server ~]# cat /etc/exports 
```
>**提示：**
NFS默认配置文件/etc/exports其实是存在的，但是没有内容，需要用户自行配置

####1.5.2 exports配置文件格式
/etc/exports文件位置格式为：
NFS共享的目录   NFS客户端地址1(参1,参2...) 客户端地址2(参1，参2...)
查看exports语法文件格式帮助的方法为：
执行man exports命令，然后切换到文件结尾，可以快速看如下样例格式：
```
[root@nfs-server ~]# cat /etc/exports 
/data	192.168.200.0/24(rw,sync)

命令说明：
/data:nfs的共享目录路径
192.168.200.0/24:允许挂载我的共享目录的IP地址段
（rw）:可读可写
（sync）：实时同步
```
修改配置文件以后，必须重启nfs服务
```
[root@nfs-server ~]# /etc/init.d/nfs reload
[root@nfs-server ~]# showmount -e
Export list for nfs-server:
/data 192.168.200.0/24
```
####1.5.3创建共享目录并给共享目录更改属主属组为nfsnobody
```
#自己随便创建一个共享目录
[root@nfs-server ~]# mkdir /data
#nfs本来就有程序用户
[root@nfs-server ~]# id nfsnobody
uid=65534(nfsnobody) gid=65534(nfsnobody) groups=65534(nfsnobody)
#给共享目录修改属主属组
[root@nfs-server ~]# chown -R nfsnobody.nfsnobody /data
[root@nfs-server ~]# ll -d /data
drwxr-xr-x 2 nfsnobody nfsnobody 4096 Feb 15 18:51 /data
```
>**特别提示：**
如果不授权属主属组，那么共享目录挂载以后将不遵守配置文件exports设定好的读写规则，虽然也能够正常挂载，但是会导致写入文件时提示没有权限。
```
touch：无法创建"/data/a":权限不够
```

####1.5.4 进行本地挂载测试
```
[root@nfs-server ~]# mount 192.168.200.137:/data /tmp
[root@nfs-server ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G 1015M   16G   7% /
tmpfs                         491M     0  491M   0% /dev/shm
/dev/sda1                     485M   33M  427M   8% /boot
/dev/sr0                      4.2G  4.2G     0 100% /media/cdrom
192.168.200.137:/data          18G 1014M   16G   7% /tmp
```
>**提示：**
没有报错就说明挂载成功了

####1.5.5 进行文件写入测试
```
[root@nfs-server ~]# cd /tmp
[root@nfs-server tmp]# touch 111
[root@nfs-server tmp]# cd /data
[root@nfs-server data]# ls
111
[root@nfs-server data]# cd /tmp
[root@nfs-server tmp]# ls
111
```
>**提示：**
当配置文件exports里设定了（rw,rsync）后，表示目录可读写，并且是实时同步的。也就是说，在其中一个挂载目录里改变了里面的内容信息，那么所有挂载目录包含源共享目录的内容信息同步改变。

**至此NFS服务器端配置完毕**

###1.6 实战配置NFS客户端配置过程 nfs-client
####1.6.1 回顾整体流程

![](https://ae01.alicdn.com/kf/H32cffbbc43374e818cb0a12e46c66e59P.png)

####1.6.2 客户端必须安装nfs-utils软件
```
[root@nfs-client ~]# yum -y install nfs-utils
```
**提示：不安装不能挂载nfs共享目录**

####1.6.3 检查远端showmount
```
[root@nfs-client ~]# showmount -e 192.168.200.137
Export list for 192.168.200.137:
/data 192.168.200.0/24
```

####1.6.4 客户端挂载
```
[root@nfs-client ~]# mount 192.168.200.137:/data /tmp
```
**提示：-t nfs 可以省略**

####1.6.5 进行文件读写及同步测试
```
[root@nfs-client ~]# ls /tmp
111
[root@nfs-client ~]# touch /tmp/yunjisuan
[root@nfs-client ~]# ls /tmp
111  yunjisuan
[root@nfs-client ~]# ssh root@192.168.200.137 "ls /data"
The authenticity of host '192.168.200.137 (192.168.200.137)' can't be established.
RSA key fingerprint is 17:3e:bd:15:31:fc:91:87:14:ac:e1:67:2f:89:d1:93.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.200.137' (RSA) to the list of known hosts.
root@192.168.200.137's password: 
111
yunjisuan
```

####1.6.6 配置开机自启动挂载nfs共享目录（/etc/fstab）
配置客户端mount挂载命令使挂载开机自动执行，这里有两种方法，如下：

**第一种方法：**将挂载命令放在/etc/rc.local里
缺点：偶尔开机挂载不上，工作中除了开机自启动配置，还要对监控
```
#最好用vim将/etc/rc.local打开，加入如下一行（防止用echo追加少打一个>，而清空了文件）
mount -t nfs 192.168.200.137:/data /mnt
一般将挂载挂到/mnt（临时目录）
```
**第二种方法:**将挂载命令放在/etc/fstab里
```
[root@nfs-client ~]# tail -1 /etc/fstab
192.168.200.137:/data	/mnt			nfs	defaults 0 0
```
**其实所谓配置方法，这里有一个误区，如下：**
**·** fstab会优先于网络被linux系统加载，网络没启动时执行fstab会导致连不上NFS服务器端，无法实现开机挂载。而且，即使是本地的文件系统，也要注意，fstab最后两列要设置0 0 否则有可能导致无法启动服务器的问题。
**·** 因此，nfs网络文件系统最好不要放到fstab里实现开机挂载。
**·** 但是，如果是在开机自启动服务里设置并**netfs**服务，放入fstab里也是可以挂载的。
例如：
```
[root@nfs-client ~]# chkconfig --list netfs
netfs          	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@nfs-client ~]# chkconfig netfs on
[root@nfs-client ~]# chkconfig --list netfs
netfs          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```
如此一来，我们也可以通过fstab进行开机挂载nfs网络文件系统了。

**至此客户端配置完毕**

###1.7 NFS配置权限设置常用参数说明

|rw|Read-write,表示可读可写权限|
|-|-|-|
|ro|Read-only，表示只读权限|
|sync|（同步，实时）请求或吸入数据时，数据同步写入到NFS Server的硬盘后才返回|
|async|（异步）写入时数据会先写到内存缓冲区，直到硬盘有空档才会写入磁盘，这样可以提升写入速率，风险为若服务器挂掉或不正常关机，会损失缓冲区未写入磁盘的数据|
|no_root_squash|访问NFS Server共享目录的用户如果是root，它对该共享目录具有root权限|
|root_squash|如果访问目录的是root，则它的权限将被压缩成匿名用户|
|all_squash|不管访问共享目录的用户身份如何，他的权限都被压缩成匿名用户|
|anonuid|指定共享文件夹里文件所有者的uid号，例如：（rw,squash,anonuid=12306,anongid=12306）|
|anongid|指定共享文件夹里文件所有者的gid号，例如：（rw,squash,anonuid=12306,anongid=12306）|

###1.8 NFS服务企业案例配置实战
**实例一：**
共享/data目录给192.168.200.0/24整个网段可读可写

**实例二：**
nfs服务器：192.168.200.137
共享下面两个目录：
/app/w:要求可读可写，同步数据，所有用户压缩为匿名用户
/app/r:要求只读，同步数据，所有用户都压缩为匿名用户

客户端上面的要求：
backup服务器，把nfs服务器的/app/r挂载到/data/r
web01服务器，把nfs服务器的/app/w挂载到/data/w

###1.9 exports配置文件相关参数的说明
exports配置文件的相关参数，摘自man exports：
```
EXAMPLE
       # sample /etc/exports file
       /               master(rw) trusty(rw,no_root_squash)
       /projects       proj*.local.domain(rw)
       /usr            *.local.domain(ro) @trusted(rw)
       /home/joe       pc001(rw,all_squash,anonuid=150,anongid=100
)
       /pub            *(ro,insecure,all_squash)
```
上述各个列的参数含义如下：
**·** NFS共享的目录：为NFS服务端要共享的实际目录，要用绝对路径，如（/data）。注意共享目录的本地权限，如果需要读写共享，一定要让本地目录可以被NFS客户端的用户（nfsnobody）读写。
**·** NFS客户端地址：为NFS服务端授权的可访问共享目录的NFS客户端地址，可以为单独的IP地址或主机名，域名等，也可以为整个网段地址，还可以用"*"来匹配所有客户端服务器，这里所谓的客户端一般来说是前端的业务服务器，例如：Web服务器。
**·** 权限参数集：对授权的NFS客户端的访问权限设置，参数具体说明见后文。
    nfs权限（共享权限）
    本地文件系统权限，挂载目录的权限
    
|客户端地址|具体地址|说明|
|-|-|-|
|授权单一客户端访问NFS|192.168.200.137|一般情况，生产环境中此配置不多|
|授权整个网段可访问NFS|192.168.200.0/24|其中的24等同于255.255.255.0，指定网段生产环境中常见的配置，配置简单，维护方便
|授权整个网段可访问NFS|192.168.200.*|指定网段的另外写法（不推荐使用）|
|授权某个域名客户端访问|nfs.yunjisuan.com|此方法生产环境中一般不常用|
|授权整个域名客户端访问|*.yunjisuan.com|此方法生产环境一般不常用|

###1.10 exports配置文件相关参数应用领域的详细解释（NFS精华重点）
1）（rw,sync）:可读可写，同步传输；
2）（ro,async）:只读，异步传输。

>**详细说明：**
rw或者ro，主要控制的是所有客户端用户（包含root）的读写权限，如果设置成ro，就算root也只有读权限，它是NFS权限设置的第一道总闸阀门。
sync：同步传输，实时进行。
async：异步传输，攒一会再传输。

3）root_squash:将root账户在共享目录里的身份降低为匿名者（默认nfsnobody）身份
4）no_root_squash:不降低root账户在共享目录的身份，身份还是root
5）all_squash:将所有访问用户在共享目录里的身份都降低为匿名者（默认nfsnobody）身份

>**详细说明**：
**·** 匿名者身份默认情况下就是NFS服务器端的虚拟账户角色，也就是nfsnobody.这是最低的身份，所有NFS客户端共享目录的访问者都被附加了这个身份，这也就意味着，如果文件的属主属组是nfsnobody的话，所有访问者对该文件都拥有全部所有权。
**·** 所谓身份并不是访问权限，而是用户在共享目录里创建的文件的属主和属组。
**·** 一旦身份被降低那么在共享目录里创建的文件的属主和属组就是变成了默认情况下的nfsnobody.这也就意味着，权限系统对你所创建的文件不做任何保护（任何访问者都可以查看，修改，删除）
**所谓root_squash:**
**·** 使用这个参数意味着root在共享目录里创建的任何文件都不受保护，任何人（所有用户）都可以读取，修改，删除。
**·** 而非root用户则不降低权限，在共享目录里创建的文件的属主和属组统一为nobody（身份隐藏了），这种情况下，所有普通用户之间只能互相查看文件，并不能任意修改和删除并且你还无法知道是谁创建的文件，每个普通用户只能修改或删除自己创建的文件。
**·** root用户虽然被降低了身份，但是并没有降低它的管理权限，也就是说它仍旧能对所有共享目录里的所有文件进行查看，修改，删除操作。
**·** 如果这类参数默认为空的化，那么NFS将默认使用这个参数。
**所谓no_root_squash:**
**·** 使用这个参数意味着不对root 进行降低身份的操作，也就是说root在共享目录里创建的文件的属主和属组仍旧是root（不能被普通用户修改和删除）
**·** 非root用户同root_squash一眼，并不降低权限。
**所谓all_squash:**
**·** 使用这个参数意味着对所有访问NFS共享目录的用户进行降低身份的操作，也就是说，所有用户只要在共享目录里创建文件，那么文件的属主属组就是默认情况下的nfsnobody.
**·** 在这个模式下，任何nfs客户端的任何访问用户都可以对共享目录里的任何文件进行查看，修改，删除操作。
6）anonuid和anongid：指定NFS虚拟账户的uid或gid
>**·** 这两个参数主要用来修改NFS默认的虚拟账户nfsnobody，可以通过指定虚拟账户的uid和gid的方式修改默认的虚拟账户的账户名称和所属组

##第二章 NFS企业级优化
###2.1 NFS配置文件优化
1）NFS客户端挂载后，网共享目录写入数据时卡住了。
2）NFS服务端，重启restart，客户端如果写入数据可住了。
解答：
1，nfs服务端重启后，共享文件夹进入grace time(无敌时间)
2，客户端在服务端重启后写入数据大概要等90秒
3，nfs配置文件：/etc/sysconfig/nfs
```
vim /etc/sysconfig/nfs
加入如下三条：
NFSD_V4_GRACE=90        #无敌时间
NFSD_V4_LEASE=90        #无敌时间
NLM_GRACE_PERIOD=90     #无敌时间

#说明：
NFSD_v4_GRACE=90    <===>/proc/fs/nfsd/nfsv4gracetime
NFSD_V4_LEASE=90    <===>/proc/fs/nfsd/nfsv4leasetime
NLM_GRACE_PERIOD=90     <===>/proc/fs/nfsd/nfsv4recoverydir
这三条是控制无敌时间的，加上后别忘了重启服务，一旦启用了这三条，/proc临时目录下便会生成对应的临时文件
```

###2.2 NFS客户端mount挂载深入
在NFS服务端可以通过cat /var/lib/nfs/etab查看服务端配置参数的细节，在NFS客户端可以通过cat /proc/mounts查看mount的挂载参数细节。

####2.2.1 mount挂载说明
通过如下命令在NFS客户端测试挂载获取的默认挂载参数：
```
[root@nfs-client ~]# grep tmp /proc/mounts
devtmpfs /dev devtmpfs rw,relatime,size=492540k,nr_inodes=123135,mode=755 0 0
tmpfs /dev/shm tmpfs rw,relatime 0 0
192.168.200.137:/data/ /tmp nfs4 rw,relatime,vers=4,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.200.161,minorversion=0,local_lock=none,addr=192.168.200.137 0 0
```
**NFS Client mount 挂载参数**
|mount参数|参数功能|默认参数|
|-|-|-|
|fg;bg|当客户端挂载时，可以选择前台fg挂载或者后台bg挂载，，后台挂载不影响前台其他操作，如果网络不稳建议bg比较妥当|fg|
|soft；hard|soft段挂载，当timeout出现时可能会造成资料丢失，不建议使用|hard|
|intr|当使用hard挂载的资源timeout后，若有指定intr参数，可以在timeout后把它中断掉，这避免出问题时系统整个被NFS锁死，建议使用intr|无|
|proto=udp|使用UDP协议来传输资料，在LAN中会有比较好的性能，若要跨越Internet的话，使用pro=tcp多传输的数据会有比较好的纠错能力|proto=tcp|

**mount -o参数对应的选项：**
|参数|参数意义|系统默认值|
|-|-|-|
|suid;nosuid|当挂载的文件系统上有任何SUID的程序时，只要使用nosuid就能够取消设置SUID的功能|suid|
|rw;ro|可以指定文件系统是只读（ro）或可写（rw）|rw|
|dev;nodev|是否可以保留装置文件的特殊功能？一般来说只有/dev才会有特殊的装置，因此可以选择nodev|dev|
|exec;noexec|是否具有执行文件的权限？如果想要挂载的仅是普通资源数据区（例如：图片，附件），那么可以选择noexec|exec|
|user;nouser|是否允许用户进行文件的挂载与卸载功能？如果要保护文件系统，最好不要提供用户进行挂载与卸载|nouser|
|auto；noauto|这个auto指的是“mount -a”时会不会被卸载的项目，如果不需要这个分区随时被挂载，可以设置为noauto|auto|

###2.3 NFS客户端mount挂载优化
某网友问：在企业生产环境中，NFS客户端挂载有没有必须要加的参数，比如，加noexec,nosuid,nodev,bg,soft,rsize,wsize等参数。
解答：
这个问题属于mount挂载优化内容（有些参数也适合其他文件系统），一般来说要适当加挂载参数，但是，最好是先做好测试，用数据来说话，才能更好地确定到底是挂载还是不挂载。

####2.3.1 有关系统安全挂载参数选项
在企业工作场景，一般来说，NFS服务器共享的只是普通静态数据（图片，附件，视频），不需要执行suid，exec等权限，挂载的这个文件系统只能作为数据存取之用，无法执行程序，对于客户端来讲增加了安全性，例如：很多木马篡改站点文件都是由上传入口上传的程序到存储目录，然后执行的。

因此在挂载的时候，用下面的命令很有必要：
`mount -t nfs -o nosuid,noexec,nodev,rw 192.168.200.137:/data /mnt`

####2.3.2 mount挂载性能优化参数选项
下面介绍几个在企业生产环境下，NFS性能优化挂载的例子。
1）禁止更新目录及文件时间戳挂载，命令如下:

`mount -t nfs -o noatime,nodiratime 192.168.200.137:/data /mnt`

2)安全加优化的挂载方式如下：

`mount -t nfs -o nosuid,noexec,nodev,noatime,nodiratime,intr,rsize=131072,wsize=131072 192.168.200.137:/data /mnt`

3)默认的挂载方式如下：

`mount -t nfs 192.168.200.137:/data /mnt`

如果是本地文件系统，使用如下命令：

`mount /dev/sdb1 /mnt -o defaults,async,noatime,data=writeback,barrier=0`

>**注意：**
如果是本地文件系统挂载时，如果加入nodiratime会报错

####2.3.3 NFS网络文件系统优化挂载的参数建议
在CentOS6.5 6.6 6.8等服务器端和客户端环境下，可使用如下命令参数：
`mount -t nfs -o noatime,nodiratime,nosuid,noexec,nodev,rsize=131072 192.168.200.137:/data /mnt`

经过实际测试，CentOS6.6 6.8默认的挂载参数性能还是不错的。

`mount -t nfs 192.168.200.137:/data /mnt`

**注意：非性能的参数越多，速度可能会变慢**

####2.3.4 NFS内核优化建议
**下面是优化选项说明：**
**·** /proc/sys/net/core/rmem_default:该文件指定了接受套接字缓冲区大小的默认值（以字节为单位），默认设置：124928 **建议：8388608**
**·** /proc/sys/net/core/rmem_max:该文件指定了接受套接字缓冲区大小的最大值（以字节为单位）**建议：16777216**
**·** /proc/sys/net/core/wmem_default:该文件指定了发送套接字缓冲区大小的默认值（以字节）默认设置：124928 **建议：8388608**
**·** /proc/sys/net/core/wmem_max:该文件指定了发送套接字缓冲区大小的最大值（以字节为单位）。默认设置：124928 **建议：16777216**

###2.4 NFS系统应用的优缺点说明
NFS服务可以让不同的客户端挂载使用同一个共享目录，也就是将其作为共享存储使用，这样可以保证不同节点客户端数据的一致性，在集群架构环境中经常会用到，如果是Windows和linux混合环境的集群系统，可以用samba来实现。

**优点：**
**·** 简单，容易上手，容易掌握
**·** NFS系统内数据是在文件系统之上的，即数据是能看见的。
**·** 部署快速，维护简单方便，且可控，满足需求的就是最好的。
**·** 可靠，从软件层面上看，数据可靠性高，经久耐用，数据是在文件系统之上的。
**·** 服务非常稳定

**局限：**
**·** 存在单点故障，如果NFS Server宕机了，所有客户端都不能访问共享目录，这个需要负载均衡及高可用来弥补
**·** 在大数据高并发的场合，NFS效率，性能有限（2千万/日以下PV（page view）的网站不是瓶颈，除非网站架构设计太差。）
**·** 客户端认证是基于IP和主机名的，权限要根据ID识别，安全性一般（用于内网则问题不大）
**·** NFS数据是明文的，NFS本身不对数据完整性做验证。
**·** 多台客户机器挂载一个NFS服务器时，连接管理维护麻烦（耦合度高）。尤其NFS服务端出问题后，所有NFS客户端都处于挂掉状态（测试环境可使用autofs自动挂载解决，正式环境可修复NFS服务或强制卸载）
**·** 涉及了同步（实时等待）和异步（解耦）的概念，NFS服务端和客户端相对来说就是耦合度有些高，网站程序也是一样，尽量不要耦合度太高，系统及程序架构师的重要职责就是为程序及架构解耦，让网站的扩展性变得更好。

**应用建议：**
大中小型网站（参考点2000万/日PV以下）线上应用，都有用武之地。门户站也会有应用，生产场景应该多把数据的访问往前推，即尽量把静态存储里的资源通过CDN或缓存服务器提供服务，如果没有缓存服务或架构不好，存储服务器数量再多也是扛不住压力的，而且用户体验会很差。

## 附录1 【NFS挂载加入fstab案例】
NFS客户端实现fstab开机自启动挂载

现象：在/etc/fstab中配置了nfs开机自动挂载，结果无法开机自动挂载nfs

解答：
1，nfs客户端挂载命令放在/etc/rc.local实现自动挂载
2.开机自启动netfs服务，然后才能实现fstab的开机自动挂载nfs文件系统（linux开机时在加载网络之前就会加载/etc/fstab）
 
 
##附录2 fstab误操作导致无法开机
1，fstab文件被错误修改，导致在开机启动linux时候出现错误，提示让你恢复系统设置。
1）开机时出现错误提示

![](https://ae01.alicdn.com/kf/H591ebad994174ab8a88daa5329eb64e7g.png)

2）输入root用户密码后，进入到用户操作界面

![](https://ae01.alicdn.com/kf/Hb780317d8b5b4fc2970e0f6b1570c2a0j.png)

3）打开vim /etc/fstab文件，我们发现fstab文件时只读的，也就是说目前只能看不能改

![](https://ae01.alicdn.com/kf/He1ee8a81b85944889333aa847c4321f9g.png)

4）退出/etc/fstab。在命令行输入命令

`mount -o remount,rw /` 的意思是将整个根目录以可读可写的方式重新挂载一遍remount

5）我们再次打开/etc/fstab就会发现只读模式没了

![](https://ae01.alicdn.com/kf/He802746c6462465496ad22f1b0f003c7T.png)

6）赶紧修改fstab然后重启服务器。

2，光盘救援模式恢复（用linux光盘修复系统）

1）调整开机bios设置光盘启动，然后挂载光盘

![](https://ae01.alicdn.com/kf/H13b7925e46c94550b50d2b5d66ccd7896.png)

2）重启系统，进入光盘救援模式

![](https://ae01.alicdn.com/kf/H8a5a162ad44b460d9fdae60dc8a74f4dv.png)

3）一路回车，不加在网络模式

![](https://ae01.alicdn.com/kf/H6abe061a2b214662ae06dfd8b1d8c3b38.png)

4）一路回车，选择第一个

![](https://ae01.alicdn.com/kf/H299ddd2e78cc42988c4e884450440ca3O.png)

进入这个页面

![](https://ae01.alicdn.com/kf/Hbd30f07a224749ccb76747a6be6a32b34.png)

5）输入命令

![](https://ae01.alicdn.com/kf/H60b1e0685a474855ad3962ba051a2a5fu.png)





  [1]: http://static.zybuluo.com/yao-yuan-ge/jk1r6gm1er962gyrj6o5d4qp/image_1e14dsthpta3bcvd4lokj16t1g.png
  [2]: http://static.zybuluo.com/yao-yuan-ge/ze5skjip7cu8n65snvifjmcz/TGZ%603%602CF$X4$XC~R%605E%7D8C.png
  [3]: http://static.zybuluo.com/yao-yuan-ge/7zlp1hfjift3np31h6hjlndl/timg.jpg