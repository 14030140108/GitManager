# 并发

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

