# zabbix  
zabbix显示中文乱码问题  
当部署完zabbix然后显示中文会出现如下图  
![image59](https://github.com/now521/zabbix_picture/blob/master/59.png)<br> 
然后此时先去windows的文字目录如 C:\Windows\Fonts 随便托个字体到桌面  
[![image40](https://github.com/now521/zabbix_picture/blob/master/40.png)](https://github.com/now521/zabbix_picture/blob/master/abc.TTF)<br/> 
然后转到Linux  
vim /usr/share/zabbix/include/defines.inc.php #修改使用的字体名字  
修改54行 define('ZBX_GRAPH_FONT_NAME', 'abc')改为abc  
![image41](https://github.com/now521/zabbix_picture/blob/master/41.png)  
修改103行define('ZBX_FONT_NAME', 'abc'');  
![image42](https://github.com/now521/zabbix_picture/blob/master/42.png)  
然后将windows桌面字体直接移到linux的/usr/share/zabbix/fonts/目录下 修改为abc.ttf  
 重启zabbix服务  
systemctl restart zabbix-server.service  
刷新web页面，中文乱码修复  
![image43](https://github.com/now521/zabbix_picture/blob/master/43.png)  
