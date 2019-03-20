## Zabbix代理服务器
### 一：代理概述

zabbix proxy 可以代替 zabbix server 收集性能和可用性数据,然后把数据汇报给 zabbix server,并且在一定程度上分担了zabbix server 的压力.

此外，当所有agents和proxies报告给一个Zabbix server并且所有数据都集中收集时，proxy 收集到数据之后，首先将数据缓存在本地,然后在一定得时间之后传递给 zabbix server，这样就不会因为服务器的任何临时通信问题而丢失数据。使用proxy是实现集中式和分布式监控的最简单方法。

zabbix proxy 使用场景:

    监控远程区域设备

    监控本地网络不稳定区域

    当 zabbix 监控上千设备时,使用它来减轻 server 的压力

    简化分布式监控的维护
![image67](https://github.com/now521/zabbix_picture/blob/master/67.png)  
zabbix proxy 仅仅需要一条 tcp 连接到 zabbix server,外网连接防火墙上仅仅需要加上一条规则即可。
![image68](https://github.com/now521/zabbix_picture/blob/master/68.png)
## 二、安装代理服务器Proxy
### 安装环境：  
1、系统环境：CentOS Linux release 7.5.1804 (Core) 
2、zabbix版本：zabbix-release-3.4-2.el7.noarch
3、测试环境，关闭了防火墙（生产环境不建议关闭，根据需求设置防火墙）
```
[root@centos78 ~]# systemctl stop firewlld.service       关闭防火墙
[root@centos78 ~]# systemctl disable firewalld.service  开机禁用防火墙启动
```
4、关闭Selinux
```
[root@centos78 ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
[root@centos78 ~]# setenforce 0
```
### 安装数据库

1、指定下载数据库版本最好跟zabbix-server一致，编辑安装包路径下载路径：
```
[root@centos78 ~]# vim /etc/yum.repos.d/base.repo  （没有base.repo可以自己创建）
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck = 1
```
2、安装10.2的mariadb
```
yum install mariadb-server
```
3、设置mariadb
```
[root@centos78 ~]# systemctl start mariadb  启动
[root@centos78 ~]# systemctl enable mariadb 设置开机启动
[root@centos78 ~]# systemctl status mariadb   查看启动状态
```
### 安装和设置Proxy服务端

1、下载和安装Zabbix
```
[root@centos78 ~]# rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm  （下载Zabbix最新版本）
[root@centos78 ~]# yum install zabbix-proxy-mysql -y
```
2、创建数据和导入数据
```
[root@centos78 ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.2.17-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> create database zabbix_proxy  character set utf8 collate utf8_bin;     创建数据库zabbix_proxy
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'zabbix';  设置zabbix_proxy权限和密码
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit    退出
Bye
```
### 导入数据库
```
[root@centos78 ~]# zcat /usr/share/doc/zabbix-proxy-mysql-3.4.14/schema.sql.gz |mysql -uzabbix -pzabbix zabbix_proxy 
```
配置数据库用户和密码
```
[root@centos78 ~]# vim /etc/zabbix/zabbix_proxy.conf   修改配置文件，设置密码
DBPassword=zabbix
[root@centos78 ~]# grep -n '^'[a-Z] /etc/zabbix/zabbix_proxy.conf 查看关键配置信息
24:Server=192.168.1.1               这里是Zabbix服务器的ip地址
42:Hostname=centos78            这里是proxy本身的主机名
84:LogFile=/var/log/zabbix/zabbix_proxy.log
95:LogFileSize=0
136:PidFile=/var/run/zabbix/zabbix_proxy.pid
146:SocketDir=/var/run/zabbix
166:DBName=zabbix_proxy
181:DBUser=zabbix
190:DBPassword=zabbix
390:SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
466:Timeout=4
508:ExternalScripts=/usr/lib/zabbix/externalscripts
544:LogSlowQueries=3000
```
启动zabbix和设置开机启动
```
[root@centos78 ~]# systemctl start zabbix-proxy
[root@centos78 ~]# systemctl enable zabbix-proxy
```
3、配置zabbix_proxy.conf文件
```
sed -i.ori '190a DBPassword=zabbix' /etc/zabbix/zabbix_proxy.conf                第190行插入DBPassword，
sed -i 's#Server=127.0.0.1#Server=192.168.1.1#' /etc/zabbix/zabbix_proxy.conf       这个是Zabbix server的IP地址（或主机名），不是Zabbix proxy的ip哦
sed -i 's#Hostname=Zabbix proxy#Hostname=centos78#' /etc/zabbix/zabbix_proxy.conf    这个Zabbix proxy的hostname，唯一的, 区分大小写的，确保server端知道其名称!允许的符号: 字母数字, '.', ' ', '_' 和 '-'。最大长度: 64，经常会在这里出错。
```
### 修改完后重启服务：
```
[root@centos78 ~]# systemctl restart zabbix-proxy.service
```
### 检查启动情况：
```
[root@centos78 ~]#  netstat -lntup |grep zabbix

tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      2190/zabbix_agentd  

tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      12789/zabbix_proxy  

tcp6       0      0 :::10050                :::*                    LISTEN      2190/zabbix_agentd  

tcp6       0      0 :::10051                :::*                    LISTEN      12789/zabbix_proxy  
```
## 三、客户端主机配置文件指向 proxy
```
[root@centos78 ~]#vim /etc/zabbix/zabbix_agentd.conf 

Server=192.168.1.78     这里指向proxy服务器IP,通过代理收集信息。

ServerActive=192.168.1.78

Hostname=centos78
```
## 四、服务端web界面：添加agent代理程序
![image69](https://github.com/now521/zabbix_picture/blob/master/69.png)  
几分钟后检测到代理服务器：
![image70](https://github.com/now521/zabbix_picture/blob/master/70.png)  
再创建一个自动发现规则： 
![image71](https://github.com/now521/zabbix_picture/blob/master/71.png)   
通过自动发现，就可以自动通过代理的客户端添加到自定义分组中，详细情况下面教程：
https://blog.51cto.com/allmrys/2287389  
