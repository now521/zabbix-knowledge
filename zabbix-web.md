# zabbix实战监控WEB网站性能  
http://www.ttlsa.com/zabbix/zabbix-web-monitor-real-life-scenario/  

所属分类：zabbix  
一直在纠结用什么实例来给大家演示呢？想来想去还是官方的好，那我们怎么用zabbix监控web性能和可用性呢？我们这边分为几个步骤：打开网站、登陆、登陆验证、退出，一共4个小step，看实例。  
检测流程  
## 1. 打开网站：如果http code为200，并且响应的html中包含Zabbix SIA表示打开成功（zabbix页面有这个标示）  
## 2. 登陆后台：post用户名和密码到index.php，如果响应200，那表示post成功。并且通过正则表达式从响应的html中匹配sid，这个sid也就是一个宏变量，退出可以使用到  
## 3. 验证登陆：打开首页，检索html中是否包含Profile（只有登陆成功，才会有Profile出现）  
## 4.退出账号：传递参数sid给index.php即可退出，响应200即表示退出成功。  
我们可以使用上节讲到的item key来获取每个step的速度以及响应时间或者说最新的一个错误消息，大家自己去研究吧，不难
### 创建WEB场景  
configuration->Host->你的主机->web->右上角Create scenario  
![image20](https://github.com/now521/zabbix_picture/blob/master/20.jpg)    
Name：监控项的名称  
Application：放到哪个应用中，《什么是Application》  
Authentication：是否有http的基本认证，大部分情况下是None，难不成用户进来还需要经过一次认证？  
Update interval：更新周期，默认60秒,多久跑一次  
Retries：重试次数  
Agetn：模拟浏览器  
HTTP proxy：代理，如果你的站点有多台服务器，那么请写上你目标服务器ip和端口，例如http://10.9.0.2:80,默认端口可不是80，别忘记80了  
Variables：宏变量，后面会用到。想了解请点《zabbix用户宏macro》  
### web监控阶段1：打开首页  
![image21](https://github.com/now521/zabbix_picture/blob/master/21.jpg)    
对step做一个说明：  
name：当前step名称，item key中可以用到  
url：需要检测的网址  
POST：你需要post提交上去的内容，例如user=123&password=123456,，或者使用宏变量user={user}&password={password}，如果支持GET，那么可以直接写到URL里面  
variables：变量，这边定义宏变量后续的step可以使用  
Timeout：超时时间，默认15秒  
Required string：响应的内容中必须包含的字符串，否则失败  
Required status codes：响应代码必须包含在里面，多个响应代码用逗号分隔，例如200,301,302  
### web监控阶段2：登陆  
![image22](https://github.com/now521/zabbix_picture/blob/master/22.jpg)    
post账号和密码上去，关于post在前面已经提过了。  
### WEB监控阶段3：验证登陆  
![image23](https://github.com/now521/zabbix_picture/blob/master/23.jpg)    
### WEB监控阶段4：退出账号  
![image24](https://github.com/now521/zabbix_picture/blob/master/24.jpg)    
### WEB网站检测配置完成  
记得保存账号  
![image25](https://github.com/now521/zabbix_picture/blob/master/25.jpg)    
### 查看结果  
monitorning->web->筛选出你的主机->查看“WEB性能监控_FOR_TTLSA”，结果如下图  
各个阶段的响应时间、速度、返回状态码以及总的响应时间  
![image26](https://github.com/now521/zabbix_picture/blob/master/26.jpg)  
下图是下载速度的图表，包含各个阶段  
![image27](https://github.com/now521/zabbix_picture/blob/master/27.jpg)  
下图是响应时间的图表  
![image28](https://github.com/now521/zabbix_picture/blob/master/28.jpg)  
以上是没问题的信息，那么出现故障是什么样子呢？我把密码改掉，演示给大家看看下图，在LOGIN IN这个step就出错了，拿不到SID  
![image29](https://github.com/now521/zabbix_picture/blob/master/29.jpg)   
那么Required String不匹配又是什么样子呢？我们把阶段3Login CHECK的required string的Profile改成Profile1试试。看看结果
![image30](https://github.com/now521/zabbix_picture/blob/master/30.jpg)  
好了，web监控的实例就完成了。  
