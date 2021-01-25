# Redis数据库

## 一、基础

### 1.1 定义

-  Redis是一个基于内存运行的高性能的Key-Value的NoSQL数据库，并支持持久化操作，是一个开源数据库。
- redis是单线程的，当一个客户端在查询时，另一个客户端的请求会阻塞

### 1.2 默认端口

- 6379

### 1.3 Redis和Jedis是什么

- Redis是服务端，Jedis是客户端，访问Redis需要Jedis的支持

### 1.4  RESP协议（Redis Serialization Protocol）

- 基于TCP的应用层协议RESP

- RESP底层采用的是TCP的连接方式，通过tcp进行数据传输，然后根据解析规则解析相应信息，完成交互。

- REST的格式

	- 数组以*号开头，后面为数组的个数
	- 以$开头，后面为字符串的长度，另一行为字符串

	```java
	*3   //3表示数组的长度为3
	    
	$3	 //4表示SET的长度为3，$表示下一行的字符串长度
	SET   //字符串内容
	$4
	deer
	$4
	deer    
	```

### 1.5 Redis的并发量

- SET：100000个请求2.12秒
- GET：100000个请求1.98秒

### 1.6 redis如何做压力测试？

- redis文件夹下有benchmark程序，可以测试

### 1.7 Redis配置文件

- redis.windows.conf更改其中的端口
- .\redis-server.exe .\xxx.conf            --服务器采用指定配置文件启动
- .\redis-cli.exe -h hostname -p port   --客户端采用指定主机名和端口启动

### 1.8 redis的分库原理

- 设计并编写redis的代理网关，将请求转发到不同的redis服务器

```java
//在端口6379、6380、6381开启三个redis服务器
//使用key值对3取余，将对应的语句代理值对应模值的redis服务器中
public class RedisTests {
    private static List<String> servers = new ArrayList<>();
    static{
        servers.add("127.0.0.1:6379");
        servers.add("127.0.0.1:6380");
        servers.add("127.0.0.1:6381");
    }
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(15000);
        System.out.println("代理网关正在监听15000端口....");
        Socket socket = null;
        while ((socket = serverSocket.accept()) != null) {
            try {
                while (true) {
                    System.out.println("一个链接");
                    InputStream inputStream = socket.getInputStream();
                    byte[] request = new byte[1024];
                    inputStream.read(request);
                    String req = new String(request);
                    System.out.println("收到请求: " + req);
                    String[] params = req.split("\r\n");
                    int keyLength = Integer.valueOf(params[4].length());
                    int mod = keyLength % servers.size();
                    System.out.println("根据Key的长度选择算法服务器：" + servers.get(mod));
                    String[] serverInfo = servers.get(mod).split(":");
                    Socket client = new Socket(serverInfo[0], Integer.valueOf(serverInfo[1]));
                    client.getOutputStream().write(request);
                    byte[] resp = new byte[1024];
                    client.getInputStream().read(resp);
                    client.close();
                    socket.getOutputStream().write(resp);
                    System.out.println("#############打印结束");
                    System.out.println();
                }
            } catch (IOException e) {
            }finally {
                socket.close();
            }
            serverSocket.close();
        }
    }
}
//Jedis的手写实现，客户端
public class JedisTest {
    private Socket client;
    public JedisTest(String ip, int port) throws IOException {
        client = new Socket(ip, port);
    }
    public static void main(String[] args) throws IOException {
        JedisTest jedisTest = new JedisTest("127.0.0.1",15000);
        System.out.println(jedisTest.set("name", "wl"));
        System.out.println(jedisTest.set("age","25"));
        System.out.println(jedisTest.set("length", "170"));
        System.out.println(jedisTest.get("name"));
        System.out.println(jedisTest.get("age"));
        System.out.println(jedisTest.get("length"));
    }
    private String set(String key, String value) throws IOException {
        StringBuffer sb = new StringBuffer();
        sb.append("*3")
                .append("\r\n")
                .append("$3")
                .append("\r\n")
                .append("SET")
                .append("\r\n")
                .append("$").append(key.length())
                .append("\r\n")
                .append(key)
                .append("\r\n")
                .append("$").append(value.length())
                .append("\r\n")
                .append(value)
                .append("\r\n");
        client.getOutputStream().write(sb.toString().getBytes());
        byte[] resp = new byte[1024];
        client.getInputStream().read(resp);
        return new String(resp);
    }
    private String get(String key) throws IOException {
        StringBuffer sb = new StringBuffer();
        sb.append("*2")
                .append("\r\n")
                .append("$3")
                .append("\r\n")
                .append("GET")
                .append("\r\n")
                .append("$").append(key.length())
                .append("\r\n")
                .append(key)
                .append("\r\n");
        client.getOutputStream().write(sb.toString().getBytes());
        byte[] resp = new byte[1024];
        client.getInputStream().read(resp);
        return new String(resp);
    }
}
```

### 1.9 redis的读写分离

- 读写分离的实现原理和分库类似
	- 首先根据RESP的规范可以得到当前请求是GET还是SET
	- 如果是SET，将请求转换至指定redis写服务器
	- 如果是GET，将请求随机发送至不同的redis读服务器
- 有一个问题，就是redis如何实现主从复制(就是在6379写入的数据，如何从6380读取出来)
	- 使用slaveof 127.0.01 6379(配置数据的来源，从哪个ip的哪个端口) 

### 1.10 redis如何实现高可用

- 高可用：就是master节点宕掉时，如何让从节点顶替上去
- 使用info replication可以知道master节点有多少个从节点 
- 实现原理
	- 首先不断检查Master节点的状态(不断的pingmaster的ip)
	- 当ping不通时，获取Master节点的所有从节点，从中选择一个作为主节点
	- Redis支持丰富的数据类型，string，list，set，hash，zset

### 1.11 Redis优势

- Redis是基于内存读写的，性能极高
- Redis支持数据的持久化，数据的备份
- Redis支持事务(MULTI和EXEC)，操作都是原子性的

### 1.12 redis和memcached的优势

- memcached所有值都是简单的字符串
- redis速度比memcached快
- redis支持数据的持久化

### 1.13 memcached和redis的区别

- 存储方式，memcached数据全部保存在内存中，断电丢失，redis支持数据的持久化
- 数据类型，memcached只支持简单的字符串，redis支持复杂的数据类型
- Memcached采用多线程模型，redis是单线程的，使用的是IO多路复用技术

### 1.14 一个字符串类型的值能存储最大容量是多少？

- 512M

## 二、redis数据类型

- 键的类型只能为字符串
- 值支持5种数据类型：字符串、列表、集合、散列表、有序集合
- String
	- set hello world   //将key为hello，value为world的键值对存入redis中
	- get hello
	- del hello
- LIST(值可以有重复值)
	- rpush list-key item1 item2  item3   //往list-key中插入列表，item1、item2和item3
	- lrange list-key 0 1    //列出key值为list-key的索引0到1所有value
	- lindex list-key 0    //列出key为list-key的索引为0的value值
	- lpop list-key   //取出列表的第一个value值
- SET(值不能重复)
	- sadd setkey item1 item2 item3  //给setkey增加集合元素item1、item2和item3
	- sismember setkey item1   //判断item1是否在setkey中
	- srem setkey item1   //从setkey中删除item1
	- smembers setkey   //获取setkey的所有value值
- HASH(值不能重复)
	- hset  hashkey subkey1 value1  //给key为hashkey的散列表增加subkey-value1的键值对
	- hget hashkey subkey1  //获取hashkey的键为subkey1的value值
	- hgetall  hashkey    //获取hashkey的所有键值对
	- hdel  hashkey  subkey1   //删除hashkey中键为subkey1的键值对
- ZSET
	- zadd zsetkey [incr] 100 item1 200 item2   //有序集合，每个value值有一个分数
	- zrem zsetkey  item2   //删除zsetkey中item2的元素
	- zrangebyscore  zsetkey  0  100 withscores  //查询zsetkey中分数为0-100的所有元素并显示分数
	- zrange zsetkey  0  1  withscores   //查询zsetkey中索引为0-1的所有元素

## 三、redis常用命令

- INCR：实现key对应的value值自增，因为redis中没有整数类型，是用字符串来表示的，如果字符串无法解析为整数报错。

	`INCR key          //将key对应的值+1`

- EXPIRE：设置key的过期时间

	·expire key 1000   //设置key的过期时间为1000秒

- PERSIST：清除key的过期时间

	`persist key            //清除key的过期时间，将key变成一个永久key`

- RENAME：重命名一个key

	` rename oldkey newkey           //将oldkey更名为newkey`

## 四、数据淘汰策略

- redis可以设置内存最大使用量，当内存使用量超出时，就会执行数据淘汰策略
	- 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰
	- 从已设置过期时间的数据记中挑选将要过期的数据淘汰
	- 从已设置过期时间的数据集中任意选择数据淘汰
	- 从所有数据集中挑选最近最少使用的数据淘汰
	- 从所有数据集中任意选择数据进行淘汰
	- 禁止驱逐数据
	- redis4.0引入了LFU策略(least frequently userd)
		- 从已设置过期时间的数据集中挑选访问频率最少的数据淘汰
		- 从所有数据集中挑选访问频率最少的数据淘汰

## 五、持久化

- Redis是内存型数据库，为了保证数据在断电后不会丢失，需要将内存中的数据持久化到硬盘中。

- RDB

	- 持久化文件位置：dir指定路径   dbfilename指定rdb文件名
	- redis database
	- 将某个时间点的所有数据都存放到硬盘上，但是会丢失最后一个创建快照之后的数据
	- 数据量很大的时候，保存快照的时间会很长
	- RDB持久化的时机
		- SHUTDOWN并且没有开启AOF
		- save 900 1   （900秒内有1次更改）
		- save 300 10    （300秒内有10次更改）
		- save 60 10000   （60秒内有10000次更改）
		- bgsave触发RDB持久化，会fork一个子进程，将数据存入一个临时文件，持久化完成之后替换上次持久化好的文件
		- save不会fork子进程，用父进程会持久化，会阻塞客户端
		- flush命令清空数据库，无意义 

- AOF

	- append only file

	- 在配置文件中设置appendonly on开启AOF持久化功能

	- 会产生一个appendonly.aof文件，用来记录所有的写命令用RESP的规范形式

	- 触发机制

		- everysec：每秒触发一次
		- always：同步持久化，每次发生数据变更时，立即记录到磁盘
		- no：等操作系统进行数据缓存同步到磁盘

	- AOF的重写机制(AOF中有些命令是没有用的，不需要执行)

		- 可以把auto-aof-rewrite-min-size 64mb的值改大
		- auto-aof-rewrite-percentage 100 当增长率达到100时，重写

	- 性能优化

		- 尽量减少AOF中rewrite的频率

		- 可以把auto-aof-rewrite-min-size 64mb的值改大
		- RDB可以用作后备用途，只开启save 900 1即可，

- RDB和AOF的比较

  - RDB丢失的数据比AOF的数据多
  - RDB适合大规模的数据恢复，数据的完整性和一致性不高
  - AOF在最恶劣的情况下，也只会丢失不到两秒的数据
  	- AOF有持续的IO，并且在rewrite的过程中由于该文件正在重写，所有新产生的数据无法写入该文件，会阻塞客户端的请求
  - 当服务器宕机时，通过redis-check-aof工具解决数据一致性问题

- 当AOF和RDB同时存在，先加载AOF

## 六 、事件

- redis是一个事件驱动程序
- 文件事件：Redis使用Reactor模式开发了自己的网络事件处理器，使用IO多路复用来同时监听多个套接字
	- 多个套接字
	- IO多路复用程序
	- 文件事件分派器
	- 事件处理器
- 时间事件
	- 定时事件：让一段程序在指定的时间之内执行一次
	- 周期性事件：让一段程序每隔指定时间执行一次
- 实现一个高性能的网络服务器
	- 多线程，弊端需要CPU上下文的切换
	- 单线程，IO多路复用技术
		- select：平时被阻塞，当有IO来数据时返回，但是无法得到是哪个IO来了数据，需要一个一个便利
			- bitmap默认大小是1024
			- 无法获知哪个IO有数据
		- poll
			- 解决了select的默认大小1024的缺点
		- epoll
			- 解决了select无法获取哪个IO流有数据的缺点
			- 使用重排的方式，当某个IO来数据时，将该IO放在第一个，返回的是来数据的IO数量
- 时间事件和文件事件的调度和执行
	- 服务器需要监听文件事件的套接字来得到待处理的文件事件，但是不能一直监听，所以监听时间应该根据距离现在最近的时间事件决定
		- 获取到达时间离当前时间最近的时间事件
		- 计算该事件还有多少秒到达
		- 在这个时间内监听文件事件
		- 处理已经到达的文件事件
		- 处理已经到达的时间事件

## 七、数据结构

- 字符串：SDS(simple dynamic string)
- list：采用双向链表
- 字典：就是有两个hash表，rehash时扩容另一个hash表，采用渐进式rehash，分多次完成
	- dict包含两个hash表，以及rehashidx，用来记录当前rehash的进度
	- 渐进式rehash会导致字典中的数据分散在两个hash表中，因此查找操作也会到两个表中查询
	- 每次对字典进行增删改查对会触发rehash
- 跳跃表：有序集合的底层实现
	- 查找：从上层指针开始查找，找到对应的区间之后再到下一层去查找
	- 插入：最左侧有一个负无穷节点，将节点插入最底层的链表，然后抛硬币决定是否往上层插入，如果是正面，则插入，继续抛，看是否继续往上层插入，遇到反面则终止，开始下一个节点的插入。
	- 删除：从顶层找到删除元素，并逐层找到每一层对应的节点并删除，当该层只有一个元素时，删除这一层

## 八、redis过期键的删除策略

- 定时删除：当键过期时，立即删除该键
- 惰性删除：当获取键时，先判断该键有没有过期，如果过期直接删除
- 定期删除：每隔一段时间就对数据库进行一次检查，删除里面过期的键。

## 九、Redis的同步机制

- 主从同步、从从同步
- 同步时，主服务器先做一次bgsave，并同时将期间的写命令记录到内存的buffer中，待完成后将rdb文件同步到从服务器，从服务器将rdb镜像加载到内存中，并通知主节点将期间修改的操作记录同步到从节点进行重放
- 同步时，从节点发送SYNC指令给主节点，主节点开始执行bgsave指令
- 指令传播：主节点数据修改后，会主动向从节点发送执行的写指令，从节点执行

## 十、Pipeline

- 可以将多次IO往返的时间缩减为一次，前提时pipeline执行的指令之间没有相关性	

## 十一、 Redis集群

- Sentinal哨兵：监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举中新的主服务器，实现主从节点的高可用
- 分布式解决方案，在单个redis发生故障和内存不足时，进行分片存储，解决单点故障和扩展性的问题
- 分片
	- 范围分片：
	- 哈希分片

## 十二、Redis如何设置密码

- config set requirepass 123456    （设置密码）
- auth 123456   （授权密码）

## 十三、Redis哈希槽

- Redis集群一共有16384个哈希槽
- 每个key通过对16384取模来决定放置在哪个槽

## 十四、 Redis的事务

- 事务是一个隔离的操作，事务在执行的过程中，不会被其他客户端发送来的命令请求所打断
- 事务是一个原子操作，事务中的命令要不全部执行，要不全部不执行
- 事务的命令：MULTI、EXEC、DISCARD、WATCH(使用了乐观锁CAS机制，首先存储旧值，执行事务之前，判断该值有没有变，如果变了则不执行该次事务)
- Redis的事务发生了exec之后的错误，也就是数据类型错误，则不影响其他命令的执行

## 十五、Redis的内存用完了会发生什么？

- 写命令会报错，读命令正常执行
- 配置redis的淘汰机制，当redis达到内存上限时会淘汰掉旧的内容。

## 十六、Redis的适用场景

- 会话缓存、计数器/排行榜、消息队列

## 十七、如何找出所有以某个固定前缀开头的key

- keys，但是因为redis是单线程的，会阻塞其他客户端请求
- 可以使用scan实现非阻塞的提取key列表，但是结果可能有重复
- sacn中的cursor之所以采用高位+1的操作是考虑了在sacn中如何发生扩容和缩容的情况，缩容的情况下会发生重复结果，当采用高位+1的操作时，当发生扩容时，从左到右的序列依旧是高位+1的顺序。

## 十八、redis缓存带来的问题

- 缓存穿透：查询数据库中一定不存在的key
- 缓存雪崩：在某一个时间端，缓存集中过期失效
- 缓存击穿：指一个key非常热点，大量的并发访问集中该点，当这个key在失效的瞬间，大量的并发直接击破缓存，直接请求数据库。

## 十九、Redis如何实现延迟队列

- 可以使用有序集合，将时间戳作为score

## 二十、Redis如何实现异步队列

- 使用list列表，rpush生产消息，lpop消费消息，当lpop没有消息时，适当sleep一会
- 如果不想sleep，可以使用brpush，blpop，在没有消息的时候会阻塞知道消息到来
- 可以使用publish/subscribe主题订阅者模式实现一次生产多次消费

## 二十一、Redis集群会有写操作丢失么？

- 过期的key被清理
- 最大内存不足，redis自动清理部分key以释放空间
- 主从节点切换时，切换期间会有数据的丢失

## 二十二、redis数据库

- redis使用整数标识数据库，默认有16个数据库
- 默认为0，可以使用select index来选择哪个数据库

## 二十三、redis的分布式锁

- 使用setnx获取锁，使用expire给key加一个过期时间
- 使用set指令可以同时获取锁以及给key设置过期时间可以防止在setnx和expire中间由于服务器宕机造成的锁无法释放的问题
- 判断value值释放与之前相等，解决当前线程可能释放其他线程加锁的问题

