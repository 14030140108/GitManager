# java基础

- Java中的无符号整数

	- Integer.toUnsignedString(int / long)

	- Integer.parseUnsignedInt(String)

- Java中包装类和基本数据类型的自动装箱和拆箱

	- 装箱  Integer.valueof
	- 拆箱  intvalue()

- 缓存池

	- new Integer()和valueof的区别，前者每次都新建一个对象，后者会使用缓存池中的对象，多次调用会取得同一个对象的引用
	- JDK1.8中缓存池的默认大小是-128-127

- String的不可变

	- 在JDK1.8时用char数组存储String，在JDK1.9使用byte数组存储
	- 数组使用final修饰，保证数组在初始化之后不能在引用其他数组， 并且String内部也没有能改变数组的方法，因此可以保证String的不可变
	- 不可变的好处
		- 可以缓存hash值，String的不可变使得hash值只需要计算一次
		- String Pool的需要，当一个String对象被创建时，就会从String Pool中获取，只有String不可变，才可以使用String Pool(字符串常量池)
		- 安全性
		- 线程安全：String的不可变使得天生具备线程安全性，可以在多个线程之间使用

- String、StringBuffer和StringBuilder

	- 可变性：String不可变，其他两个是可变的
	- 线程安全：String和StringBuffer是线程安全的(StringBuffer内部使用了synchronized)，StringBuilder是线程不安全的

- String Pool(字符串常量池)

	- 当使用字面量("bbb")这种形式创建字符串时，会自动将其加入String Pool中
	- 可以调用String中的intern在运行时将其添加入String Pool中，当String Pool中存在时，返回该字符串的引用
	- Java 7 之前String Pool被放在运行时常量池中，属于永久代，在Java 7中被移入了堆中，这是因为永久代的空间有效，在大量使用字符串的场景下会导致OutOfMemoryError的错误

- String str = new String("123")

	- 在这个过程中，如果字符串常量池中没有123这个字面量，那么这行代码一共创建两个对象，第一个是在常量池中创建一个，new String会在堆中创建一个对象
	- 将字符串作为另一个字符串对象的构造函数参数时，内部的char数组不是复制的，而是直接指向同一个char数组

- 值传递和引用传递

- 隐式类型转换

	- i += j 和 i = i + j ：i = i + j 可能编译不通过，但是 i += j可以通过，因为 i += j相当于 i =(type of i) i + j

- Switch

	- 从Java7开始运行Switch条件判断语句中使用String对象
	- Switch不支持long类型
	- 在JDK1.5之前只支持byte、short、int和char
	- JDK1.5增加了枚举类型和四个的包装类
	
- final关键字

	- 变量：基本类型变量数值不变，引用类型变量无法引用其他值
	- 方法：final声明的方法不能被子类重写，private方法被隐含的指定为final
	- 类：类无法被继承

- static

	- 静态变量
	- 静态方法：静态方法必须有实现，不能是抽象方法
	- 静态代码块
	- 静态内部类：不依赖于外部类的实例，不能访问外部类非静态的变量和方法
	- 静态导包：在使用静态变量和方法时不用在指明ClassName
	- 初始化顺序：静态变量和静态代码块优先于实例变量和普通语句块，静态变量和静态代码块的初始化顺序取决于在代码中的顺序
		- 存在继承的情况下：父类的(静态变量和静态代码块) -> 子类的(静态变量和静态代码块) -> 父类的(实例变量和实例代码块) -> 父类的构造函数 -> 子类的实例变量和实例代码块 -> 子类的构造函数

- Object的通用方法

	- int hashCode():相同的对象一定要有相同的hashCode值

	- boolean equals(Object obj)

		```java
		//1.检查是否为同一个对象的引用，如果是直接返回 true；
		//2.检查是否是同一个类型，如果不是，直接返回 false；
		//3.将 Object 对象进行转型；
		//4.判断每个关键域是否相等。
		public class EqualExample {
		
		    private int x;
		    private int y;
		    private int z;
		
		    public EqualExample(int x, int y, int z) {
		        this.x = x;
		        this.y = y;
		        this.z = z;
		    }
		
		    @Override
		    public boolean equals(Object o) {
		        if (this == o) return true;
		        if (o == null || getClass() != o.getClass()) return false;
		
		        EqualExample that = (EqualExample) o;
		
		        if (x != that.x) return false;
		        if (y != that.y) return false;
		        return z == that.z;
		    }
		}
		
		```

		

	- Object clone()：clone方法为Object的protected方法，如果一个类不显示的重写该方法，其他类就不能直接调用该类实例的clone方法，并且必须实现cloneable接口，如果一个类没有实现cloneable接口又调用了clone方法，就会报错CloneNotSupportedException

		- 浅拷贝(直接调用父类的clone默认为浅拷贝)  - 拷贝对象和原始对象的引用为同一个

			```java
			public class ShallowCloneExample implements Cloneable {
			
			    private int[] arr;
			
			    public ShallowCloneExample() {
			        arr = new int[10];
			        for (int i = 0; i < arr.length; i++) {
			            arr[i] = i;
			        }
			    }
			
			    public void set(int index, int value) {
			        arr[index] = value;
			    }
			
			    public int get(int index) {
			        return arr[index];
			    }
			
			    @Override
			    protected ShallowCloneExample clone() throws CloneNotSupportedException {
			        return (ShallowCloneExample) super.clone();
			    }
			}
			
			
			ShallowCloneExample e1 = new ShallowCloneExample();
			ShallowCloneExample e2 = null;
			try {
			    e2 = e1.clone();
			} catch (CloneNotSupportedException e) {
			    e.printStackTrace();
			}
			e1.set(2, 222);
			System.out.println(e2.get(2)); // 222
			
			```

		- 深拷贝

		- clone的替代方案

			- 使用拷贝构造函数

	- void finalize()

	- Class<?> getClass()

	- String toString()

	- void notify()

	- void notifyAll()

	- void wait()

	- void wait(long timeout)

	- void wait(long timeout,int nanos)

- 继承

	- 子类如果重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别，这是为了确保可以使用父类实例的地方都可以用子类去替代，满足里氏替换原则
	- 字段最好不要是公有的

- 抽象类与接口

	- 抽象类

		- 一个类中如果有抽象方法，就必须声明为抽象类
		- 抽象类不能被实例化，只能被继承

	- 接口

		- 接口在Java8之前，不能有方法的实现，Java8中新增默认方法，为了维护方便
		- 接口中的成员方法默认为public，abstract 
		- 接口中的字段都是static和final的

	- 比较

		- 抽象类需要满足里氏替换原则，即子类要能替换父类，接口不需要
		- 一个类可以实现多个接口，但是不能继承多个抽象类
		- 抽象类中没有字段和方法的访问级别的限制

	- super

		- 访问父类的构造函数
		- 访问父类的成员方法

	- 重写与重载

		- 重写

			- 为了满足里氏替换原则，子类重写父类方法需要有三个原则
				- 子类方法的访问级别不能低于父类
				- 子类方法的返回值必须为父类返回值或者其子类型
				- 子类方法抛出的异常必须是父类异常或者其子异常
			- 使用@Override注解，可以帮忙检查上面的三个限制

		- 在调用一个方法时，先从本类中查找是否有对应的方法，如果没有再到父类中查看，看是否从父类继承来，否则就要对参数进行转型，转成父类之后查看是否有对应的方法，如果还是没有，则参数继续向上转型判断

			- this.func(this)
			- super.func(this)
			- this.func(super)
			- super.func(super)

			```java
			class A {
			
			    public void show(A obj) {
			        System.out.println("A.show(A)");
			    }
			
			    public void show(C obj) {
			        System.out.println("A.show(C)");
			    }
			}
			
			class B extends A {
			
			    @Override
			    public void show(A obj) {
			        System.out.println("B.show(A)");
			    }
			}
			
			class C extends B {
			}
			
			class D extends C {
			}
			
			
			public static void main(String[] args) {
			
			    A a = new A();
			    B b = new B();
			    C c = new C();
			    D d = new D();
			
			    // 在 A 中存在 show(A obj)，直接调用
			    a.show(a); // A.show(A)
			    // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
			    a.show(b); // A.show(A)
			    // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
			    b.show(c); // A.show(C)
			    // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
			    b.show(d); // A.show(C)
			
			    // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
			    A ba = new B();
			    ba.show(c); // A.show(C)
			    ba.show(d); // A.show(C)
			}
			
			```

	- 重载

		- 存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数和顺序至少有一个不同
		- 应该注意，返回值不同其他均相同的不算是重载

	- 反射

	  - Class和java.lang.reflect一起对反射提供了支持，在java.lang.reflect中有三个类：

	  	- Field：可以使用get和set方法修改Field关联的字段
	  	- Method：可以使用invoke方法调用与Method对象关联的方法
	  	- Constructor：可以用Constructor和newInstance创建新的对象

	  	```java
	  	//通过反射调用构造函数、成员函数和变量
	  	Class<?> pClass = Class.forName("Person");
	  	Person person = (Person)pClass.getConstructor(String.class).newInstance("wanglin");
	  	pClass.getMethod("functionName",String.class).invoke(person,"temp");
	  	Field field = pClass.getDeclaredField("FieldName");
	  	field.setAccessible(true);
	  	field.set(person,"haha");
	  	field.get(person);
  
	  	
    	```
	
	  //通过反射调用私有的构造函数
	  	public class Father {
	  	
	  	    private String name;
	  	    private Father() {
	  	        name = "wanglin";
	  	    }
	  	
	  	    public String tosay() {
	  	        return name;
	  	    }
	  	
	  	    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
	  	        Constructor<Father> constructor = Father.class.getDeclaredConstructor();
	  	        constructor.setAccessible(true);
	  	        Father father1 = constructor.newInstance();
	  	        System.out.println(father1.tosay());
	  	    }
	  	}
	  	```
	
	- 异常
	
		- 不受检查异常：编译器不要求强制处理的异常，RuntimeException及其子类和Error类
		- 检查异常：编译器要求必须处置的异常，要不try-catch捕获，要不throws抛出，否则编译不通过

# java容器

- set

	- HashSet(内部元素的插入顺序是无序的)
	- LinkedHashSet(使用双向链表维护元素的插入顺序)
	- TreeSet -> SortedSet（有序，基于红黑树）

- List

	- ArrayList
		- Fail-Fast
		- 扩容是原来大小的1.5倍
		- 不是线程安全的，可以使用Collections.synchronizedList()得到一个线程安全的List
		- 也可以使用concurrent包下的CopyOnWriteArrayList()类
			- 读写分离，写的时候会复制一个数组，并且加锁，结束之后需要把原始数组指向新的数组
			- CopyOnWriteArrayList在写操作的同时可以读，大大提高了读操作的性能，适合读多写少的应用场景
			- 缺陷
				- 内存占用：写操作需要复制一个数组，所以内存占用是原来的两倍
				- 数据不一致：读操作不能读取实时性数据，因为部分写操作的数据还未同步到数组中
			- 因此CopyOnWriteArrayList不适合内存敏感以及实时性很高的应用场景
	- Vector
		- 构造函数可以传入扩容的增长量，默认是两倍
		- 方法使用了synchronized进行同步
	- LinkedList
		- 基于双向链表
		- 数组支持随机访问，但是插入和删除代价很高，需要移动元素
		- 链表不支持随机访问，但插入和删除只需要改变指针

- Map

	- HashMap

		- 使用Entry类型存储，该类是一个链表

		- 使用拉链法并且在插入Entry时采用头插法

		- HashMap允许插入键为null的键值对，但是因为无法调用null的hashcode方法，所以只能强制指定下标来存放，HashMap使用桶下标为0的桶来存放键为null的键值对

		- 确定桶下标

			- 先计算hash值
			- 在通过hash值计算桶下标，计算桶下标时，需要用hash值对table的长度进行取模，如果table的长度正好是2的n次方，那么就可以将取模运算转换成位运算

			```java
			x = 1 << 4;
			x - 1 = 00001111;
			y = 10110010
			y & x-1 = 00000010;
			y % x = 00000010;
			//所以当x为2的n次方时，对x的取模运算可以转换为对x-1的与运算(位运算)
			```

		- 扩容

			- capacity：table的容量，默认为16，必须保证为2的n次方

			- size：键值对的数量

			- threshold：size的临界值，当size大于等于threshold时就必须进行扩容

			- loadFactor：装载因子，table能够使用的比例，threshold = capacity * loadFactor，默认为0.75

			- 扩容默认大小是原来的两倍，并且需要把oldTable的所有键值对重新插入newTable中

			- 扩容的时候需要重新计算桶下标，处于相同桶的Entry只需要计算一个桶下标即可，但是需要把每一个Entry都采用头插法插入newTable中

				- capacity为2的n次方，可以简化重新计算桶下标的速度

				```java
				Capacity = 00010000
				NewCapacity = 00100000
				//当hash值的第5位为0时
				Hash % Capacity = Hash % NewCapacity  //此时桶的位置和原位置相同
				//当hash值的第5位为1时
				Hash % NewCapacity = Hash % Capacity + 16  //此时桶的位置 = 原位置  + 16
				    
				//通过这种方法，可以直接判断hash对应位置的值，如果是0则就是原位置，否则为原位置+16
				//hash & Capacity == 0
				```

				- resize中的loHead，loTail，hiHead、hiTail

				```java
				//loHead和loTail是桶链表中位置不变的所有Node节点组成的链表
				//hiHead和hiTail是桶链表中位置需要+oldTable容量的所有结点组成的链表
				do {
				    next = e.next;
				    if ((e.hash & oldCap) == 0) {
				        if (loTail == null)
				            loHead = e;
				        else
				            loTail.next = e;
				        loTail = e;
				    }
				    else {
				        if (hiTail == null)
				            hiHead = e;
				        else
				            hiTail.next = e;
				        hiTail = e;
				    }
				} while ((e = next) != null);
				```

		- 计算数组的容量

			- HashMap的构造函数允许用户传入的容量不是2的n次方，因为它可以自动的将传入的容量转换为2的n次方

			```java
			static final int tableSizeFor(int cap) {
			    int n = cap - 1;   //如果cap已经是2的n次方，cap-1为了保证返回的是原始值
			    n |= n >>> 1;
			    n |= n >>> 2;
			    n |= n >>> 4;
			    n |= n >>> 8;
			    n |= n >>> 16;
			    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
			}
			```

		- 链表转为红黑树

			- 在JDK1.8开始，一个桶存储的链表长度大于等于8时会将链表转为红黑树

		- 与HashTable的比较

			- HashTable使用Synchronized来进行同步
			- HashMap允许插入键值为null的Entry
			- HashMap的迭代器是Fail-Fast迭代器
			- HashMap不能保证随着时间的推移Map中的元素次序是不变的

	- LinkedHashMap

		- 继承自HashMap

		- 内部维护了一个双向链表，用来维护插入顺序或者LRU顺序(最少使用)

			- accessOrder：默认为false，此时维护的是插入顺序
			- tail：链表的尾节点
			- head：链表的头节点
			- afterNodeAccess(Node<K,V> p)：当一个节点被访问之后调用该函数
				- 当accessOrder为true时，将当前访问节点移动到链表最后，那么链表首部就是最近最久未使用的节点
			- afterNodeInsertion()：当一个节点被插入之后调用该函数
				- 该方法中调用removeEldestEntry()函数，该函数默认返回false，表示不移除链表首结点，如果需要它为true，需要继承LinkedHashMap去重写这个方法的实现，这在实现LRU缓存中特别有用，通过移除最近未使用的结点，保证缓存空间的足够，并且缓存的数据都是热点数据。
			- 使用LinkedHashMap实现LRU缓存

			```java
			public class LRUCache<K, V> extends LinkedHashMap<K, V> {
			
			    private static final int DEFAULTL_SIZE = 3;
			
			    @Override
			    protected boolean removeEldestEntry(Map.Entry eldest) {
			        return size() > DEFAULTL_SIZE;
			    }
			
			    public LRUCache() {
			        super(DEFAULTL_SIZE, 0.75f, true);
			    }
			}
			```

			

	- HashTable

	- TreeMap -> SortedMap

	- ConcurrentHashMap

		- 实现原理和HashMap类似，不同地方在于ConcurrentHashMap加入了segment(分段锁)，每个segment维护几个桶
		- count为每个segment中的键值对数量，当计算size操作时，将所有segment中的count累计起来
			- 在执行size操作时，先尝试不加锁，如果两次结果一致，则认为结果正确，尝试次数如果超过3次，则给所有segment加锁，并计算。

- 复制数组

	- System.arraycopy(object src,int srcPos,object dest,int destPos,int length)
		- 将src数组中从srcPos的位置复制到dest数组从destPos的位置，复制的长度为length
	- Arrays.copyOf(object src,int newlength)
		- 将src数组复制到新的数组中，新的数组长度为newlength，底层调用arraycopy
	- Arrays.copyOfRange(object src,int from,int to)
		- 复制src数组从from位置到to位置(不包括)，底层调用arraycopy

- ObjectOutputStream和ObjectInputStream

	- ObjectOutputStream的writeObject方法在传入的对象有writeObject()时会通过反射调用该方法
	- readObject和writeObject类似的原理

	```java
	ArrayList list = new ArrayList();
	ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
	oos.writeObject(list);
	//writeObject中判断传入的list对象中是否存在writeObject方法，如果存在oos会通过反射调用该方法
	```

- Fail-Fast

	- Java容器的一种保护机制，能够防止多个进程同时修改同一个容器的内容，一旦它发现其他进程修改了容器，就会抛出ConcurrentModificationException异常
		- 原理就是容器中有一个modCount变量，当该变量的值与预期值不符合时，抛出该异常
	- 所以删除list中的元素时，可以使用for循环和iterator自带的remove

	```java
	for(int i=0;i<list.size();i++){
	    //删除傻强
	    if("傻强".equals(list.get(i))){
	        list.remove(i);
	    }
	}
	System.out.println(list);
	
	Iterator<String> iterator = list.iterator();
	while(iterator.hasNext()){
	    String s= iterator.next();
	    //删除lisi
	    if("傻强".equals(s)){
	        iterator.remove();//使用迭代器的remove
	    }
	}
	System.out.println(list);
	
	```

# Java并发

- 使用线程的三种方式

	- 实现Runnable接口，并实现run方法
	- 实现Callable接口，并实现call方法
	- 继承Thread，实现run方法

- 实现接口 VS 继承Thread

	- 实现接口更好一些
		- Java不支持多重继承，因此继承了Thread就无法继承其他类了
		- 继承Thread开销很大

- Executor

	- CachedThreadPool：一个任务创建一个线程
	- FixedThreadPool：所有任务只能使用固定大小的线程
	- SingleThreadExecutor：相当于大小为1的FixedThreadPool，只有一个线程执行所有任务

	```java
	ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
	```

- Daemon

	- 但程序没有一个非守护线程时，程序将退出，并杀死所有守护进程
	- main属于非守护线程
	- 在线程启动之前使用setDaemon方法可以将一个线程设置为守护线程

- sleep

	- 会休眠当前正在执行的线程，单位为毫秒
	- 线程可能会抛出InterruptedException异常，因为异常无法跨线程传播会main中，所有只能try-catch捕获

- yield

	- 对静态方法yield的调用声明了当前线程已经完成了声明周期中最重要的部分，可以切换给其他线程来执行，该方法只是对线程调度器的一个建议，而且也只是建议其他具有相同优先级的线程可以运行

- 中断

	- InterruptedException
		- 通过一个线程调用interrupt()来中断函数，如果该线程正处于阻塞，限期等待或者无限等待状态，调用interrupt函数就会抛出中断异常，从而提前结束线程运行，但是不能中断IO阻塞和synchronized阻塞
	- Interrupted()
		- 如果一个线程的run方法执行一个无限循环，并且没有执行sleep等可以抛出interruptedException的操作，那么调用interrupt()方法就无法使线程提前结束
		- 但是调用了interrupt方法会设置线程的中断标记，此时调用interrupted()会返回true，因此可以在循环体中使用interrupted方法来判断线程是否处于中断状态，从而提前结束线程
		- 并且该方法在调用过之后，会清除中断标记，也就是说如果连续两次调用该函数，第二次将返回false
		- 该函数作用与当前线程
	- isInterrupted()
		- 判断该线程是否被中断，不会清除中断标记(与interrupted()的区别)
		- 线程如果中断，则该函数会被忽略，因为在中断时线程不活动将会被此方法返回false
	- Executor的中断操作
		- ExecutorService中的shutdown方法等待所有线程都执行完毕之后在关闭，shutdownNow方法则相当于调用每个线程的interrupted方法
		- 可以通过使用ExecutorService.submit()方法提交一个线程，该方法返回Future对象，通过该对象调用cancel(true)就可以中断线程

- 互斥同步

	- synchronized
	- ReentrantLock
	- 比较
		- synchronized是由JVM实现的，ReentrantLock是由JDK实现的
		- 当持有锁的线程长期不释放锁的时候，synchronized等待的线程不可中断，ReentrantLock可以中断
		- synchronized和ReentrantLock是不公平锁，synchronized是可重入锁，即可以对一个锁多次加锁，多次释放
			- 公平锁：多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁
			- 不公平锁：多个线程同时竞争同一个锁
	- 当同一个对象里面有static方法加了synchronized，有一个普通方法加了synchronized，如果这时有2个线程，其中一个线程先获取了static方法上的锁，另一个线程还可以在第一个线程释放锁之前获取普通方法的锁么？反过来呢？
		- 可以，因为static上获取的是类锁，普通方法获取的是对象锁，这是两把锁，互不影响
	- 尽量优先使用synchronized

- Join

	- 当调用了t.join后，会挂起当前调用该方法的线程，先执行t线程，等t线程执行完后继续执行挂起的那个线程

- wait、notify和notifyAll

	- wait方法只能在同步代码块或者同步方法中执行，wait方法会让当前线程进入等待状态，直到调用了该对象的notify或者notifyAll方法
	- wait是object的方法
	- wait会释放对象锁
	
- await、signal和signalAll

	- await是Condition的方法，使用lock.newCondition可以创建一个对象，调用await方法会释放当前lock，并进入等待队列，知道调用signal或者signalAll之后该线程获取到锁为止

		```java
		public class AwaitSignalExample {
		    private Lock lock = new ReentrantLock();
		    private Condition condition = lock.newCondition();
		    public void before() {
		        lock.lock();
		        try {
		            System.out.println("before");
		            condition.signalAll();
		        } finally {
		            lock.unlock();
		        }
		    }
		    public void after() {
		        lock.lock();
		        try {
		            //释放lock锁
		            condition.await();
		            System.out.println("after");
		        } catch (InterruptedException e) {
		            e.printStackTrace();
		        } finally {
		            lock.unlock();
		        }
		    }
		    public static void main(String[] args) {
		        ExecutorService executorService = Executors.newCachedThreadPool();
		        AwaitSignalExample example = new AwaitSignalExample();
		        executorService.execute(() -> example.after());
		        executorService.execute(() -> example.before());
		    }
		}
		//output:
		before
		after
		```

		

- 线程状态

  - NEW：创建后尚未启动
  - RUNNABLE：正在Java虚拟机中运行
  - BLOCKED：阻塞状态
  - WAITING：该状态是主动的，阻塞是被动的
  - TIMED_WAITING
  - TERMINATED：死亡，可以是线程结束任务后自己结束，也可以是产生了异常结束

- JUC(Java.Util.Concurrent)

	- CountDownLatch：用来控制一个或者多个线程等待多个线程，内部维护了一个计数器，当计数器减到0时，那些因为调用await方法而在等待的线程才会执行

	```java
	public class CountdownLatchExample {
	    public static void main(String[] args) throws InterruptedException {
	        final int totalThread = 10;
	        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
	        ExecutorService executorService = Executors.newCachedThreadPool();
	        for (int i = 0; i < totalThread; i++) {
	            executorService.execute(() -> {
	                System.out.print("run..");
	                countDownLatch.countDown();
	            });
	        }
	        countDownLatch.await();
	        System.out.println("end");
	        executorService.shutdown();
	    }
	}
	
	//run.. ...    end
	```

	- CyclicBarrier

		- CyclicBarrier用来控制多个线程互相等待，当调用了该类中的await方法时，计数器减一，当减到0时，等待的线程开始执行,并且计数器恢复初始值，调用reset可以重置该值
		- CyclicBarrier(int ,Runnable)，后面的参数表示当最后一个线程到达栅栏后要完成的任务

		```java
		import java.util.concurrent.*;
		
		public class Parallelimit {
		    public static void main(String[] args) throws InterruptedException {
		        int totalThread = 5;
		        ExecutorService service = Executors.newCachedThreadPool();
		        CyclicBarrier cyc = new CyclicBarrier(totalThread,()->
		        {
		            System.out.println(Thread.currentThread().getName() + "完成任务");
		        });
		        for (int i = 0; i < totalThread; i++) {
		            service.execute(()->{
		                try {
		                    Thread.sleep(1000);
		                    System.out.println(Thread.currentThread().getName() + " 到达栅栏 A");
		                    cyc.await();
		                    System.out.println(Thread.currentThread().getName() + " 冲破栅栏 A");
		
		                    Thread.sleep(2000);
		                    System.out.println(Thread.currentThread().getName() + " 到达栅栏 B");
		                    cyc.await();
		                    System.out.println(Thread.currentThread().getName() + " 冲破栅栏 B");
		                } catch (Exception e) {
		                    e.printStackTrace();
		                }
		            });
		        }
		    }
		}
		
		//output:
		pool-1-thread-3 到达栅栏 A
		pool-1-thread-1 到达栅栏 A
		pool-1-thread-2 到达栅栏 A
		pool-1-thread-5 到达栅栏 A
		pool-1-thread-4 到达栅栏 A
		pool-1-thread-4完成任务
		pool-1-thread-1 冲破栅栏 A
		pool-1-thread-2 冲破栅栏 A
		pool-1-thread-3 冲破栅栏 A
		pool-1-thread-5 冲破栅栏 A
		pool-1-thread-4 冲破栅栏 A
		pool-1-thread-2 到达栅栏 B
		pool-1-thread-4 到达栅栏 B
		pool-1-thread-5 到达栅栏 B
		pool-1-thread-3 到达栅栏 B
		pool-1-thread-1 到达栅栏 B
		pool-1-thread-1完成任务
		pool-1-thread-1 冲破栅栏 B
		pool-1-thread-5 冲破栅栏 B
		pool-1-thread-4 冲破栅栏 B
		pool-1-thread-2 冲破栅栏 B
		pool-1-thread-3 冲破栅栏 B
		```

	- CyclicBarrier和CountDownLatch的区别

		- CountDownLatch中的线程的职责是不一样的，有的在倒计时，有的在等待倒计时
		- CyclicBarrier中的线程的职责是一样的
		- CyclicBarrier中的计数器可以循环利用

	- Semaphore

		- 类似于操作系统中的信号量，控制对互斥资源的访问线程数

		```java
		import java.util.concurrent.*;
		
		public class Parallelimit {
		    public static void main(String[] args) throws InterruptedException {
		        int clientCount = 3;
		        int totalRequestCount = 10;
		        ExecutorService service = Executors.newCachedThreadPool();
		        Semaphore semaphore = new Semaphore(clientCount);
		        for (int i = 0; i < totalRequestCount; i++) {
		            service.execute(() ->{
		                try {
		                    semaphore.acquire();
		                    System.out.println(semaphore.availablePermits() + " ");
		                } catch (InterruptedException e) {
		                    e.printStackTrace();
		                }finally {
		                    semaphore.release();
		                }
		            });
		        }
		    }
		}
		
		//output:
		2 1 1 1 2 2 1 2 2 2
		```

	- FutureTask

		- FutureTask可用于异步获取执行结果或取消执行任务的场景，当一个计算任务需要很长时间，那么就可以用FutureTask来封装这个任务，主线程在完成任务之后在获取返回值结果

		```java
		public static void main(String[] args) throws ExecutionException, InterruptedException {
		        CountDownLatch cnt = new CountDownLatch(1);
		        FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {
		            @Override
		            public Integer call() throws Exception {
		                int result = 0;
		                for (int i = 0; i < 100; i++) {
		                    Thread.sleep(10);
		                    result += i;
		                }
		                cnt.countDown();
		                return result;
		            }
		        });
		        Thread computerThread = new Thread(futureTask);
		        computerThread.start();
		        cnt.await();
		        System.out.println(futureTask.get());
		 }
		//output：4950
		```

	- BlockingQueue

		- 阻塞队列
			- LinkedBlockingQueue
			- ArrayBlockingQueue(固定长度)
			- PriorityBlockingQueue
		- 提供了阻塞的take()和put()方法

		```java
		class ProducerConsumer {
		    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);
		
		    public static class Producer extends Thread {
		        @Override
		        public void run() {
		            try {
		                queue.put("product");
		            } catch (InterruptedException e) {
		                e.printStackTrace();
		            }
		            System.out.println("produce..");
		        }
		    }
		
		    private static class Consumer extends Thread {
		        @Override
		        public void run() {
		            try {
		                queue.take();
		            } catch (InterruptedException e) {
		                e.printStackTrace();
		            }
		            System.out.println("consume..");
		        }
		    }
		
		    public static void main(String[] args) {
		        for (int i = 0; i < 2; i++) {
		            Producer producer = new Producer();
		            producer.start();
		        }
		
		        for (int i = 0; i < 5; i++) {
		            Consumer consumer = new Consumer();
		            consumer.start();
		        }
		
		        for (int i = 0; i < 3; i++) {
		            Producer producer = new Producer();
		            producer.start();
		        }
		    }
		}
		```

	- ForkJoin

		- ForkJoin使用ForkJoinPool来启动，ForkJoinPool是一个特殊的线程池，线程数量取决于CPU核数，ForkJoinPool使用工作窃取算法提高CPU利用率，允许空闲的线程从其他的线程中窃取一个任务来执行，窃取的任务必须是最晚的任务。

		```java
		import java.util.concurrent.ExecutionException;
		import java.util.concurrent.ForkJoinPool;
		import java.util.concurrent.Future;
		import java.util.concurrent.RecursiveTask;
		
		public class ForkJoinExample extends RecursiveTask<Integer> {
		
		    private final int threshold = 5;
		    private int first;
		    private int last;
		
		    public ForkJoinExample(int first, int last) {
		        this.first = first;
		        this.last = last;
		    }
		
		    @Override
		    protected Integer compute() {
		        int result = 0;
		        if (last - first <= threshold) {
		            // 任务足够小则直接计算
		            for (int i = first; i <= last; i++) {
		                result += i;
		            }
		        } else {
		            // 拆分成小任务
		            int middle = first + (last - first) / 2;
		            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
		            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
		            leftTask.fork();
		            rightTask.fork();
		            result = leftTask.join() + rightTask.join();
		        }
		        return result;
		    }
		
		    public static void main(String[] args) throws ExecutionException, InterruptedException {
		        ForkJoinExample example = new ForkJoinExample(1, 10000);
		        ForkJoinPool forkJoinPool = new ForkJoinPool();
		        Future result = forkJoinPool.submit(example);
		        System.out.println(result.get());
		    }
		}
		```

	- 线程不安全

	- Java内存模型

		- 每个线程都有自己的工作内存，不同线程交互需要通过主内存，工作内存是存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本的拷贝。
		- 内存模型定义了8个操作来完成主内存和工作内存之间的交互操作
		- 内存模型三大特性
			- 原子性：每一个操作都具有原子性，但是多个原子性操作结合不一定还是原子性，AtomicInteger可以保证多个线程修改的原子性，使用synchronized也可以保证操作的原子性(它对应的内存间操作为lock和unlock)
			- 可见性：指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改，Java内存模型是通过在变量修改后将新值同步回主内存中，在变量读取前从主内存刷新变量值来实现可见性
			- 有序性：在本线程内观察，所有操作都是有序的，在一个线程观察另一个线程，所有操作都是无序的，无序是因为指令发生了重排序
				- volatile关键字可以通过添加内存屏障的方法禁止指令重排
		- 先行发生原则
			- 单一线程原则
			- 管程锁定原则：一个unlock操作先行发生于后面对同一个锁的lock操作
			- volatile变量规则：对一个volatile变量的写操作先行发生于后面对这个变量的读操作
			- 线程启动规则：Thread对象的start方法先行发生于此线程的每一个动作
			- 线程加入规则：Thread对象的结束先行发生于join方法的返回
			- 线程中断规则：对线程interrupt方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过interrupted方法检测到是否有中断发生
			- 对象终结规则：一个对象的初始化完成(构造函数执行结束)先行发生于它的finalize方法的开始
			- 传递性： 如果操作A先行发生于操作B，操作B先行发生于操作C，则操作A先行发生于操作C

	- 线程安全

		- 多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为
		- 不可变的类型
			- final修饰的基本数据类型
			- String类型
			- 枚举类型
			- Number部分子类：如Long和Double等数值包装类，BigInteger和BigDecimal等大数据类型，AtomicInteger和AtomicLong是可变的
			- 对于集合类型，可以使用Collections.unmodifiableXXX()方法来获取一个不可变的集合
		- 互斥同步
			- synchronized
			- ReentrantLock
		- 同步
			- 阻塞同步
			- 非阻塞同步：先进行操作，如果没有其他线程争用共享数据，那操作就成功了，否则采取补偿措施，不断的重试，知道成功为止
			- CAS指令：需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B
			- AtomicInteger：该类中使用了Unsafe类的CAS操作
			- ABA：CAS中的ABA问题，如果一个变量初始值为A，它的值被改成B，后来又被改为A，那么CAS操作依旧会成功执行，使用AtomicStampedReference可以解决这个问题。
				- ABA问题举例：内存位置为链表的头结点node1，线程2操作头结点变化了两次，很可能是先修改头结点为node2，再将node1插入表头成为新的结点，对于线程1，头结点仍然是node1，CAS操作成功，但是子链表的状态已经不可知
		- 无同步方案
			- 局部变量存储在虚拟机栈中，属于线程私有的
			- 线程本地存储：每个线程都有一个ThreadLocal.ThreadLocalMap对象，当调用set方法时，先获取当前线程的Map对象，然后存储value值

	- 锁优化

		- 自旋锁：互斥同步进入阻塞状态的开销很大，应该尽量避免，共享数据的锁定状态只会持续很短的一段时间，自旋锁的思想就是让一个线程在请求一个共享数据的锁时，执行忙循环(自旋)，如果在这段时间内能获得锁，就可以避免进入阻塞
		- 锁消除
		- 锁粗化
		- 轻量级锁
			- JDK1.6引入了轻量级锁和偏向锁
		- 偏向锁
			- 偏向于第一个获取锁对象的线程，当有另一个线程去尝试获取锁对象时，偏向状态结束，此时撤销偏向状态恢复到未锁定状态或轻量级锁状态
		

# Java虚拟机

- 运行时数据区域

	<img src="D:\软件\typora\笔记\img\Java虚拟机.png" style="zoom:50%;" />

	- 程序计数器：记录正在执行的虚拟机字节码指令的地址，如果正在执行的是本地方法则为空
	- Java虚拟机栈：每个Java方法在执行的同时会创建一个栈帧用来存储局部变量表、操作数栈和常量池引用的，从方法的调用到执行完成的过程，对应着一个栈帧在Java虚拟机栈中入栈和出栈的过程
		- 可以使用-Xss虚拟机参数来指定每个线程Java虚拟机栈的大小，JDK1.4中默认为256K，而在JDK1.5+则为1M
		- 该区域可能抛出两个异常
			1. 当线程请求的栈深度超过最大值，抛出StackOverflowError异常
			2. 栈进行动态扩展时如果无法申请到足够内存，抛出OutOfMemoryError异常
	- 本地方法栈：本地方法栈是为本地方法服务，本地方法一般是用其他语言编写的，并且编译为基于本机硬件和操作系统的程序。
	- 堆
		- 所有对象都在这里分配内存，是垃圾收集的主要区域(GC堆)
		- 堆可以分为两块：新生代和老生代
		- 可以动态增加堆内存，增加失败会抛出OutOfMemoryError异常
		- 可以使用-Xms和-Xmx来指定一个程序的堆内存大小，第一个是初始堆内存大小，第二个是最大堆内存大小
	- 方法区
		- 用于存放已加载的类信息、常量、静态变量和即时编译器编译后的代码
		- 从JDK1.8开始移除永久代，并把方法去移至元空间，它位于本地内存中，而不是虚拟机内存中
		- 在JDK1.8之后，原来永久代的数据被分到了堆和元空间，元空间存储类信息，常量和静态变量存储在堆中
	- 运行时常量池
		- 运行时常量池是方法区的一部分
		- Class文件中的常量池(编译器生成的字面量和符号引用)会在类加载后被放入这个区域，除了在编译期生成的常量外，还允许动态生成，例如String的intern方法
	- 直接内存

- 垃圾收集

	- 垃圾收集主要针对堆和方法区进行，程序计数器、Java虚拟机栈和本地方法栈为线程私有的，只存在于线程的生命周期内，线程结束就会消失
	- 判断一个对象是否可被回收
		- 引用计数算法：可能出现两个对象循环引用，此时这两个对象永远无法被回收，所以Java虚拟机不使用计数算法
		- 可达性分析算法：以GC Roots为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收
			- GC Roots一般包含以下内容
				1. 虚拟机栈中局部变量表引用的对象
				2. 本地方法栈中JNI中引用的对象
				3. 方法区中类静态属性引用的对象
				4. 方法区中常量引用的对象
		- 方法区的回收
			- 方法区主要是对常量池的回收和对类的卸载
			- 类的卸载条件
				1. 该类的所有实例已被回收，此时堆中不存在该类的任何实例
				2. 加载该类的ClassLoader已经被回收
				3. 该类对应的class对象没有被任何引用，也就无法在任何地方通过反射访问该类
		- finalize()
			- 一个对象在回收的时候会调用该方法，可以在该方法中实现自救，即把自身的引用加载到GC Roots链上
			- finalize方法具有不确定性，可以还在执行的过程中就被回收了
			- 该方法只会调用一次
		- 对象的引用类型
			- 强引用：new出来的对象
			- 软引用：SoftReference
			- 弱引用：WeakReference
			- 虚引用：PhantomReference(不会影响对象的生存时间，作用仅仅时在对象回收时可以收到一条系统通知)
		- 垃圾收集算法
			- 标记-清除：在标记阶段，检查每个对象是否存活，并进行标记，之后将可回收的对象进行回收，并将该块加入空闲链表中(相邻的分块会进行合并)
				- 会产生大量不连续的内存碎片，无法给大对象分配内存，标记和清除效率不高
			- 标记-整理：标记阶段和上述一致，之后会将存活的对象移至一端，然后直接清理调端边界以外的内存
				- 需要大量移动对象，效率较低
			- 复制：将内存分为一块较大的Eden空间和2块较小的Survivor空间，每次使用Eden和其中一块Survivor，在回收时，将两块空间上存活的对象都复制到另一个Survivor上，最后清理调Eden和使用过的那一块Survivor。HotSpot虚拟机的Eden和Survivor的比例为8:1
			- 分代收集
				- 新生代：复制算法
				- 老年代：标记-清除或者标记-整理
		- 垃圾收集器
			1. Serial收集器：单线程收集器，串行工作(当执行垃圾收集的时候必须停顿用户程序)，是client场景下默认的新生代收集器
			2. ParNew收集器：Serial收集器的多线程版本，是Server场景下默认的新生代收集器
			3. Parallel Scavenge收集器：
			4. Serial Old收集器
			5. Parallel Old收集器
			6. CMS收集器
			7. G1收集器
		- 内存分配与回收策略
			- Minor GC：回收新生代，新生代对象存活时间短，执行的速度一般也比较块
			- Full GC：回收老年代和新生代，很少执行，执行速度比Minor GC慢很多
			- 内存分配策略
				- 对象优先分配在Eden上，当Eden空间不够时，发起Minor GC
				- 大对象和长期存活的对象直接进入老年代
				- 动态对象年龄判断
				- 空间分配担保
					- 在发生Minor GC之前，判断老年代的最大连续可用空间是否大于新生代所有对象总空间，条件成立则Minor GC是安全的，否则就查看是否允许担保失败，如果允许，判断老年代可用连续空间是否大于历次晋升到老年代对象的平均大小，如果大于尝试冒险进行一次Minor GC，否则进行Full GC
			- Full GC的触发条件
				- 调用System.gc()
				- 老年代空间不足
				- 空间担保分配失败
				- Concurrent Mode Failure：执行CMS GC的过程需要有对象放入老年代，老年代空间不足就会触发Full GC
				- JDK1.7之前永久代空间不足

# 类加载机制

- 类是在运行期间第一次使用时动态加载的，类的生命周期包括：加载  验证  准备  解析  初始化 使用  卸载

- 类加载过程是前5个阶段

	- 加载

		- 通过类的完全限定名称获取定义该类的二进制字节流
		- 将该字节流表示的静态存储结构转换为方法区的运行时存储结构
		- 在内存中生成一个代表该类的Class对象，作为方法区中该类各种数据的访问入口

	- 验证

	- 准备

		- 在该阶段为类变量分配内存并设置初始值
		- 实例变量不会在这个阶段分配内存，它会在对象实例化时随着对象一起被分配在堆中，初始化值一般为0，引用值为null
		- 如果类变量是常量，那么它将初始化为表达式所定义的值

	- 解析

		- 将常量池的符号引用替换为直接引用的过程，有时候解析过程在某些情况下可以在初始化阶段之后在执行，这是为了支持Java的动态绑定
	
- 初始化
	
	- 初始化阶段才真正的开始执行类中定义的Java代码，初始化阶段是虚拟机执行类构造器<clinit>方法的过程在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，会将类变量进行真正的赋值
		- <clinit>是由编译器自动收集类变量的所有赋值动作和静态代码块中的语句合并而成的，特别注意的是，静态代码块之能访问到在它之前定义的类变量，在它之后定义的类变量，只能赋值，不能访问。
		- 当有继承关系时，先执行父类的<clinit>
		- 接口中不可以使用静态代码块，但是仍然有类变量的赋值操作，所有仍然会生成<clinit>方法，但接口与类不同，执行接口的<clinit>的方法时不需要先执行父接口的<clinit>方法，只有当父接口中定义的变量使用时，父接口才会初始化，另外接口的实现类在初始化时也一样不会执行接口的<clinit>方法。
		- 虚拟机会保证一个类的<clinit>方法在多线程环境下被正确的加锁和同步
	
- 类初始化时机
	
	- 主动引用
	
		- 使用new关键字实例化对象时，读取或设置一个类的静态字段(被final修饰、已在编译期把结果放入常量池的静态字段除外)，调用一个类的静态方法
			- 对类进行反射调用时
			- 当初始化一个类时，发现父类还没有初始化
			- 虚拟机启动时，先加载主类(main方法所在的类)
	
	- 被动引用(下面的场景引用类时将不会进行初始化)
	
		- 通过子类直接访问父类的静态字段，不会导致子类的初始化
			- 通过数组定义来引用类，不会导致类的初始化
	
		```java
			SuperClass[] sca = new SuperClass[10]
		```
	
		- 常量在编译阶段会存入调用类的常量池中
	
- 类加载器的分类
	
	- 启动类加载器(Bootstrap ClassLoader)：此类加载器负责将
		- 扩展类加载器(Extension ClassLoader)：它负责将JAVA_HOME/lib/ext或者java.ext.dir系统变量所指定路径中的所有类库加载到内存中
		- 应用程序类加载器(Application ClassLoader)：由于这个类加载器是ClassLoader中的getSystemClassLoader方法的返回值，因此一般称为系统类加载器，它负责加载用户路径Classpath上所指定的类库，如果应用程序中没有自定义过类加载器，一般情况下这个就是程序中默认的类加载器
	
- 双亲委派模型
	
	- 工作过程：一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成加载时，才尝试自己加载
		- 好处：使得Java类随着它的类加载器一起具有一种带着优先级的层次关系，从而使得基础类得到统一
			- 例如：java.lang.Object存放在rt.jar中，如果编写另外一个java.lang.Object并放到Classpath中，程序可以编译通过，由于双亲委派模型的存在，在rt.jar中的object比在Classpath中的object优先级更高，这是因为rt.jar中的Object是由启动类加载器加载的，而Classpath中的Object是由系统类加载器加载的，那么程序中所有的Object都是rt.jar中的Object
		- 实现：ClassLoader中loadClass方法采用双亲委派模型，先检查类是否已经被加载过，如果没有则让父类加载器去加载，当父类加载器加载失败时，此时尝试自己加载(调用findClass方法)
	
- 自定义类加载器实现
	
	- 需要继承ClassLoader，并重写findClass函数
	
	```java
		public class FileSystemClassLoader extends ClassLoader {
		
		    private String rootDir;
		
		    public FileSystemClassLoader(String rootDir) {
		        this.rootDir = rootDir;
		    }
		
		    protected Class<?> findClass(String name) throws ClassNotFoundException {
		        byte[] classData = getClassData(name);
		        if (classData == null) {
		            throw new ClassNotFoundException();
		        } else {
		            return defineClass(name, classData, 0, classData.length);
		        }
		    }
		
		    private byte[] getClassData(String className) {
		        String path = classNameToPath(className);
		        try {
		            InputStream ins = new FileInputStream(path);
		            ByteArrayOutputStream baos = new ByteArrayOutputStream();
		            int bufferSize = 4096;
		            byte[] buffer = new byte[bufferSize];
		            int bytesNumRead;
		            while ((bytesNumRead = ins.read(buffer)) != -1) {
		                baos.write(buffer, 0, bytesNumRead);
		            }
		            return baos.toByteArray();
		        } catch (IOException e) {
		            e.printStackTrace();
		        }
		        return null;
		    }
		
		    private String classNameToPath(String className) {
		        return rootDir + File.separatorChar
		                + className.replace('.', File.separatorChar) + ".class";
		    }
		}
	```
	
	

# Java IO

- Java IO可以分为以下几类
	1. 磁盘操作：File
	2. 字节操作：InputStream和OutputStream
	3. 字符操作：Reader和Writer
	4. 对象操作：Serializable
	5. 网络操作：Socket
	6. 新的输入输出：NIO
- 磁盘操作
- 字节操作
- 字符操作
- 对象操作
- 网络操作
- NIO
	- 通道：双向的，流是单向的
	- 