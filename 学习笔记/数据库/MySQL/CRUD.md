## 一、常见命令

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

## 二、数据操作

### 2.1 插入数据记录

#### 2.1.1 插入完整数据记录

```mysql
INSERT INTO tableName(field1,field2,...,fieldn) values(value1,value2,...,valuen)
```

#### 2.1.2 插入部分数据记录

#### 2.1.3 插入多条完整数据记录

```mysql
INSERT INTO tableName(field1,field2,...,fieldn) values(value1,value2,...,valuen),(value1,value2,...,valuen)
```

### 2.2 更新数据记录

#### 2.2.1 更新数据记录

```mysql
UPDATE tableName SET field1=value1,field2=value2 WHERE CONDITION
```

### 2.3 删除数据记录

```mysql
DELETE DROP tableName WHERE CONDITION
```

## 三、数据查询

### 3.1 简单查询

```mysql
SELECT field1,field2,field3 FROM tableName WHERE CONDITION1 GROUP BY fieldm HAVING CONDITION2 ORDER BY fieldn ASC|DESC
```

#### 3.1.1 DISTINCT查询

```mysql
# 查询结果去重
SELECT DISTINCT field1,field2,field3 FROM t_student
```

#### 3.1.2 IN查询

```mysql
SELECT field1,field2,field3 FROM tableName WHERE fieldm IN(value1,value2,value3)
```

#### 3.1.3 NOT IN查询

```mysql
SELECT field1,field2,field3 FROM tableName WHERE fieldm NOT IN(value1,value2,value3)
```

- 在具体使用关键字IN时，查询的集合中如果存在NULL，则不会影响查询结果，使用关键字NOT IN，查询的集合中如果存在NULL，则不会有任何的查询结果

#### 3.1.4 BETWEEN AND查询

```mysql
SELECT field1 FROM tableName where fieldm between minvalue and maxvalue
```

#### 3.1.5 不符合范围的数据记录查询

```mysql
SELECT field1 FROM tableName where fieldm not between minvalue and maxvalue
```

#### 3.1.6 LIKE模糊查询

```mysql
select field1 from tableName where field like value
```



