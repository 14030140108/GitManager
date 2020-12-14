# Linux命令

## 1. grep命令

```shell
1. grep "被查找的字符串" filename   # 从指定的文件名中查找字符串

2.  grep -r "..." .   # 从当前目录文件递归查询指定的字符串 
2.1 find . -name *.* | xargs grep "..."   #与上述的含义一样
```

