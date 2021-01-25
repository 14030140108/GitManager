# Java并发面试题

## 1. Java线程状态

## 2. Sleep和wait方法区别

- wait需要获取对象锁，sleep不需要
- sleep是Thread的静态方法，wait是object的方法
- sleep运行结束后线程进入Runnable状态，wait执行之后进入Blocked状态

## 3. ConcurrentHashMap的工作原理及代码实现

- ConcurrentHashMap采用分段锁来提高并发度，每个锁维护几个桶，默认段的大小为16
- CHM在计算size和ContainValue方法时，采用乐观锁的机制，先尝试计算3次之后如果还是失败，则给每段进行加锁。
- JDK1.8使用了CAS操作来支持更高的并发度，当CAS操作失败的时候使用内置Synchronized
- JDK1.8的实现也在链表过长时转换为红黑树

## 4. Java间线程的通信方式

- 锁机制：Synchronized、ReentrantLock
- 条件变量机制：使用volatile关键字标识的boolean类型的变量或者AtomicInteger类的对象等进行线程间的同步
- 基于锁机制衍生的一些通信方式，比如基于Synchronized衍生的wait-notify，比如基于Reentrantlock衍生出的Condition.await和signal机制(可以用来实现ArrayBlockingQueue)
- JUC包中的工具类也可以用来实现线程间的通信(CountdownLatch,CyclicBarrier,Semphore)

## 5. 并发编程

- 多线程同步执行
- 两种情况
	- 线程之间存在竞争
	- 线程之间是交替执行的

## 6. Synchronized

- Syn在JDK1.6之前为重量级锁，同步的原理需要调用操作系统的函数，使得CPU需要切换为内核态
- 在JDK1.6之后对Synchronized进行了锁优化，并提出了无锁状态、轻量级锁、偏向锁、重量级锁

## 7. ReenTrantLock

- 内部有一个Sync类继承AQS(AbstractQueuedSynchronizer)
- ReenTrantLock将交替执行的并发情况可以在JDK级别处理，不用调用OS的函数，Sync在JDK1.6之前必须调用OS函数

### 7.1  AQS

- AbstractQueuedSynchronizer
- 该类是一个抽象类，里面最重要的几个字段：head tail 
- 最重要的函数：hasQueuedPredecessor(该函数在公平锁并且state=0时需要判断是否需要插入队列)
- 内部类：Node节点：包含pre、next和Thread当前线程,waitStatus
- AQS中的waitSatus
	- AQS的结点在陷入阻塞时会将前驱结点的waitStatus的值设置为SIGNAL，当前驱结点释放锁时，根据自己的waitStatus状态判断释放要唤醒后继节点(SIGNAL时会唤醒)

### 7.2. ReentrantLock原理

- 基于AQS实现，AQS是一个抽象类，内部维护了一个队列
- 公平锁实现
	- 加锁
		1. 获取当前锁状态，如果为自由状态
			- 如果AQS队列为空，则执行CAS操作
			- 如果队列不为空，则将该线程加入队列
		2. 如果锁为加锁状态，则将线程加入队列，如果该线程为第一个排队的线程，会先自旋两次
	- 解锁
- 非公平锁
	- 加锁
		1. 直接尝试CAS操作，失败则开始正常判断锁状态
		2. 非公平锁当锁状态为自由时，不会考虑同步队列中的其他线程，直接尝试CAS操作，公平锁会考虑需不需要加入同步队列中，通过HasQueuedPredecessors方法来实现的

### 7.3 ReentrantLock可中断的原理

- ReentrantLock提供了一个API，LockInterruptibly该接口必须捕获异常
- 该接口中调用了和lock接口一样的方法parkAndCheckInterrupt，所以lock为了兼容LockInterruptibly，所以parkAndCheckInterrupt的返回值为Thread.interrupted();

## 8. Java中如何避免死锁

- 使用乐观锁机制
- 减少锁粒度
- 控制加锁顺序(如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生，但是需要提前知道所有可能用到的锁)
- 在尝试获取锁的时候设置一个超时时间：如果某个线程没有在给定的时间内获得锁，则会进行回退并释放所有已经获得的锁
- 死锁后程序可以自动检测
	- 检测出来之后可以直接释放所有锁，并且等待一段时间后去重试
	- 给这些线程设置随机的优先级，优先级高的继续执行

## 9. 元空间相比永久代的优势

- 字符串常量池存在永久代，容易出现性能问题和内存溢出
- 类的方法和信息大小难以确定，给永久代大小的指定带来困难
- 永久代会为GC带来复杂性，效率低

## 10. Java运行时数据区

- 线程私有的：程序计数器、java虚拟机栈、本地方法栈
- 线程共享的：堆和元空间
- JDK1.8没有方法区的概念，以前方法区中的字面量和符号引用分配到了堆中的常量池，类加载信息存储在元空间，元空间使用的本地内存

## 11. Java内存模型

- Java内存模型中有主内存和工作内存等概念

## 12. AtomicInteger如何实现？什么是CAS操作

- AtomicInteger是底层是利用CAS来实现的
- CAS就是比较内存值与自己传入的期待值是否相同，如果相同则直接更改为新值，否则更改失败
- AtomicStampedReference可以解决CAS的ABA问题，其实就是增加了一个stamp的东西，相当于版本号

## 13. 一个线程能不能start两次

- 不能，线程在启动之前，会先判断状态是否为NEW，如果不是，则抛出异常

## 14. Executors提供的四种线程池

- newCachedThreadPool：每个任务创建一个线程，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程
- newFixedThreadPool：固定工作线程数量的线程池
- newSingleThreadExecutor：使用单线程对工作队列中的任务进行处理
- newScheduledThreadPool：
- 线程池的任务拒绝策略
	- 丢弃任务并抛出异常(AbortPolicy)
	- 丢弃任务不抛出异常(DiscardPolicy)
	- 丢弃任务队列最前面的任务(DiscardOldestPolicy)
	- 由启动线程池的线程处理该任务
- shutdown和shutdownNow的区别
	- shutdown会继续执行任务队列中的任务，但是不会在接受新的任务了
	- shutdownnow直接中断所有线程，并且清空待处理的任务队列，直接返回

## 15. 缓存一致性

- 总线加锁：性能低下
- MESI缓存一致性协议：当某个线程修改了某个值之后，立马写入主内存中，其他CPU通过总线嗅探机制探知该变量已经被修改，从而将自己缓存里的数据失效。

## 16. volatile

- volatile可以保证有序性和可见性
	- 可见性：在一个线程中对某个变量的修改，对其他线程可见
	- 有序性：禁止指令的重排序
		- 防止指令重排序带来的并发安全问题
		- 在每个 volatile 写操作的前面插入一个 StoreStore 屏障
			- store1 storestore store2：确保store1在store2之前输入主内存
		- 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障
			- 该屏障为万能屏障，包含了其他三个屏障的功能
			- store1 storeload load2：确保store1在load2之前先刷入内存
		- storestore可以保证volatile变量的写操作之前的操作先刷入内存
		- storeload保证volatile变量的写操作先于之后的读操作发生
		- 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障
			- load1 loadload load2：确保load1先于load2装载
		- 在每个 volatile 读操作的后面插入一个 LoadStore 屏障
			- load1 loadstore store2：确保load1先于后续store2刷新至内存  
- volatile不保证原子性
	- 原子性：一个操作或者多个操作要不全部执行，要不全部不执行，不会被其他线程干扰

## 17. JUC下的包

- 原子类：Atomic类
	- 里面的值是使用volatile修饰的，保证了可见性和有序性
	- 在自增的时候使用CAS操作实现了操作的原子性
- Collections：集合框架
- Lock：锁框架
- ThreadPoolExecutor：线程池
- 同步器：CountdownLatch、Cyclicbarrier、Semphore

## 18. happens-before原则(6大原则)

- 解释：前一个操作的结果可以被后一个操作的结果获取
- 程序顺序规则
	- 同一个线程中前面的写操作对后面的操作可见
- 锁规则
	- 如果线程1解锁了锁a，接着线程2加锁了a，那么线程1解锁a之前的写操作都对线程2可见
- volatile变量规则
	- 如果线程1对变量v进行了写入，接着线程2读取了v，那么线程1中对v写入及之前的写操作都对线程2可见
- 传递性
- start()规则
	- 线程A在开启线程B之前的所有修改对线程B可见
- join规则
	- 线程A执行t2.join并成功返回，那么t2中所有的写操作对线程A可见

## 19. 基于Synchronized的notify机制与Condition的signal机制的差异

- notify随机唤醒等待线程
- signal方法将会唤醒在等待队列中等待时间最长的结点(首节点)，将这个节点从等待队列中移到同步队列，之后唤醒该线程

## 20. Synchronized和Lock的区别

- Lock可以非阻塞的获取锁，tryLock，并且tryLock中可以指定超时时间
- Lock可以获取锁的状态，isLocked
- Lock由公平锁和非公平所，Syn是非公平锁
- Lock可以被阻塞，Syn不可以被阻塞
- Lock是JDK实现的，Syn是JVM实现的

## 21. Java中有哪些锁？

- 公平锁，非公平锁，读写锁，共享锁，互斥锁，自旋锁，偏向锁，轻量级锁，重量级锁

## 22.Java对象布局

- MarkWord
	- 在32位计算机上为4个字节，在64位计算机上为8个字节
	- 64位计算机：
		- 前25bit没有用，hash占用31bit，CMS_free占用1bit，age占用4bit()，是否是偏向锁1bit，锁标志位2bit
- Class Pointer
	- 类的元数据：该对象对应的类信息

## 23. Java对象锁状态

- 无状态
	- 0 01
		- 在无锁状态下，前25位用来存储对象的hashcode
- 偏向锁
	- 1 01
	- 只有一个线程进入了临界区，适用于只有一个线程访问同步块场景
	- 第一个获取到锁的线程之后不需要进行同步，也不需要进行CAS操作，当有另一个线程去尝试获取这个锁时，偏向锁状态结束
	- 偏向锁状态下，32位下前25中的前23位存储线程的ID
- 轻量级锁
	- 00
	- 多个线程没有出现竞争，使用CAS操作获取锁对象成功，变成轻量级锁
	- 轻量级锁下，前25位用来指向线程栈中锁记录指针Lock Record的内存地址，Lock Record中有一个owner会指向锁对象
- 重量级锁
	- 10
	- 当两个以上线程出现争用同一个锁时，轻量级锁变为重量级锁
- GC标记
	- 11
- Syn加锁的逻辑流程
	- 首先编译器将Syn编译成JVM指令moniterenter，moniterexit
	- 判断锁对象的markword区域中锁的状态
		- 如果是无锁状态，则加偏向锁，并使用CAS操作将前23位设置为线程ID
		- 如果是偏向锁状态，则查看线程ID是否为当前线程ID，如果是没啥事，如果不事，那么持有偏向锁的线程会判断自己是否已经执行完同步块，如果没有，则将锁改为轻量级锁(需要锁对象的markword中前30位指向线程虚拟机栈中的lockrecord区域，并且owner指向锁对象)，如果已经执行完，则改为无锁状态
		- 如果是轻量级锁，那么线程会尝试使用CAS操作将锁对象的markword中前30位指向线程虚拟机栈中的lockrecord区域，如果CAS失败，则会进行一定次数的自旋，如果自旋阶段还没有获取到锁，则将锁改为重量级锁，线程阻塞
		- 在moniterexit中如果是重量级锁，会去唤醒阻塞队列中的线程，开启新一轮的锁竞争

## 24. ReentrantReadWriteLock

- state中的高16位用来记录读锁，低16位用来记录写锁
- 节点中多了一个mode模式，取值有shared和exclusive，分别表示写锁和读锁
- 如果当前线程获取读锁，则等待队列中的第一个节点一定是写锁等待节点
- 锁降级：当线程1获取写锁后，之后获取读锁可以成功，当线程1释放写锁后，其他线程看到线程1的状态为获取了读锁
- 写锁加锁
	- 锁状态为无锁：和ReentrantLock思路一致，判断是否需要加入等待队列(公平锁和非公平锁)
	- 锁状态为有锁
		- 有读锁了，加入等待队列
		- 有写锁并且不是当前请求锁的线程，加入等待队列
		- 有写锁且当前获取写锁的线程与当前请求锁的线程为同一线程，直接获取锁，并将锁状态值加1
- 写锁释放锁
	- 每次调用解锁方法，减少重入次数，当减少到0时，唤醒等待队列中的第一个节点，如果第一个是读节点，会继续传播唤醒状态
- 读锁加锁
	- 锁状态为无锁状态：获取锁，firstReader指向无锁状态下第一个获取读锁的线程，firstReaderHoldCount记录第一个获取读锁线程持有当前锁的计数
	- 有锁状态
		- 当前锁处于读锁状态且等待队列为空：直接获取锁状态并且读锁状态+1
			- 如果当前线程不是第一个获取读锁的线程，使用ThreadLocal存储当前线程的重入次数
		- 当前锁处于读锁等待队列不为空：公平锁的话加入等待队列，非公平锁的话如果第一个为写锁则加入等待队列(防止写锁饿死)，第一个等待节点一定为exclusive模式，
		- 当前锁处于写锁状态：加入等待队列
- 读锁释放
	- 获取当前线程的重入次数，减一(ThreadLocal)
	- 当前读取的获取次数减一，使用CAS操作
	- 如果读状态位0，则唤醒后续等待节点

## 25. JMM和JVM

- JMM定义了线程工作内存和主内存之间的抽象关系，程序中的变量存储在主内存中，每个线程拥有自己的工作内存并存放变量的拷贝，JMM就是规定了工作内存和主内存之间对于共享变量访问的细节，它指定了一个规范，保证Java编写的多线程在不同平台的JVM上运行都符合这个规范
- JVM也是一个规范，它规定了JVM内存中的运行时数据区，HotSpot就是实现了这个规范