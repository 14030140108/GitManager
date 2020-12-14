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

## 二、Springboot中使用技巧

### 2.1 new出来的的对象中如何使用Spring管理的bean

```java
//实现一个GetBeanUtil工具类,实现ApplicationContextAware接口
@Component
public class GetBeanUtil implements ApplicationContextAware {

    private static ApplicationContext context = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (GetBeanUtil.context == null) {
            context = applicationContext;
        }
    }

    public static Object getBean(String name) {
        return context.getBean(name);
    }

    public static <T> T getBean(Class<T> c) {
        return context.getBean(c);
    }

    public static ApplicationContext getContext() {
        return context;
    }
}

//自己要new的类
public class Test{
	PhoenixService phoenixService = GetBeanUtil.getBean(PhoenixService.class);    
}

```

