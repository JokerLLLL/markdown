初始化一个仓库      git init

查看仓库状态        git status

预提交仓库          git add *

移除缓存            git rm -cached

正式更新提交        git commit -m 'first commit'

查看所有commit记录  git log

查看分支所在位置     git branch

新建分支            git branch test1

切换分支            git checkout test1

                   git checkout v1.0

                   git checkout 8739df5828a9a4c2ef3de8fa83880f73e0f920d2   [commit_id ]

                   git checkout abc  [将未add的文件进行还原]

新建切换           git checkout -b test2

合并分支到master   git merge  test1

删除分支           git branch -d test1

                  git branch -D test1  [强制删除]

标签历史           git tag

新建标签           git tag v1.0

SSH链接远程仓库    ssh-keygen -t rsa
生成SSH key  生成id_rsa和id_rsa.pub        
Linux/Mac 系统 在 ~/.ssh 下，win系统在 /c/Documents and Settings/username/.ssh
然后将id_rsa.pub 的内容添加到 GitHub

Push & Pull

克隆远程仓库      git clone git@github.com:JokerLLLL/gitTest.git

本地执行 add commit

更新远程仓库       git push origin master  

关联本地和远程仓库 git remote add origin    [origin]为仓库设置的名称

设置全局姓名邮箱   git config --global user.name "JokerLLLL"

                  git config --global user.email "145645529@qq.com"

别名配置alias     git config --global alias.co checkou #别名

                  git config --global alias.ci commit

                  git config --global alias.st status

                  git config --global alias.br branch

                  git config --global alias.psm 'push origin master'

                  git config --global alias.plm 'pull origin master'

   #git lg        git config --global alias.lg 'log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative'

其他配置          git config --global core.editor "vim"

                  git config --global color.ui true

                  git config --global core.quotepath false #设置显示中文文件名

~/.gitconfig 文件 git config -l

比较命令diff(比较add之前的改动）
                  git diff
