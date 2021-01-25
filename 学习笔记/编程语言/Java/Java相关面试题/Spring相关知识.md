# 一、SpringBoot

## 1. 介绍一下SpringBoot的好处？

- SpringBoot可以用来独立的开发Spring应用
- SpringBoot内嵌了Tomcat、Jetty等web容器
- SpringBoot可以省略XML文件的配置，使用starter依赖项实现一键自动配置(零配置)

## 2. SpringMVC中常用的配置文件

- web.xml
	- ContextLoaderListener：负责初始化Spring上下文(Context-param给其提供参数)
	- Servlet：给Tomcat、Jetty注册一个Servlet，拦截所有请求(Tomcat和Jetty是Servlet容器，相当于代码中的ServletContext)
	- 该xml文件在tomcat启动的时候加载
- applicationContext.xml
	- 配置扫描业务类DAO层
- springmvc.xml
	- 配置扫描Controller
	- 配置Controller层返回的视图解析语言(不必要)

## 3. 注册Servlet的方法

- Servlet标签在xml文件中
- @WebServlet(在Serlvet-api3.0以上支持)

## 4. SpringBoot如何注册DispatcherServlet

- 通过实现WebApplicationInitializer接口，并重写onStartup方法，在该方法中初始化DispatcherServlet

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletCxt) {
        // Load Spring web application configuration
        //第二行这行代码是在非web项目中初始化spring上下文，第一行是在web项目中初始化spring上下文
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        //AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();
        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

## 5. SpringBoot怎样把SpringMVC原始的三个配置文件替代的？

- web.xml：使用上述的代码完成初始化spring和注册DispatcherServlet
- applicationContext.xml：通过使用注解@ComponentScan
- springmvc.xml：将Controller放在注解@ComponentScan能扫描到的地方，则该注解可以一并扫描

## 6. Tomcat和Jetty为什么可以充当Web容器，Nginx不行

- 因为Tomcat和Jetty的实现符合Servlet规范
- Tomcat7之前基本遵循的是serlvet3.0之前的规范
- Tomcat8往后遵循Servlet3.0以后的规范
- Servlet3.0规范中定义了如果META-INF\services\javax.servlet.ServletContainerInitializer这几个层级存在，在javax.servlet.ServletContainerInitializer文件中有ServletContainerInitializer接口具体的实现类，那么容器在启动时必须调用该实现类的onStartup方法(SPI规范)
- 所以Tomcat要想遵循servlet3.0，就必须做到上述的规范

## 7. 第四题中的WebApplicationInitializer的onStartUp方法是如何在Tomcat启动时被加载的？

- Spring根据Servlet3.0中的规范，知道第六题中的那个功能，所以通过实现ServletContainerInitializer并重写onStartup方法，该类有一个HandlesTypes注解，在注解中将WebApplicationInitializer.class加载进去，那么所有的WebApplicationInitializer的实现类将作为参数传给ServletContainerInitializer的onStartup方法，在该方法中对WebApplicationInitializer的所有实现类调用onStartup方法，这样就成功实现了DispatcherServlet的注册和Spring的加载
- HandlesTypes注解也是Servlet3.0的规范，意思是说使用该注解，在onStartup中会传入注解中接口的所有实现类，以Set集合的形式
- 必须是web项目才会按上述步骤加载

## 8. Spring的IOC和AOP体现在哪个项目？

- Spring FrameWork

## 9. Spring FrameWork的核心技术？

- IOC和AOP SpEL等

## 10. 什么是Spring的上下文

- 就是在初始化AnnotationConfigApplicationContext时，各种组件组合在一起可以维护Spring的正常工作。
- 组件有：RootBeanDefinition(用来描述SpringBean的)、BeanDefinitionMap(用来存放RootBeanDefinition的)

## 11. Spring执行流程

- 实例化AnnotationConfigApplicationContext 
- refresh 
- invokeBeanFactoryPostProcessors (将扫描到的类封装为BeanDefinition放入map(Spring单例池)中，之后看开发人员有没有实现BeanFactoryPostProcessor接口，如果实现了则执行其中的方法)
- 调用很多方法
- 调用finishBeanFactoryInitialization方法，该方法中会执行很多后置处理器

## 12. Bean和对象的区别

- bean是Spring创建的，包含了一套完整的生命周期
- 对象只是Spring在创建Bean的过程中的第一步

## 13. Spring如何解决循环依赖的？

- Spring支持循环依赖，但是必须是必须是单例情况下，原型情况下不支持循环依赖(因为原型情况下实在getBean时才实例化对象)

- Spring在创建Bean的过程中有一个Set集合singletonsCurrentlyInCreation，该集合中存放所有正在创建的beanName
- 第一次实例化A，调用getSingleton时结果为空(该函数中判断SingletonObjects是否存在A并且A是否正在创建过程中)，之后会调用getSingleton的重载方式，如果SingletonObjects依然没有A，则判断A是否 在创建过程中，如果不在则将A设置为正在创建过程中，并将A实例化，在之后发现A依赖于B，与A相同的方法创建B实例，之后发现B依赖A，此时继续创建A，当执行getSingleton时，此时发现SingletonObjects中没有A实例并且显示A正在创建过程中，则判断此时出现循环依赖，并生成一个临时的A对象给B。B完成创建，A完成创建。

## 14. SingletonBeanRegistry中的getSingleton()方法

- 该方法可以用来解决Bean的循环依赖问题，解决方法如问题13所述。

## 15. BeanPostPrecessor可以用来干什么？

- 它的实现类可以用来干预Bean生命周期的初始化过程

## 16. AOP的使用场景？

事务、日志记录，异常处理，权限验证，性能检查

## 17. 什么是AOP？

- 在传统的面向过程的开发过程中，我们的业务逻辑是自上而下的，但是在这个自上而下的过程当中，会产生一些横切性问题，比如日志记录，权限验证，事务处理，这些横切性问题跟我们的主业务逻辑没有直接关系，AOP就是把这些横切性问题模块成为一个切面，开发人员去关注这些切面执行的时机和顺序。
- 传统的OOP开发中的代码逻辑是自上而下的，而这些过程中会产生一些横切性问题，这些横切性问题和我们的业务逻辑关系不大，这些横切性问题不会影响到主要逻辑实现，但是会散落到代码的各个部分，难以维护，AOP就是处理这些横切性问题的，它的思想就是把这些问题和主业务逻辑分开，达到与主业务逻辑解耦的目的，使代码的重用性和开发效率更高。

## 18. AOP专业术语

- aspect：就是join point、advice、point cut所在的类，描述增强的业务逻辑、增强的时机，什么时候增强的类(InnovacationHandler)
- join point：程序执行的一个点，在Sping AOP中代表一个方法
- advice：代码增加的逻辑以及位置，在原本功能之前还是之后，还是
- Point Cut：连接点的集合
- Target project：被代理的目标对象
- AOP proxy：代理对象
- weaving：把一个功能进行增强，就叫对这个功能进行了织入

## 19. Spring AOP使用

```java
@Component
@Aspect
public class TestAspect {

    //切点的方法只是一个载体，让注解@PointCut有地方写
    @Pointcut("execution(* com.wl.Service..*.*(..))")
    public void pointCut() {
    }

    @Before("pointCut()")
    public void adviceService() {
        System.out.println("query before DB--------------------------");
    }

    //Advice中写真正的代码增强逻辑
    @After("pointCut()")
    public void adviceService1() {
        System.out.println("query after DB--------------------------");
    }
}
```

## 20. Spring AOP和IOC容器的关系

- Spring AOP当中的对象必须在IOC容器中