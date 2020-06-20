# 并发

## 一、锁

### 1.1 ReentrantLock锁原理

#### 1.1.1 加锁

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

#### 1.1.2 解锁

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
