# GitHub和Git详解

## 一、GItHub名词

- 仓库(repository)：仓库就是项目，当你在github上开源一个项目时，那就必须新建一个repository
- 收藏(star)：你的项目被别人收藏的人数
- 复制(fork)：你的项目被别人复制的次数

```shell
#当你fork一个别人的项目时，会在你自己的个人中心多出一个同名仓库并显示其来源，你可以在自己fork出来的项目中随便修改，不会影响原项目，当你改完之后你可以发起一个pull request请求，表示希望原项目可以合并其修改，原项目作者可以看到其pull request(PR),同意则合并。
```

- 关注(Watch)：你可以观察某个项目，当这个项目有修改的时候，就会通知你
- 事务卡片(Issue)：发现代码bug时使用，可以用来讨论

## 二、Git名词

- 工作区(Work Directory) ：添加、编辑、修改文件等动作
- 暂存区：暂存已经修改的文件最后统一提交到git仓库中
- Git仓库(Git Repository)：最终确定的文件保存到仓库，成为一个新的版本，并对他人可见

### 2.1 集中式(SVN)和分布式(GIT)

#### 2.1.1 SVN

- SVN每次存储版本之间的差异，需要的空间相对小一些，可是在回滚的时候速度会很慢
- 代码放在单一的服务器上，便于项目的管理
- 服务器宕机或者损坏时，会丢失记录

#### 2.1.2 GIT

- Git每次存储的是项目的完整快照(索引)，需要的硬盘空间相对大一些(Git团队对代码做了压缩，最终需要的实际空间大小比SVN多不了太多)，可是Git回滚的速度极快、
- 断网的情况下也可以进行，因为git是在本地进行的
- 使用github进行团队协作，如果github挂了，每个客户端也保存的是整个完整的项目(包含历史记录)

## 三、git操作

### 3.1 Git基本命令

#### 3.1.1 git删除文件并同步远程仓库

```shell
git rm test.txt
git commit -m "删除test文件"
git push -u origin master
```

#### 3.1.2 git 拉取远程仓库内容

```shell
git pull origin master
```

#### 3.1.3 检出命令

```shell
git checkout fileName   # 撤销命令，先从暂存区恢复文件，如果没有，从版本库恢复内容(工作区就是还没有add的时候						的操作，暂存区就是已经调用了git add操作之后的区域，版本库就是git commit之后的区域)
```

#### 3.1.4 删除本地文件，从远程仓库获取

```shell
git fetch --all        #拉取远程仓库不合并
git reset --hard origin/master   #
git pull origin master   #拉取远程内容到本地仓库
```

#### 3.1.5 查看提交的历史记录

```shell
git log --oneline   # 查看提交的历史记录
```



### 3.2 分支操作

#### 3.2.1 创建分支

```shell
git branch name   # 创建分支
```

#### 3.2.2 切换分支

```shell
git checkout name   # 切换分支
```

#### 3.2.3 删除分支

```shell
git branch -D name  # 强制删除分支
```

#### 3.2.4 显示分支列表

```shell
git branch  # 显示分支列表
```



### 3.3 git对象

- git对象在git中是键值对存在的，key是value对应的hash值

```shell
echo "test content" | git hash-object -w --stdin   # 将“test content”保存到数据库中并返回其对应的key


find ./  -type p   # 查看git保存数据的路径

git cat-file -p hasnKey  # 可以查看key对应的内容
git cat-file -t hasnKey  # 可以查看git对象的类型


# 一般使用git是管理文件
git hash-object -w ./test.txt   # 将test.txt文件保存到数据库中并返回hash值

```

