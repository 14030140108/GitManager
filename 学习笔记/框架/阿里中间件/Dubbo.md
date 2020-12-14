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

public class MainApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("consumer.xml");
        OrderService bean = ioc.getBean(OrderService.class);
        List<UserAddress> userAddresses = bean.initOrder("1");
        for (UserAddress userAddress : userAddresses) {
            System.out.println(userAddress);
        }
        System.in.read();
    }
}
```

## 三、Dubbo和Springboot的整合

- Springboot和Dubbo整合的三种方式
  - 利用dubbo提供的API
  - 注解
  - XML  

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

## 四、配置覆盖关系

- 方法级优先，接口级次之，全局配置在次之
- 如果级别一样，则消费者优先，提供方次之()

```xml
<!--配置关系为数字标号，数字越小优先级越高-->
消费者：(方法级优先，接口级次之，全局最后)
<dubbo:method></dubbo:method>                     (1)
<dubbo:reference></dubbo:reference>				 (3)
<dubbo:consumer></dubbo:consumer>				 (5)

提供方：(方法级优先，接口级次之，全局最后)
<dubbo:method></dubbo:method>					(2)
<dubbo:service></dubbo:service>					(4)
<dubbo:provider></dubbo:provider>				(6)
```

## 五、配置属性

### 5.1 timeout

### 5.2 version

### 5.3 retries

### 5.4 本地存根

- 消费者调用服务端的服务之前，先调用本地存根的代码逻辑做一些参数验证等

## 六、负载均很

### 6.1 基于权重

### 6.2 基于轮询

### 6.3 基于一致性Hash

### 6.4 基于权重的随机负载均衡

## 七、服务降级

### 7.1 不发起远程调用，在客户端层面直接返回为空

### 7.2 超时之后返回为空

 ## 八、Dubbo原理

### 8.1  标签解析

```java
//该方法位于AbstractApplicationContext的refresh中，标签的解析就在这一步中实现
ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
```

- obtainFreshBeanFactory函数中调用了refreshBeanFactory函数，该函数中调用loadBeanDefinitions进行了BeanDefinition的加载
- 加载之后进行注册BeanDefinition，最后进行解析parseCustomElement，该方法中首先获取NameSpaceHandler，之后调用该Handler的parse
- DubboNamespaceHandler中初始化了每个标签对应的解析类，实现了NameSpaceHandler接口

```java
public void init() {
        registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
    }
```

- 上述标签和解析类的映射关系用map进行存储

### 8.2 服务暴露

- 服务暴露的过程发生在refresh中的finishRefresh方法中
- ServiceBean为解析标签之后生成的配置类，里面包含所有的标签配置信息

```java
/* 1. ServiceBean类实现了InitializingBean接口，通过调用接口中的afterPropertiesSet方法实现将标签中的配置信息全部存放在ServiceBean中，发生在refresh中的finishBeanFactoryInitialization方法中
*  2. ServiceBeans类实现了ApplicationListener监听器，服务的暴露就是通过调用接口中的onApplicationEvent方法的export来实现的
*/
public class ServiceBean<T> extends ServiceConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware {
}
```

- export方法

```java
/* export方法最终会调用DubboProtocol和RegisterProtocol中的export方法
*  1. 其中DubboProtocol负责开启netty服务器，并监听配置文件定义的ip和端口(默认20880)
*  2. RegisterProtocol将接口和接口实现的包装类存储在本地CHM中，并注册到注册中心中
*/
```

### 8.3 服务引用



### 8.4 服务调用



 