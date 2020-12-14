

# Phoenix

## 一、 Java连接Phoenix

### 1.1 导入Maven依赖

```xml
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.10.0-HBase-0.98</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.14</version>
</dependency>
```

### 1.2 application.yml中增加phoenix的数据源配置

```yml
spring:
  datasource:
    driver-class-name: org.apache.phoenix.jdbc.PhoenixDriver
    url: jdbc:phoenix:master:2181
    type: com.alibaba.druid.pool.DruidDataSource
    data-username:
    data-password:
```

### 1.3 使用MyBatis操作Phoenix

```java
@Repository
public interface PhoenixMapper {

    @Select("SELECT * FROM \"user\" WHERE id = #{userId}")
    User getUser(@Param("userId") int userId);

   // @Update("CREATE TABLE IF NOT EXISTS ${tableName} (id BIGINT NOT NULL PRIMARY KEY,name CHAR(20),age INTEGER)")
    @UpdateProvider(type = com.wl.mapper.MapperProvider.class, method = "create")
    void create(@Param("tableName") String tableName, List<String> fields);

    @Update("UPSERT INTO \"${tableName}\" VALUES(${user.id},'${user.name}',${user.age})")
    void insert(@Param("tableName") String tableName, @Param("user") User user);
}


public class MapperProvider {

    public String create(String tableName, List<String> fields) {
        String result = "CREATE TABLE IF NOT EXISTS \"" +
                tableName + "\" ( ";
        String temp = "\"";
        for (int i = 0; i < fields.size(); i++) {
            if (i == 0) {
                temp += fields.get(i) + "\" VARCHAR(50) NOT NULL PRIMARY KEY, \"";
            } else if (i != fields.size() - 1) {
                temp += fields.get(i) + "\" VARCHAR(50), \"";
            } else {
                temp += fields.get(i) + "\" VARCHAR(50))";
            }
        }
        return result + temp;
    }
}
```

## 二、Phoneix中常用SQL语法

### 2.1 SQL语法

```sql
# 创建表语句
CREATE TABLE IF NOT EXISTS GeoTest(GeoID BIGINT NOT NULL PRIMARY KEY,Latitude DOUBLE,Longitude DOUBLE,Time DATE);

# 插入语句
UPSERT INTO GeoTest VALUES (1,35.3,120.6,TO_DATE("2020-12-13 11:03:23"));

# 查询语句
```

### 2.2 实现自增

```sql
# sequence的用法：https://www.cnblogs.com/hbase-community/p/8853573.html

create table if not exists user(id integer not null primary key,name varchar);
create sequence id_sequence            //创建一个序列，从1开始，自增的大小为1
upsert into user values(next value for id_sequence,'root');
```



