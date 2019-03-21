# 一、系统环境
yum update升级以后的系统版本为
```
[root@yl-web yl]# cat /etc/redhat-release 
CentOS Linux release 7.1.1503 (Core) 
```
# 二、mysql安装

一般网上给出的资料都是
```
#yum install mysql
#yum install mysql-server
#yum install mysql-devel
```
安装mysql和mysql-devel都成功，但是安装mysql-server失败，如下：
```

[root@yl-web yl]# yum install mysql-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sina.cn
 * extras: mirrors.sina.cn
 * updates: mirrors.sina.cn
No package mysql-server available.
Error: Nothing to do

```

查资料发现是CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用mariadb代替了。

有两种解决办法：
###1、方法一：安装mariadb

MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。

安装mariadb，大小59 M。
```
[root@yl-web yl]# yum install mariadb-server mariadb 
```
mariadb数据库的相关命令是：
```

systemctl start mariadb  #启动MariaDB

systemctl stop mariadb  #停止MariaDB

systemctl restart mariadb  #重启MariaDB

systemctl enable mariadb  #设置开机启动
```

所以先启动数据库
```
[root@yl-web yl]# systemctl start mariadb
```
然后就可以正常使用mysql了
```

[root@yl-web yl]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.41-MariaDB MariaDB Server

Copyright (c) 2000, 2014, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> 

```

安装mariadb后显示的也是 MariaDB [(none)]> ，可能看起来有点不习惯。下面是第二种方法。  
###2、方法二：官网下载安装mysql-server
```
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum install mysql-community-server
```
安装成功后重启mysql服务。
```
service mysqld restart
```
初次安装mysql，root账户没有密码。
```
[root@yl-web yl]# mysql -u root 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)

mysql> 
```
设置密码
```
mysql> set password for 'root'@'localhost' =password('password');
Query OK, 0 rows affected (0.00 sec)
mysql> 
```
不需要重启数据库即可生效。

在mysql安装过程中如下内容：
```
Installed:
  mysql-community-client.x86_64 0:5.6.26-2.el7                mysql-community-devel.x86_64 0:5.6.26-2.el7                
  mysql-community-libs.x86_64 0:5.6.26-2.el7                  mysql-community-server.x86_64 0:5.6.26-2.el7               

Dependency Installed:
  mysql-community-common.x86_64 0:5.6.26-2.el7                                                                            

Replaced:
  mariadb.x86_64 1:5.5.41-2.el7_0          mariadb-devel.x86_64 1:5.5.41-2.el7_0   mariadb-libs.x86_64 1:5.5.41-2.el7_0  
  mariadb-server.x86_64 1:5.5.41-2.el7_0  
```
所以安装完以后mariadb自动就被替换了，将不再生效。
```
[root@yl-web yl]# rpm -qa |grep mariadb
```
## 方法三 先卸载在安装
### 1、卸载mariadb，否则安装mysql会出现冲突
执行命令
```
rpm -qa | grep mariadb
```
列出所有被安装的mariadb rpm 包；
执行命令
rpm -e --nodeps 包名称（比如：rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64）
逐个将所有列出的mariadb rpm 包给卸载掉
### 2、添加官方的yum源
以centos7安装mysql5.6为例：
创建并编辑mysql-community.repo文件
```
vi /etc/yum.repos.d/mysql-community.repo
```
将以下内容粘贴进去并保存
```
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```
注意：gpgcheck是GPG加密校验，官方文档中,值为1，但check会报错误，所以这里改为0跳过检查，对安装无影响。
同理，其他centos版本安装其他版本的mysql只需要改为对应的baseurl即可：
centos7安装mysql5.7：baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
centos6安装mysql5.6：baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
centos6安装mysql5.7：baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
### 3、安装
执行命令
```
sudo yum install mysql-community-server
```
### 4、启动
执行命令
```
sudo service mysqld start
```
### 5、改mysql 的root密码
```
mysqladmin -u root -p password 你的新密码
```
初始密码为空，直接按回车即可
注意：mysql5.7的初始密码是随机生成的，放在了 /var/log/mysqld.log
## 三、配置mysql
### 1、编码
mysql配置文件为/etc/my.cnf
最后加上编码配置
```
[mysql]
innodb_file_per_table=1
default-character-set =utf8
```
这里的字符编码必须和/usr/share/mysql/charsets/Index.xml中一致。
![image39](https://github.com/now521/zabbix_picture/blob/master/39.png)
### 2、远程连接设置
把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户。
```
mysql> grant all privileges on *.* to root@'%'identified by 'password';
```
如果是新用户而不是root，则要先新建用户
```
mysql>create user 'username'@'%' identified by 'password';  
```
此时就可以进行远程连接了。
创建zabbix数据库并把导入一些sql表
#进入数据库
```
mysql
mysql>CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;
mysql>GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost IDENTIFIED BY 'zabbix';
mysql>SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| zabbix             |
+--------------------+
4 rows in set (0.00 sec)
```
### 3、安装php5.6和Apache服务
#获取yum源
```
rpm -ivh http://repo.webtatic.com/yum/el6/latest.rpm
```
#安装下列所有包
```
yum -y install httpd php56w php56w-gd php56w-mysqlnd php56w-bcmath php56w-mbstring php56w-xml php56w-ldap
```
编辑php的ini文件（vim /etc/php.ini）并修改一下内容，注意date.timezone一定要写对，否则在配置完zabbix后，显示的界面全部报错
```
post_max_size = 16M
max_execution_time = 300
max_input_time = 300
date.timezone = Asia/Shanghai
always_populate_raw_post_data = -1
```
配置/etc/httpd/conf/httpd.conf
```
DocumentRoot "/var/www/html/zabbix"
<Directory "/var/www/html/zabbix">
ServerName 127.0.0.1
DirectoryIndex index.html index.html.var index.php
```
### 4、创建zabbix用户和组
```
groupadd zabbix
useradd -g zabbix zabbix
```
### 5、yum安装zabbix
引导zabbix4.1.1的yum源
rpm -ivh http://repo.zabbix.com/zabbix/4.1/rhel/7/x86_64/zabbix-release-4.1-1.el7.noarch.rpm
若一台服务器上即是zabbix_server又是zabbix_agent和web Server则需要安装以下程序包
```
yum install zabbix zabbix-agent zabbix-get zabbix-sender zabbix-server zabbix-server-mysql zabbix-web zabbix-web-mysql -y
```
若服务器是zabbix_server
```
yum install zabbix zabbix-get zabbix-server zabbix-server-mysql zabbix-sender -y
```
若服务器是zabbix_agent
```
yum install zabbix zabbix-agent zabbix-get zabbix-sender -y
```
若服务器是web Server
```
yum install zabbix zabbix-get zabbix-sender zabbix-web zabbix-web-mysql -y
```
6、往数据库中导入一些数据
```
cd /usr/share/doc/zabbix-server-mysql-3.4.3/create/
```
#把下面三个表给导入zabbix数据库中
```
mysql -uroot -proot123 zabbix < schema.sql
mysql -uroot -proot123 zabbix < images.sql
mysql -uroot -proot123 zabbix < data.sql
```
7、修改/etc/zabbix/zabbix_server.conf 和 zabbix_agentd.conf
#若是本机
```
DBHost=localhost
```
#若不是本机
```
DBHost=x.x.x.x
```
#设置数据库用户名和密码，可自己在数据库中设置用户和密码
```
DBUser=root
DBPassword=root123
```
8、浏览器访问zabbix页面并进行初始化，详见上面步骤
总结：两种安装方法大同小异，核心思想还是一样的，就是在安装的时候可能会遇到很多坑，有报错又解决不了就找度娘吧，一般都能解决的！

参考博客：   
http://blog.csdn.net/mingjie1212/article/details/54619461  
http://blog.csdn.net/mingjie1212/article/details/52704987   
http://blog.csdn.net/linux_player_c/article/details/52287921  
