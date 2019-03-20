一、zabbix agent主动模式与被动模式的区别
zabbix agent的运行模式有以下两种：
1、被动模式：此模式为zabbix默认的工作模式，由zabbix server 向zabbix agent 发出指令获取数据，zabbix agent被动地去获取数据并返回给zabbix server，zabbix server会周期性地向agent索取数据。此模式的最大问题就是会增加zabbix server的工作量，在大量的服务器环境下，zabbix server不能及时获取到最新的数据。
2、主动模式：即由zabbix agent 主动采集数据并返回给zabbix server，不需要zabbix server 的另行干预，因此使用主动模式能在一定程序上减轻zabbix server的压力。
二、主动模式和被动模式的配置
![image72](https://github.com/now521/zabbix_picture/blob/master/72.webp)   
                      部署拓扑图

1、编译安装zabbix server
获取zabbix server的源码包并复制保存到/usr/local/src目录下后解压缩创建相应的软链接：
[root@zabbix-server ~]# tar xf /usr/local/src/zabbix-3.0.18.tar.gz -C /usr/local/src/

安装依赖包：
[root@zabbix-server ~]# yum install gcc libxml2-devel net-snmp net-snmp-devel curl curl-devel php php-bcmath php-mbstring mariadb mariadb-devel -y

安装数据库：
[root@zabbix-server ~]# yum install -y mariadb-server
[root@zabbix-server ~]# systemctl restart mariadb
[root@zabbix-server ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@"192.168.0.%" identified by '123456';
Query OK, 0 rows affected (0.00 sec)

安装jdk包：
#事先获取jdk源码包并放在在/usr/local/src目录下
[root@zabbix-server ~]# cd /usr/local/src/
[root@zabbix-server src]# tar xf jdk-10.0.1_linux-x64_bin.tar.gz
[root@zabbix-server src]# ln -sv /usr/local/src/jdk-10.0.1 /usr/local/jdk
"/usr/local/jdk" -> "/usr/local/src/jdk-10.0.1"

#在/etc/profile文件中添加环境变量：
[root@zabbix-server src]# vim /etc/profile
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
[root@zabbix-server src]# source /etc/profile
[root@zabbix-server src]# java -version  #命令若执行成功，说明jdk已成功安装
java version "10.0.1" 2018-04-17
Java(TM) SE Runtime Environment 18.3 (build 10.0.1+10)
Java HotSpot(TM) 64-Bit Server VM 18.3 (build 10.0.1+10, mixed mode)

编译安装zabbix-server：
[root@zabbix-server ~]# cd /usr/local/src/zabbix-3.0.18
[root@zabbix-server zabbix-3.0.18]# useradd zabbix -s /sbin/nologin
[root@zabbix-server zabbix-3.0.18]# ./configure --prefix=/usr/local/zabbix --enable-server --enable-agent --with-mysql --with-net-snmp --with-libcurl --with-libxml2 --enable-java
***********************************************************
*            Now run 'make install'                       *
*                                                         *
*            Thank you for using Zabbix!                  *
*              <http://www.zabbix.com>                    *
***********************************************************
#如无编译无报错，则应出现上述提示
[root@zabbix-server zabbix-3.0.18]# make install

将zabbix的数据库模板按顺序导入数据库：
[root@zabbix-server zabbix-3.0.18]# mysql -uzabbix -p123456 -h192.168.0.81 zabbix < /usr/local/src/zabbix-3.0.18/database/mysql/schema.sql 
[root@zabbix-server zabbix-3.0.18]# mysql -uzabbix -p123456 -h192.168.0.81 zabbix < /usr/local/src/zabbix-3.0.18/database/mysql/images.sql 
[root@zabbix-server zabbix-3.0.18]# mysql -uzabbix -p123456 -h192.168.0.81 zabbix < /usr/local/src/zabbix-3.0.18/database/mysql/data.sql 

复制启动脚本到/etc/init.d/目录下并更改相应的配置：
[root@zabbix-server zabbix-3.0.18]# cp /usr/local/src/zabbix-3.0.18/misc/init.d/fedora/core/zabbix_server /etc/init.d/
[root@zabbix-server zabbix-3.0.18]# vim /etc/init.d/zabbix_server
        # Zabbix-Directory
        BASEDIR=/usr/local/zabbix
        ......
start() {
        if [ $RUNNING -eq 1 ]
                then
                echo "$0 $ARG: $BINARY_NAME (pid $PID) already running"
        else
                action $"Starting $BINARY_NAME: " $FULLPATH -c /usr/local/zabbix/etc/zabbix_server.conf    #另外可以在此处用-c选项指定zabbix_server的配置文件路径，默认为/usr/local/zabbix/etc/zabbix_server.conf
                touch /var/lock/subsys/$BINARY_NAME
    fi
}
......

配置zabbix_server的配置文件：
[root@zabbix-server zabbix-3.0.18]# mkdir /var/log/zabbix 
[root@zabbix-server zabbix-3.0.18]# chown zabbix:zabbix /var/log/zabbix/ -R
[root@zabbix-server ~]# vim /usr/local/zabbix/etc/zabbix_server.conf
#修改新增以下配置
LogFile=/var/log/zabbix/zabbix_server.log
DBHost=192.168.0.81
DBName=zabbix
DBUser=zabbix
DBPassword=123456
Timeout=30
LogSlowQueries=3000

#最后启动zabbix_server
[root@zabbix-server ~]# /etc/init.d/zabbix_server start

安装httpd：
[root@zabbix-server ~]# yum install -y httpd
#创建zabbix目录
[root@zabbix-server ~]# mkdir /var/www/html/zabbix
#复制zabbix相关的php文件到指定目录
[root@zabbix-server ~]# cp -a /usr/local/src/zabbix-3.0.18/frontends/php/* /var/www/html/zabbix/

安装httpd的相关依赖包：
[root@zabbix-server ~]# yum install php-gettext php-session php-ctype php-xmlreader php-xmlwriter php-xml php-net-socket php-gd php-mysql -y

修改/etc/php.ini文件的配置：
[root@zabbix-server ~]# vim /etc/php.ini
#找到相关配置并修改配置为如下
post_max_size = 16M
max_execution_time = 300
max_input_time = 300
date.timezone = Asia/Shanghai

启动httpd服务：
[root@zabbix-server ~]# systemctl start httpd
[root@zabbix-server ~]# systemctl stop firewalld 
[root@zabbix-server ~]# setenforce 0

此时访问相应的http://192.168.0.81/zabbix页面会显示检查依赖关系的结果，如无报错就可以继续完成安装；否则需要按照报错的按照查找指定的解决办法。
![image73](https://github.com/now521/zabbix_picture/blob/master/73.webp)
zabbix安装之检查依赖关系状态
![image74](https://github.com/now521/zabbix_picture/blob/master/74.webp)
zabbix安装之添加数据库
![image75](https://github.com/now521/zabbix_picture/blob/master/75.webp)
zabbix安装之填写zabbix服务器信息
![image76](https://github.com/now521/zabbix_picture/blob/master/76.webp)

zabbix安装之信息汇总
![image77](https://github.com/now521/zabbix_picture/blob/master/77.webp)
zabbix安装之安装配置文件
![image78](https://github.com/now521/zabbix_picture/blob/master/78.webp)
到此步，需要把相应的配置文件下载保存为指定的路径/var/www/html/zabbix/conf/zabbix.conf.php。
![image79](https://github.com/now521/zabbix_picture/blob/master/79.webp)  
安装完成

配置完成后即可登录到zabbix的web管理页面，默认的登录账号为Admin，密码为zabbix。
2、以主动模式部署zabbix_agent并使用zabbix server进行监控
安装相关的依赖包：
[root@zbx-active ~]# yum install -y gcc

解压缩源码包并编译安装：
[root@zbx-active ~]# cd /usr/local/src/

[root@zbx-active src]# tar xf zabbix-3.0.18.tar.gz
[root@zbx-active zabbix-3.0.18]# cd zabbix-3.0.18
[root@zbx-active zabbix-3.0.18]# useradd zabbix -s /sbin/nologin
[root@zbx-active zabbix-3.0.18]# ./configure --prefix=/usr/local/zabbix --enable-agent
[root@zbx-active zabbix-3.0.18]# make install

复制启动进程到/etc/init.d目录下并编辑修改相应的配置：
[root@zbx-active zabbix-3.0.18]# cp /usr/local/src/zabbix-3.0.18/misc/init.d/fedora/core/zabbix_agentd /etc/init.d/
[root@zbx-active zabbix-3.0.18]# vim /etc/init.d/zabbix_agentd
        # Zabbix-Directory
        BASEDIR=/usr/local/zabbix

接着修改zabbix_agented的配置文件：
[root@zbx-active zabbix-3.0.18]# vim /usr/local/zabbix/etc/zabbix_agentd.conf
#找到对应的配置，并修改为如下
LogFile=/tmp/zabbix_agentd.log
Server=192.168.0.81    #被动模式所接受的服务器Ip，此处是为了启用监听10050端口，从而监测到zabbix主机的zbx状态。
ListenPort=10050
StartAgents=1    #默认启动的zabbix_agentd pre-fork进程，如果为0的话表示停用被动模式
ServerActive=192.168.0.81  #主动模式的服务器IP
Hostname=192.168.0.83  
Timeout=30
UnsafeUserParameters=1

最后启动zabbix_agented：
[root@zbx-active zabbix-3.0.18]# /etc/init.d/zabbix_agentd start    #此时监听端口10050也应该启动了
[root@zbx-active ~]# systemctl stop firewalld
[root@zbx-active ~]# setenforce 0


接着在zabbix server上添加相应的zabbix 主机：
![image80](https://github.com/now521/zabbix_picture/blob/master/80.webp)
创建zabbix主机
![image81](https://github.com/now521/zabbix_picture/blob/master/81.webp)
关联相应的主动模式监控模板

最后等待数据刷新，一段时间过后应该会显示相关的数据。
![image82](https://github.com/now521/zabbix_picture/blob/master/82.webp)
获取到对应主机的最新数据
![image83](https://github.com/now521/zabbix_picture/blob/master/83.webp)
生成了对应数据的数据图形

3、以被动模式部署zabbix_agent并使用zabbix server进行监控
被动模式的部署方式比主动模式简单，因为其本身就是zabbix agent的默认工作模式。
zabbix agent的编译安装过程这里就不重复演示了，可参考主动模式的编译安装。
zabbxi agent被动模式的配置文件配置如下：
LogFile=/tmp/zabbix_agentd.log
Server=192.168.0.81
ListenPort=10050
StartAgents=5
#ServerActive=192.168.0.81   #注释掉主动模式的配置
Hostname=192.168.0.84
Timeout=30
UnsafeUserParameters=1

最后启动zabbix_agent：
[root@zbx-passive ~]# /etc/init.d/zabbix_agentd start
Reloading systemd:                                         [  OK  ]
Starting zabbix_agentd (via systemctl):                    [  OK  ]
[root@zbx-passive ~]# systemctl stop firewalld
[root@zbx-passive ~]# setenforce 0

最后在zabbix server主机上添加相应的主机。
![image84](https://github.com/now521/zabbix_picture/blob/master/84.webp)
添加主机
![image85](https://github.com/now521/zabbix_picture/blob/master/85.webp)
添加主机监控模板

等待一段时间过后，应该就出现相应的监控数据。
![image86](https://github.com/now521/zabbix_picture/blob/master/86.webp)
通过被动模式抓取的最新数据
![iage87](https://github.com/now521/zabbix_picture/blob/master/87.webp)
被动模式主机的图形化数据

三、zabbix proxy
zabbix是一个分布式的监控系统，其支持通过代理服务器zabbix proxy 来收集zabbix agent的数据，然后zabbix proxy再把收集保存在本地数据库的数据送往zabbix server进行统一存储和展示。单一的zabbix server+ zabbix agent的环境，因受限于zabbix server性能瓶颈的缘故，其能管控的zabbix agent是有限的。而zabbix proxy则把zabbix server从收集数据这一功能中抽离出来，使得zabbix server专注于对数据进行存储和展示，提升了zabbix server所能管控的zabbix agent数据及提高了部署的灵活性。
1、zabbix proxy服务器的部署
![iage88](https://github.com/now521/zabbix_picture/blob/master/88.webp)
zabbix proxy模式部署拓扑图

新增zabbix proxy服务器，修改之前部署配置的zabbix server和agent来调用zabbix proxy服务器，从而让zabbix server 从zabbix proxy上获取其收集到的zabbix agent数据。
首先安装proxy所依赖的程序包：
[root@proxy ~]# yum install gcc libxml2-devel net-snmp net-snmp-devel curl curl-devel php php-bcmath php-mbstring mariadb mariadb-devel -y

安装jdk：
#事先下载相应的源码包并存放到指定目录
[root@proxy ~]# cd /usr/local/src/
[root@proxy src]# tar xf jdk-10.0.1_linux-x64_bin.tar.gz
[root@proxy src]# ln -sv /usr/local/src/jdk-10.0.1 /usr/local/jdk

添加jdk环境变量：
[root@proxy src]# vim /etc/profile
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
[root@proxy src]# source /etc/profile、
[root@proxy src]# java -version
java version "10.0.1" 2018-04-17
Java(TM) SE Runtime Environment 18.3 (build 10.0.1+10)
Java HotSpot(TM) 64-Bit Server VM 18.3 (build 10.0.1+10, mixed mode)

安装数据库：
[root@proxy src]# yum install -y mariadb-server
[root@proxy src]# systemctl start mariadb
MariaDB [(none)]> create database zabbix_proxy character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all privileges on zabbix_proxy.* to proxy@'192.168.0.%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

编译安装zabbix proxy：
[root@proxy ~]# useradd zabbix -s /sbin/nologin
[root@proxy ~]# cd /usr/local/src/
[root@proxy src]# tar xf zabbix-3.0.18.tar.gz 
[root@proxy src]# cd zabbix-3.0.18
[root@proxy zabbix-3.0.18]# ./configure --prefix=/usr/local/zabbix --enable-proxy --enable-agent --with-mysql --with-net-snmp --with-libcurl --with-libxml2 --enable-java
[root@proxy zabbix-3.0.18]# make install

导入数据库：
[root@proxy zabbix-3.0.18]# mysql -uproxy -p123456 -h192.168.0.89 zabbix_proxy < /usr/local/src/zabbix-3.0.18/database/mysql/schema.sql 

编辑配置zabbix proxy的配置文件：
[root@proxy ~]# vim /usr/local/zabbix/etc/zabbix_proxy.conf
ProxyMode=0    #指定代理那种模式的agent，0为主动模式，1位被动模式；
Server=192.168.0.81  #zabbix server服务器的地址或主机名
Hostname=GD_SZ_Proxy  #代理服务器的名称
LogFile=/tmp/zabbix_proxy.log  
DBHost=192.168.0.89  #数据库服务器地址
DBName=zabbix_proxy
DBUser=proxy
DBPassword=123456
ProxyLocalBuffer=3
ProxyOfflineBuffer=24
HeartbeatFrequency=60    #心跳间隔检测时间，默认为60秒，范围为0-3600秒，被动模式不使用
ConfigFrequency=5    #间隔多久从zabbix server获取监控信息
DataSenderFrequency=5  #数据发送时间间隔，默认为1秒，被动模式不使用
StartPollers=10    #启动的线程数据，建议与客户端的数量保持一致
Timeout=30
LogSlowQueries=3000

最后启动zabbix proxy 服务：
[root@proxy ~]# /usr/local/zabbix/sbin/zabbix_proxy -c /usr/local/zabbix/etc/zabbix_proxy.conf
[root@proxy ~]# systemctl stop firewalld
[root@proxy ~]# setenforce
[root@proxy ~]# setenforce 0

2、在zabbix server上添加代理服务器
![image89](https://github.com/now521/zabbix_picture/blob/master/89.webp)
添加proxy服务器
![image90](https://github.com/now521/zabbix_picture/blob/master/90.webp)
修改监控主机的使用代理监控
![image91](https://github.com/now521/zabbix_picture/blob/master/91.webp)
修改此前为被动模式监控的192.168.0.84的监控模板

3、修改zabbxi agent的配置
在192.168.0.83上修改其对应的agent配置：
[root@zbx-active ~]# vim /usr/local/zabbix/etc/zabbix_agentd.conf
#修改其server和serveractive的配置均指向代理服务器
Server=192.168.0.89
ServerActive=192.168.0.89

随后重启zabbix agent服务：
[root@zbx-active ~]# /etc/init.d/zabbix_agentd restart

在192.168.0.84上修改其对应的agent配置：
[root@zbx-passive ~]# vim /usr/local/zabbix/etc/zabbix_agentd.conf
Server=192.168.0.89
ServerActive=192.168.0.89

随后重启zabbix agent服务：
[root@zbx-passive ~]# /etc/init.d/zabbix_agentd restart

4、观察监控情况及数据
![image91](https://github.com/now521/zabbix_picture/blob/master/91.webp)
刷新周期过后，监控主机的连接性应能正常绿了
![image92](https://github.com/now521/zabbix_picture/blob/master/92.webp)
相关主机的图表正常刷新

链接：https://www.jianshu.com/p/c735e9bb1c66
