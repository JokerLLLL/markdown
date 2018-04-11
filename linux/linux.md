tail -f /tmp/date.txt  #实时更新
tail -3 /tmp/date.txt  #最后三行
date +%w               #返回星期几
test 1 = 0             #测试语句
echo $?                #返回最后一个语句生成的值
cat /proc/cpuinfo      #查看cup
fdisk  -l              #查看硬盘
ifconfig -a            #查看网卡


#查看进程
ps aux|grep server.php        #查看进程
pstree -ap|grep server.php    #查看进程树

ll /proc/进程号               #通过进程号查看进程信息       

## kill命令
kill -l    所有信号
以下为中断信号：
    HUP     1    终端断线
    INT     2    中断（同 Ctrl + C）
    QUIT    3    退出（同 Ctrl + \）
    TERM   15    终止
    KILL    9    强制终止
    CONT   18    继续（与STOP相反， fg/bg命令）
    STOP   19    暂停（同 Ctrl + Z）

kill -SIGUSR1 7259  #重启配置文件

#端口查看
lsof -i:9505
netstat -anp|grep 9505
netstat -luntp|grep 80   #查看监听端口
      -a (all)显示所有选项，默认不显示LISTEN相关
      -t (tcp)仅显示tcp相关选项
      -u (udp)仅显示udp相关选项
      -n 拒绝显示别名，能显示数字的全部转化成数字。
      -l 仅列出有在 Listen (监听) 的服務状态

      -p 显示建立相关链接的程序名
      -r 显示路由信息，路由表
      -e 显示扩展信息，例如uid等
      -s 按各个协议进行统计
      -c 每隔一个固定时间，执行该netstat命令。

#tar命令
tar -xzvf file.tar.gz              -C  /tmp/ #指定解压目录      -c打包 -x解包 -v显示过程 -f指定打包后名
tar -xjvf file.tar.bz2

#rar
rar -x file.rar 

#编译安装
./configure
make
make install
make test       #测试是否完成安装

#文件命令
cp  xxx.tar.gz  /usr/local/    #复制文件
mv  *.php  1/                  #移除文件到1下*****
mv  1.txt  1.html              #重名
rm  -rf   1.html               #强制删除

#swoole编译安装
phpize                                #关联到PHP拓展库
./configure   --enable-async-redis    #启用异步redis  需要安装hiredis  C客户端的支持
make                                  #编译
sudo make install                     #编译安装

#find 查找
find / -name nginx*                   #匹配所有nginx的目录以及文件
