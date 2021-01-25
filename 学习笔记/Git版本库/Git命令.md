### git常用命令

```git
# 表示在使用git push命令时，将本地当前分支的内容推送到远程的master分支(之后可以直接使用git push)
1. git push --set-upstream origin master 

# 表示在使用git pull命令时，将本地当前分支的内容关联到远程指定的master分支(之后可以直接使用git pull)
2. git branch --set-upstream-to=origin/master

# 查看git配置的远程仓库的URL
3. git remote -v 

# 删除远程的dev分支
4. git push origin --delete dev    or   git push origin :dev (:前面为本地分支，后面为远程分支，推送一个空分支到远程分支，相当于删除远程分支)

# 创建本地dev分支并关联远程的dev分支
5. git checkout -b dev origin/dev

# 从远程仓库拉取内容到本地
6. git pull origin master --allow-unrelated-histories

7. git reset的三种模式 --hard --soft --mixed(默认)
	# 修改HEAD和master的hash为指定hash值，并更新工作区和暂存区和版本库
	- git reset --hard HEAD^(hash值) 
	# 修改HEAD和master的hash为指定hash值，，修改版本库内容为head所指向的内容，不会覆盖工作区和暂存区
	- git reset --soft HEAD^   
	# 修改HEAD和master的hash为指定hash值，修改版本库和暂存区内容为head所指向的内容，不会覆盖工作区
	- git reset --mixed HEAD^
   
# 查看暂存区中的内容
8. git ls-files -s

# 删除远程分支上的文件
9. git rm -r --cached /target

# 强制推送本地内容到远程仓库
10. git push -f origin master
```

