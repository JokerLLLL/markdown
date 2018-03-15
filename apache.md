<VirtualHost *:80>
    ServerName hz.com
    DocumentRoot "D:\www"                #不能是根目录         
    <Directory D:\www>   
     Options Indexes FollowSymLinks      #选项  没有index时显示文件夹 允许文件链接
             AllowOverride all           #文件重写 .htaccess起作用
             order deny，allow           #用于ip的访问许可
             allow from all               允许所以ip
             Require all granted         #允许所以请求
    </Directory>
</VirtualHost>