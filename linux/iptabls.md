INPUT链 – 处理来自外部的数据。
OUTPUT链 – 处理向外发送的数据。
FORWARD链 – 将数据转发到本机的其他网卡设备上

service iptables start            #开启
service iptables stop             #停止
service iptables restart          #重启
service iptables status           #查看状态
service iptalbes save             #保存
/etc/sysconfig/iptables-config    #服务配置文件
/etc/sysconfig/iptables           #规则保存文件
echo "1"> /proc/sys/net/ipv4/ip_forward   #打开iptables转发(默认0)


iptables [ -t 表名] 命令选项 [链名] [条件匹配] [-j 目标动作或跳转]
表名：
    FIlter(默认-过滤功能),NAT,Mangle,Raw
命令选项：
    -A	在指定链的末尾添加（ –append ）一条新的规则
    -D	删除（ –delete ）指定链中的某一条规则，按规则序号或内容确定要删除的规则
    -I	在指定链中插入（ –insert ）一条新的规则，默认在链的开头插入
    -R	修改、替换（ –replace ）指定链中的一条规则，按规则序号或内容确定
    -L	列出（ –list ）指定链中的所有的规则进行查看，默认列出表中所有链的内容
    -F	清空（ –flush ）指定链中的所有规则，默认清空表中所有链的内容
    -N	新建（ –new-chain ）一条用户自己定义的规则链
    -X	删除指定表中用户自定义的规则链（ –delete-chain ）
    -P	设置指定链的默认策略（ –policy ）
    -n	用数字形式（ –numeric ）显示输出结果，若显示主机的 IP 地址而不是主机名
    -P	设置指定链的默认策略（ –policy ）
    -v	查看规则列表时显示详细（ –verbose ）的信息
    -V	查看 iptables 命令工具的版本（ –Version ）信息
    -h	查看命令帮助信息（ –help ）
    –line-number	查看规则列表时，同时显示规则在链中的顺序号  
链名：
  FIlter表
    INPUT链 – 处理来自外部的数据。
    OUTPUT链 – 处理向外发送的数据。
    FORWARD链 – 将数据转发到本机的其他网卡设备上。
条件匹配：
    -p	指定规则协议，如 tcp, udp,icmp 等，可以使用 all 来指定所有协议
    -s	指定数据包的源地址参数，可以使 IP 地址、网络地址、主机名
    -d	指定目的地址
    -i	输入接口
    -o	输出接口
目标值：
    ACCEPT：允许数据包通过。
    DROP：直接丢弃数据包，不给出任何回应信息。
    REJECT：拒绝数据包通过，必须时会给数据发送端一个响应信息。
    LOG：在/var/log/messages 文件中记录日志信息，然后将数据包传递给下一条规则。
    QUEUE：防火墙将数据包移交到用户空间
    RETURN：防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)

#常见命令：
iptables -F                               #删除所有规则
iptables -L(iptables -L -V -n)            #查看规则
iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
                                          #增加规则到最后
iptables -I INPUT 2 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
                                          #添加一条规则到指定位置
iptables -D INPUT 2                       #删除一条规则
iptables -R INPUT 3 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
                                          #修改一条规则
iptables -P INPUT DROP                    #修改默认策略

#基本配置:
a)删除现有规则
iptables -F

b)配置默认链策略
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

c)允许远程主机进行 SSH 连接
iptables -A INPUT -i eth0 -p tcp –dport 22 -m state –state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp –sport 22 -m state –state ESTABLISHED -j ACCEPT

d)允许本地主机进行 SSH 连接
iptables -A OUTPUT -o eth0 -p tcp –dport 22 -m state –state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -p tcp –sport 22 -m state –state ESTABLISHED -j ACCEPT

e)允许 HTTP 请求
iptables -A INPUT -i eth0 -p tcp –dport 80 -m state –state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp –sport 80 -m state –state ESTABLISHED -j ACCEPT


#使用 iptables 抵抗常见攻击
1. 防止 syn 攻击
思路一：限制 syn 的请求速度（这个方式需要调节一个合理的速度值，不然会影响正常用户的请求）
iptables -N syn-flood
iptables -A INPUT -p tcp --syn -j syn-flood
iptables -A syn-flood -m limit --limit 1/s --limit-burst 4 -j RETURN
iptables -A syn-flood -j DROP
思路二：限制单个 ip 的最大 syn 连接数
iptables –A INPUT –i eth0 –p tcp --syn -m connlimit --connlimit-above 15 -j DROP

2. 防止 DOS 攻击
利用 recent 模块抵御 DOS 攻击
iptables -I INPUT -p tcp -dport 22 -m connlimit --connlimit-above 3 -j DROP
单个 IP 最多连接 3 个会话
iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH  
只要是新的连接请求，就把它加入到 SSH 列表中
Iptables -I INPUT -p tcp --dport 22 -m state NEW -m recent --update --seconds 300 --hitcount 3 --name SSH -j DROP  
5 分钟内你的尝试次数达到 3 次，就拒绝提供 SSH 列表中的这个 IP 服务。被限制 5 分钟后即可恢复访问。

3. 防止单个ip访问量过大
iptables -I INPUT -p tcp --dport 80 -m connlimit --connlimit-above 30 -j DROP

4. 木马反弹
iptables –A OUTPUT –m state --state NEW –j DROP

5. 防止ping攻击
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/m -j ACCEPT
