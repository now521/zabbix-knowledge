# zabbix执行远程命令

## 概述
监控,有的人只把他当做报警使用,出现问题之后打开跑回家打开电脑，巴拉巴拉的处理掉，大多数时候都是一些小问题，为何不让zabbix帮你把这些事情处理掉呢？和朋友具体，收到xx硬盘空间慢了、xx服务器高负载等问题，你要回家处理？多扫兴

瞧瞧zabbix远程执行命令可以做些什么吧:

重启应用（Apache、nginx、MySQL等等）
使用IPMI接口重启服务器
自动释放磁盘空间（删除老文件，清除/tmp目录等等）
CPU过载时讲一个虚拟机迁移到另外一台物理服务器
云环境下，一台服务器CPU\硬盘\内存\其他硬件资源不足的情况下，可以自动添加过去
创建一个报警，记得使用邮件报警吗？呵呵，实际上，我们把发送邮件的操作改成执行远程命令就行了

备注：zabbix代理不支持远程命令

远程命令最大长度为255字符，同时支持多个远程命令，如需要执行多条命令，只需要另起一行写命令即可，还有，远程命令可以使用宏变量。

接下来我将一步一步告诉大家如何设置远程命令

## 配置
首先我们需要在zabbix客户配置文件开启对远程命令的支持，编辑zabbix_agentd.conf，修改
```
EnableRemoteCommands = 1
EnableRemoteCommands = 1
```
重启客户端

备注：Aive zabbix不支持远程命令

然后配置action，Configuration->Actions，选择Operation选项卡，operation type改成Remote Command，选择远程命令类似 (IPMI, Custom script, SSH, Telnet, Global script)，输入远命令

配置Action

在Operations选显卡中选择Remote command
选择远程命令类型(IPMI, Custom script, SSH, Telnet, Global script)
写上远程命令
示例:
```
sudo /etc/init.d/apache restart
sudo /etc/init.d/apache restart
```
上面例子用来在出现状况的情况下重启Apache，务必全包zabbix agent能够执行这个命令.

备注：

1.sudo不用多说了，zabbix用户没有运行某些命令的权限,必须加上.

2.远程命令，自然是在远程的主机后台运行。
Conditions选项卡定义了什么条件下，这个远程命令会被执行，其实这个和前面说的action真没什么区别，大家都能看懂。下图的意思是，在非维护时间Apache应用出现状况，并且严重性级别为Disaster。满足条件之后，就开始执行命令了。

## 访问权限
确保你的zabbix用户有执行权限，如果某些命令需要root权限，那么请使用sudo

#visudo

#visudo
编辑sudoer文件，zabbix用户便可以执行Apache restart命令了
```
allows 'zabbix' user to run all commands without password.
 zabbix ALL=NOPASSWD: ALL
allows 'zabbix' user to restart apache without password.
 zabbix ALL=NOPASSWD: /etc/init.d/apache restart

allows 'zabbix' user to run all commands without password.
 zabbix ALL=NOPASSWD: ALL
allows 'zabbix' user to restart apache without password.
 zabbix ALL=NOPASSWD: /etc/init.d/apache restart
```
备注：在某些情况下，zabbix需要sudo才能执行命令，请先在/etc/sudoer开启requiretty.具体的方法，请百度或者google.

使用多种接口执行远程命令
如果目标系统支持多种接口：zabbix agent、IPMI、远程命令（默认），请看如下一些实例
示例1

通过zabbix检测到的一些问题，然后自动重启windows

参数	                       描述
Operation type        	Remote command
Type	                  Custom script
Command	         c:\windows\system32\shutdown.exe -r -
示例2

使用IPMI重启服务器

参数	                描述
Operation type	   Remote command
Type	                 IPMI
Command	             reset on
示例三

使用IPMI关机

参数	                   描述
Operation type	    Remote command
Type                  	IPMI
Command	              power off
