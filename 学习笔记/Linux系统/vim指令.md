# 一、VIM指令

## 1.1. 剪切

```shell
# 快捷方式操作
dd:删除并剪切当前行
n + dd:删除并剪切从当前行开始的n行
将光标移动至目标行，按p粘贴(p粘贴到当前行的下一行，P可以粘贴到当前行的上一行)
# 命令行方式
:1,10 m 12    # 将1至10行内容剪切到12行
```

## 1.2. 复制

```shell
# 快捷方式操作
yy:复制当前行
n + yy:复制从当前行开始的n行
将光标移动至目标行，按p粘贴(p粘贴到当前行的下一行，P可以粘贴到当前行的上一行)
# 命令行方式
:1,10 co 12    # 将1至10行内容复制到12行
```

# 二、Ubuntu系统指令

## 2.1 查看某个被占用端口

```shell
netstat -ntlp | grep port
# -n表示以数字的形式显示地址和端口号
e.g: netstat -tlp | grep 4040
	output: tcp		0		0 localhost:4040 		*:*				LISTEN		11775/mysql-proxy
	netstat -ntlp | grep 4040
	output: tcp		0		0 127.0.0.1:4040 		0.0.0.0:*		 LISTEN		 11775/mysql-proxy

# -t表示tcp协议
# -l表示显示正在监听的tcp
# -p表示显示程序进程号和程序名
```

## 2.2 shell编程中输出指定PID

```shell
#!/bin/bash
PID=$(ps -e | grep mysql-proxy | awk '{printf $1}') #其他的地方可以使用${PID}来引用这个变量
```

