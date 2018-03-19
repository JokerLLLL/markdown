#基础篇
/etc/logrotate.d/nginx   配置文件  日志轮转配置
/etc/nginx               配置文件目录
源码包安装配置目录：
/usr/local/nginx/conf/nginx.conf
/usr/local/nginx/conf/nginx.conf.default
/usr/local/nginx/conf/fastcgi_params                   配置文件   cgi  fastcgi
/usr/local/nginx/conf/uwsgi_params
/usr/local/nginx/conf/scgi_params
/usr/local/nginx/conf/koi-utf                          编码转化文件
/usr/local/nginx/conf/mime.types                       http协议content-type与扩展名对应关系
/sys/fs/cgroup/systemd/system.slice/nginx.service      守护进程文件

yum源配置：
/etc/nginx/


编译参数：
--prefix=/usr/local/nginx             安装目录或路径
--user=www                            所属人
--group=www                           所属组


nginx.conf语法说明：
user    
worker_process  工作进程数
error_log       nginx错误日志
pid             nginx服务启动pid

events{
  worker_connenctions   进程允许链接最大数
  use                   工作进程数
}
