# zabbix添加监控磁盘io的脚本
## 1、创建目录
```
cd /etc/zabbix/
mkdir scripts
```
## 2、编辑监控脚本脚本
vim disk_io.sh
```
#!/bin/bash
Disk=$1
Option=$2
case $Option in
rrqm)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $2}'
;;
wrqm)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $3}'
;;
rps)
iostat -dxk 1 2|grep "\b$Disk\b"|tail -1|awk '{print $4}'
;;
wps)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $5}'
;;
rKBps)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $6}'
;;
wKBps)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $7}'
;;
avgrq-sz)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $8}'
;;
avgqu-sz)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $9}'
;;
await)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $10}'
;;
svctm)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $11}'
;;
util)
iostat -dxk 1 2|grep "\b$Disk\b" |tail -1|awk '{print $12}'
;;
esac

```
## 3、修改配置文件
```
cp -arp zabbix_agentd.conf zabbix_agentd.conf.bak
vim zabbix_agentd.conf（rhel7版本的这个数字不能重复）
```
![image31](https://github.com/now521/zabbix_picture/blob/master/31.png)
```
vim /etc/sudoers
root    ALL=(ALL)       ALL
zabbix  ALL=NOPASSWD:ALL
#Defaults    requiretty（rhel7需要注释掉这个）
ln -s /usr/local/zabbix /etc/zabbix
/etc/init.d/zabbix-agent restart
sudo /etc/zabbix/scripts/disk_io.sh sda rrqm
```
##4、安装zabbix-get测试工具
```
yum install zabbix-get.x86_64
zabbix_get -s 10.156.1.11 -p 10050 -k disk.io[sda,util]
```
server端的zabbix-agent.conf 必须指向127.0.0.1，
![image32](https://github.com/now521/zabbix_picture/blob/master/32.png)
![image33](https://github.com/now521/zabbix_picture/blob/master/33.png)

## 5、页面设置：
![image34](https://github.com/now521/zabbix_picture/blob/master/34.png)
![image35](https://github.com/now521/zabbix_picture/blob/master/35.png)
![image36](https://github.com/now521/zabbix_picture/blob/master/36.png)
## 6、磁盘io使用率  diskio[sda,util]
![image37](https://github.com/now521/zabbix_picture/blob/master/37.png)
![image38](https://github.com/now521/zabbix_picture/blob/master/38.png)
