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

## 二、应用

### 2.1 Spring注入Date类型

- Spring支持自定义属性编辑器(实现PropertyEditorSupport)，自定义属性编辑注册器(PropertyEditorRegistrar)，开发人员实现的上述类均放入CustomEditorConfigurer(该类实现了BeanFactoryPostProcessor)中，在spring的特定过程中通过执行BeanFactoryPostProcessor的postProcessorBeanFactory方法，将CustomEditorConfigurer中存储的上述两种自定义属性编辑器和注册器存储至BeanFactory中，当Spring在注入bean遇到类型不一致时，如果预期的类型和自定义属性编辑器中value的类型一直，便调用该编辑器完成属性格式的转换，并注入之前的bean中。

- 实例工厂

  ```java
  //除了构造函数注入，也可以使用setter注入
  public class DateBean {
  	Date brithday;
  
  	public DateBean(Date brithday) {
  		this.brithday = brithday;
  	}
  
  	public Date getBrithday() {
  		return brithday;
  	}
  }
  ```

  ```xml
  //实例化一个SimpleDateFormat对象，传入构造参数的值
  <bean id="simpleDateFormat" class="java.text.SimpleDateFormat">
  		<constructor-arg value="yyyy-MM-dd"/>
  </bean>
  
  //使用实例工厂的方法产生Date类型的bean并通过构造函数注入DateBean中
  <bean id="dateBean" class="org.springframework.beans.DateBean">
      <constructor-arg>
          <bean factory-method="parse" factory-bean="simpleDateFormat">
              <constructor-arg value="2015-11-12"/>
          </bean>
      </constructor-arg>
  </bean>
  ```

  - 自定义Stirng到Date转换器

    ```java
    public class DatePropertyEditor extends PropertyEditorSupport {
    
    	private String format = "yyyy-MM-dd";
        
    	@Override
    	public void setAsText(String text) throws IllegalArgumentException {
    		try {
    			this.setValue(new SimpleDateFormat(format).parse(text));
    		} catch (ParseException e) {
    			e.printStackTrace();
    		}
    	}
    }
    ```

    ```xml
    //将自定义实现的PropertyEditorSupport类注入CustomEditorConfigurer类的customEditors集合中
    //注入map结合中的entry的value值是calss对象，无法使用bean注入，所以必须拥有默认构造函数
    <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
        <property name="customEditors">
            <map>
                <entry key="java.util.Date" value="org.springframework.beans.DatePropertyEditor">
                </entry>
            </map>
        </property>
    </bean>
    
    <bean id="dateBean" class="org.springframework.beans.DateBean">
        <constructor-arg value="2015-12-12"/>
    </bean>
    ```

    - 实现PropertyEditorRegistrar接口，并重写registerCustomEditors方法，并将Spring自带的CustomDateEditor注入propertyEditorRegistrars

      ```java
      public class DatePropertyEditorRegistrar implements PropertyEditorRegistrar {
      
      	@Override
      	public void registerCustomEditors(PropertyEditorRegistry registry) {
      		registry.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
      	}
      }
      ```

      ```xml
      <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
          <property name="propertyEditorRegistrars">
              <list>
                  <bean class="org.springframework.beans.DatePropertyEditorRegistrar"/>
              </list>
          </property>
      </bean>
      
      <bean id="dateBean" class="org.springframework.beans.DateBean">
          <constructor-arg value="2015-12-12"/>
      </bean>
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

### 2.4  Spring BeanPostProcessor

- Spring在Bean的整个生命周期中在很多地方调用了后置处理器，其中包括Spring自己实现的后置处理器，以及开发人员实现的后置处理器

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

### 3.3 Spring自定义属性编辑器，原理解析

- DateBean类需要注入一个Date格式的属性

```java
//DateBean中使用setter方法注入，所以string转换为Date的时间发生在属性注入
//当使用构造函数注入时，String转换为Date的时间发生在反射创建bean之后，instantiateBean()函数调用之后
public class DateBean {
	Date brithday;

	public void setBrithday(Date brithday) {
		this.brithday = brithday;
	}

	public Date getBrithday() {
		return brithday;
	}
}
```

- 两种方式实现

```java
// (1) 自定义属性编辑器
public class DatePropertyEditor extends PropertyEditorSupport {

	private String format = "yyyy-MM-dd";

	public void setFormat(String format) {
		this.format = format;
	}

	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		try {
			this.setValue(new SimpleDateFormat(format).parse(text));
		} catch (ParseException e) {
			e.printStackTrace();
		}
	}
}

// (2) 自定义属性编辑注册器
public class DatePropertyEditorRegistrar implements PropertyEditorRegistrar {

	@Override
	public void registerCustomEditors(PropertyEditorRegistry registry) {
		registry.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
	}
}

```

- 上面两种方式的bean会放入CustomEditorConfigurer的集合中，通过xml配置文件

  ```xml
  <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
      <property name="customEditors">
          <map>
              <entry key="java.util.Date" value="org.springframework.beans.DatePropertyEditor">
              </entry>
          </map>
      </property>
  </bean>
  
  <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
      <property name="propertyEditorRegistrars">
          <list>
              <bean class="org.springframework.beans.DatePropertyEditorRegistrar"/>
          </list>
      </property>
  </bean>
  ```

- CustomEditorConfigurer实现了BeanFactoryPostProcessor接口

  ```java
  invokeBeanFactoryPostProcessors()    //在该接口中，会激活所有BeanFactory的后置处理器，其中CustomEditorConfigurer会将属性编辑注册器和属性编辑器这两个集合放入BeanFactory的集合中
      
  //CustomEditorConfigurer.java
  @Override
      public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
      if (this.propertyEditorRegistrars != null) {
          for (PropertyEditorRegistrar propertyEditorRegistrar : this.propertyEditorRegistrars) {
              beanFactory.addPropertyEditorRegistrar(propertyEditorRegistrar);
          }
      }
      if (this.customEditors != null) {
          this.customEditors.forEach(beanFactory::registerCustomEditor);
      }
  }
  ```

- 在通过反射创建bean的过程中，会遍历BeanFactory中的两个集合， 并将类型转换器注入每个bean中,所有的类型转换器保存在PropertyEditorRegistrySupport中

  ```java
  //AbstractBeanFactroy.java
  protected void registerCustomEditors(PropertyEditorRegistry registry) {
      PropertyEditorRegistrySupport registrySupport =
          (registry instanceof PropertyEditorRegistrySupport ? (PropertyEditorRegistrySupport) registry : null);
      if (registrySupport != null) {
          registrySupport.useConfigValueEditors();
      }
      if (!this.propertyEditorRegistrars.isEmpty()) {
          //遍历所有属性编辑注册器，调用接口方法，该方法就是将指定的类型转换器注入bean对应的map中，bean在这里已经用BeanWrapperImpl包装过，而BeanWrapperImpl继承自PropertyEditorRegistrySupport，里面有个map用来存储类型转换器
          for (PropertyEditorRegistrar registrar : this.propertyEditorRegistrars) {
              try {
                  registrar.registerCustomEditors(registry);
              }
              catch (BeanCreationException ex) {
                  Throwable rootCause = ex.getMostSpecificCause();
                  if (rootCause instanceof BeanCurrentlyInCreationException) {
                      BeanCreationException bce = (BeanCreationException) rootCause;
                      String bceBeanName = bce.getBeanName();
                      if (bceBeanName != null && isCurrentlyInCreation(bceBeanName)) {
                          if (logger.isDebugEnabled()) {
                              logger.debug("PropertyEditorRegistrar [" + registrar.getClass().getName() +
                                           "] failed because it tried to obtain currently created bean '" +
                                           ex.getBeanName() + "': " + ex.getMessage());
                          }
                          onSuppressedException(ex);
                          continue;
                      }
                  }
                  throw ex;
              }
          }
      }
      //将集合中的类型转换器加入每个bean对应的map集合中
      if (!this.customEditors.isEmpty()) {
          this.customEditors.forEach((requiredType, editorClass) ->
                                     registry.registerCustomEditor(requiredType, BeanUtils.instantiateClass(editorClass)));
      }
  }
  ```

  - 开始类型转换，本次实例代码采用setter方式注入，所以类型转换发生在属性注入期间

    ```mermaid
    graph TD
    A[populateBean] --> B[applyPropertyValues]
    	B[applyPropertyValues]  --> C[convertForProperty]
    	C[convertForProperty]  --> D[...]
    	D[...]  --> E[doConvertValue]
    	E[doConvertValue] --> F[doConvertTextValue]
    	F[doConvertTextValue] --> G[setAsText]
    ```

    

    