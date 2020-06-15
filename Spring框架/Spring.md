# Spring应用

## 一、spring配置

### 1.1 基于XML的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="myTestBean" class="org.springframework.beans.MyTestBean"/>
</beans>
```

#### 1.1.1 XML配置中标签的用法及说明

- lookup-method标签

  ```xml
  <bean id="myTestBean" class="org.springframework.beans.MyTestBean">
  		<!--该标签可以更改bean中一个方法返回的bean类型-->
  		<lookup-method name="getTestStr" bean="user"/>
  </bean>
  <bean id="teacher" class="org.springframework.beans.Teacher"/>
  <bean id="user" class="org.springframework.beans.User"/>
  
  <!--其中myTestBean中的getTestStr方法可以为抽象方法，也可以为正常的方法-->
  ```

- replaced-method标签

  ```xml
  <bean id="myTestBean" class="org.springframework.beans.MyTestBean">
  	<!--该标签可以将myTestBean中的某个方法替换为teacher,teacher必须实现MethodReplacer接口-->
  	<replaced-method name="getTestStr" replacer="teacher"/>
  </bean>
  <bean id="teacher" class="org.springframework.beans.Teacher"/>
  ```

  ```java
  public class Teacher extends User implements MethodReplacer {
  	@Override
  	public void showMe() {
  		System.out.println("i am teacher");
  	}
  	//无法重复调用method方法，会出现循环调用
  	@Override
  	public Object reimplement(Object obj, Method method, Object[] args) throws Throwable {
  		return new Teacher();
  	}
  }
  ```

- constructor-arg标签

- Qualifier标签

  ```xml
  <context:annotation-config/>
  <bean id="myTestBean" class="org.springframework.beans.MyTestBean">
  </bean>
  <bean class="org.springframework.beans.Teacher">
  	<qualifier value="teacher1"/>
  </bean>
  <bean class="org.springframework.beans.Teacher">
  	<qualifier value="teacher2"/>
  </bean>
  ```

  ```java
  //使用@Autowired 和 @Qualifier实现多个相同bean的注入
  public class MyTestBean {
  	
  	@Autowired
  	@Qualifier("teacher1")
  	private Teacher teacher;
  
  	public Teacher getTeacher() {
  		return teacher;
  	}
  }
  ```

# Spring源码分析

## 一、字段说明

```java
-- spring中的属性字段说明
-- DefaultSingletonBeanRegistry.java
//用于存储在Spring内部所使用的beanName -> 对象工厂的引用，当对象通过(ObjectFactory.getObject)被创建，此引用信息将被删除
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

//用于存储在创建bean早期对创建的原始bean的一个引用，即使用工厂方法或者构造方法创建出来的对象，一旦对象最终创建好，此引用信息将删除
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

//用于存储bean的实例化对象，也是常说的ioc容器
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

//当创建一个bean的时候，会将beanName加入set集合中，在循环依赖的时候会用到
private final Set<String> singletonsCurrentlyInCreation =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));

-- DefaultListableBeanFactory.java
//用于存储bean解析后的BeanDefinition
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```

## 二、Spring中的相关类说明

### 2.1 FactoryBean

```java
@Setter
@Getter
@ToString
public class Car {
	private int maxSpeed;
	private String brand;
	private double price;
}

//bean的实现类为实现了FactoryBean接口
public class CarFactoryBean implements FactoryBean<Car> {

	@Getter @Setter
	private String carInfo;
	@Override
	public Car getObject() throws Exception {
		Car car = new Car();
		String[] infos = carInfo.split(",");
		car.setBrand(infos[0]);
		car.setPrice(Integer.valueOf(infos[1]));
		car.setMaxSpeed(Integer.valueOf(infos[2]));
		return car;
	}

	@Override
	public Class<?> getObjectType() {
		return Car.class;
	}

	@Override
	public boolean isSingleton() {
		return false;
	}
}

public class MainApplication {
	public static void main(String[] args) {
		ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("beans.xml");
        //当对应id的bean的实现类实现了FactoryBean接口时，返回的getbean为该接口中getObject的bean对象
		Car car = (Car) ioc.getBean("carFactoryBean");
        //当id前加&时，返回该bean的实例化对象
        CarFactoryBean carFactoryBean = (CarFactoryBean) ioc.getBean("@carFactoryBean");
		System.out.println(car.toString());
	}
}
```

```xml
<bean id="carFactoryBean" class="org.springframework.beans.CarFactoryBean">
	<property name="carInfo" value="超级跑车,400,2000000"/>
</bean>
```

### 2.2 BeanFactory

### 2.3 AbstractApplicationContext

## 三、源码分析

### 3.1 Spring的循环依赖

```java
//DefaultSingletonBeanRegistry.java
/*
* 该函数中的代码是解决循环依赖的核心
* 1. 当出现循环依赖的时候，BeanA的创建会再次回到该函数，并且此时isSingletonCurrentlyInCreation(beanName)返回
* 为true，此时会从singletonFactories中获取ObjectFactory并创建一个beanA(该beanA其实还未完全走完生命周期的整个* 过程，但是因为引用是相同的，所以最终的beanA是同一个)
* 2. 其中ObjectFactory是在最初创建beanA的时候在刚刚实例化出来的beanA对象时，将其加入singletonFactories中
*/
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

### 3.2 Spring中bean的生命周期

-  扫描所有配置的bean，将其解析为BeanDefinition，并存储在BeanDefinitionMap中

  ```java
  //该函数中已经将之前BeanFactory的内容全部功能
  ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
  ```

- 准备BeanFactory

  ```java
  //增加SPEL语言支持，注册属性编辑器，忽略一些依赖接口，注册一些依赖
  prepareBeanFactory(beanFactory);
  ```

- BeanFactoryPostProcessor的注册和激活

  ```java
  //在这一步注册并激活所有BeanFactoryPostProcessor(包括硬编码 + 配置文件注册的)，激活配置文件的BeanFactory后置处理器时还可以排序
  invokeBeanFactoryPostProcessors(beanFactory);
  ```

- 注册一些BeanPostProcessor到BeanFactory中

  ```java
  registerBeanPostProcessors(beanFactory);
  ```

- 初始化消息资源(当需要开发一个支持多国语言的web应用程序，要求可以根据客户端系统的语言类型返回对应的界面)

  ```java
  initMessageSource();
  ```

- 实例化非延迟加载的单例bean

  ```java
  /*
  * bean的实例化和初始化过程，是整个bean加载的核心，其中之前注册的BeanPostProcessor就是在这里面调用的
  * 1. 从singletonObjects中第一次尝试获取，如果存在，直接返回
  * 2. 如果不存在，则开始创建，首先将beanName加入正在创建的列表
  * 3. 通过反射实例化beanName对象
  * 4. 将bean对象的ObjectFactory加入singletonFactories中
  * 5. 属性注入(这里其实就是获取所有的BeanPostProcessor,并将调用特定实例的某个方法)
  *  	- @Autowired注入的属性就会被AutowiredAnnotationBeanPostProcessor.postProcessProperties()成* * 功注入
  * 6. bean的生命周期回调
  * 	- invokeAwareMethods调用，其中会调用BeanNameAware接口方法，BeanFactoryAware接口方法
  * 	- applyBeanPostProcessorsBeforeInitialization调用，其中就是获取所有的BeanPostProcessor，并调用before方法
  * 	- invokeInitMethods调用，包括实现了InitializingBean接口的afterPropertiesSet方法
  *   - 使用init-method自定义的初始化方法
  *   - applyBeanPostProcessorsAfterInitialization调用，其中本质就是获取所有的BeanPostProcessor，并调用after方法，其中实现AOP就是通过AnnotationAwareAspectJAutoProxyCreator类实现的
  * 7. 将beanName对应的bean从正在创建的列表中删除
  * 8. 将bean加入singletonObjects集合中
  */
  finishBeanFactoryInitialization(beanFactory);
  ```

  ### 3.3 getObjectForBeanInstance解析

  ```java
  /*
  * 该函数的作用是产生最终的bean
  * 从singletonObjects中获取的对象只是原始对象，例如一个bean如果继承了FactoryBean，那么singletonObjects中* * 存储的是FactoryBean的实例化对象，但是最终返回的bean应该是getObject对应的bean。
  * 创建出来bean之后，并对bean进行后置处理
  */
  protected Object getObjectForBeanInstance(
  			Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {
  		//isFactoryDereference函数用来判断name是否以&开头
  		if (BeanFactoryUtils.isFactoryDereference(name)) {
  			if (beanInstance instanceof NullBean) {
  				return beanInstance;
  			}
  			//如果name以&开头，但是不是FactoryBean的实例，则抛出异常
  			if (!(beanInstance instanceof FactoryBean)) {
  				throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
  			}
  		}
  
  		//如果不是FactoryBean的实例，则直接返回
  		//如果是，则判断name是否以&开头，true则返回beanInstance，false则不返回
  		if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
  			return beanInstance;
  		}
  
  		//下面的情况为bean为FactoryBean的实例，但是name不以&开头
  		Object object = null;
  		if (mbd == null) {
  			object = getCachedObjectForFactoryBean(beanName);
  		}
  		if (object == null) {
  			// Return bean instance from factory.
  			FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
  			// Caches object obtained from FactoryBean if it is a singleton.
  			if (mbd == null && containsBeanDefinition(beanName)) {
  				mbd = getMergedLocalBeanDefinition(beanName);
  			}
  			boolean synthetic = (mbd != null && mbd.isSynthetic());
  			// 真正调用FactoryBean.getObject()的方法返回bean
  			// 判断beanName是否正在创建，如果正在创建。说明是循环依赖则直接返回
  			// 否则将beanName加入正在创建bean的列表中，并调用bean的后置处理，最后从正在创建列表中移除bean
  			object = getObjectFromFactoryBean(factory, beanName, !synthetic);
  		}
  		return object;
  	}
  ```

  

