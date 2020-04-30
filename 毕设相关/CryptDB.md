# CryptDB加密数据库

## 一、安装CryptDB

### 1.1 安装准备

- 参考链接：[https://leeyuxun.icu/CryptDB%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8.html](https://leeyuxun.icu/CryptDB安装与使用.html)

```shell
# 虚拟机：Ubuntu12.04

# 安装git、ruby
apt-get install git ruby
```

### 1.2 克隆CryptDB到本地

```shell
git clone -b public git://g.csail.mit.edu/cryptdb
```

### 1.3 安装CryptDB

```shell
cd cryptdb
./scripts/install.rb .   # 后面的.表示cryptDB的安装路径

# 在安装过程中需要设置mysql-server的密码，cryptDB设置的默认密码为letmein，如果设置的密码和这个不一致，可以更改文件实现
cd cryptdb/mysqlproxy
vim wrapper.lua

#  找到其中  os.getenv("CRYPTDB_PASS") or "letmein"  ， 将其中的letmein改为你设置的mysql密码，重新到cryptDB中执行make指令
cd cryptdb
make   
```

### 1.4 设置EDBDIR环境变量

```shell
export EDBDIR=/root/software/cryptdb
```

### 1.5 启动3个终端

```shell
# 终端1：运行mysql-proxy代理，默认监听3307端口的SQL语句并进行加密，将加密内容传输给mysql-server服务端
# 终端2：启动3307端口的mysql客户端，用于给mysql-server发送SQL语句，但是语句会被终端1的mysql-proxy代理拦截(明文)
# 终端2：启动3306端口的mysql客户端，用来查看最终结果(密文)


# 1. 启动mysql-proxy代理
root@ubuntu: /root/software/cryptdb/bins/proxy-bin/bin/mysql-proxy  \
   --plugins=proxy --event-threads=4             \
   --max-open-files=1024                         \
   --proxy-lua-script=$EDBDIR/mysqlproxy/wrapper.lua \
   --proxy-address=127.0.0.1:3307                \
   --proxy-backend-addresses=localhost:3306
   
# 2. 启动3307端口的mysql客户端
mysql -uroot -p -h 127.0.0.1 -P 3307   #  (password: letmein)

# 3. 启动3306端口的mysql客户端
mysql -uroot -p                        #  (password: letmein)


# 操作 & 结果
# 在终端2中输入SQL语句，在终端1中可以看到这些SQL语句以及加密后的语句，在终端3中可以看到加密后的数据库表以及内容



```



