# 一、常见命令

```mysql
# 创建user表
create table user(
	id int(11) PRIMARY KEY,
	username VARCHAR(20),
	sex VARCHAR(20)
)ENGINE= INNODB,CHARACTER SET utf8;

# 修改表结构，在user表中增加，删除，修改字段(字段类型+字段名称)
ALTER TABLE `user` ADD COLUMN passwd VARCHAR(20);
ALTER TABLE `user` DROP COLUMN passwd;
ALTER TABLE `user` MODIFY COLUMN sex VARCHAR(30);
ALTER TABLE `user` CHANGE sex1 sex VARCHAR(30);
```

