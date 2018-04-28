firewalld自身并不具备防火墙的功能，而是和iptables一样需要通过内核的netfilter来实现，也就是说firewalld和 iptables一样，他们的作用都是用于维护规则，而真正使用规则干活的是内核的netfilter，只不过firewalld和iptables的结构以及使用方法不一样罢了。



一个重要的概念：区域管理

通过将网络划分成不同的区域，制定出不同区域之间的访问控制策略来控制不同程序区域间传送的数据流。例如，互联网是不可信任的区域，而内部网络是高度信任的区域。网络安全模型可以在安装，初次启动和首次建立网络连接时选择初始化。该模型描述了主机所连接的整个网络环境的可信级别，并定义了新连接的处理方式。有如下几种不同的初始化区域：

阻塞区域（block）：任何传入的网络数据包都将被阻止。

工作区域（work）：相信网络上的其他计算机，不会损害你的计算机。

家庭区域（home）：相信网络上的其他计算机，不会损害你的计算机。

公共区域（public）：不相信网络上的任何计算机，只有选择接受传入的网络连接。

隔离区域（DMZ）：隔离区域也称为非军事区域，内外网络之间增加的一层网络，起到缓冲作用。对于隔离区域，只有选择接受传入的网络连接。

信任区域（trusted）：所有的网络连接都可以接受。

丢弃区域（drop）：任何传入的网络连接都被拒绝。

内部区域（internal）：信任网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接。

外部区域（external）：不相信网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接。

注：FirewallD的默认区域是public。

firewalld默认提供了九个zone配置文件：block.xml、dmz.xml、drop.xml、external.xml、 home.xml、internal.xml、public.xml、trusted.xml、work.xml，他们都保存在“/usr/lib /firewalld/zones/”目录下。

配置方法
firewalld的配置方法主要有三种：firewall-config、firewall-cmd和直接编辑xml文件，其中 firewall-config是图形化工具，firewall-cmd是命令行工具，而对于linux来说大家应该更习惯使用命令行方式的操作，所以 firewall-config我们就不给大家介绍了。

安装配置：


1、安装firewalld

root执行 # yum install firewalld firewall-config

 

2、运行、停止、禁用firewalld

启动：# systemctl start  firewalld

查看状态：# systemctl status firewalld 或者 firewall-cmd --state

停止：# systemctl disable firewalld

禁用：# systemctl stop firewalld

systemctl mask firewalld

systemctl unmask firewalld

 

4、配置firewalld

查看版本：$ firewall-cmd --version

查看帮助：$ firewall-cmd --help

查看设置：

显示状态：$ firewall-cmd --state

查看区域信息: $ firewall-cmd --get-active-zones

查看指定接口所属区域：$ firewall-cmd --get-zone-of-interface=eth0

拒绝所有包：# firewall-cmd --panic-on

取消拒绝状态：# firewall-cmd --panic-off

查看是否拒绝：$ firewall-cmd --query-panic

 

更新防火墙规则：# firewall-cmd --reload

          # firewall-cmd --complete-reload

    两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务

 

将接口添加到区域，默认接口都在public

# firewall-cmd --zone=public --add-interface=eth0

永久生效再加上 --permanent 然后reload防火墙

 

设置默认接口区域

# firewall-cmd --set-default-zone=public

立即生效无需重启

 

打开端口（貌似这个才最常用）

查看所有打开的端口：

# firewall-cmd --zone=dmz --list-ports

加入一个端口到区域：

# firewall-cmd --zone=dmz --add-port=8080/tcp

若要永久生效方法同上

 

打开一个服务，类似于将端口可视化，服务需要在配置文件中添加，/etc/firewalld 目录下有services文件夹，这个不详细说了，详情参考文档

# firewall-cmd --zone=work --add-service=smtp

 

移除服务

# firewall-cmd --zone=work --remove-service=smtp

 

还有端口转发功能、自定义复杂规则功能、lockdown





iptables 是与最新的 3.5 版本 Linux 内核集成的 IP 信息包过滤系统。如果 Linux 系统连接到因特网或 LAN、服务器或连接 LAN 和因特网的代理服务器， 则该系统有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。