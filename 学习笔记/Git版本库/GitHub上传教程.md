# GitHub上传教程

## 一、本地Git的操作

### 1.1 下载Git

` ` 下载地址：https://git-scm.com/downloads

### 1.2 配置邮箱和姓名

```shell
git config --global user.name "your name"
git config --global user.email "your email"

# 查看配置的邮箱和用户名
git config user.name
git config user.email
```

### 1.3 生成ssh私钥和公钥

```shell
ssh-keygen -t -rsa -C "your email"
```

### 1.4 新建本地仓库

```shell
cd D:\software\git\content    #本地仓库的目录
git init                   # 创建本地仓库
```

### 1.5 提交本地代码

```shell
git add text.txt             # 将text文件增加到暂存区
git commit -m "示例文件"      # 提交代码并配置注释说明
git remote add origin git@github.com:14030140108/GitManager.git   # 配置远程github仓库的地址，以及别名origin
# git remote rm origin     # 删除配置的远程仓库别名和地址(一般情况不用)
# git pull --rebase origin master  (拉取远程仓库的所有文件至本地目录)
git push -u origin master     # 提交文件到远程仓库
```

### 1.6 非第一次提交代码步骤

```shell
git add text.txt
git commit -m "示例说明"
git push -u origin master   # origin为之前连接远程仓库别名，需要记住，如果没有记住，重新增加一个别名即可
```



## 二、Github上配置

### 2.1 增加ssh密钥

```shell
1. 点击setting
2. 将C:\Users\LinWang\.ssh\id_rsa.pub中的内容增加到Github的ssh密钥列表中
```

### 2.2 创建仓库

```shell
# 按照Github步骤操作即可
```

## 三、git操作

### 3.1 git删除文件并同步远程仓库

```shell
git rm test.txt
git commit -m "删除test文件"
git push -u origin master
```

### 3.2 git 拉取远程仓库内容

```shell
git pull origin master
```

### 3.3 检出命令

```shell
git checkout fileName   # 撤销命令，先从暂存区恢复文件，如果没有，从版本库恢复内容(工作区就是还没有add的时候						的操作，暂存区就是已经调用了git add操作之后的区域，版本库就是git commit之后的区域)
```

### 3.4 删除本地文件，从远程仓库获取

```shell
git fetch --all        #拉取远程仓库不合并
git reset --hard origin/master   #
git pull origin master   #拉取远程内容到本地仓库
```

