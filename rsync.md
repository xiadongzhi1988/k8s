本文作者：**wuXing**
QQ:** 1226032602**
E-mail:** 1226032602@qq.com**




1. **服务器规划**
| 服务器说明   | 数量   | 名称规划   | 
|:----|:----:|:----|:----:|:----|:----:|
| 负载均衡服务器   | 2   | 对访问网站的流量进行分流，减少流量对某台服务器的压力   | 
| web服务器   | 2   | 处理用户页面访问请求（Nginx，apache）   | 
| NFS存储   | 1   | 存储图片  附件 头像等静态数据   | 
| 备份服务器（rsync）   | 1   | 对全网服务器数据，进行实时与定时备份   | 
| 数据库服务器（mysql）   | 1   | 对动态变化数据进行存储（文本内容）   | 
| 管理服务器   | 1   | 1、作为yum仓库服务器，提供全网服务器的软件下载  2、跳板机、操作审计  3、vpn（pptp）  4、监控（zabbix）  5、兼职批量分发和管理   | 
| 说明：总计需要服务器8台，完成本次项目   |    |    | 





2. **主机IP规划表**
| 服务器说明   | 外网IP（NAT）   | 内网IP（NAT）   | 主机名称规划   | 
|:----|:----:|:----|:----:|:----|:----:|:----|:----:|
| A1-nginx负载服务器01   | **10.0.0.5/24**   | 172.16.1.5/24   | lb01   | 
| A2-nginx负载服务器02   | **10.0.0.6/24**   | 172.16.1.6/24   | lb02   | 
| B1-nginx web服务器 01   | **10.0.0.7/24**   | 172.16.1.7/24   | web01   | 
| B2-nginx web服务器 02   | **10.0.0.8/24**   | 172.16.1.8/24   | web02   | 
| C3-mysql 数据库服务器   | **10.0.0.51/24**   | 172.16.1.51/24   | db01   | 
| C1-NFS存储服务器   | **10.0.0.31/24**   | 172.16.1.31/24   | nfs01   | 
| **C2-rsync****同步服务器**   | 10.0.0.41/24   | 172.16.1.41/24   | backup   | 
| X-管理服务器   | **10.0.0.61/24**   | 172.16.1.61/24   | m01   | 

提示：
1、和老师保持高度完全一致
2、灰色临时使用，企业场景里可以没有（仅限于ssh使用）
3、负载均衡器的VIP  10.0.0.3/24
4、带外网ip的服务器的内网ip不配网关和DNS
5、外部ip该配啥配啥




|    |    | 
|:----|:----:|:----|:----:|
| 1、搭建yum仓库、定制rpm包、收集架构所需软件   | 2、配置ntp时间服务器   | 
| 3、配置zabbix或nagios监控全网服务   | 4、解决nginx负载均衡的单点（keepalived）   | 
| 5、部署pptpvpn远程连接及管理服务器   | 6、实现mysql主从复制及主故障自动切换到备服务   | 
| 7、解决NFS单点故障自动切换（keepalived）   | 8、实现所有无外部ip的内网服务器可以上外网   | 
| 9、利用shell脚本及saltstack一键自动化安装整个集群   | 10、无人值守批量安装linux（kickstart/cobbler）   | 
| 11、解决web集群会话保持问题（Memcached）   | 12、搭建跳板机（用shell跳板机或jumpserver或CrazyEye）   | 










3. ** 系统基础优化**
1. ** #hosts文件**

\cp /etc/hosts{,.bak}
```
cat >/etc/hosts<<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.1.5      lb01
172.16.1.6      lb02
172.16.1.7      web01
172.16.1.8      web02
172.16.1.9      web03
172.16.1.51     db01
172.16.1.31     nfs01
172.16.1.41     backup
172.16.1.61     m01
172.16.1.81     firewall
EOF
```



2. ** #1、关闭selinux**
```
sed -i.bak 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
grep SELINUX=disabled /etc/selinux/config 
setenforce 0
getenforce
```



3. ** #2、关闭iptables **
```
chkconfig iptables off
```

```
systemctl disable firewalld.service
```



4. ** #3、精简开机自启动服务**
```
export LANG=en
chkconfig|egrep -v "crond|sshd|network|rsyslog|sysstat"|awk '{print "chkconfig",$1,"off"}'|bash
chkconfig --list|grep 3:on
```



5. ** #4、提权oldboy可以sudo**
```
useradd oldboy
echo 123456|passwd --stdin oldboy
\cp /etc/sudoers /etc/sudoers.ori
echo "oldboy  ALL=(ALL) NOPASSWD: ALL " >>/etc/sudoers
tail -1 /etc/sudoers
visudo -c
```

6. ** #5、英文字符集**
```
cp /etc/sysconfig/i18n /etc/sysconfig/i18n.ori
echo 'LANG="en_US.UTF-8"'  >/etc/sysconfig/i18n
source /etc/sysconfig/i18n
echo $LANG
```


7. ** #6、时间同步**
```
echo '#time sync by oldboy at 2010-2-1' >>/var/spool/cron/root
echo '*/5 * * * * /usr/sbin/ntpdate pool.ntp.org >/dev/null 2>&1' >>/var/spool/cron/root
crontab -l
```



8. ** #7、命令行安全（可不设置）**

#echo 'export TMOUT=300' >>/etc/profile
#echo 'export HISTSIZE=5' >>/etc/profile
#echo 'export HISTFILESIZE=5' >>/etc/profile
#tail -3 /etc/profile
#. /etc/profile


9. ** #8、加大文件描述**

临时设置
ulimit -n 65536
查看
ulimit -n
永久设置
echo '*               -       nofile          65535 ' >>/etc/security/limits.conf 
tail -1 /etc/security/limits.conf


10. ** #9、内核优化**
```
cat >>/etc/sysctl.conf<<EOF
net.ipv4.tcp_fin_timeout = 2
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.ip_local_port_range = 4000    65000
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.route.gc_timeout = 100
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
net.core.somaxconn = 16384
net.core.netdev_max_backlog = 16384
net.ipv4.tcp_max_orphans = 16384
#以下参数是对iptables防火墙的优化，防火墙不开会提示，可以忽略不理。
net.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_tcp_timeout_established = 180
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
EOF
```

sysctl -p



11. ** #10、安装其他小软件**

yum install lrzsz nmap tree dos2unix nc telnet -y

12. ** #11、ssh连接速度慢优化**
```
sed -i.bak 's@#UseDNS yes@UseDNS no@g;s@^GSSAPIAuthentication yes@GSSAPIAuthentication no@g'  /etc/ssh/sshd_config
/etc/init.d/sshd restart
```


4. ** 确认/etc/services**

/etc/services 
# /etc/services:
# $Id: services,v 1.55 2013/04/14 ovasik Exp $
#
# Network services, Internet style
# IANA services version: last updated 2013-04-10
#
# Note that it is presently the policy of IANA to assign a single well-known
# port number for both TCP and UDP; hence, most entries here have two entries
# even if the protocol doesn't support UDP operations.
# Updated from RFC 1700, ``Assigned Numbers'' (October 1994).  Not all ports
# are included, only the more common ones.
#
# The latest IANA port assignments can be gotten from
#       http://www.iana.org/assignments/port-numbers
# The Well Known Ports are those from 0 through 1023.
# The Registered Ports are those from 1024 through 49151
# The Dynamic and/or Private Ports are those from 49152 through 65535
#
# Each line describes one service, and is of the form:
#
# service-name  port/protocol  [aliases ...]   [# comment]

tcpmux          1/tcp                           # TCP port service multiplexer
tcpmux          1/udp                           # TCP port service multiplexer
rje             5/tcp                           # Remote Job Entry
rje             5/udp                           # Remote Job Entry
echo            7/tcp
echo            7/udp






5. **  ****Linux基础优化与安全**

1）不用root 登录管理系统，而以普通用户登录通过sudo 授权管理。
2）更改默认的远程连接SSH 服务端口，禁止root 用户远程连接，甚至要更改SSH 服务只监听内网IP。
3）定时自动更新服务器的时间，使其和互联网时间同步。
4）配置yum 更新源，从国内更新源下载安装软件包。
5）关闭SELinux 及iptables（在工作场景中，如果有外部IP 一般要打开iptables，高并发高流量的服务器可能无法开启）。
6）调整文件描述符的数量，进程及文件的打开都会消耗文件描述符数量。
7）定时自动清理邮件临时目录垃圾文件，防止磁盘的inodes 数被小文件占满（注意Centos6 和
Centos5 要清除的目录不同）。
8）精简并保留必要的开机自启动服务（如crond、sshd、network、rsyslog、sysstat）。
9）Linux 内核参数优化/etc/sysctl.conf，执行sysctl -p 生效。
10）更改系统字符集为“zh_CN.UTF-8”，使其支持中文，防止出现乱码问题。
11）锁定关键系统文件如/etc/passwd、/etc/shadow、/etc/group、/etc/gshadow、/etc/inittab， 处理以上内容后把chattr、lsattr 改名为oldboy，转移走，这样就安全多了。
12）清空/etc/issue、/etc/issue.net，去除系统及内核版本登录前的屏幕显示。
13）清除多余的系统虚拟用户账号。
14）为grub 引导菜单加密码。
15）禁止主机被ping。
16）打补丁并升级有已知漏洞的软件。






# **企业linux运维场景数据同步方案**
## **文件级别同步方案**
scp、NFS、SFTP、http、rsync、csync2、union
http://oldboy.blog.51cto.com/2561410/775056
思想：
    1、文件级别也可以利用mysql，mongodb等软件
    2、两个服务器同时写数据，双写就是一个同步机制
## **文件系统级别同步**
 drbd（基于文件系统同步网络RAID1），同步几乎任何业务数据
    mysql 数据库的官网推荐drbd同步数据，所有单点服务（NFS，MFS、DRBD）
等都可以用drbd



## **数据库同步方案**
### **自身同步机制**
mysql replication， mysql主从复制（逻辑的sql从写）
oracle  dataguar（物理的磁盘块，逻辑的sql语句从写）

### **第三方drbd**
http://oldboy.blog.51cto.com/2561410/1240412
分布式文件系统集群：  NFS、glusterfs、FASTDFS





# **rsync**
6. ** 什么是rsync**

http://rsync.samba.org/documentation.html
http://www.samba.org/ftp/rsync/rsync.html
https://download.samba.org/pub/rsync/rsyncd.conf.html

http://vsftpd.beasts.org/vsftpd_conf.html


[https://download.samba.org/pub/rsync/rsyncd.conf.html](https://download.samba.org/pub/rsync/rsyncd.conf.html)




7. ** **** Rsync的特性（7个特性信息说明）**

①. 支持拷贝普通文件与特殊文件如链接文件，设备等。
②. 可以有排除指定文件或目录同步的功能，相当于打包命令tar的排除功能。
    #tar zcvf backup_1.tar.gz  /opt/data  -exclude=oldboy    
    说明：在打包/opt/data时就排除了oldboy命名的目录和文件。
③. 可以做到保持原文件或目录的权限、时间、软硬链接、属主、组等所有属性均不改变-p。
④. 可实现增量同步，既只同步发生变化的数据，因此数据传输效率很高（tar -N）。
    # 将备份/home 目录自 2008-01-29 以来修改过的文件
    # tar -N 2008-01-29 -zcvf /backups/inc-backup_$(date +%F).tar.gz /home
    # 将备份 /home 目录昨天以来修改过的文件
    # tar -N $(date -d yesterday "+%F") -zcvf /backups/inc-backup_$(date +%F).tar.gz /home
# 添加文件到已经打包的文件
    # tar -rf all.tar *.gif
   说明：这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。
⑤. 可以使用rcp,rsh,ssh等方式来配合进行隧道加密传输文件（rsync本身不对数据加密）
⑥. 可以通过socket(进程方式)传输文件和数据（服务端和客户端）*****。重点掌握
⑦. 支持匿名的或认证（无需系统用户）的进程模式传输，可实现方便安全的进行数据备份及镜像。



8. ** rsync功能**

本地和远程两台主机数据同步和备份
本地相当于cp，rm   远程相当于scp ，cp 和 scp都是全量的， rsync是全量和增量，本地和远程都支持
只有一个操作对象，相当于 ls -l

rsync  当作 cp或者scp  时，可以自动创建一级目录

**rsync  用于存储的实时同步**

9. ** 应用**
13. **  两台服务器之间数据同步  **

cron+ rsync 实现


14. **把所有服务器数据同步到备份服务器**

    生产场景集群架构服务器备份方案
cron+ rsync 实现



15. **  数据实时同步  **

1、rsync + inotify
2、 rsync + sersync








10. ** rsync  参数**

-a, --archive               archive mode; equals -rlptgoD
-v, --verbose               increase verbosity
-z, --compress              compress file data during the transfer  
-R, --relative              use relative path names   使用相对路径名
-l      只传输链接文件
-L    传输链接文件,并将源文件中的内容放入链接文件中
--bwlimit=KBPS          limit I/O bandwidth; KBytes per second
--bwlimit=300

-partial    解决断点续传

-e, --rsh=command        指定使用rsh、ssh方式进行数据同步 

--delete                 删除那些DST中SRC没有的文件 








# **rsync 工作方式**
SYNOPSIS
       Local:  rsync [OPTION...] SRC... [DEST]

       Access via remote shell:
         Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST:DEST


       Access via rsync daemon:
         Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
               rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]

         Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
               rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST


11. ** 本地数据传输模式  （类似cp）**

rsync  [OPTION...]  SRC...  [DEST]

[root@oldboy ~]# cp /etc/hosts /tmp/
[root@oldboy ~]# **rsync /etc/hosts /tmp/**
[root@oldboy ~]# rm -f /tmp/hosts 
[root@oldboy ~]# rsync /etc/hosts /tmp/
[root@oldboy ~]# ll /tmp/hosts 
-rw-r--r-- 1 root root 158 May 21 13:56 /tmp/hosts










16. ** rsync  保持属性**

[root@oldboy ~]#** rsync -avz /etc/hosts /tmp/**
sending incremental file list
hosts

sent 124 bytes  received 31 bytes  310.00 bytes/sec
total size is 158  speedup is 1.02
[root@oldboy ~]# ll /tmp/hosts 
-rw-r--r-- 1 root root 158 Jan 12  2010 /tmp/hosts




17. ** rsync  删除**

[root@oldboy data]# ll
total 8
-rw-r--r--. 1 root root  30 Mar  6 19:44 a.txt
-rw-r--r--. 1 root root   0 Mar  6 19:44 b.txt
-rw-r--r--. 1 root root 118 Mar  6 19:30 oldboy.txt
[root@oldboy ~]# mkdir /null
[root@oldboy ~]#** rsync -r --delete /null/ /data/        说明： null 后面 /**
[root@oldboy ~]# ll /data/
total 0
[root@oldboy ~]# mkdir /null/ddd
[root@oldboy ~]#** rsync -r --delete /null/ /data/**
[root@oldboy ~]# ll /data/
total 4
drwxr-xr-x 2 root root 4096 May 21 14:23 ddd







12. ** 远程**

rsync  scp功能

1、推
rsync [OPTION...] SRC... [USER@]HOST:DEST
[root@oldboy ~]# **rsync /etc/hosts -e 'ssh' oldgirl@10.0.0.33:~/**
The authenticity of host '10.0.0.33 (10.0.0.33)' can't be established.
RSA key fingerprint is 40:48:b0:e7:0f:75:d7:58:d4:ba:a3:5a:ef:3b:a1:71.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.33' (RSA) to the list of known hosts.
oldgirl@10.0.0.33's password: 
[root@oldboy ~]# ll /home/oldgirl/
total 4
-rw-r--r-- 1 oldgirl oldgirl 158 May 21 14:33 hosts
[root@oldboy ~]# **rsync -avz /etc/hosts -e 'ssh' oldgirl@10.0.0.33:~/** 
oldgirl@10.0.0.33's password: 
sending incremental file list
hosts

sent 67 bytes  received 37 bytes  13.87 bytes/sec
total size is 158  speedup is 1.52
[root@oldboy ~]# ll /home/oldgirl/
total 4
-rw-r--r-- 1 oldgirl oldgirl 158 Jan 12  2010 hosts

2、拉
rsync [OPTION...] [USER@]HOST:SRC... [DEST]
[root@oldboy ~]#** rsync -avz  -e 'ssh' oldgirl@10.0.0.33:~/hosts /tmp/**
oldgirl@10.0.0.33's password: 
receiving incremental file list

sent 14 bytes  received 57 bytes  20.29 bytes/sec
total size is 158  speedup is 2.23
[root@oldboy ~]# ll /tmp/
total 4
-rw-r--r-- 1 oldgirl oldgirl 158 Jan 12  2010 hosts


13. **以守护进程（socket）的方式传输数据（重点）**

备份拓扑
![图片](https://images-cdn.shimo.im/xrw62em5cgQOID5S/image.image/png!thumbnail)







18. **  安装rsync**

yum install rsync -y

19. **  配置文件（服务端）**

**/etc/rsyncd.conf**
```
uid = rsync
gid = rsync
use chroot = no
max connections = 200
timeout = 300
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[backup]
path = /backup/
ignore errors
read only = false
list = false
hosts allow = 172.16.1.0/24
auth users = rsync_backup
secrets file = /etc/rsync.password
```





fake super = yes  #rsync3.1配置

**/etc/rsyncd.conf**
```
uid = rsync
gid = rsync
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
ignore errors
read only = false
list = false
hosts allow = 172.16.1.0/24
auth users = rsync_backup
secrets file = /etc/rsync.password
[backup]
path = /backup
```




20. **  rsync启动**
```
rsync  --daemon   --config=FILE
```

```
systemctl start rsyncd
```


21. **  添加到开机启动**

echo "/usr/bin/rsync --daemon" >> /etc/rc.local


22. **  rsync 端口**

[root@oldboy ~]# netstat -lntup |grep 873
tcp        0      0 0.0.0.0:**873**                 0.0.0.0:*                   LISTEN      3939/rsync          
tcp        0      0 :::**873**                      :::*                        LISTEN      3939/rsync


23. ** 创建同步目录、rsync虚拟用户、设置权限、添加用户密码文件**
```

mkdir /oldboy
useradd rsync -s /sbin/nologin -M
chown -R rsync.rsync /oldboy
echo "rsync_backup:123" > /etc/rsync.password
chmod 600 /etc/rsync.password 

ll /etc/rsync.password 
-rw------- 1 root root 17 May 21 17:38 /etc/rsync.password
```









24. **  客户端配置**

[root@client01 ~]# rpm -qa rsync
rsync-3.0.6-12.el6.x86_64

配置密码文件
[root@client01 ~]# echo "123" >/etc/rsync.password             
[root@client01 ~]# cat /etc/rsync.password 
123
[root@client01 ~]# chmod 600 /etc/rsync.password

rsync密码变量
export RSYNC_PASSWORD=123456




**同步执行命令都是在客户端**
Access via rsync daemon:
Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
     **rsync -avz rsync_backup@10.0.0.41::oldboy  /data1 --password-file=/etc/rsync.password**
     rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
     **rsync -avz rsync://rsync_backup@10.0.0.41/oldboy /data1 --password-file=/etc/rsync.password**

Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
   **rsync -avz /data1/ rsync_backup@10.0.0.41::oldboy  --password-file=/etc/rsync.password**
     rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST
 **rsync -avz /data1/ rsync://rsync_backup@10.0.0.41/oldboy  --password-file=/etc/rsync.password**
拉同步：
[root@client01 ~]#** rsync -avz rsync_backup@10.0.0.41::oldboy  /data1    ****oldboy为服务端配置文件中的模块**
Password: 
receiving incremental file list
./
test/
test/a/
test/a/b/
test/a/b/c/

sent 83 bytes  received 179 bytes  74.86 bytes/sec
total size is 0  speedup is 0.00

[root@client01 ~]#** rsync -avz rsync_backup@10.0.0.41::oldboy  /data1 --password-file=/etc/rsync.password **
receiving incremental file list

sent 68 bytes  received 164 bytes  154.67 bytes/sec
total size is 0  speedup is 0.00








25. **测试：**

服务器端：
[root@oldboy ~]# cd /oldboy/
[root@oldboy oldboy]# 
[root@oldboy oldboy]# ll
total 4
drwxr-xr-x 3 root root 4096 May 21 18:04 test
[root@oldboy oldboy]# touch {a..g}
[root@oldboy oldboy]# ls
a  b  c  d  e  f  g  test


客户端：
[root@client01 ~]# **rsync -avz rsync_backup@10.0.0.41::oldboy  /data1 --password-file=/etc/rsync.password** 
receiving incremental file list
./
a
b
c
d
e
f
g

sent 204 bytes  received 484 bytes  1376.00 bytes/sec
total size is 0  speedup is 0.00
[root@client01 ~]# ls /data1/
a  b  c  d  e  f  g  test







推同步：
客服端：
[root@client01 ~]# touch /data1/{1..10}
[root@client01 ~]# **rsync -avz /data1/ rsync_backup@10.0.0.41::oldboy  --password-file=/etc/rsync.password**        
sending incremental file list
./
1
10
2
3
4
5
6
7
8
9
sent 604 bytes  received 205 bytes  1618.00 bytes/sec
total size is 0  speedup is 0.00


服务端：
[root@oldboy oldboy]# ls /oldboy/
1  10  2  3  4  5  6  7  8  9  a  b  c  d  e  f  g  test

26. ** 小结**

rsync server：
1、vim  /etc/rsyncd.conf  (用户，目录，模块，虚拟用户及密码文件)
2、 创建共享目录  /oldboy
3、创建rsync用户并且授权访问/oldboy
4、创建密码文件，复制配置文件里的路径，然后添加密码内容
 内容虚拟用户名:密码
5、密码文件权限600
6、rsync  --daemon 然后放入 /etc/rc.local
7、tail  /var/log/rsyncd.log



rsync  client (多个)
1、密码文件和服务端没任何关系
  --password-file=/etc/rsync.password  内容：密码
2、/etc/rsync.password     权限  600

3、同步：
   Access via rsync daemon:
Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
     **rsync -avz rsync_backup@10.0.0.41::oldboy  /data1 --password-file=/etc/rsync.password**
     rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
     **rsync -avz rsync://rsync_backup@10.0.0.41/oldboy /data1 --password-file=/etc/rsync.password**

Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
   **rsync -avz /data1/ rsync_backup@10.0.0.41::oldboy  --password-file=/etc/rsync.password**
     rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST
 **rsync -avz /data1/ rsync://rsync_backup@10.0.0.41/oldboy  --password-file=/etc/rsync.password**












27. **  rsync排除**

排除单个文件
[root@client01 data1]#** rsync -avz ****--exclude=a**** /data1/ rsync://rsync_backup@10.0.0.33/oldboy  --password-file=/etc/rsync.passwordsending incremental file list**
./
1
10
2
3
4
5
6
7
8
9
b
c
d
e
f
g
oldboy
test/
sent 858 bytes  received 338 bytes  2392.00 bytes/sec
total size is 0  speedup is 0.00


排除多个文件或目录
```
rsync -avz /oldboy/  --exclude=d  --exclude=c/4.txt  rsync_backup@172.16.1.41::backup  --password-file=/etc/rsync.password
```






28. **  rsync  排除多个文件**

一、 客户端排除
1、
rsync -avz --exclude={a,b} /data1/ rsync://rsync_backup@10.0.0.33/oldboy  --password-file=/etc/rsync.password

2、
[root@client01 ~]# seq 10 > paichu.txt
[root@client01 ~]# cat paichu.txt 
1
2
3
4
5
6
7
8
9
10

```
rsync -avz --exclude-from=/root/paichu.txt /data1/ rsync://rsync_backup@10.0.0.33/oldboy  --password-file=/etc/rsync.password    
```
sending incremental file list
./
a
b
c
d
e
f
g
oldboy
test/
test/a/
test/a/b/
test/a/b/c/
sent 466 bytes  received 179 bytes  258.00 bytes/sec
total size is 0  speedup is 0.00


二、服务端排除
/etc/rsyncd.conf  配置文件中添加
**exclude=a b**

29. ** rsync 无差异同步**

--delete
rsync -avz **--delete** /data1/  rsync://rsync_backup@10.0.0.33/oldboy  --password-file=/etc/rsync.password

rsync推送企业工作场景：
1、备份  --delete 风险
本地有啥，远端有啥
本地没有的远端有也要删除

rsync拉取企业工作场景：
1、代码发布，下载  --delete 风险
远端有啥，本地（客户端）就有啥
远端没有的本地有也要删除



















14. ** rsync共享多个目录**

[root@backup data]# cat** /etc/rsyncd.conf**
```
uid = rsync
gid = rsync
use chroot = no
max connections = 200
timeout = 300
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
ignore errors
read only = false
list = false
hosts allow = 172.16.1.0/24
#hosts deny = 0.0.0.0/32
auth users = rsync_backup
secrets file = /etc/rsync.password
[backup]
path = /backup/
[data]
path = /data/
```











# **xinetd服务启动rsync守护进程**
15. **检查安装xinetd服务**

rpm -qa |grep xinetd

yum install -y xinetd


16. ** 修改配置文件**

/etc/xinetd.d/rsync
disable = yes    修改为  disable = no
17. ** 启动**

/etc/init.d/xinetd start

# **rsync优缺点**
18. ** rsync优点**

1、增量备份，支持socket（daemon），集中备份（支持推拉，都是以客户端为参照物）
2、远程shell通道模式还可以加密（ssh）传输，socket（daemon）需要加密传输，可以
   利用vpn服务或ipsec服务
19. ** rsync缺点**

1、大量小文件同步的时候，对比时间较长，有时候，rsync进程可能会停止
2、同步大文件，10G这样的大文件有事也会出问题，中断，未完整同步前，是隐藏文件，
   可以通过续传(--partial)等参数实现传输
   rsync 3.0.9版本，推送大文件时，发生过服务器端文件系统只读问题（BUG）
3、  一次性远程拷贝可以用scp  打包后再拷贝

# **rsync帮助**
http://rsync.samba.org/
http://rsync.samba.org/ftp/rsync/rsync.html
http://rsync.samba.org/ftp/rsync/rsyncd.conf.html
man rsync
man rsyncd.conf

http://mirrors.ustc.edu.cn/help/rsync-guide.html



# rsync指定配置文件运行
[root@backup ~]# vim /usr/lib/systemd/system/rsyncd.service 
```
[Unit]
Description=fast remote file copy program daemon
ConditionPathExists=/tmp/rsyncd.conf

[Service]
EnvironmentFile=/etc/sysconfig/rsyncd
ExecStart=/usr/bin/rsync --daemon --config=/tmp/rsyncd.conf --no-detach "$OPTIONS"

[Install]
WantedBy=multi-user.target
```



[root@backup ~]# vim /tmp/rsyncd.conf
```
uid = rsync
gid = rsync
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
ignore errors
read only = false
list = true
hosts allow = 172.16.1.0/24
auth users = rsync_backup
secrets file = /etc/rsync.password
[backup]
path = /backup
[wuxing]
path = /wuxing
```


# rsync无认证
[root@backup backup]# vim /tmp/rsyncd.conf 
```
uid = rsync
gid = rsync
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
ignore errors
read only = false
list = true
hosts allow = 172.16.1.0/24
[backup]
path = /backup
[wuxing]
path = /wuxing
```



# rsync使用其它端口


## 服务端
[root@backup backup]# vim /tmp/rsyncd.conf 
```
uid = rsync
gid = rsync
port = 874
fake super = yes
use chroot = no
max connections = 200
timeout = 600
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
ignore errors
read only = false
list = true
hosts allow = 172.16.1.0/24
#auth users = rsync_backup
#secrets file = /etc/rsync.password
[backup]
path = /backup
[wuxing]
path = /wuxing
```



## 客户端
```
[root@nfs01 ~]# rsync -avz  rsync://172.16.1.41:874/backup 
receiving incremental file list
drwxr-xr-x             20 2019/06/12 17:10:54 .
-rw-r--r--          1,267 2019/03/07 17:58:23 passwd

sent 24 bytes  received 88 bytes  224.00 bytes/sec
total size is 1,267  speedup is 11.31
```


```
[root@nfs01 ~]# rsync -avz  --port=874 172.16.1.41::backup 
receiving incremental file list
drwxr-xr-x             20 2019/06/12 17:10:54 .
-rw-r--r--          1,267 2019/03/07 17:58:23 passwd

sent 24 bytes  received 88 bytes  224.00 bytes/sec
total size is 1,267  speedup is 11.31
```









# **同步公网yum源**

```
/usr/bin/rsync -az --bwlimit=1000 rsync://mirrors.ustc.edu.cn/centos/6/os/x86_64/  /data/yum_data/centos/6/os/x86_64/
```


```
/usr/bin/rsync -az --bwlimit=1000 rsync://mirrors.ustc.edu.cn/epel/6/x86_64/ /data/yum_data/epel/6/x86_64/
```



# **全网备份**
备份全网服务器数据生产架构方案案例模型
企业案例：rsync上机实战考试题： 
某公司里有一台Web服务器，里面的数据很重要，但是如果硬盘坏了，数据就会丢失，现在领导要求你把数据在其他机器上做一个周期性定时备份。要求如下：
每天晚上00点整在Web服务器A上打包备份网站程序目录并通过rsync命令推送到服务器B上备份保留（备份思路可以是先在本地按日期打包，然后再利用rsync推到备份服务器上）。
具体要求如下：
所有服务器的备份目录必须都为/backup
要备份的系统配置文件包括但不限于：
  a)	定时任务服务的配置文件		(/var/spool/cron/root)	（适合web和nfs服务器）。
  b)	开机自启动的配置文件			(/etc/rc.local)			（适合web和nfs服务器）。
  c)	日常脚本的目录				(/server/scripts)。
  d)	防火墙iptables的配置文件	(/etc/sysconfig/iptables)。
  e)	自己思考下还有什么需要备份呢？
Web服务器站点目录假定为(/var/html/www)。
Web服务器访问日志路径假定为（/app/logs）
Web服务器保留打包后的7天的备份数据即可(本地留存不能多于7天，因为太多硬盘会满)
备份服务器上,保留最近7天的备份数据，同时保留6个月内每周一的所有数据副本
备份服务器上,要按照备份数据服务器的内网IP为目录保存备份，备份的文件按照时间名字保存。
需要确保备份的数据尽量完整正确，在备份服务器上对备份的数据进行检查，把备份的成功及失败结果信息发给系统管理员邮箱中




20. **备份脚本演变过程**

backup_rsync.sh 
#!/bin/bash

mkdir -p /backup/${HOSTNAME}_$(hostname -i)_$(date +%F)
cp -a /etc/passwd /etc/fstab /etc/rc.d/rc.local /var/spool/cron /backup/${HOSTNAME}_$(hostname -i)_$(date +%F)
rsync -avz /backup/ rsync_backup@backup::backup --password-file=/etc/rsync.password


backup_rsync.sh 
#!/bin/bash

Path=/backup
Host=$(hostname -i)
Date=$(date +%F)
Dest=${Path}/${HOSTNAME}_${Host}_${Date}

mkdir -p $Dest
cp -a /etc/passwd /etc/fstab /etc/rc.d/rc.local /var/spool/cron $Dest
rsync -avz ${Path}/ rsync_backup@backup::backup --password-file=/etc/rsync.password


backup_rsync.sh 
#!/bin/bash

Path=/backup
Host=$(hostname -i)
Date=$(date +%F)
Dest=${Path}/${HOSTNAME}_${Host}_${Date}
Rsync_User=rsync_backup
Rsync_Addr=backup
Rsync_Module=backup
export RSYNC_PASSWORD=123456

[ -d $Dest ] || mkdir -p $Dest
cp -a /etc/passwd /etc/fstab /etc/rc.d/rc.local /var/spool/cron $Dest/
cp -a /server $Dest/
cp -a /etc/rsyncd.conf $Dest/
rsync -avz ${Path}/ ${Rsync_User}@${Rsync_Addr}::${Rsync_Module}



for i in {1..24};do date -s "2018/07/$i" && bash /server/scripts/backup_rsync.sh;done







rsync环境变量
```
strings /usr/bin/rsync |grep RSYNC_
```

```
RSYNC_MODULE_NAME
RSYNC_HOST_NAME
RSYNC_HOST_ADDR
RSYNC_USER_NAME
RSYNC_MODULE_PATH
RSYNC_PASSWORD
```




backup_rsync.sh 
```
#!/bin/bash

Path=/backup
Host=$(hostname -i)
Date=$(date +%F)
Dest=${Path}/${HOSTNAME}_${Host}_${Date}
Rsync_User=rsync_backup
Rsync_Addr=backup
Rsync_Module=backup
export RSYNC_PASSWORD=123456

[ -d $Dest ] || mkdir -p $Dest
cp -a /etc/passwd /etc/fstab /etc/rc.d/rc.local /var/spool/cron $Dest/
cp -a /server $Dest/
cp -a /etc/rsyncd.conf $Dest/
rsync -avz ${Path}/ ${Rsync_User}@${Rsync_Addr}::${Rsync_Module}
find ${Path}/ -type d -name "*$(Host)*" -mtime +7 -exec rm -fr {} \;
```


```

#!/bin/bash

RSYNC_MODULE_NAME="backup"
RSYNC_HOST_NAME="backup"
IP="$(hostname -I |awk '{print $2}')"
RSYNC_USER_NAME="rsync_backup"
RSYNC_PASSWORD="123"
BACK_PATH="/backup"
DATE="$(date +%F)"
WEEK="$(date +%w)"

mkdir $BACK_PATH/$IP -p
cd /
tar zcfh $BACK_PATH/$IP/${HOSTNAME}_bak_${DATE}_week$WEEK.tar.gz etc/rc.local var/spool/cron/ server/scripts/
md5sum $BACK_PATH/$IP/*.tar.gz > $BACK_PATH/$IP/md5.txt
rsync -az $BACK_PATH/$IP $RSYNC_USER_NAME@$RSYNC_HOST_NAME::$RSYNC_MODULE_NAME
find $BACK_PATH -type f -mtime +7 -name "*.tar.gz" ! -name "*week1.tar.gz" -delete
```






21. **全网备份完整**

[root@web01 scripts]# cat** bak.sh** 
```
#/bin/sh
source /etc/profile
if [ `date +%w` -eq 0 ];then
   DATE="$(date +%F)_sun"
else
   DATE=$(date +%F)
fi

IP=$(ifconfig eth1|awk -F  '[ :]+' 'NR==2{print $4}')
[ ! -d /backup/$IP ] && mkdir -p /backup/$IP
#bak
cd /&&\
tar zcfh /backup/$IP/backup_${DATE}.tar.gz var/html/www/ app/logs/ etc/rc.local server/scripts/ var/spool/cron/ &&\
find /backup -type f -name "back*${DATE}.tar.gz"|xargs md5sum >/backup/$IP/${DATE}_${IP}_flag.tar.gz
#bak to bak server
rsync -az /backup/ rsync_backup@172.16.1.41::backup/ --password-file=/etc/rsync.password 
#del before 7days
find /backup -type f -name "*.tar.gz" -mtime +7|xargs rm -f
```



00 00 * * * /bin/sh /server/scripts/bak.sh >/dev/null 2>&1



22. **验证文件（md5sum）**

[root@backup 172.16.1.8]# **md5sum -c 2016-05-29_172.16.1.8_flag.tar.gz** 
/backup/172.16.1.8/backup_2016-05-29.tar.gz: 确定

[root@backup 172.16.1.8]# find /backup/ -type f -name "*$(date +%F)*_flag*"|xargs md5sum -c
/backup/172.16.1.8/backup_2016-05-29.tar.gz: OK




23. **邮件配置两种方法**
30. ** 方法1**

修改/etc/mail.rc最后一行加入
/etc/mail.rc
```
set from=740256835@qq.com
set smtp=smtps://smtp.qq.com:465
set smtp-auth-user=740256835@qq.com
set smtp-auth-password=ihppxhssqsiqbfie
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/
```



31. ** 方法2**

[root@backup ~]# /etc/init.d/postfix start
Starting postfix:                                          [  OK  ]
[root@backup ~]# /etc/init.d/postfix status
master (pid  1566) is running...

注意两种方法只能配置一种，不能同时配置




发邮件
```
mail -s "host" 1226032602@qq.com < /etc/hosts
```




docker容器中启动postfix服务
[root@backup ~]# grep "^inet_" /etc/postfix/main.cf
inet_interfaces = localhost
inet_protocols = all
改为
[root@backup ~]# grep "^inet_" /etc/postfix/main.cf
inet_interfaces = all
inet_protocols = all



![图片](https://images-cdn.shimo.im/9zG1klUaUN4gVFd2/image.image/png!thumbnail)










24. **发邮件**

[root@backup scripts]# cat check.sh 
find /backup/ -type f -name "*$(date +%F)*_flag*"|xargs md5sum -c >/tmp/$(date +%F)_mail.log
mail -s "$(date) mail" 1226032602@qq.com </tmp/$(date +%F)_mail.log

03 00 * * * /bin/sh /server/scripts/check.sh >/dev/null 2>&1



25. ** 实验成功邮件**

![图片](https://images-cdn.shimo.im/uVsW3I1AYrEpdIUA/image.image/png!thumbnail)



26. ** 备份服务器上删除180天以前**

[root@backup scripts]# cat del.sh 
find /backup/ -type f -name "backup*.tar.gz" ! -name "*sun*" -mtime +180|xargs rm -f 

00 06 * * * /bin/sh /server/scripts/del.sh >/dev/null 2>&1






27. ** 本地打包并推送脚本**

[root@web01 scripts]# cat backup_rsync.sh 
#!/bin/bash
. /etc/profile

IP="`ifconfig eth1|awk -F '[ :]+' 'NR==2{print $4}'`"
BAK_PATH="/backup"
DATE1=`date +%u -d "-1day"`
DATE2=`date +%F -d "-1day"`
if [ $DATE1 -eq 1 ];then
   DATE=${DATE2}_MON
else
   DATE=$DATE2
fi

[ ! -d ${BAK_PATH}/$IP ] && mkdir -p ${BAK_PATH}/$IP &> /dev/null

#tar
cd / && \
tar zcfh ${BAK_PATH}/$IP/bak_${DATE}.tar.gz var/spool/cron/root etc/rc.local server/scripts etc/sysconfig/iptables boot/grub/grub.conf var/html/www app/logs

find $BAK_PATH -type f -name "*${DATE}.tar.gz"|xargs md5sum > $BAK_PATH/$IP/bak_${DATE}.flag


#rsync
rsync -az /$BAK_PATH/$IP/ rsync_backup@backup::backup/$IP/ --password-file=/etc/rsync.password

#delete
find /$BAK_PATH/$IP/ -type f -mtime +7|xargs rm -f






28. ** 备份服务器发邮件脚本**

[root@backup scripts]# cat mail_del180.sh 
```
#!/bin/bash
. /etc/profile

IP="`ifconfig eth1|awk -F '[ :]+' 'NR==2{print $4}'`"
BAK_PATH="/backup"
DATE1=`date +%u -d "-1day"`
DATE2=`date +%F -d "-1day"`
if [ $DATE1 -eq 1 ];then
   DATE=${DATE2}_MON
else
   DATE=$DATE2
fi

[ ! -d ${BAK_PATH}/$IP ] && mkdir -p ${BAK_PATH}/$IP &> /dev/null

find $BAK_PATH -type f -name "*${DATE}*flag"|xargs md5sum -c > /$BAK_PATH/mail_${DATE}.log

mail -s "bak_check_${DATE}" 1226032602@qq.com < /$BAK_PATH/mail_${DATE}.log 

find $BAK_PATH -type f  ! -name "*MON.tar.gz" -mtime +180 |xargs rm -f
```




---


# lsyncd (实时备份)
[https://axkibe.github.io/lsyncd/](https://axkibe.github.io/lsyncd/)
[https://axkibe.github.io/lsyncd/manual/config/file/](https://axkibe.github.io/lsyncd/manual/config/file/)

[https://www.cnblogs.com/zxci/p/6243574.html](https://www.cnblogs.com/zxci/p/6243574.html)


```
yum install lsyncd
```

模板
```
ll /usr/share/doc/lsyncd*/examples/
```


远程目录同步，rsync + rsyncd daemon

/etc/lsyncd.conf
```
settings {
 logfile = "/var/log/lsyncd/lsyncd.log",
 statusFile = "/var/log/lsyncd/lsyncd.status",
 inotifyMode = "CloseWrite",
 maxProcesses = 8,
}
sync {
 default.rsync,
 source = "/tmp/src",
 target = "rsync_backup@172.16.1.41::backup",
 delete="running",
 exclude = { ".*", ".tmp" },
 delay = 3,
 init = false,
rsync = {
 binary = "/usr/bin/rsync",
 archive = true,
 compress = true,
 verbose = true,
 password_file = "/etc/rsync.pwd",
 _extra = {"--bwlimit=200"}
}
}
sync {
 default.rsync,
 source = "/tmp/src",
 target = "rsync_backup@172.16.1.41::wuxing",
 delete="running",
 exclude = { ".*", ".tmp" },
 delay = 3,
 init = false,
rsync = {
 binary = "/usr/bin/rsync",
 archive = true,
 compress = true,
 verbose = true,
 password_file = "/etc/rsync.pwd",
 _extra = {"--bwlimit=200"}
}
}
```


delete
```
delete	=	true       #在目标上删除源中没有的内容。在启动时以及在正常操作期间删除的内容
delete	=	false      #不会删除目标上的任何文件。不在启动时也不在正常操作上
delete	=	'startup'
delete	=	'running'
```







```
settings {
 logfile = "/var/log/lsyncd/lsyncd.log",
 statusFile = "/var/log/lsyncd/lsyncd.status",
 inotifyMode = "CloseWrite",
 maxProcesses = 8,
}
sync {
 default.rsync,
 source = "/data",
 target = "rsync_backup@backup::data",
 delete="startup",
 exclude = { ".*" },
 delay = 1,
rsync = {
 binary = "/usr/bin/rsync",
 archive = true,
 compress = true,
 verbose = true,
 password_file = "/etc/rsync.pwd",
 _extra = {"--bwlimit=200"}
}
}
```




























