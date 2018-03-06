npm install hexo-deployer-git --save

hexo clean

hexo generate

hexo deploy

hexo new "postName" #新建文章

hexo new page"pageName" #新建页面

hexo generate #生成静态页面至public目录

hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）

hexo deploy #将.deploy目录部署到GitHub

hexo help # 查看帮助

hexo version #查看Hexo的版本


.
├── .deploy       #需要部署的文件

├── node_modules  #Hexo插件

├── public        #生成的静态网页文件

├── scaffolds     #模板

├── source        #博客正文和其他源文件, 404 favicon CNAME 等都应该放在这里

|    ├── _drafts   #草稿

|    └── _posts    #文章

├── themes        #主题

├── _config.yml   #全局配置文件

└── package

#更新
hexo g
hexo d
