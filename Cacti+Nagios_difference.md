# Cacti与Nagios进行网络监控的区别
Cacti与Nagios进行网络监控的区别

Cacti和Nagios是现在使用比较多的网络监控软件了，对于这两款监控软件的区别，应该说是侧重点的不同。

Cacti比较着重于直观数据的监控，易于生成图形，用来监控网络流量、cpu使用率、硬盘使用率等可以说很在合适不过。

而Nagios则比较注重于主机和服务的监控，并且有很强大的发送报警信息的功能。

把两者结合起来，既可以使报警机制高效及时，又可以很容易的查看各项数据的情况。

由于工作的关系，我在前一家公司主要是用FreeBSD来架构网络监控程序，最早使用的是MRTG，然后开始用RRDTOOL，后来发现了Cacti，爱不释手啊。

而现在的公司，一开始是老板要求用Nagios来进行主机和服务监控，但是后来觉得Nagios设置起来实在不方便，所以改用了Cacti，并且使用Plugin来构建报警机制，但是效果不甚理想。

于是就在找一个比较合适的解决办法，前一段在网上看到Nagios For Cacti的Plugin终于有了更新，决定试一下看看。

1. 安装必须的软件

2. 安装Cacti

3. 安装Cacti Plugins Arch

4.安装NPC，Settings和Thold

5. 安装Nagios

6. 安装NDoutils

如果，你管理的系统是一个30台服务器规模以下的小公司，那么也许你自己写的监控脚本是最好的解决办法，但是，如果，服务器达到30台以上的，而且分布到各个地域，那么使用一些开源的监控工具就非常合适了。

这里只说自己用过的两种监控工具，这两种工具可以配合使用，一个是cacti,另一个是nagios。

这两个工具最好是都装在linux系统上，cacti需要通过snmp协议收集被监控服务器的信息，nagios 则有自己的agent去收集信息。cacti虽然可以安装在windows上，其实那也是模拟了一个linux的类环境。

cacti偏重于网络流量，系统负载方面的监控。而 nagios偏重于系统服务方面的监控，你可以在被监控的机器上写自己的程序(shell,c 或 perl都可以) 。nagios则通过这些脚本来对服务进行监控。nagios可以和短信发送机配合用来监控规模较大的网站。  
安装方法链接：https://blog.csdn.net/shedong1011/article/details/48354837  
https://blog.csdn.net/allen_a/article/details/50719296
