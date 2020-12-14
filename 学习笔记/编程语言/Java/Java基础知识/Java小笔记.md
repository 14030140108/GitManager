# Java笔记

### 一、Static变量带来的线程不安全

### 二、日志框架

```java
/*
 * Commons Logging 和 Log4j 其中Commons Logging是日志的接口，Log4j是日志的实现，使用log4j.properties配置日志参数
 * Slf4j 和 Logback 其中Slf4j是日志的接口，Logback是日志的实现，使用logback.xml来配置日志参数
 */
```

#### 2.1 Commons Logging

```java
// 使用Commons Logging(自动检测是否有log4j的实现)
// 在静态方法中引用Log:
public class Main {
    static final Log log = LogFactory.getLog(Main.class);
    static void foo() {
        log.info("foo");
    }
}

// 在实例方法中引用Log:
public class Person {
    //使用getClass()的方式来获取类的好处时，子类也可以直接使用log变量来记录日志
    protected final Log log = LogFactory.getLog(getClass());
    void foo() {
        log.info("foo");
    }
}

```

#### 2.2 slf4j

```java
//使用slf4j(自动检测是否有logback的实现)
public class Main {
    static final Logger log = LoggerFactory.getLogger(Main.class);
    static void foo() {
        log.info("foo");
    }
}
```

##### 2.2.1 Logback.xml

```xml
// https://www.cnblogs.com/gavincoder/p/10091757.html  logback.xml详解

<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
    <!--滚动记录文件，先将日志记录到文件中，当满足某个条件时，将日志记录到其他文件中-->
	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<encoder>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
			<charset>utf-8</charset>
		</encoder>
		<file>log/output.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
			<fileNamePattern>log/output.log.%i</fileNamePattern>
		</rollingPolicy>
		<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
			<MaxFileSize>1MB</MaxFileSize>
		</triggeringPolicy>
    </appender>
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</configuration>
```

### 三、Java中接口新特性

```java
// https://blog.csdn.net/yubin1285570923/article/details/108462920 
/*
 * java8新特性中接口除了常量和抽象方法外新增了(静态方法和默认方法)
 * -- 当实现类中出现同名同参数的方法调用时，应该遵循以下原则：
 *     1. 类优先原则(只针对方法，属性不起作用)
 *     2. 子接口优先
 *     3. 具体指定某个接口中的方法
 */
```

