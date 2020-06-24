# 并发

## 一、锁

### 1.1 ReentrantLock锁原理

#### 1.1.1 加锁lock

![](D:\software\git\repository\GitManager\img\原理图解\Java并发\ReentrantLock原理.png)

- 公平锁

  acquire函数就是用来加锁判断的，根据不同的情况有不同的处理方式

  ```java
  /**
       * ReentrantLock.lock的步骤
       * 1，tryAcquire首先尝试获取锁，
       *      - 当获取成功的时候，返回true，此时if判断直接结束，线程获取锁成功
       *      - 当获取失败的时候，返回false，此时将线程加入队列
       * 2. addWaiter表示将当前线程加入队列中
       *      - 加入队列时，如果队列的中啥都没有，则会先创建一个空Node，然后将当前线程封装好的Node加入队列中，并设置好指针
       *      - 如果队列不为空，则直接将代表当前线程的Node加入队列最后
       *
       * 3. acquireQueued表示将加入队列的线程进行park，没有获取到锁就会被阻塞在该函数中，返回值为线程在阻塞的过程中是否被中断
       *      - 如果需要park的线程是第二个线程，那么首先自旋一次，之后通过判断前一个Node的ws状态来再次自旋一次，如果两次都没有获取锁则park
       *      - 如果需要park的线程不是第二个线程，那么也会进行一个空转，判断当前线程所在的Node的前驱Node是否是head，不是则park，是的话和上条一致
       * 4. selfInterrupt()函数是lock函数为了兼容lockInterruptibly函数
       *      - 当线程阻塞被唤醒时，通过返回值可以知道该线程之前是否被阻塞，但是会清除中断标志位，所以该函数的作用就是恢复中断标志位为true
       * @param arg
       */
  public final void acquire(int arg) {
      if (!tryAcquire(arg) &&
          acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
          selfInterrupt();
  }
  ```

  tryAcquire函数会根据锁的状态来判断是否可以成功获取锁

  ```java
  /**
           * 尝试获取锁状态，返回结果为true表示获取锁成功，返回结果为false表示获取锁失败
           * 1. getState()获取锁状态
           *      当state=1表示获取锁失败，返回false
           *      当state=0表示当前锁处于自由状态，此时需要进行判断hasQueuedPredecessors函数(表示该线程是否需要入队，true表示需要，false表示不需要入队)
           *              1.1 该线程需要入队列，此时返回false
           *              1.2 该线程不需要入队列，此时使用CAS操作直接修改state状态，并设置当前线程为持有锁线程
           * 2. current == getExclusiveOwnerThread()
           *      该代码表示ReentrantLock支持重入锁
           * @param acquires
           * @return
           */
  protected final boolean tryAcquire(int acquires) {
      final Thread current = Thread.currentThread();
      int c = getState();
      if (c == 0) {
          if (!hasQueuedPredecessors() &&
              compareAndSetState(0, acquires)) {
              setExclusiveOwnerThread(current);
              return true;
          }
      }
      //可重入锁的实现代码
      else if (current == getExclusiveOwnerThread()) {
          int nextc = c + acquires;
          if (nextc < 0)
              throw new Error("Maximum lock count exceeded");
          setState(nextc);
          return true;
      }
      return false;
  }
  ```

- 非公平锁

  非公平锁会直接进行CAS操作，失败才会去获取锁acquire

  ```java
  final void lock() {
      if (compareAndSetState(0, 1))
          setExclusiveOwnerThread(Thread.currentThread());
      else
          acquire(1);
  }
  ```

  非公平锁的逻辑与公平锁的唯一区别在：当锁状态为自由状态时，公平锁需要判断当前线程是否需要入队，非公平锁直接CAS操作

#### 1.1.2 解锁unlock

公平锁和非公平锁的解锁操作是一样的

```java
/**
     * 释放锁，当释放锁成功(成功的判断条件是锁状态为0)时返回true，否则返回失败
     * 1. tryRelease函数是将当前state的值减一，当值为0时返回true，不为0则返回false
     *      - 必须当state的值为0时，才会去唤醒队列中的线程，之后返回true
     *      - 如果tryRelease后state的值依然大于0，那么一定时重入锁导致的，需要多次释放锁才会去唤醒线程
     * 2. unparkSuccessor函数唤醒队列中第二个位置的节点(第一个位置的节点的thread是null)
     * @param arg 
     * @return
     */
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

//唤醒node.next的线程
private void unparkSuccessor(Node node) {
     
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
    
        Node s = node.next;
        //如果一个线程代表的Node中ws大于0表示这个线程已经被取消，此时从队列中从后向前遍历寻找最后一个线程Node的ws为<=0的Node，并唤醒该线程
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

#### 1.1.3 可中断锁lockInterruptibly

```java
/**该函数与acquireQueued的逻辑是一致的，区别在于：
     *    1. acquireQueued中当不会直接抛出异常
     *        - 该函数中直接抛出异常，此时线程中断标识为false(因为调用的是Thread.interrupted,会清除标志位)
     *        - 因为lock函数中阻塞线程调用的也是parkAndCheckInterrupt函数，因为会清除标志位，所以为了兼容只能调用一次selfInterrupt函数来将标志位重新设置为true
     *    2. 将addWaiter函数和acquireQueued函数进行了封装
     */
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        
        
        if (failed)
            cancelAcquire(node);
    }
}
```

### 1.2 Synchronized锁

#### 1.2.1 锁升级

```java
1. 对象刚刚new出来时，为无锁状态  0 0 1
2. 给对象上锁，没有出现竞争，此时为偏向锁 1 0 1，markword中会存储当前线程ID
3. 当发生竞争之后，变为轻量级锁(竞争的过程：撤销偏向锁,每个线程有自己的锁记录LR，对象中的markword会记录抢到的线程对应的LR) 0 0
4. 竞争加剧会自动升级为重量级锁(调用操作系统的函数，CPU会从用户态切换到内核态) 1 0 
5. 1 1 GC标记信息
```

#### 1.2.2 锁消除

```java
public void add(String str1,String str2){
    StringBuffer sb = new StringBuffer();
    sb.append(str1).append(str2);
}
//StringBuffer是线程安全的，内部方法被synchronized修饰，但是当JVM检测到sb只在add方法中使用，不可能被其他线程引用(因为是局部变量，栈私有)，JVM会自动消除对象内部的锁
```

#### 1.2.3 锁粗化

#### 1.2.4 Synchronized实现过程

```java
moniterenter moniterexit   -->  lock comxchg指令
```

### 1.3 volatile原理

```java
保证了内存模型的可见性和有序性，无法保证原子性
```



## 二、并发基础 知识

### 2.1 线程状态

```java
NEW: 一个还没开始的线程处于这种状态
RUNNABLE：一个正在执行的线程，当调用start函数之后的线程处于这种状态
//waiting和blocked区别：blocked更像是被动的进入这种状态，waiting更像是自己主动进入这种状态
//blocked是虚拟机认为程序还不能进入临界区，进去会有问题
//Wait操作是已经进入了临界区，但是因为某些其他资源没有准备充足，自己就先等等
BLOCKED：处于这个状态的线程一般是在等待获取监视器上的锁，通常由两种情况：一、执行synchronized方法等待监视器上的锁;二、调用object.wait方法之后被notify，等待重新获取synchronized上的锁
WAITINT：一个线程等待另一个线程执行一个特殊的操作，例如：object.wait方法，join方法、park方法
TIMED-WAITING：处于waiting状态，但最多waiting到指定的超时时间，例如：Thread.sleep,object.wait(time)
TERMINATED：执行结束的线程处于这种状态
```

### 2.2 Thread.sleep与Object.wait方法有何区别

```java
1. sleep是Thread的静态方法，同时也是一个本地方法，wait方法是object上的final方法，不可被重写
2. wait执行的时候需要获取到object的锁，sleep不需要
```

### 2.3 Java中线程间通信的方法

```java
1. 使用锁机制(Synchronized,ReentrantLock,ReentrantReadWriteLock)
2. 条件变量机制：volatile变量，atomicInteger类的对象
3. 基于锁机制衍生出的一些通信方式：基于Synchronized衍生的wait-notify机制，基于ReentrantLock衍生出的Condition.wait和signal机制
4. JUC包下的一些工具类：CountDownLatch、CyclicBarrier、Semphore(这些底层都是基于AQS框架来实现的)
```

### 2.4  对象布局

```java
Object o = new Ojbcet();        //在内存中一共占16个字节
// 对象头(markword)  8个字节,存储synchronized的信息
// 类型指针  4个字节
//实例数据
//对齐(8的整数倍)
```



## 一、实现循环输出AB

### 1.1 使用wait和notify

```java
public static void main(String[] args) {
        Object o = new Object();
        Thread t1 = new Thread(() -> {
            while (true) {
                synchronized (o) {
                    System.out.println("A");
                    try {
                        o.notify();
                        o.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "thread-A");
        Thread t2 = new Thread(() -> {
            while (true) {
                synchronized (o) {
                    System.out.println("B");
                    try {
                        o.notify();
                        o.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "thread-B");

        t1.start();
        t2.start();
    }
```

### 1.2 使用Exchanger

```java
public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();
        Thread t1 = new Thread(() -> {
            String str = "A";
            while (true) {
                System.out.println(str);
                try {
                    str = exchanger.exchange(str);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "thread-A");
        Thread t2 = new Thread(() -> {
            String str = "B";
            while (true) {
                System.out.println(str);
                try {
                    str = exchanger.exchange(str);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "thread-B");

        t1.start();
        t2.start();
    }
```

### 1.3 使用CAS操作

```java
public static void main(String[] args) {
        AtomicInteger a = new AtomicInteger(1);
        Object o = new Object();
        Thread t1 = new Thread(() -> {
            while (true) {
                synchronized (o) {
                    if (a.compareAndSet(1, 0)) {
                        System.out.println("A");
                    }
                }
            }
        }, "thread-A");
        Thread t2 = new Thread(() -> {
            while (true) {
                synchronized (o) {
                    if (a.compareAndSet(0, 1)) {
                        System.out.println("B");
                    }
                }
            }
        }, "thread-B");

        t1.start();
        t2.start();
    }
```

### 1.4 使用contidion

```java
//和wait、notify的基本一致
public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();
        Thread t1 = new Thread(() -> {
            while (true) {
                lock.lock();
                try {
                    System.out.println("A");
                    condition.signal();
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }, "thread-A");
        Thread t2 = new Thread(() -> {
            while (true) {
                lock.lock();
                try {
                    System.out.println("B");
                    condition.signal();
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        }, "thread-B");

        t1.start();
        t2.start();
    }
```
