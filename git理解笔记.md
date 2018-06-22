#版本回退
git reset --hard 版本号或Head^
退回某个版本或上几个版本

git reflog
查看历史命令 (带上版本号)

git checkout -- [filename]
把工作区回退到最后一次 git add 或 git commit 的状态

git reset head
把当前目录的 缓冲区里(git add 提交的) 的撤销

#文件删除
git status
查看是否有文件删除

git rm [filename]
提交到缓冲区(会删除工作区文件)

git checkout -- [filename]
如果想从版本库恢复

git commit -m 'delete file'
提交版本库

#分支
git checkout -b develop
新建分支develop并把head头指向develop [-b表示 branch]

git checkeout master
把head头指向master(切换分支)

git branch develop
创建develop分支

git branch
查看分支

git branch -d develop
删除分支

git branch -D feature
强制删除分支 {feature有提交，但未进行合并，会进行数据丢失}

git merge develop
在master分支中执行，合并 {这种合并会是fast forward,会破会树形结构,或会产生recursive报警=此时效果和--no-ff参数一样}

git merge develop --no-ff -m 'meger master by jokerl'
--no-ff参数表示不使用fast forward提交，而进行了一次commit


#隐藏工作区
git stash
需要切换其他分支 但工作区文件未并不想先提交 {对于新建文件 未追踪文件要先 git add 进行增加}

git stash list
查看备份

git stash apply
恢复工作区

git stash drop
丢弃备份

git stash pop
恢复+丢弃

#远程仓库 + 多人合作
git remote
得到远程默认仓库名，一般都为origin

git remote -v
查看具体可以push 和pull的源

git clone 127.0.0.1:demo.git
克隆远程仓库

git checkout -b develop origin/develop
创建本地的develop并关联远程仓库的devel的分支

git branch --set-upstream-to=origin/develop
关联本地分支和远程分支

git push [origin] [master]
在哪个分支下使用 push 就推送那个分支 或向这样完整写明

#标签功能
git tag
查看所以标签

git tag v1.0 [commit_id]
添加当前标签或某个提交点标签

git show v1.0
查看标签节点变得信息

git tag -a v2.0 -m '版本2.0' [commit_id]
添加标签和备注

git push origin v1.0
推送标签

git push origin --tags
推送所有标签

git tag -d v1.0
删除标签

git push origin :refs/tags/v1.0
删除远程标签
