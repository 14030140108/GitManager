# MySQL安装

## 一、Ubuntu下MySQL安装

### 1.1 安装包

```shell
mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz

// 解压
xz -d mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz   # 将.tar.xz文件解压为.tar文件
tar xvf mysql-8.0.19-linux-glibc2.12-x86_64.tar    # 将.tar文件解压缩
```

### 1.2 配置my.cnf文件

```shell
[client]
default-character-set=utf8

[mysqld]

basedir = /root/software/mysql
datadir = /root/software/mysql/data
socket = /tmp/mysql.sock
log-error = /root/software/mysql/data/error.log
pid-file = /root/software/mysql/data/mysql.pid
tmpdir = /tmp
port = 3306
character_set_server=utf8
user=root   # 很重要，如果不创建mysql账户，那么必须指定一个用户，因为这个参数默认是mysql用户
# skip-grant-tables
```

### 1.3 其他配置

```shell
# 创建data文件夹
mkdir data
# 创建错误文件
touch error.log
```

### 1.4 初始化Mysql数据库

```shell
# 初始化Mysql数据库，获取初始密码
./mysqld --initialize --user=root --basedir=/root/software/mysql --datadir=/root/software/mysql/data
```

### 1.5 启动mysqld服务

```shell
# 为了安全考虑，不会直接启动mysqld服务，而是启动mysqld_safe
# 通过启动   mysql安装目录/support-files/mysql.server 也可以实现启动Mysqld服务，mysql.server中就是通过启动mysqld_safe来实现的

# 开机自动启动mysqld服务
cp /root/software/mysql/support-files/mysql.server /etc/init.d/mysql  # 将mysql.server复制到/etc/init.d,并重命名为msyql，之后开机会自动调用mysql脚本启动mysql服务

# 启动服务的三种方式：
1.  ./mysqld_safe     # 在mysql安装目录下/bin
2.  ./mysql.server start | stop | status | restart    # 在mysql安装目录下/support-files/mysql.server
3.  service mysql start | stop | restart | status    # 有第二种方式中的脚本复制到/etc/init.d,可以开机自启，也可以通过service mysql start开启
```

### 1.6 修改初始密码，配置远程登录

```shell
mysql -uroot -p  # 此时的密码为刚刚初始化数据库生成的随机密码
alter user 'root'@'localhost' identified by 'password';  # 更改root，localhost的登录密码

create user 'root'@'%' identified by 'password'   # 创建登录mysql的用户
# 授予该用户所有权限，
grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
flush privilegs;
```

## 二、Windows下MySQL安装

### 2.1 安装包

```shell
mysql-5.7.29-winx64.zip
```

### 2.2 配置my.ini文件

```shell
[mysqld]
port = 3306
basedir=D:/software/mysql/mysql-5.7.29-winx64
datadir=D:/software/mysql/mysql-5.7.29-winx64/data
max_connections=200
character-set-server=utf8
default-storage-engine=INNODB
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
# skip-grant-tables

[mysql]
default-character-set=utf8
```

### 2.3 下载缺失的库文件

```shell
# 下载链接
https://www.microsoft.com/zh-CN/download/details.aspx?id=40784
```

### 2.4 安装mysqld服务

```shell
mysqld -install
```

### 2.5 初始化MySQL数据库

```shell
mysqld --initialize-insecure --user=mysql
```

### 2.6 启动Mysqld服务

```shell
net start | stop mysql
```

### 2.7 修改密码，配置远程登录

```shell
# 在my.ini文件中mysqld中添加
skip-grant-tables   # 可以实现免密码登录

mysql -uroot -p  # 此时密码为空
alter user 'root'@'localhost' identified by 'password';  # 更改root，localhost的登录密码

create user 'root'@'%' identified by 'password'   # 创建登录mysql的用户
# 授予该用户所有权限，
grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
flush privilegs;
```

