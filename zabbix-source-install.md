# Centos6.5源码安装zabbix<br>
## 前言<br>
最近在学习Zabbix，发现安装zabbix需要的yum源在官方库里只有centos7以上的，centos6以下的已经没有了，所以centos6只能通过源码包来安装，以下是源码包安装zabbix的步骤说明<br>
## MySQL5.7的安装配置<br>
### 第一步：下载mysql<br>
在Linux终端使用wget命令下载网络资源：wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz （也可在windows中下载后传输到Linux）<br>
### 第二步：解压文件<br>
由于我是在我本机software目录中下载的文件，为了方便管理首先将此文件移动到/usr/local 目录下<br>
```
mv /software/mysql-5.7.17-linux-glibc2.5-x86_64.tar /usr/local
```
接下来去到移动后的目录
```
cd /usr/local 
```
然后解压 
```
tar zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar
```
解压后为了方便后面操作可把解压后文件名修改为mysql：　　mv mysql-5.7.17-linux-glibc2.5-x86_64 mysql<br>
### 第三步：配置启动文件<br>
去到之前解压后并改名为mysql的目录下会有以下文件<br>
然后去到support-files目录下<br>
1、复制my.cnf 到 /etc/my.cnf (mysqld启动时自动读取)<br>
```
cp my-default.cnf /etc/my.cnf
```
注意：如果你在安装时Linux虚拟机时同时安装了默认的mysql，此时操作以上步骤，终端将会提示你文件已存在是否覆盖，输入yes覆盖即可。<br>
2、配置数据库编码<br>
```
vi /etc/my.cnf
```
在这份文件中可以添加以下配置信息(如果有修改即可)<br>
```
[mysql]
default-character-set=utf8
[mysqld]
default-storage-engine=INNODB
character_set_server=utf8
```
3、复制mysql.server 到/etc/init.d/  目录下【目的想实现开机自动执行效果】<br>
```
cp mysql.server /etc/init.d/mysqld
```
4、修改 /etc/init.d/mysql 参数<br>
```
vi /etc/init.d/mysql
```
给与2个目录位置<br>
```
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```
5、出于安全便利，创建一个操作数据库的专门用户<br>
　　1)、groupadd mysql #建立一个mysql的组<br>
　　2)、useradd -r -g mysql mysql #建立mysql用户，并且把用户放到mysql组<br>
　　3)、passwd mysql #给mysql用户设置一个密码<br>
　　4)、给目录/usr/local/mysql 更改拥有者　　chown -R mysql:mysql /usr/local/mysql/<br>
### 第四步：初始化 mysql 的数据库<br>
首先去到mysql的bin目录<br>
1.初始化<br>
```
./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```
生成出一个data目录，代表数据库已经初始化成功<br>
并且mysql的root用户生成一个临时密码：SHNq8Qvd2g>L(最好先记录这个临时密码)w+9bukXmMFH!<br>
2.给数据库加密<br>
```
./mysql_ssl_rsa_setup --datadir=/usr/local/mysql/data
```
3.启动mysql(为了不让进程卡主，可在启动mysql的命令后加上&代表此进程在后台运行)<br>
```
./mysqld_safe --user=mysql
```
4.检查ps -ef|grep mysql<br>
发现有以上进程便代表启动成功。<br>
### 第五步：进入客户端<br>
1.登录 ./mysql -uroot -p回车后输入之前的临时密码<br>
2.修改密码<br>
```
set password=password('新密码');
```
### 第六步：设置远程访问<br>
1，在远程访问之前需先配置防火墙systemctl stop firewalld.service(不推荐，可配置开通3306端口)<br>
2，授权<br>
```
mysql>grant all privileges on *.* to 远程访问用户名@'%' identified by '用户密码';
mysql>select host,user from user; 【多出1条远程登录用户记录】
mysql>flush privileges;(刷新)
```
此时使用远程机器进行访问<br>
解析：使用mysql -h主机ip -u用户名 -p密码即可进行远程访问<br>
### 第七步：设置开机自启动<br>
添加服务mysql<br>
```
chkconfig --add mysql
```
2、设置mysql服务为自动<br>
```
chkconfig mysql on
```
3、重启查看进程<br>
```
ps -ef|grep mysql
```
### 第八步：配置环境变量
为了方便操作，配置环境变量还是有必要的。
```
vi /etc/profile
export PATH=$JAVA_HOME/bin:/usr/local/mysql/bin:$PATH
```
## 安装zabbix
### 1、获取zabbix安装包<br>
https://sourceforge.net/projects/zabbix/files/latest/download?source=files  
由于更新原因，现在最新的版本已经不是4.0.4。有些命令在粘贴复制的时候需要自己手动改版本号  
解压<br>
```
tar -zxvf zabbix-4.0.4.tar.gz
```
安装依赖包：<br>
```
yum install mysql-devel unixODBC-devel net-snmp-devel libssh2-devel openldap openldap-devel OpenIPMI* libpcre* -y
```
### 2 安装php5.6和Apache服务
补充说明：<br>
今天我测试了一下下面安装的php56 yum源(rpm 安装的latest.rpm)，有点问题，我看了生成的三个repo文件，里面的mirrolist都是https的，于是我进浏览器测试访问，结果访问失败，故我把三个repo文件里面的https全部替换为http，再次测试就成功了。yum源也可以正常访问。若是有出现此类情况的网友可以参照这种方法解决。<br> 
获取yum源
```
rpm -ivh http://repo.webtatic.com/yum/el6/latest.rpm
```
安装下列所有包<br>
```
yum -y install httpd php56w php56w-gd php56w-mysqlnd php56w-bcmath php56w-mbstring php56w-xml php56w-ldap
```
编辑php的ini文件（vim /etc/php.ini）并修改一下内容，注意date.timezone一定要写对，否则在配置完zabbix后，显示的界面全部报错 <br>
```
post_max_size = 16M
max_execution_time = 300
max_input_time = 300
date.timezone = Asia/Shanghai
always_populate_raw_post_data = -1
```
配置/etc/httpd/conf/httpd.conf<br>
```
DocumentRoot "/var/www/html/zabbix"
<Directory "/var/www/html/zabbix">
ServerName 127.0.0.1
DirectoryIndex index.html index.html.var index.php
```
![iage64](https://github.com/now521/zabbix_picture/blob/master/64.png)
![image65](https://github.com/now521/zabbix_picture/blob/master/65.png)
![image66](https://github.com/now521/zabbix_picture/blob/master/66.png)
设置web前端<br>
```
mkdir /var/www/html/zabbix
cp -a zabbix-4.0.4/frontends/php/* /var/www/html/zabbix/
```
设置apache的执行和所有者<br>
```
chown -R apache:apache /var/www/html/zabbix
```
赋予可执行权限<br>
```
chmod +x /var/www/html/zabbix/conf/
```
### 3 创建zabbix用户和组<br>
```
groupadd zabbix
useradd -g zabbix zabbix
### 4 编译并安装zabbix
cd zabbix-4.0.4/
./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --enable-java --with-net-snmp --with-libcurl --with-libxml2 --with-unixodbc --with-ssh2 --with-openipmi --with-openssl --prefix=/usr/local/zabbix
```
可能会出现以下的报错信息，建议直接全部安装
```
yum install gcc* mysql-devel libxml2-devel net-snmp* java* curl-devel libevent-devel -y
（1）configure: error: in `/zabbix/zabbix-4.0.4':#configure: error: no acceptable C compiler found in $PATH
yum install gcc* -y
（2）configure: error: MySQL library not found
yum install mysql-devel -y
（3）configure: error: LIBXML2 library not found
yum install libxml2-devel -y
（4）configure: error: Invalid Net-SNMP directory - unable to find net-snmp-config
yum install net-snmp* -y
（5）configure: error: Unable to find "javac" executable in path
```
yum install java* -y#在装java*的时候，我看到要安装1个多G的东西，于是我就改成了javac*，发现后来编译还是出错，所以还是老老实实装java*吧，虽然装的包有点多，但至少能编译成功啊！<br>
```
（6）configure: error: Curl library not found
yum install curl-devel -y
（7）configure: error: SSH2 library not found
yum install -y libssh2-devel
```
以上是在编译的过程中可能会报错的信息及解决方法，当然也可能还有其它报错这里没列举出来，有问题找度娘，没毛病！<br>
编译成功会出现  
```
************************************************************            Now run 'make install'                       **                                                         **            Thank you for using Zabbix!                  **              <http://www.zabbix.com>                    ************************************************************

make install
```
### 5 修改配置文件zabbix_server.conf
```
vim /usr/local/zabbix/etc/zabbix_server.conf
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
```
### 6 添加Zabbix服务器和Zabbix代理启动脚本
```
cp zabbix-4.0.4/misc/init.d/fedora/core/zabbix_server /etc/init.d/zabbix_server
cp zabbix-4.0.4/misc/init.d/fedora/core/zabbix_agentd /etc/init.d/zabbix_agentd
```
修改 /etc/init.d/zabbix_server /etc/init.d/zabbix_agentd的BASEDIR=/usr/local/为BASEDIR=/usr/local/zabbix<br>
sed -i 's#BASEDIR=/usr/local/#BASEDIR=/usr/local/zabbix#g' /etc/init.d/zabbix_{server,agentd}<br>
### 7 创建zabbix数据库并把导入一些sql表<br>
进入数据库，刚装的mysql密码为空，直接回车就行
```
#进入数据库，刚装的mysql密码为空，直接回车就行
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
mysql>use zabbix;
mysql>source zabbix-3.4.3/database/mysql/schema.sql;
mysql>source zabbix-3.4.3/database/mysql/images.sql;
mysql>source zabbix-3.4.3/database/mysql/data.sql;
mysql>show tables;
+----------------------------+
| Tables_in_zabbix           |
+----------------------------+
| acknowledges               |
| actions                    |
| alerts                     |
| application_discovery      |
| application_prototype      |
| application_template       |
| applications               |
| auditlog                   |
| auditlog_details           |
| autoreg_host               |
| conditions                 |
| config                     |
| corr_condition             |
| corr_condition_group       |
| corr_condition_tag         |
| corr_condition_tagpair     |
| corr_condition_tagvalue    |
| corr_operation             |
| correlation                |
| dashboard                  |
| dashboard_user             |
| dashboard_usrgrp           |
| dbversion                  |
| dchecks                    |
| dhosts                     |
| drules                     |
| dservices                  |
| escalations                |
| event_recovery             |
| event_tag                  |
| events                     |
| expressions                |
| functions                  |
| globalmacro                |
| globalvars                 |
| graph_discovery            |
| graph_theme                |
| graphs                     |
| graphs_items               |
| group_discovery            |
| group_prototype            |
| groups                     |
| history                    |
| history_log                |
| history_str                |
| history_text               |
| history_uint               |
| host_discovery             |
| host_inventory             |
| hostmacro                  |
| hosts                      |
| hosts_groups               |
| hosts_templates            |
| housekeeper                |
| httpstep                   |
| httpstep_field             |
| httpstepitem               |
| httptest                   |
| httptest_field             |
| httptestitem               |
| icon_map                   |
| icon_mapping               |
| ids                        |
| images                     |
| interface                  |
| interface_discovery        |
| item_application_prototype |
| item_condition             |
| item_discovery             |
| item_preproc               |
| items                      |
| items_applications         |
| maintenances               |
| maintenances_groups        |
| maintenances_hosts         |
| maintenances_windows       |
| mappings                   |
| media                      |
| media_type                 |
| opcommand                  |
| opcommand_grp              |
| opcommand_hst              |
| opconditions               |
| operations                 |
| opgroup                    |
| opinventory                |
| opmessage                  |
| opmessage_grp              |
| opmessage_usr              |
| optemplate                 |
| problem                    |
| problem_tag                |
| profiles                   |
| proxy_autoreg_host         |
| proxy_dhistory             |
| proxy_history              |
| regexps                    |
| rights                     |
| screen_user                |
| screen_usrgrp              |
| screens                    |
| screens_items              |
| scripts                    |
| service_alarms             |
| services                   |
| services_links             |
| services_times             |
| sessions                   |
| slides                     |
| slideshow_user             |
| slideshow_usrgrp           |
| slideshows                 |
| sysmap_element_trigger     |
| sysmap_element_url         |
| sysmap_shape               |
| sysmap_url                 |
| sysmap_user                |
| sysmap_usrgrp              |
| sysmaps                    |
| sysmaps_elements           |
| sysmaps_link_triggers      |
| sysmaps_links              |
| task                       |
| task_acknowledge           |
| task_close_problem         |
| task_remote_command        |
| task_remote_command_result |
| timeperiods                |
| trends                     |
| trends_uint                |
| trigger_depends            |
| trigger_discovery          |
| trigger_tag                |
| triggers                   |
| users                      |
| users_groups               |
| usrgrp                     |
| valuemaps                  |
| widget                     |
| widget_field               |
+----------------------------+
140 rows in set (0.00 sec)

mysql>\q

```
### 8 启动所有服务，并设置开机自启
启动Apache服务<br>
service httpd start<br>
启用mysql服务
```
service mysqld start
/etc/init.d/zabbix_server start
/etc/init.d/zabbix_agentd start
```
设置开机自启<br>
```
chkconfig httpd on
chkconfig mysqld on
chkconfig --add /etc/init.d/zabbix_server
chkconfig --add /etc/init.d/zabbix_agentd
chkconfig zabbix_server on
chkconfig zabbix_agentd on
```
查看端口号80、3306、10050（zabbix_agentd）、10051（zabbix_server）是否监听
```
ss -tnul
```
### 9 在所有其他服务器上都部署上zabbix
操作同上面安装zabbix一致
### 10 浏览器访问zabbix页面并进行初始化
### 11 登陆zabbix安装界面<br>
![image46](https://github.com/now521/zabbix_picture/blob/master/46.png)<br>
### 12 进入下一步<br>
![image47](https://github.com/now521/zabbix_picture/blob/master/47.png)<br>
### 13 填写数据库信息并进入下一步<br>
![image48](https://github.com/now521/zabbix_picture/blob/master/48.png)<br>
![image49](https://github.com/now521/zabbix_picture/blob/master/49.png)<br>
![image50](https://github.com/now521/zabbix_picture/blob/master/50.png)<br>
### 14 发现问题，查找原因，按照提示下载配置文件并放到指定目录<br>
![image51](https://github.com/now521/zabbix_picture/blob/master/51.png)<br>
### 15 完成安装<br>
![image52](https://github.com/now521/zabbix_picture/blob/master/52.jpg)<br>
### 16 登陆zabbix，用户名：Admin密码：zabbix。<br>
![image53](https://github.com/now521/zabbix_picture/blob/master/53.png)<br>
### 17 修改语言为中文<br>
![image54](https://github.com/now521/zabbix_picture/blob/master/54.png)<br>
![image55](https://github.com/now521/zabbix_picture/blob/master/55.png)<br>
### 18 添加主机<br>
![image56](https://github.com/now521/zabbix_picture/blob/master/56.png)<br>
![image57](https://github.com/now521/zabbix_picture/blob/master/57.png)<br>
![image58](https://github.com/now521/zabbix_picture/blob/master/58.png)<br>
### 19 添加监控项<br>
![image60](https://github.com/now521/zabbix_picture/blob/master/60.png)<br>
![image61](https://github.com/now521/zabbix_picture/blob/master/61.png)<br>
![image62](https://github.com/now521/zabbix_picture/blob/master/62.png)<br>
![image59](https://github.com/now521/zabbix_picture/blob/master/59.png)<br>
![image63](https://github.com/now521/zabbix_picture/blob/master/63.png)<br>
