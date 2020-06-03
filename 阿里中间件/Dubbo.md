# Dubbo

## 一、Dubbo详解

### 1.1 Dubbo定义

- Apache Dubbo 是一个基于Java的高性能，轻量级的RPC框架。Dubbo提供了三个关键功能，包括基于接口的远程呼叫，容错和负载平衡以及自动服务注册和发现。

### 1.2 Dubbo流程图

![](E:\框架\Dubbo\img\Dubbo框架执行流程.png)

### 1.3  Dubbo与Springboot整合依赖

```xml
<dependency>
	<groupId>com.alibaba.boot</groupId>
	<artifactId>dubbo-spring-boot-starter</artifactId>
	<version>0.2.0</version>
</dependency>
```

## 二、基于XML的Dubbo

### 2.1 注册中心

```java
package com.wl.service;

import com.wl.bean.UserAddress;

import java.util.List;

//将服务的接口注册在zookeeper中
public interface OrderService {
    public List<UserAddress> initOrder(String userId);
}

```

### 2.2 提供者

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--指定当前服务的名字-->
    <dubbo:application name="userService"/>

    <!--指定通信协议以及通信端口-->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!--指定注册中心的位置-->
    <dubbo:registry  protocol="zookeeper" address="127.0.0.1:2181"/>

    <dubbo:service interface="com.wl.service.UserService" ref="userServiceImp"/>

    <bean id="userServiceImp" class="com.wl.service.UserServiceImp"/>

</beans>
```

```java
@Service   //此注解是dubbo与springboot整合时使用
public class UserServiceImp implements UserService {
    @Override
    public List<UserAddress> getUserAddress(String userId) {
        UserAddress address1 = new UserAddress("李华", "1", "西安电子科技大学北校区", "123456");
        UserAddress address2 = new UserAddress("李华", "1", "西安电子科技大学长安校区", "123456");
        return Arrays.asList(address1, address2);
    }
}

public class MainApplication {
    public static void main(String[] args){
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("provider.xml");
        ac.start();
        System.in.read();
    } 
}
```

### 2.3 消费者

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.wl.service"/>
    <!-- consumer's application name, used for tracing dependency relationship (not a matching criterion),
    don't set it same as provider -->
    <dubbo:application name="OrderService"/>
    <!-- use multicast registry center to discover service -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <!-- generate proxy for the remote service, then demoService can be used in the same way as the
    local regular interface -->
    <dubbo:reference id="userService" check="false" interface="com.wl.service.UserService"/>
</beans>
```

```java
@Service
public class OrderServiceImp implements  OrderService {

    @Autowired
    UserService userService;

    public List<UserAddress> initOrder(String userId) {
        return userService.getUserAddress(userId);
    }
}
```

## 三、Dubbo和Springboot的整合

### 3.1 提供者

```properties
dubbo.application.name=userService-springboot

dubbo.protocol.name=dubbo
dubbo.protocol.host=192.168.3.120
dubbo.protocol.port=20880

dubbo.registry.protocol=zookeeper
dubbo.registry.address=127.0.0.1:2181
```

```java
@Service   //使用dubbo提供的注解可以自动将实现类远程注册至zookeeper中
public class UserServiceImp implements UserService {
    @Override
    public List<UserAddress> getUserAddress(String userId) {
        UserAddress address1 = new UserAddress("李华", "1", "西安电子科技大学北校区", "123456");
        UserAddress address2 = new UserAddress("李华", "1", "西安电子科技大学长安校区", "123456");
        return Arrays.asList(address1, address2);
    }
}
```

### 3.2 消费者

```properties
server.port=8081

dubbo.application.name=OrderService-Springboot
dubbo.registry.address=zookeeper://127.0.0.1:2181
```

```java
@Service
public class OrderServiceImp implements  OrderService {

    @Reference     //使用dubbo提供的注解可以从远程zookeeper中依赖注入接口
    UserService userService;

    public List<UserAddress> initOrder(String userId) {
        return userService.getUserAddress(userId);
    }
}

@EnableDubbo    //记得开启dubbo支持注解
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class);
    }
}
```

