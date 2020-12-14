# HashMap

1. transfer函数

    ```java
    void transfer(Entry[] newTable,boolean rehash){
        int newCapacity = newTable.length;
        for(Entry<K,V> e : table){
            while(null != e){
                Entry<K,v> next = e.next;               --(1)
                if(rehash){
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                //求出e在newTable中的新索引位置
                int i = indexFor(e.hash,newCapacity);
                //JDK1.7之前使用头插法重新将结点插入计算后的newTable中
                e.next = newTable[i]; //将newTable中的结点放入e中的next域中，头插法就在这里 --(2)
                newTable[i] = e; //将构造好的e放入newTable中
                e = next; // 继续下一个结点
            }
        }
    }
    ```

    - 在JDK1.8之前使用头插法，在多线程下形成循环列表：
        - 假设线程1执行到 (1) 处，CPU调度完成，假设e指向的值为3，next为7
        - 开始执行线程2，并且线程2的transfer函数执行完毕，已经完成索引
        - 继续执行线程1，此时继续往newTable中插入结点e，由于线程2已经完成扩容(并且由于是头插法，在索引为3的位置next和e的位置已经颠倒，即 7 --> 3)，当线程1插入结点e时，将会继续指向7(即 3 -->7)，此时循环链表形成
        - 图解网址：https://blog.csdn.net/Chisunhuang/article/details/79041656
    - 线程不安全原因：
        - 两个线程在put的时候，假如两个线程put的key的hash值相同，不论是采用头插法还是尾插法，假如A获取了插入位置X，但是还未插入，此时B也计算出待插入位置为X，则不论AB插入的先后顺序肯定有一个丢失。
        - 上述所讲的循环列表问题

