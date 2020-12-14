# Spring文档

## 一、IOC容器

### 1.1 Spring IOC容器和Bean简介

org.springframework.beans和org.springframework.context是Spring IOC容器的基础，BeanFactory接口提供了能够管理任何类型对象的高级配置机制，ApplicationContext是BeanFactory的子接口，增加了：

- 容易与Spring AOP结合
- 消息资源处理(用于国际化)
- 应用层特定的上下文，例如webApplicationContext
- 事件发布

简而言之，BeanFactory提供了配置框架和基本功能，ApplicationContext增加了更多针对企业的功能，是对BeanFactory的完整超集。

在Spring中，构成应用程序主干并由Spring IOC容器管理的对象称为bean，Bean是由Spring IOC容器实例化，组装和以其他方式管理的对象，Bean及其之间的依赖关系反映在容器使用的配置元数据中(配置元数据为spring配置文件，可以是XML文件，Java注解或者java代码)

### 1.2 容器概述

ApplicationContext代表Spring IOC容器，并负责实例化、配置和组装bean。容器通过读取配置元数据来获取有关要实例化，配置和组装哪些对象的指令。配置元数据以XML，Java批注或Java代码表示。它使你能够表达组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

ApplicationContext提供了该接口的几种实现，通常使用ClassPathXmlApplicationContext或者FileSystemXmlApplicationContext，可以通过提供少量XML配置来声明性的启用这些其他元数据格式的支持，从而指示容器将Java注释或代码用作元数据格式

![](D:\software\git\repository\GitManager\img\SpringIOC容器.png)

#### 1.2.1 配置元数据

如上图所示，Spring IoC容器使用一种形式的配置元数据。这个配置元数据表示您作为应用程序开发人员如何告诉Spring容器在应用程序中实例化，配置和组装对象。 

// **question:** 通常，不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是DAO和业务逻辑的职责。但是，您可以使用Spring和AspectJ的集成来配置在IOC容器控制之外创建的对象。

- 基于XML配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id="..." class="...">  
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  
      <bean id="..." class="...">
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  
      <!-- more bean definitions go here -->
  
  </beans>
  ```
  
- 

#### 1.2.2 实例化容器

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

- services.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <!-- services -->
  
      <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
          <property name="accountDao" ref="accountDao"/>
          <property name="itemDao" ref="itemDao"/>
          <!-- additional collaborators and configuration for this bean go here -->
      </bean>
  
      <!-- more bean definitions for services go here -->
  
  </beans>
  ```

- daos.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id="accountDao"
          class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
          <!-- additional collaborators and configuration for this bean go here -->
      </bean>
  
      <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
          <!-- additional collaborators and configuration for this bean go here -->
      </bean>
  
      <!-- more bean definitions for data access objects go here -->
  
  </beans>
  ```

服务层由PetStoreServiceImpl类和JpaAccountDao和JpaItemDao(基于JPA对象关系映射标准，JPA是Java 持久化API，是将POJO持久化到数据库的一种标准规范，有注解和XML两种方式来描述对象和关系的映射)。该`property name`元素是指JavaBean属性的名称，以及`ref`元素指的是另一个bean定义的名称。`id`和`ref`元素之间的这种联系表达了协作对象之间的依赖性。

**组成基于XML的配置元数据**

 使用一个或多个出现的``元素从另一个文件中加载bean定义。以下示例显示了如何执行此操作： 

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

#### 1.2.3 使用容器

 该`ApplicationContext`是一个维护bean定义以及相互依赖的注册表的高级工厂的接口。通过使用方法 `T getBean(String name, Class requiredType)`，您可以检索bean的实例。 

```java
//第一种方式
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);
// use configured instance
List<String> userList = service.getUsernameList();


//第二种方式
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();

```

### 1.3 bean概述

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的。

在容器本身内，这些bean定义表示为`BeanDefinition` 对象，其中包含（除其他信息外）以下元数据：

|         Property         | Explained in ..                  |
| :----------------------: | -------------------------------- |
|          Class           | 实例化的bean类型                 |
|           Name           | beanName                         |
|          Scope           | Bean Scope(Singleton ,prototype) |
|  Constructor arguments   | Dependency Injection             |
|        Properties        | Dependency Injection             |
|     Autowiring mode      |                                  |
| Lazy initialization mode |                                  |
|  Initialization method   | initialization Callbacks         |
|    Destruction method    | Destruction Callbacks            |

 `ApplicationContext`实现还允许注册在容器外部（由用户）创建的现有对象。这是通过通过方法访问ApplicationContext的BeanFactory来完成的`getBeanFactory()`，该方法返回BeanFactory `DefaultListableBeanFactory`实现。`DefaultListableBeanFactory` 通过`registerSingleton(..)`和 `registerBeanDefinition(..)`方法支持此注册。但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用(因为容器之外的bean的注册时间是IOC容器初始化完成之后，手动调用registerSingleton实现，而此时属性注入已经完成，所以无法将该bean与其他的常规bean一起使用)

#### 1.3.1 Naming Beans

```xml
<alias name="fromName" alias="toName"/>
```

#### 1.3.2 实例化bean

```java
1. 指定class属性为实例化对象的类型
2. 如果bean定义为static嵌套类
eg. com.example.SomeThing$otherThing，用$分隔外部类和内部类
```

**构造函数实例化**

当通过构造函数创建一个bean时，所有普通类都可以被Spring使用并与之兼容，也就是说，正在开发的类不需要实现特定的接口或以特定的方式进行编码，只需指定bean类就足够了，但是该bean需要一个默认的构造函数。

**静态工厂方法实例化**

定义使用静态工厂方法创建的bean时，使用factory-method指定静态方法的名称，使用class指定静态工厂类的全限定名

```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

**使用实例工厂方法实例化**

使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean，要使用此机制，请将class属性保留为空，并在 `factory-bean`属性中指定当前（或父容器或祖先容器）中包含要创建对象的实例方法的Bean的名称。使用`factory-method`属性设置工厂方法本身的名称。 

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

注：在spring文档中，factory bean是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。相反，FactoryBean是指特定FactoryBean的实现类。

**确定Bean的运行时类型**

Bean元数据定义中指定的类只是初始类引用，可能与声明的工厂方法结合使用，或者是FactoryBean可能导致Bean的运行时类型不同的类，此外AOP代理可以使用基于接口的代理包装bean实例，而目标bean的实际类型(仅仅是其实现的接口)

 找出特定bean的实际运行时类型的推荐方法是`BeanFactory.getType`调用指定的bean名称。这考虑了以上所有情况，并返回了`BeanFactory.getBean`要针对相同bean名称返回的对象的类型。 

### 1.4 依赖关系

#### 1.4.1 依赖注入

**基于构造函数的依赖注入**

 基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项。调用`static`带有特定参数的工厂方法来构造Bean几乎是等效的，并且本次讨论将构造函数和`static`工厂方法的参数视为类似。以下示例显示了只能通过构造函数注入进行依赖项注入的类： 

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;
    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    // business logic that actually uses the injected MovieFinder is omitted...
}
```

- **构造函数参数解析**

构造函数参数解析匹配是通过使用参数的类型进行，如果bean定义的构造函数参数不存在任何歧义，则在实例化bean时，在bean定义中定义构造函数参数的顺序就是默认注入的顺序

```java
package x.y;

public class ThingOne {
    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>
    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当使用简单类型时，Spring无法确定值的类型，因此在没有帮助的情况下无法按照类型进行匹配

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>

or

<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>

or
//目前下面的方法可以成功，之前由于在运行时参数名称通常是不可用的，所以无法区分getX和getY如何与name中的名称对应，
因此需要使用@ConstructorProperties注解与get方法进行对应
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

```java
@ToString
public class ExampleBean {

	private String years;

	private String ultimateAnswer;

    //使用该注解与get方法相对应，从而获取对应的字段
	@ConstructorProperties({"years","ultimateAnswer"})  
	public ExampleBean(String years, String ultimateAnswer) {
		this.years = years;
		this.ultimateAnswer = ultimateAnswer;
	}
}
```

**基于Setter的依赖注入**

```java
public class SimpleMovieLister {
    
    private MovieFinder movieFinder;
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

### 1.9 基于注释的容器配置

####  1.9.1 @Required

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
//Spring Framework5.1开始正式弃用了@Required批注，使用构造函数注入或者bean实现InitializingBean.afterPropertiesSet()
```

#### 1.9.2 @Autowired

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

注：从Spring Framework4.3开始，@Autowired如果目标bean仅定义一个开始的构造函数，则不需要添加注释，但是如果有几个构造函数，并且没有默认的构造函数，则必须至少注释一个构造函数，@Autowired以指示容器使用哪个构造函数

可以将@Autowired用于传统的setter方法，可以将注释应用于具有任意名称和多个参数的方法

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

可以将@Autowired应用于字段，甚至可以将其与构造函数混合使用

注：如果希望数组的bean按照特定顺序排序，则目标bean可以实现该Ordered接口或使用@Order或标准@Priority注释

从Spring Framework5.0开始，可以使用@Nullable注释

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```



 ### 1.12 基于Java的容器配置

#### 1.12.1 基本概念@Bean和@Configuration

@bean注释被用于指示一个方法实例，可以配置，并初始化到由SpringIOC容器进行管理的新对象

@Configuration表明其主要目的是作为Bean定义的来源，@Configuration类允许通过调用@Bean同一类中的其他方法来定义Bean之间的依赖关系。

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

//等效于xml中以下配置
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>

```

#### 1.12.2 使用实例化Spring容器AnnotationConfigApplicationContext

当实例化 ClassPathXmlApplicationContext 时使用Spring XML 文件，当实例化AnnotationConfigApplicationContext时可以用@Configuration作为输入

```java
public static void main(String[] args) {
    //AppConfig.class中可以注册多个Bean
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

**通过使用编程方式构建容器**

可以通过AnnotationConfigApplicationContextde无参构造函数实例化，然后使用register方法进行配置

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

**使用组建扫描**

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}

<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
        
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}   

```

**支持web应用程序**AnnotationConfigWebApplicationContext

```xml
web.xml
<web-app>
    <!--将WebApplicationContext作为参数传入ContextLoaderListener中 -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- 配置contextConfigLocation,WebApplicationContext将去指定的全限定类中注册bean -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--配置DispatcherServlet中使用AnnotationConfigWebApplicationContext作为Web程序上下文 -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 1.12.3 使用@Bean注释

@Bean是方法级别的注释，是XML<bean/>元素的直接模拟，注释支持所提供的一些属性：init-method、destroy-method

可以使用@Bean注释在@Configuration或@Component类中

**声明一个Bean**

采用@Bean注解时，Bean定义的类型为方法返回值的类型，bean名称与方法名称相同，可以通过声明的服务接口引用类型

```java
@Configuration
class AppConfig {

    @Bean
    fun transferService(): TransferService {
        return TransferServiceImpl()
    }
}
```

**Bean依赖**

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

**接收生命周期回调**

@Bean注释的类支持生命周期回调，可以使用JSR-250中的@PostConstruct和@PreDestroy

 还完全支持常规的Spring [生命周期](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-nature)回调。如果bean实现`InitializingBean`，`DisposableBean`或`Lifecycle`，则容器将调用它们各自的方法

 也完全支持标准`*Aware`接口集（例如[BeanFactoryAware](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-beanfactory)， [BeanNameAware](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware)， [MessageSourceAware](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#context-functionality-messagesource)， [ApplicationContextAware](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware)等)

 该`@Bean`注释支持指定任意初始化和销毁回调方法，就像Spring XML中的`init-method`和`destroy-method`属性的`bean`元素，如下面的示例所示： 

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

**指定Bean的范围**

Spring包含@Scope注释，以便您可以指定bean的范围

**使用@Scope注释**

默认范围是singleton，但是可以使用@Scope覆盖它

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

**自定义Bean的名称**

```java
@Configuration
public class AppConfig {

    @Bean(name = "myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

**Bean别名**

 有时希望为单个Bean提供多个名称，否则称为Bean别名。 为此`name`，`@Bean`注释的属性接受String数组。以下示例显示了如何为bean设置多个别名： 

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

**Bean Description**

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

