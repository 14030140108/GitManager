# SpringBoot

## 一、springboot必备

### 1.1  依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
</dependency>

<!--该依赖包是restful项目开发需要，web项目-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 1.2 启动

```java
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args){
        SpringApplication.run(MainApplication.class);
    }
}

//上述方法简化了下列Spring + XML配置
public class MainApplication {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("consumer.xml");
        ioc.start();
        OrderService orderService = ioc.getBean(OrderService.class);
        orderService.initOrder("1");
    }
}
```

