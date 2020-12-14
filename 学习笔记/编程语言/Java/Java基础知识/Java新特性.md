# Java新特性

## 1. Lambda表达式

1. 使用Lambda表达式需要"函数式接口"的支持
    - 函数式接口：接口中只有一个抽象方法的接口
        - 计算一个接口中的抽象方法时的三点原则
            1. 函数式接口只有一个抽象方法
            2. default方法为默认实现，不属于抽象方法
            3. 接口重写了Object的公共方法也不算在内
    - 可以使用Lambda表达式创建该接口的对象

2. 使用lambda表达式对List进行排序

```java
public class Java8Tester {
    public static void main(String[] args) {
        List<String> names1 = new ArrayList<String>();
        names1.add("Google ");
        names1.add("Runoob ");
        names1.add("Taobao ");
        names1.add("Baidu ");
        names1.add("Sina ");

        List<String> names2 = new ArrayList<String>();
        names2.add("Google ");
        names2.add("Runoob ");
        names2.add("Taobao ");
        names2.add("Baidu ");
        names2.add("Sina ");

        Java8Tester tester = new Java8Tester();
        
        System.out.println("使用 Java7 的语法");
        tester.sortUsingJava7(names1);
        System.out.println(names1);

        System.out.println("使用 Java8 的语法");
        tester.sortUsingJava8(names2);
        System.out.println(names2);
    }
    //使用Java7进行字符串排序
    public void sortUsingJava7(List<String> names) {
        Collections.sort(names, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.compareTo(o2);
            }
        });
    }
	//使用Java8进行字符串排序
    public void sortUsingJava8(List<String> names) {
        Collections.sort(names,(s1,s2) -> s1.compareTo(s2));
    }
}
```

- Lambda表示也可称为闭包，允许把函数作为一个方法的参数，是代码变的更加的简洁紧凑

3. Lambda表达式的新特性
    - 可选类型声明：不需要声明参数类型，编译器可以同意识别参数值
    - 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号
    - 可选的大括号：如果主体包含了一个语句，就不需要使用大括号
    - 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值

4. 变量作用域

    - lambda表达式可以引用类的实例变量 和类变量，并且可以被修改

    - lambda表达式可以访问局部变量，并且可以不用final修饰，但是不能被修改(隐性的具有final的语义)，可以用数组或者对象代替局部变量

        - 具体其实是局部变量中的基本数据类型无法修改，因为基本数据类型无法通过自身修改其值，所以在lambda表达式中如果修改了基本数据类型的值后无法传递到外层
        - lambda表达式中需要的数据是参数的拷贝，因为局部变量保存在栈中，lambda表达式调用时，变量可能已经被释放，所以必须先进行参数的拷贝

        ```java
        public class LambdaDemo {
            static String str = "Hello! ";
            public static void main(String[] args) {
                int num = 1;
                LambdaDemo tester = new LambdaDemo();
                MathOperation mathOperation = message -> System.out.println(num + message);
                mathOperation.sayMessage("Runoob");
                //num在任何地方都不能修改，具有final的语义
                num = 5
            }
        }
        interface MathOperation {
            void sayMessage(String message);
        }
        ```

    - lambda表达式不允许声明一个与局部变量同名的参数或者局部变量

5. 函数式接口

    - 消费型接口

        ```java
        //Consumer<T>
        //函数：void accept(T t)
        //需求：消费
        public class FuncInterfaceDemo {
            public static void main(String[] args) {
                eat(100D,m -> System.out.println("一共消费了" + m + "元"));
            }
            public static void eat(Double money, Consumer<Double> consumer) {
                consumer.accept(money);
            }
        }
        ```

    
    - 供给型接口
    
        ```java
        //Supplier<T>
        //函数：T get()
        //需求：产生基本数据
        public static List<Integer> getNumList(int num, Supplier<Integer> sup) {
                List<Integer> list = new ArrayList<>();
                for (int i = 0; i < num; i++) {
                    Integer n = sup.get();
                    list.add(n);
                }
                return list;
            }
            public static void main(String[] args) {
                List<Integer> numList = getNumList(10, () ->(int)(Math.random() * 100));
                //System.out.println(PPrint.pformat(numList));
                System.out.println(numList);
            }
        ```
    
    - 函数型接口
    
        ```java
        //Function<T,R>
        //函数：R apply(T t)
        //需求：字符串处理
        public static String strHandler(String str, Function<String, String> fun) {
                return fun.apply(str);
            }
            public static void main(String[] args) {
                String newStr = strHandler("\t\t\t Function   ", str -> str.trim());
                System.out.println(newStr);
                String subStr = strHandler("Function", str -> str.substring(2, 5));
                System.out.println(subStr);
            }
        ```
    
    - 断言型接口
    
        ```java
        //Predicate<T t>
        //函数：Boolean test(T t)
        //需求：筛选满足条件的字符串
        public static List<String> filterStr(List<String> list, Predicate<String> predicate) {
                List<String> strList = new ArrayList<>();
                for (String s : list) {
                    if (predicate.test(s)) {
                        strList.add(s);
                    }
                }
                return strList;
            }
            public static void main(String[] args) {
                List<String> list = Arrays.asList("Hello", "predicate", "Lambda", "www", "ok");
                List<String> strList = filterStr(list, (s) -> s.length() <= 3);
                System.out.println(strList);
            }
        ```

##  2. 方法引用

- [方法引用参考](https://www.jianshu.com/p/62465b26818f)

- 方法引用是用来直接访问类或者实例的已经存在的方法或者构造函数。方法引用提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文，计算时，方法引用会创建函数式接口的一个实例。

1. 构造器引用
   
- Class::new
  
    - 构造器引用适用于lambda表达式主体中仅仅调用了某个类的构造函数返回实例的场景
    
    - ```java
        //lambda表达式使用：
        Supplier<List<String>> supplier1 = () -> new ArrayList<>();
        //构造器引用：
        Supplier<List<String>> supplier = ArrayList::new;
        ```
    
2. 静态方法引用
   
    - Class::static_method
    
    - 静态方法引用适用于lambda表达式主体中仅仅调用了某个类的静态方法的情形
    
    - ```java
        public class MethodReference {
            public static void main(String[] args) {
                //lambda表达式使用：
                Arrays.asList(new String[]{"a", "c", "b"}).stream().forEach(s -> MethodReference.println(s));
                //静态方法引用
                Arrays.asList(new String[]{"a", "c", "b"}).stream().forEach(MethodReference::println);
            }
            public static void println(String s) {
                System.out.println(s);
            }
        }
        ```
    
3. 特定类任意对象的方法引用
   
    - Class::method
    
    - lambda表达式的第一个入参(接口中抽象方法的第一个参数)为实例方法的调用者，后面的入参与实例方法的入参一致
    
    - ```java
        public class MethodReference {
            public static void main(String[] args) {
                TestInterface testInterface = Student::setNameAndScore;
                testInterface.set(new Student(), "wl", 100);
            }
        }
        
        @FunctionalInterface
        interface TestInterface {
            void set(Student d, String name, Integer score);
        }
        
        class Student {
            private String name;
            private Integer score;
        
            public void setNameAndScore(String name, Integer score) {
                this.name = name;
                this.score = score;
                System.out.println("Student " + name + "'s score is " + score);
            }
        }
        ```
    
4. 特定对象的方法引用
   
    - instance::method
    
    - 特定对象的实例方法引用适用于lambda表达式的主体中仅仅调用了某个对象的某个实例方法的场景
    
    - ```java
        public class MethodReference {
            public static void main(String[] args) {
                MethodReference mr = new MethodReference();
                //lambda表达式使用：
                Arrays.asList(new String[]{"a","c","b"}).stream().forEach(s -> mr.println(s));
                //特定对象的实例方法使用：
                Arrays.asList(new String[]{"a","c","b"}).stream().forEach(mr::println);
            }
        
            public void println(String s) {
                System.out.println(s);
            }
        }
        ```
    
5. 通常使用lambda表达式来创建匿名方法，然而，有时候仅仅是调用一个已存在的方法，在这时候，可以使用方法引用来简化lambda表达式。所以方法引用的本质也是一个lambda表达式，

6. 方法引用中一般情况下，方法的入参和返回值与lambda表达式实现的函数式接口的入参和返回值一致，但是如果入参的对象是个类，方法的入参可以为空，因为该方法可能是通过className.functionName的形式调用的。

## 3. 默认方法

- 默认方法就是接口可以有实现方法，而且不需要实现类去实现其方法
- 接口的好处是面向抽象而不是面向具体编程，缺陷是当需要修改接口时，需要修改全部实现该接口的类

- 当继承的多个接口有相同的默认方法时：
    - 在实现类中重写默认方法
    - 使用 Class.super.methodName()语法调用指定接口的默认方法
- Java8的另一个特性是接口可以声明静态方法，并且可以实现该方法

## 4. Stream

- IntConsumer是int消费型函数式接口

    ```java
    public class IntConsumerDemo {
        public static void main(String[] args) {
            IntConsumer c1 = System.out::println;
            //andThen函数是该接口中的一个默认方法，可以生成一个序列
            IntConsumer c2 = c1.andThen(c1);
            IntConsumer c4 = c2.andThen(c2);
            c4.accept(5);
        }
    }
    //output:
    5
    5
    5
    5   
    ```

2. Stream支持的聚合操作

    - filter

        - 接收的参数是一个断言型接口，通过test函数的返回值过滤流中的元素

    - map

        - 接收的参数是一个函数型接口，映射每个元素到对应的结果

    - limit

        - 获取指定数量的流

    - sorted

        - 对流进行排序,接收的是一个比较器Comparator函数式接口

        ```java
        List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
                List<Integer> sortedNum = numbers.stream().sorted((s1, s2) -> s1 - s2).collect(Collectors.toList());
                System.out.println(sortedNum);
        ```

    - parallel

        - 并行流

    - Collectors

        - 将流转换成集合和聚合元素，可用于返回列表或字符串

    - 统计

        ```java
         public static void main(String[] args) {
                List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
                IntSummaryStatistics stats = numbers.stream().mapToInt(x -> x).summaryStatistics();
                System.out.println("列表中最大的数 : " + stats.getMax());
                System.out.println("列表中最小的数 : " + stats.getMin());
                System.out.println("所有数之和 : " + stats.getSum());
                System.out.println("平均数 : " + stats.getAverage());
         }
        ```

## 5. Optional类

 1. 一个可以用来检测空指针异常的容器

    ```java
    public class OptionalDemo {
        public static void main(String[] args) {
            Integer value1 = null;
            Integer value2 = new Integer(10);
            Optional<Integer> a = Optional.ofNullable(value1);
            Optional<Integer> b = Optional.of(value2);
            OptionalDemo optionalDemo = new OptionalDemo();
            System.out.println(optionalDemo.sum(a, b));
        }
        public Integer sum(Optional<Integer> a, Optional<Integer> b) {
            System.out.println("第一个参数值存在: " + a.isPresent());
            System.out.println("第二个参数值存在: " + b.isPresent());
            Integer value1 = a.orElse(new Integer(0));
            Integer value2 = b.get();
            return value1 + value2;
        }
    }
    //output:
    第一个参数值存在: false
    第二个参数值存在: true
    10
    ```

## 6. Nashorn

1. Hashorn是一个javascript引擎

2. JJS是一个Nashorn引擎的命令行工具

3. 在java中调用JavaScript代码

    ```java
    public class InvokeJavaScript {
        public static void main(String[] args) {
            ScriptEngineManager scriptEngineManager = new ScriptEngineManager();
            ScriptEngine nashorn = scriptEngineManager.getEngineByName("nashorn");
            String name = "Runoob";
            Integer result = null;
    
            try {
                nashorn.eval("print('" + name + "')");
                result = (Integer) nashorn.eval("10 + 2");
            } catch (ScriptException e) {
                System.out.println("执行脚本错误：" + e.getMessage());
            }
            System.out.println(result);
        }
    }
    //output:
    Runoob
    12
    ```

## 7. 日期和时间的API

- JDK中常用的时间日期类库在java.time中

1. JDK8中如果不考虑时区的情况下：

    - 时间使用LocalTime
    - 日期使用LocalDate
    - 日期和时间使用LocalDateTime

    ```java
    public class Java8DateDemo {
        public static void main(String[] args) {
            LocalDateTime currentTime = LocalDateTime.now();
            System.out.println("当前时间：" + currentTime);
    
            LocalDate date1 = currentTime.toLocalDate();
            System.out.println("date1：" + date1);
    
            System.out.println(currentTime.getDayOfMonth());
            System.out.println(currentTime.getDayOfWeek());
            System.out.println(currentTime.getDayOfYear());
            System.out.println(currentTime.getYear());
    
            LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012).withMonth(1);
            System.out.println("date2: " + date2);
    
            LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12);
            System.out.println("date3: " + date3);
    
            LocalTime date4 = LocalTime.of(16, 26, 05);
            System.out.println("date4: " + date4);
    
            LocalTime date5 = LocalTime.parse("20:15:30");
            System.out.println("date5: " + date5);
        }
    }
    //output:
    当前时间：2019-11-20T16:27:02.693
    date1：2019-11-20
    20
    WEDNESDAY
    324
    2019
    date2: 2012-01-10T16:27:02.693
    date3: 2014-12-12
    date4: 16:26:05
    date5: 20:15:30
    ```

2. 考虑时区的情况下:

    - 使用ZonedDateTime类

    ```java
    public class Java8ZoneDateDemo {
        public static void main(String[] args) {
            ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
            System.out.println("date1：" + date1);
            ZoneId id = ZoneId.of("Europe/London");
            System.out.println("ZoneId: " + id);
            //获取指定时区的时间
            LocalDateTime date2 = LocalDateTime.now(id);
            ZonedDateTime date3 = ZonedDateTime.now(id);
            System.out.println("date2: " + date2);
            System.out.println("date3: " + date3);
            ZoneId currentZone = ZoneId.systemDefault();
            System.out.println("当前时区：" + currentZone);
        }
    }
    //output:
    date1：2015-12-03T10:15:30+08:00[Asia/Shanghai]
    ZoneId: Europe/London
    date2: 2019-11-20T09:10:01.326
    date3: 2019-11-20T09:10:01.328Z[Europe/London]
    当前时区：Asia/Shanghai
    ```

## 8. Base64

1. 网络上传输的字符并不全是可打印的字符，比如二进制文件、图片等，base64的出现就是为了解决此问题，它是基于64个可打印的字符来表示的二进制数据的一种方法
2. 电子邮件刚问世的时候，只能传输英文，但后来随着用户的增加，中文、日文等文字的用户也有需求，但这些字符并不能被服务器或网关有效处理，因此Base64在URL、Cookies、网页传输少量二进制文件中也有相应的使用
3. 64个字符分别是：A-Za-Z0-9+/，=为填充字符
4. 将字符串每3个字节看成一组，一共24位，之后每6位一组，分为4组，每组6bit，之后每组的最高位填两个0，变成32位，最后按照base64表编码

```java
public class Base64Demo {
    public static void main(String[] args) {
        String man = "Man";
        //使用Base64编码
        byte[] result = Base64.getEncoder().encode(man.getBytes());
        System.out.println(new String(result));
        //使用BASE64Encoder编码
        BASE64Encoder base64Encoder = new BASE64Encoder();
        String dst = base64Encoder.encode(man.getBytes());
        System.out.println(dst);
        
        String str = "TWFu";
        //使用Base64解码
        byte[] decode = Base64.getDecoder().decode(str);
        //使用BASE64Encoder解码
        BASE64Decoder base64Decoder = new BASE64Decoder();
        byte[] decode = base64Decoder.decodeBuffer(str);
        System.out.println(new String(decode));
    }
}
//output：
TWFu
TWFu
Man
Man
```

