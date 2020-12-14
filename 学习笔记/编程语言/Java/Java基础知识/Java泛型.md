# 泛型

- Java泛型是一种语法糖，是为了在编译时尽可能发现出错的地方并减少程序员需要做的强制类型转换

## 一、泛型的种类

- 修饰类

```java
public class GenericDemo<T>{
    ...
}
```

- 修饰接口

```java
public interface GenericType<T> {
   ....
}
```

- 修饰方法

```java
public class GenericDemo<T>{
    
    //在一个类中静态方法不能引用泛型T，必须定义一个新的泛型，如下所示
    public static <T> void staticMethod(T t){
        //该T和类的T不是一个同一个泛型
        ...
    }

    //正确，形参中的T为类的泛型T
   	public void method nonStaticMethod(T t){
        ...
    }
	
}
```

## 二、类型擦除

- 编译器会将所有的泛型擦除为Object

- 类型擦除的局限性

	- 无法使用基本类型
	- 无法获取带泛型的Class,也不能判断带泛型的Class

	```java
	GenericDemo<String> s = ...;
	GenericDemo<Integer> i = ...;
	Class c1 = s.getClass();
	Class c2 = i.getClass();
	//c1和c2是相等的，都是Generic.class
	
	//Compile error
	if(c1 instanceof GenericDemo<String>.class)
	```

	- 不能实例化T类型，必须借助Class<T>

	```java
	clazz.newInstance();
	```

## 三、extends和super通配符

### 3.1  extends通配符

- 上界不存
- extends通配符一般用在泛型方法上

```java
public class GenericDemo<T>{
}

public class GenericHelper{
    public static void main(String[] args){
        ...
    }
    public void method(GenericDemo<? extends Number> g){
        //此处不能调用set，因为传入的g为Number的子类，不能把Number增加到Number的子类中
        p.setFirst();  //Compile error
        p.setFirst(null) //允许
    }
}
```



### 3.2 super通配符

- 下界不取
- super通配符一般用在泛型方法上

```java
public class GenericDemo<T>{
}

public class GenericHelper{
    public static void main(String[] args){
        ...
    }
    public void method(GenericDemo<? super Number> g){
        //此处不能调用get，因为g为Number的父类，无法将Number的父类强制转换为Number
        g.getFirst();   //错误
        Object o = g.getFirst()  //允许
    }
}
```

## 四、桥方法

- 桥方法的存在是为了解决泛型类型擦除与多态之间的冲突

```java
// 1. GenericDemoDay2经过类型擦除后setFirst方法的形参变为Object，但是IntegerGeneric的setFirst方法的形参是Integer，并没有起到覆盖重写的作用，在把Integer使用多态转换为父类GenericDemoDay2的时候依然调用的是父类的setFirst方法
   2. 编译器会在IntegerGeneric中增加一个桥方法，如下所示:
public void setFirst(Object o){
    setFirst((Integer) o);
}

public class GenericDemoDay2<T> {
    private T first;
    public T getFirst() {
        return first;
    }
    public void setFirst(T first) {
        this.first = first;
    }
}

public class IntegerGeneric extends GenericDemoDay2<Integer> {
    @Override
    public Integer getFirst() {
        return super.getFirst();
    }
    @Override
    public void setFirst(Integer first) {
        super.setFirst(first);
    }
    public static void main(String[] args) {
        GenericDemoDay2 genericDemoDay2 = new IntegerGeneric();
        genericDemoDay2.setFirst(20);
        System.out.println(((IntegerGeneric) genericDemoDay2).getFirst());
    }
}
```

## 五、协变

- Orange是Fruit的子类，但是List<Orange>不是List<Fruit>的子类，泛型中使用通配符来解决泛型无法协变的问题
- 数组支持协变