# RocketMQ

## 一、下载与安装

- 系统：Centos 7.2.1511 

	` cat /etc/redhat-release  //可以查看centos版本号`

- RocketMQ的版本是4.3.2

### 1.1 下载

- 下载地址：http://rocketmq.apache.org/dowloading/releases/

### 1.2 安装

- 解压

	` unzip rocketmq-all-4.3.2-bin-release.zip`

- 修改配置文件

	- broker默认是生产环境配置，直接分配8G内存，一般学习测试的服务器没有那么大内存，所以修改该默认值
	- 修改bin目录下的runbroker.sh文件

	```shell
	#修改其中的JVM参数如下
	#JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
	JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn128m"
	```
	- 修改runserver.sh文件

	```shell
	#修改JVM参数如下
	#JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
	JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn64m -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=128m"
	
	```

- 启动nameserver

	```shell
	./mqnamesrv
	```

- 启动broker

	```shell
	./mqbroker -n 192.168.31.185:9876
	```

### 1.3 安装RocketMQ-Console

- 安装地址：https://github.com/StyleTang/rocketmq-console-ng
- 浏览器中：192.168.31.185：8082(默认端口为8080)

## 二、RocketMQ API调用

### 2.1 Maven依赖

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.3.0</version>
</dependency>
```

### 2.2 发送消息

#### 2.2.1 发送同步消息

```java
public class SyncProducerDemo {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException, UnsupportedEncodingException {
        DefaultMQProducer producer = new DefaultMQProducer("myGroup");

        producer.setNamesrvAddr("192.168.31.185:9876");
        producer.setSendMsgTimeout(10000);
        producer.start();

        String msg = "我的第一个消息";
        Message message = new Message("MyTopic","MyTag",msg.getBytes("UTF-8"));
        SendResult sendResult = producer.send(message);
        System.out.println("消息ID：" + sendResult.getMsgId());
        System.out.println("消息队列：" + sendResult.getMessageQueue());
        System.out.println("消息offset：" + sendResult.getQueueOffset());
        System.out.println("消息ID的offset：" + sendResult.getOffsetMsgId());
        System.out.println(sendResult);
        producer.shutdown();
    }
}

//output:
消息ID：C0A81F62355818B4AAC26A2B63340000
消息队列：MessageQueue [topic=MyTopic, brokerName=master, queueId=5]
消息offset：0
消息ID的offset：C0A81FB900002A9F000000000003D63A
SendResult [sendStatus=SEND_OK, msgId=C0A81F62355818B4AAC26A2B63340000, offsetMsgId=C0A81FB900002A9F000000000003D63A, messageQueue=MessageQueue [topic=MyTopic, brokerName=master, queueId=5], queueOffset=0]

```

#### 2.2.2 发送异步消息

```java
public class AsyncProducerDemo {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("myGroup");

        producer.setNamesrvAddr("192.168.31.185:9876");
        producer.setSendMsgTimeout(10000);
        producer.start();

        String msg = "我的第一个异步消息";
        Message message = new Message("MyTopic", msg.getBytes("UTF-8"));

        CountDownLatch cn = new CountDownLatch(1);
        producer.send(message, new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                System.out.println("消息发送成功" + sendResult);
                System.out.println("消息ID：" + sendResult.getMsgId());
                System.out.println("消息队列：" + sendResult.getMessageQueue());
                System.out.println("消息offset：" + sendResult.getQueueOffset());
                System.out.println("消息ID的offset：" + sendResult.getOffsetMsgId());
                cn.countDown();
            }

            @Override
            public void onException(Throwable throwable) {
                System.out.println("发送消息失败：" + throwable);
                cn.countDown();
            }
        });

        cn.await();
        producer.shutdown();
    }
}

//output:
消息发送成功SendResult [sendStatus=SEND_OK, msgId=C0A81F622BD818B4AAC26A41AE0C0000, 		             offsetMsgId=C0A81FB900002A9F000000000003D7A1, messageQueue=MessageQueue [topic=MyTopic, brokerName=master, queueId=7], queueOffset=0]
消息ID：C0A81F622BD818B4AAC26A41AE0C0000
消息队列：MessageQueue [topic=MyTopic, brokerName=master, queueId=7]
消息offset：0
消息ID的offset：C0A81FB900002A9F000000000003D7A1
```

### 2.3 Consumer消费消息

```java
public class ConsumerDemo {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConsumerGroup");
        consumer.setNamesrvAddr("192.168.31.185:9876");
        consumer.subscribe("MyTopic", "*");
        consumer.registerMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            System.out.println("消息接受成功" + list);
            try {
                for (MessageExt messageExt : list) {
                    System.out.println("消息：" + new String(messageExt.getBody(), "UTF-8"));
                }
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });
        consumer.start();
    }
}

//output:
消息接受成功[MessageExt [queueId=4, storeSize=178, queueOffset=0, sysFlag=0, bornTimestamp=1587454213530, bornHost=/192.168.31.98:7757, storeTimestamp=1587454213538, storeHost=/192.168.31.185:10911, msgId=C0A81FB900002A9F000000000003D903, commitLogOffset=252163, bodyCRC=841937032, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='MyTopic', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=1, CONSUME_START_TIME=1587454243789, UNIQ_KEY=C0A81F62306018B4AAC26A52D5990000, WAIT=true}, body=[-26, -120, -111, -25, -102, -124, -25, -84, -84, -28, -72, -128, -28, -72, -86, -27, -68, -126, -26, -83, -91, -26, -74, -120, -26, -127, -81, 50], transactionId='null'}]]
消息：我的第一个异步消息2
消息接受成功[MessageExt []]
消息：我的第一个异步消息
消息接受成功[MessageExt []]
消息：我的第一个异步消息3
```

### 2.4 创建Topic

```java
public class TopicDemo {
    public static void main(String[] args) throws MQClientException {
        DefaultMQProducer producer = new DefaultMQProducer("rocketMQ-topic");
        producer.setNamesrvAddr("192.168.31.185:9876");
        producer.start();
        //key为broker的名称，newTopic为topic的名称，queueName为队列数量
        producer.createTopic("master","MyTopic",8);
        System.out.println("Topic创建成功");
        producer.shutdown();
    }
}
```

### 2.5 消息过滤器

- 默认不开启自定义属性的过滤查询

```shell
# 在broker的配置文件中增加一行下面的语句
enablePropertyFilter=true
```

```java
//Consumer端
public class ConsumerDemo {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConsumerGroup");
        consumer.setNamesrvAddr("192.168.31.185:9876");
        //consumer.subscribe("MyTopic", "*");
        //订阅的时候采用选择器
        consumer.subscribe("MyTopic", MessageSelector.bySql("sex='男' and age >=18 and age <= 25"));
        consumer.registerMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            System.out.println("消息接受成功" + list);
            try {
                for (MessageExt messageExt : list) {
                    System.out.println("消息：" + new String(messageExt.getBody(), "UTF-8"));
                }
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });
        consumer.start();
    }
}
//Producer端，其余内容不变，设置了消息的属性

 String msg = "用户ID：1003";
Message message = new Message("MyTopic", msg.getBytes("UTF-8"));
message.putUserProperty("sex","男");
message.putUserProperty("age","23");

```

## 三、Producer详解

### 3.1 顺序消息

- Producer端口

```java
//通过模拟100条消息(每条消息由所属的订单ID)，相同的订单ID应该送往同一个MQ中
public class OrderProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer = new DefaultMQProducer("myGroup");
        producer.setNamesrvAddr("192.168.31.185:9876");
        producer.setSendMsgTimeout(10000);
        producer.start();

        for (int i = 0; i < 100; i++) {
            int orderId = i % 10;
            String msg = "order --> " + i + ", orderID " + orderId;
            Message message = new Message("MyTopic", "MyTag", msg.getBytes());
            SendResult sendResult = producer.send(message, (list, message1, o) -> {
                Integer id = (Integer) o;
                return list.get(id % list.size());
            }, orderId);
            System.out.println(sendResult);
        }
        producer.shutdown();
    }
}
```

- Consumer端

```java
//启动线程消费消息，线程的数量和MQ的数量一致，每一个线程负责一个MQ中消息的读取
public class OrderConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConsumerGroup");
        consumer.setNamesrvAddr("192.168.31.185:9876");

        consumer.subscribe("MyTopic","*");
        consumer.registerMessageListener((MessageListenerOrderly) (list, consumeOrderlyContext) -> {
            for (MessageExt messageExt : list) {
                try {
                    System.out.println(Thread.currentThread().getName() + ": " + messageExt.getQueueId() + " " + new String(messageExt.getBody(), "UTF-8"));
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
            return ConsumeOrderlyStatus.SUCCESS;
        });
        consumer.start();
    }
}
```

### 3.2 分布式事务

- 事务是指单个逻辑单元执行 的一系列操作要么全部执行，要么全部不执行

- 分布式事务：事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上，从本质上来讲，分布式事务就是为了保证不同数据库的数据一致性
	- 基于单个JVM，数据库分库分表了(包含多个数据库)
	- 基于多个JVM，服务拆分了(不跨数据库)，目前系统的设计都越来越服务化，包含的模块增多，也会设计到分布式事务，例如：用户的积分和优惠券可能由不同的团队管理，积分增加，优惠券应该减少，积分和优惠券就应该是不同的服务
	- 基于多个JVM，服务拆分了并且数据库也分库分表了
	
- MQ解决分布式事务的原理
	
	- A(存在DB操作)，B(存在DB操作)两方需要保证分布式事务一致性，通过引入中间层MQ，A和MQ保持事务的一致性(异常情况下通过MQ反查A接口实现check)，B和MQ保证事务一致(通过重试)，从而达到最终事务一致性，RocketMQ无法通过事务端发起服务器的回滚(例如订单系统已经支付成功，即使其他系统失败，订单也不会回滚)
	- RocketMQ解决分布式事务的流程图
	
	<img src="D:\软件\typora\笔记\img\RocketMQ解决分布式事务.jpg" style="zoom:50%;" />

```java
// 流程说明
1. 在扣款之前，先发送预备消息
2. 发送预备消息成功后，执行本地扣款事务
3. 扣款成功后，在发送确认消息(Commit或者Rollback)
4. 消费端可以看到确认消息，消费此消息，进行加钱
```

- RocketMQ的回查
	- 当本地事务执行成功，但是给MQ发送确认消息时失败时，MQ可以通过回调来判断本地事务的执行状态，如果成功就发送Commit，失败则Rollback
	- Map(事务ID，事务状态)中保存了事务的状态，可以通过事务ID直接在表中直接查询事务的状态

#### 3.2.1 事务生产者

```java
public class TransactionProducer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, InterruptedException {
        TransactionMQProducer producer = new TransactionMQProducer("TransactionProducer");
        producer.setNamesrvAddr("192.168.31.185:9876");
        producer.setSendMsgTimeout(10000);
        producer.setTransactionListener(new TransactionListenerImpl());
        producer.start();

        Message message = new Message("MyTopic", "用户A给用户B转账1000元".getBytes("UTF-8"));
        producer.sendMessageInTransaction(message, null);

        producer.shutdown();
    }
}
```

#### 3.2.2 事务回调实现类

```java
public class TransactionListenerImpl implements TransactionListener {

    private static Map<String, LocalTransactionState> STATE_MAP = new HashMap<>();
    //Producer通过回调调用TransactionListener的该方法
    @Override
    public LocalTransactionState executeLocalTransaction(Message message, Object o) {
        try {
            System.out.println("用户A账户减1000元");
            Thread.sleep(1000);
            System.out.println("用户B账户增加1000元");
            Thread.sleep(1000);
            STATE_MAP.put(message.getTransactionId(),
                    LocalTransactionState.COMMIT_MESSAGE);
            return LocalTransactionState.COMMIT_MESSAGE;
        } catch (InterruptedException e) {
            e.printStackTrace();
            STATE_MAP.put(message.getTransactionId(),
                    LocalTransactionState.ROLLBACK_MESSAGE);
        }
        return LocalTransactionState.ROLLBACK_MESSAGE;
    }

    //回查时通过该接口获取事务的执行状态
    @Override
    public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
        return STATE_MAP.get(messageExt.getTransactionId());
    }
}
```

#### 3.2.3 消费者

```java
public class TransactionConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConsumerGroup");
        consumer.setNamesrvAddr("192.168.31.185:9876");
        consumer.subscribe("MyTopic", "*");
        consumer.registerMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            for (MessageExt messageExt : list) {
                try {
                    System.out.println(new String(messageExt.getBody(), "UTF-8"));
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        consumer.start();
    }
}
```

