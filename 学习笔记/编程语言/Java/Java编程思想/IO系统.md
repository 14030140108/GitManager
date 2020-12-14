# IO系统

## 1.1 File类

1. FileNameFilter：文件名过滤器接口
    - File类中的list函数可以接受一个FileNameFilter的对象作为参数
    - list会对目录下的所有文件名调用该对象中的accept回调函数来判断是否匹配文件
    
2. net.mindview.util Jar包

    - 该包中TextFile类中包含常用的读写文件的操作

    - 该包中PPrint类为“灵巧打印机”类，会自动缩排list等容器中的元素，使其更易阅读

        ```java
        File file = new File(".");
        File[] files = file.listFiles();
        List<File> files1 = Arrays.asList(files);
        System.out.println(files1);
        //该类中的pfomat方法可以从Collection中产生格式化的String，
        //pprint方法调用pformat执行
        System.out.println(PPrint.pformat(files1));
        
        //output:
        [
          .\.idea
          .\Hello.txt
          .\JavaIODemo.iml
          .\lib
          .\pom.xml
          .\src
          .\target
          .\text.txt
        ]
        ```

3. 字符串数组的排序

    ```java
    Arrays.sort(list_Strings,String.CASE_INSENSITIVE_ORDER);
    ```

4. File中的函数

    ```java
    File file = new File("D:\\software");
    System.out.println(file.getName());    //output: software
    System.out.println(file.toString());	//output: D:\software
    System.out.println(file.getPath());		//output: D:\software
    System.out.println(file.getAbsolutePath());		//output: D:\software
    System.out.println(file.length());		//output: 4096
    System.out.println(file.getParent());	//output: D:\
    System.out.println(file.getParentFile());	//output: D:\
    ```

### 1.1.2 目录实用工具

1. 匹配当前指定目录下特定正则表达式的文件名或目录名

### 1.1.3 目录的检查及创建

1. idea中输出错误高亮

    `System.err.println("error!")`

2. renameTo()

    - 用来将一个文件移动或者重命名到由参数指定的另一个路径下面

## 1.2 输入和输出

1. 任何自inputStream或者Reader派生出来的类都含有read方法
2. 任何自outputStream或者Writer派生出来的类都含有write方法
3. 所有与输入有关的类都应该从InputStream继承，与输出有关的类都应该从outputStream继承

### 1.2.1 InputStream类型

<img src="D:\软件\typora\笔记\Java学习总结\img\InputStream.png" alt="InputStream子类继承结构图" style="zoom: 80%;" />

1. inputStream的作用是用来表示那些从不同数据源产生输入的类

    - 字节数组
    - String对象
    - 文件
    - 管道
    - 一个由其他种类的流组成的序列，以便我们可以将它们收集合并到一个流内
    - 其他数据源，如internet连接等

2. 每一种数据源都有相应的InputStream子类

    - ByteArrayInputStream类( 内存缓冲区(输入流)   <--- 字节数组 <====  内存缓冲区(输出流))

        - 输入流就是把输入数据转化为流，在流中可以进行read等提供的基本操作，对输入数据进行读取，切片等，就是有能力接受数据的接收端对象，实现手段就是在内存中开辟一个字节数组，用来保存读入的数据
        - 字节数组输入流就是将字节数组转化为流，并且可以进行一系列基本操作

         [ByteArrayInputStream类知识图](img\ByteArrayInputStream类.xmind) 

    - StringBufferInputStream类
      
        - 将String转换成InputStream
    - FileInputStream类
      
        - 用于从文件中读取信息
    - PipedInputStream类
      
        - 产生用于写入相关PipedOutputStream的数据，实现“管道化”概念
    - SequenceInputStream类
      
        - 实现将两个或多个InputStream对象转换成单一的InputStream
    - FilterInputStream类
      
        - 抽象类，作为“装饰器”的接口，其中装饰器为其他的InputStream类提供有用功能
        - DataInputStream类
            - 可以读取基本数据类型
            - 构造器中需要传入一个InputStream对象
        - BufferedInputStream类
            - 使用它可以防止每次读取时都得进行实际写操作，代表使用缓冲区
        
        - LineNumberInputStream类
            - 跟踪输入流中的行号
        - PushbackInputStream类
            - 具有能弹出一个字节的缓冲区
        
        

### 1.2.2 OutputStream类型

<img src="D:\软件\typora\笔记\Java学习总结\img\OutputStream.png" alt="OutputStream子类继承结构图" style="zoom:80%;" />

- ByteArrayOutputStream类(所有送往流的数据都放置在此缓冲区)
    - 输出流就是有能力产生数据的数据源对象，实现手段就是在内存缓冲区中开辟了一个字节数组用来保存流中的数据
    - s字节数组输出流就是可以自己产生数据，创造数据，并把这些数据生成为字节数组
    - write()
        - 将一个int型整数写入输出流中
    - write(byte[],off,length)
        - 将byte数组从off开始写入length长度到输出流
    - reset()
        - 与字节数组输入流一致
    - writeTo()
        - 将输出流写入另一个输出流中
    
- FileOutputStream类
  
    - 用于将信息写入文件
    
- PipedOutputStream类
  
    - 任何写入其中的信息都会自动作为相关PipedInputStream的输出，实现管道化的概念
    
- FilterOutputStream类
  
    - 抽象类，作为装饰器类的接口
    - DataOutputStream类
        - 按照可移植方式向流中写入基本类型数据
        - 构造器中需要传入要给outputStream对象
    
    - PrintStream类
        - 用于产生格式化输出
    - BufferedOutputStream类
        - 使用它以避免每次发送数据时都要进行实际的写操作(使用缓冲区技术)

### 1.2.3 LineNumberInputStream和LineNumberReader类

- 这两个类增加了两个方法，可以得到当前的行数以及设置行数(并不能改变当前流的读取位置)
    - getLineNumber
    - serLineNumber：仅仅影响getLineNumber得到的值

## 1.3 Reader和Writer

<img src="D:\软件\typora\笔记\Java学习总结\img\Reader.png" alt="Reader子类继承结构图" style="zoom:80%;"/>

<img src="D:\软件\typora\笔记\Java学习总结\img\Writer.png" alt="Writer子类继承结构图" style="zoom:80%;"/>

1. InputStreamReader可以将InputStream转化为Reader
2. OutputStreamWriter可以将OutputStream转化为Writer
3. BufferedReader和BufferedWriter一般经常使用，可以提高速度，使用缓冲区技术，在发送数据给流时不需要实际的写操作(和BufferedInputStream和BufferedOutputStream功能一致)

![Stream流常用类型(面向字节)](D:\software\git\repository\GitManager\Java学习总结\Java编程思想\img\Stream流常用类型(面向字节).png)

[IO流继承关系图](https://blog.csdn.net/qq_37969433/article/details/79886991)

- 在上述网址中的图中，FilterOutputStream的子类还有一个PrintStream，在Writer子类中还有一个PrintWriter

4. 图中在流中缺少 InputStream和OutputStream以及PrintStream，右边缺少InputStreamReader和OutputStreamWriter以及PrintWriter

## 1.4 RandomAccessFile类

1. 该类是可以在文件中移动及修改记录的位置，支持搜寻方法，只适用于文件

## 1.5 标准I/O

1. 标准输入System.in的类型为inputStream
2. 标准输出和错误System.out和System.err的类型均为printStream
3. 标准I/O重定向：对标准输入、输出和错误进行重定向

## 1.6 进程控制

1. 使用java程序执行其他程序，使用mindview中的OSExecute类

    ```java
    public class OSExecute {
        public OSExecute() {
        }
    
        public static void command(String command) {
            boolean err = false;
    
            try {
                Process process = (new ProcessBuilder(command.split(" "))).start();
                BufferedReader results = new BufferedReader(new InputStreamReader(process.getInputStream()));
    
                String s;
                while((s = results.readLine()) != null) {
                    System.out.println(s);
                }
    
                for(BufferedReader errors = new BufferedReader(new InputStreamReader(process.getErrorStream())); (s = errors.readLine()) != null; err = true) {
                    System.err.println(s);
                }
            } catch (Exception var6) {
                if (command.startsWith("CMD /C")) {
                    throw new RuntimeException(var6);
                }
    
                command("CMD /C " + command);
            }
    
            if (err) {
                throw new OSExecuteException("Errors executing " + command);
            }
        }
    }
    
    //OSExecuteDemo.java
    public class OSExecuteDemo {
        public static void main(String[] args) {
            OSExecute.command("javap App");
        }
    }
    ```

## 1.7 新I/O

### 1.7.1 ByteBuffer类

1. 该类中的属性

    - Position ：表示当前位置的指针
    - Capacity：缓冲区最大容量
    - limit：当前最大使用量，或者说时有效数据的EOF位置，不包含该值对应的值
    - mark：标记

2. 该类中的函数

    - clear函数：将position指针指向0，limit指向容量
    - flip函数：将limit指向当前位置，position指向0
    - rewind函数：position指针指向0
    - compact函数：压缩数据， 比如当前EOF是6，当前指针指向2 （即0,1的数据已经写出了，没用了）,那么compact方法将把2,3,4,5的数据挪到0,1,2,3的位置， 然后指针指向4的位置。这样的意思是，从4的位置接着再写入数据。 

3. 在JDK1.4中引入了nio包，其目的是为了提高速度，，速度的提高所使用的结构更接近与操作系统执行IO的方式：通道(channel)和缓冲器(ByteBuffer)

    ```java
    public class GetChannel {
    
        public static final int BSIZE = 1024;
        public static void main(String[] args) throws IOException {
            FileChannel fc = new FileOutputStream("data.txt").getChannel();
            fc.write(ByteBuffer.wrap("some text ".getBytes()));
            fc.close();
    
            fc = new RandomAccessFile("data.txt", "rw").getChannel();
            fc.position(fc.size());
            fc.write(ByteBuffer.wrap("some more".getBytes()));
            fc.close();
    
            fc = new FileInputStream("data.txt").getChannel();
            ByteBuffer buff = ByteBuffer.allocate(BSIZE);
            fc.read(buff);
            buff.flip();
            while (buff.hasRemaining()) {
                System.out.print((char) buff.get());
            }
        }
    }
    
    /* FileChannel的函数
    * 1. read函数: 从通道中读取字节数据到ByteBuffer中,每次读完之后，如果要写需要调用ByteBuffer的flip函数,
    *			  如果之后继续读，需要清空缓冲区clear函数
    * 2. write函数: 将ByteBuffer中的字节数据写入通道中
    */
    
    /* 实现两个文件的复制有两种方式：
    * 1.使用read和write实现文件之间的读写
    * 2.使用transferTo或者transFrom函数
    */
    
    
    public class ChannelCopy {
        private static final int BSIZE = 1024;
        
        //使用read和write函数实现文件内容的拷贝
        public static void main(String[] args) throws IOException {
            FileChannel in = new FileInputStream("data.txt").getChannel();
            FileChannel out = new FileOutputStream("data1.txt").getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
            //当返回-1时，表示已经达到了输入的末尾
            while (in.read(buffer) != -1) {
                buffer.flip();
                out.write(buffer);
                buffer.clear();
            }
        }
        
        //使用transferTo或者transferForm函数实现文件内容的拷贝
         public static void main(String[] args) throws IOException {
            FileChannel in = new FileInputStream("data.txt").getChannel();
            FileChannel out = new FileOutputStream("data1.txt").getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
            in.transferTo(0, in.size(), out);
            //Or: 
            //out.transferFrom(in, 0, in.size());
        }
    }
    
    
    
    
    ```

### 1.7.2 转换数据

1. java.nio.CharBuffer类

    - CharBuffer类定义缓冲区中可以每次读取一个字符的数据

    - 将ByteBuffer转换为CharBuffer时,使用ByteBuffer中的asCharBuffer函数

        - 转换的时候，要不在输入时对其进行编码，要不输出时对其解码

        - 直接使用CharBuffer输入数据

            ```java
            public class BufferToText {
                public static void main(String[] args) throws IOException {
                    //读取data.txt文件
                    FileChannel fc = new FileInputStream("data.txt").getChannel();
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    fc.read(buffer);
                    //没有输出结果
                    System.out.println(buffer.asCharBuffer());
            		
                    //重置buffer，对buffer数据进行解码
                    buffer.rewind();
                    String encoding = System.getProperty("file.encoding");
                    System.out.println(Charset.forName(encoding).decode(buffer));
            
                    //将buffer输入的数据进行编码
                    fc = new FileOutputStream("data.txt").getChannel();
                    fc.write(ByteBuffer.wrap("some more".getBytes("UTF-16BE")));
                    fc.close();
                    fc = new FileInputStream("data.txt").getChannel();
                    buffer.clear();
                    fc.read(buffer);
                    buffer.flip();
                    //正确输出结果
                    System.out.println(buffer.asCharBuffer());
            
                    //使用CharBuffer存放数据
                    fc = new FileOutputStream("data.txt").getChannel();
                    buffer = ByteBuffer.allocate(24);
                    buffer.asCharBuffer().put("some some");
                    fc.write(buffer);
                    fc.close();
                    fc = new FileInputStream("data.txt").getChannel();
                    buffer.clear();
                    fc.read(buffer);
                    buffer.flip();
                    System.out.println(buffer.asCharBuffer());
                }
            }
            
            //output:
                                                          
            Hello World                               
            some more
            some some   
            ```

        2. 输出charset类中所有编码的别名

            ```java
            public class AvailableCharsets {
                public static void main(String[] args) {
               //sortedMap是一个有序的Map，默认的排序是根据key值进行升序排序，可以重写comparator方式来根据value进行排序
                    SortedMap<String, Charset> charsets = Charset.availableCharsets();
                    Iterator<String> it = charsets.keySet().iterator();
                    while (it.hasNext()) {
                        String csName = it.next();
                        Iterator aliases = charsets.get(csName).aliases().iterator();
                        if (aliases.hasNext()) {
                            System.out.print(csName + ": ");
                        }
                        while (aliases.hasNext()) {
                            System.out.print(aliases.next());
                            if (aliases.hasNext()) {
                                System.out.print(",");
                            }
                        }
                        System.out.println();
                    }
                }
            }
            ```

### 1.7.3 获取基本类型

```java
public class GetData {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        buffer.asCharBuffer().put("Hello World!");
        char c;
        while ((c = buffer.getChar()) != 0) {
            System.out.print(c + " ");
        }

        System.out.println();
        buffer.rewind();

        buffer.asShortBuffer().put((short) 471142);
        System.out.println(buffer.getShort());

        buffer.rewind();
        buffer.asFloatBuffer().put(471142F);
        System.out.println(buffer.getFloat());

        buffer.rewind();
        buffer.asDoubleBuffer().put(471142D);
        System.out.println(buffer.getDouble());
    }
}
```

### 1.7.4 视图缓冲器

1. 视图缓冲器可以让我们通过某个特定的基本数据类型的视窗查看其底层的ByteBuffer
2. 可以在同一个ByteBuffer中建立不同的视图缓冲器，将同一个字节序列翻译为不同的数据类型(P558页)

```java
public class DoubleBufferDemo {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        DoubleBuffer db = buffer.asDoubleBuffer();
        db.put(new double[]{1.2, 2.3, 3.4, 4.5, 5.6, 6.7, 7.8, 8.9});
        System.out.println(db.get(3));
        db.put(3, 10.110);
        db.flip();
        while (db.hasRemaining()) {
            System.out.print(db.position() + "->" + db.get() + " , ");
        }
    }
}

//output:
4.5
0->1.2 , 1->2.3 , 2->3.4 , 3->10.11 , 4->5.6 , 5->6.7 , 6->7.8 , 7->8.9
```

3. 字节存放次序

    - 高位优先(类似于大端存储)
    - 低位有限(类似于低位存储)

    ```java
    public class Endians {
        public static void main(String[] args) {
            ByteBuffer buffer = ByteBuffer.wrap(new byte[12]);
            buffer.asCharBuffer().put("abcdef");
            System.out.println(Arrays.toString(buffer.array()));
            buffer.rewind();
            buffer.order(ByteOrder.BIG_ENDIAN);
            buffer.asCharBuffer().put("abcdef");
            System.out.println(Arrays.toString(buffer.array()));
            buffer.rewind();
            buffer.order(ByteOrder.LITTLE_ENDIAN);
            buffer.asCharBuffer().put("abcdef");
            System.out.println(Arrays.toString(buffer.array()));
        }
    }
    
    //output:
    [0, 97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102]
    [0, 97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102]
    [97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102, 0]
    ```

### 1.7.5 用缓冲器操纵数据

1. ByteBuffer是将数据移进移出通道的唯一方式
2. 我们只能创建一个独立的基本类型缓冲器，或者使用as方法从ByteBuffer中获得，也就是说，我们不能把基本类型的缓冲器转换成ByteBuffer。然而，我们可以通过视图缓冲器将基本类型数据移进移出ByteBuffer中。

### 1.7.6 缓冲器的细节

 ```java
public class Endians {
    public static void main(String[] args) {
        char[] data = "UsingBuffers".toCharArray();
        ByteBuffer buffer = ByteBuffer.allocate(data.length * 2);
        CharBuffer charBuffer = buffer.asCharBuffer();
        charBuffer.put(data);
        System.out.println(charBuffer.rewind());
        symmetricScramble(charBuffer);
        System.out.println(charBuffer.rewind());
        symmetricScramble(charBuffer);
        System.out.println(charBuffer.rewind());
    }
	//交换两个相邻字符
    private static void symmetricScramble(CharBuffer buffer) {
        while (buffer.hasRemaining()) {
            buffer.mark();
            char c1 = buffer.get();
            char c2 = buffer.get();
            buffer.reset();
            buffer.put(c2).put(c1);
        }
    }
}

//output:
UsingBuffers
sUniBgFuefsr
UsingBuffers
 ```

### 1.7.7 内存映射文件

1. 内存映射文件允许我们创建和修改那些因为太大而不能放入内存的文件

    ```java
    public class LargeMappedFiles {
        static int length = 0x7ffffff;   //128M
        public static void main(String[] args) throws IOException {
            MappedByteBuffer out = new RandomAccessFile("test.dat", "rw").getChannel()
                    .map(FileChannel.MapMode.READ_WRITE, 0, length);
            for (int i = 0; i < length; i++) {
                out.put((byte) 'x');
            }
            System.out.println("Finished writing");
            for (int i = length / 2; i < length / 2 + 6; i++) {
                System.out.println((char) out.get(i));
            }
        }
    }
    
    //output:
    Finished writing
    x
    x
    x
    x
    x
    x
    ```

    2. 尽管旧IO流用nio实现后性能有所提高，但是映射文件访问往往可以更加显著的加快速度
    
    ```java
    public class MappedIO {
        private static int numOfInts = 4000000;
        private static int numOfUbuffInes = 200000;
        //静态内部抽象类
        private abstract static class Tester {
            private String name;
            public Tester(String name) {
                this.name = name;
            }
            public void runTest() {
                System.out.print(name + ": ");
                try {
                    long start = System.nanoTime();
                    test();
                    double duration = System.nanoTime() - start;
                    System.out.format("%.2f\n", duration / 1.0e9);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            public abstract void test() throws IOException;
        }
        //静态内部抽象子类数组
        private static Tester[] tests = {
                new Tester("Stream Write") {
                    @Override
                    public void test() throws IOException {
                        DataOutputStream out = new DataOutputStream(
                                new BufferedOutputStream(
                                        new FileOutputStream("temp.tmp")));
                        for (int i = 0; i < numOfInts; i++) {
                            out.writeInt(i);
                        }
                        out.close();
                    }
                },
                new Tester("Mapped Write") {
                    @Override
                    public void test() throws IOException {
                        FileChannel fc = new RandomAccessFile(new File("temp.tmp"), "rw").getChannel();
                        IntBuffer ib = fc.map(FileChannel.MapMode.READ_WRITE, 0, fc.size()).asIntBuffer();
                        for (int i = 0; i < numOfInts; i++) {
                            ib.put(i);
                        }
                        fc.close();
                    }
                },
                new Tester("Stream Read") {
                    @Override
                    public void test() throws IOException {
                        DataInputStream in = new DataInputStream(
                                new BufferedInputStream(
                                        new FileInputStream("temp.tmp")));
                        for (int i = 0; i < numOfInts; i++) {
                            in.readInt();
                        }
                        in.close();
                    }
                },
                new Tester("Mapped Read") {
                    @Override
                    public void test() throws IOException {
                        FileChannel fc = new RandomAccessFile(new File("temp.tmp"), "r").getChannel();
                        IntBuffer ib = fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size()).asIntBuffer();
                        while (ib.hasRemaining()) {
                            ib.get();
                        }
                        fc.close();
                    }
                },
                new Tester("Stream Read/Write") {
                    @Override
                    public void test() throws IOException {
                        RandomAccessFile raf = new RandomAccessFile("temp.tmp", "rw");
                        raf.writeInt(1);
                        for (int i = 0; i < numOfUbuffInes; i++) {
                            raf.seek(raf.length() - 4);
                            raf.writeInt(raf.readInt());
                        }
                        raf.close();
                    }
                },
                new Tester("Mapped Read/Write") {
                    @Override
                    public void test() throws IOException {
                        FileChannel fc = new RandomAccessFile("temp.tmp", "rw").getChannel();
                        IntBuffer ib = fc.map(FileChannel.MapMode.READ_WRITE, 0, fc.size()).asIntBuffer();
                        ib.put(1);
                        for (int i = 1; i < numOfUbuffInes; i++) {
                            ib.put(ib.get(i - 1));
                        }
                    }
                }
        };
        public static void main(String[] args) {
            for (Tester test : tests) {
                test.runTest();
            }
        }
    }
    
    //output:
    Stream Write: 0.31
    Mapped Write: 0.03
    Stream Read: 0.47
    Mapped Read: 0.01
    Stream Read/Write: 3.89
    Mapped Read/Write: 0.01
        
    //从结果中可以看出，使用映射文件访问可以显著的提高IO访问的速度
    ```

### 1.7.8 文件加锁

1. 文件加锁demo

```java
//JDK1.4中引入了文件加锁机制，允许我们同步访问某个作为共享资源的文件
public class FileLocking {
    public static void main(String[] args) throws IOException, InterruptedException {
        RandomAccessFile fos = new RandomAccessFile("file.txt","rw");
        FileLock fl = fos.getChannel().tryLock();
        if (fl != null) {
            System.out.println("Locked File");
            TimeUnit.MILLISECONDS.sleep(100);
            System.out.println("Released File");
        }
        fos.close();
    }
}
```

2.  对映射文件的部分加锁

```java
public class LockingMappedFiles {
    static final int LENGTH = 0x7ffffff;
    static FileChannel fc;

    public static void main(String[] args) throws IOException {
        fc = new RandomAccessFile("test.dat", "rw").getChannel();
        MappedByteBuffer out = fc.map(FileChannel.MapMode.READ_WRITE, 0, LENGTH);
        for (int i = 0; i < LENGTH; i++) {
            out.put((byte) 'x');
        }
        new LockAndModify(out, 0, 0 + LENGTH / 3);
        new LockAndModify(out, LENGTH / 2, LENGTH / 2 + LENGTH / 4);
    }
    private static class LockAndModify extends Thread {
        private ByteBuffer buffer;
        private int start, end;

        LockAndModify(ByteBuffer mbb, int start, int end) {
            this.start = start;
            this.end = end;
            mbb.limit(end);
            mbb.position(start);
            buffer = mbb.slice();
            start();
        }
        public void run() {
            try {
                FileLock fl = fc.lock(start, end, false);
                System.out.println("Locked: " + start + " to " + end);
                while (buffer.position() < buffer.limit() - 1) {
                    buffer.put((byte) (buffer.get() + 1));
                }
                fl.release();
                System.out.println("Released: " + start + " to " + end);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

//output:
Locked: 67108863 to 100663294
Locked: 0 to 44739242
Released: 67108863 to 100663294
Released: 0 to 44739242
```

### 1.7.9 压缩

1. 利用GZIP进行简单压缩

```java
public class GZIPcompress {
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(
                new FileReader("test.txt"));
        BufferedOutputStream out = new BufferedOutputStream(
                new GZIPOutputStream(
                        new FileOutputStream("text.gz")));
        System.out.println("writing file");
        int c;
        while ((c = in.read()) != -1) {
            out.write(c);
        }
        in.close();
        out.close();
        BufferedReader in2 = new BufferedReader(
                new InputStreamReader(
                        new GZIPInputStream(
                                new FileInputStream("text.gz"))));
        System.out.println("reading file");
        String s;
        while ((s = in2.readLine()) != null) {
            System.out.println(s);
        }
    }
}
```

2. Java档案文件

    ```java
    //将当前目录下的所有txt文件压缩为jar包
    jar cvf myApp.jar *.txt 
    ```

### 1.7.10 对象序列化

```java
public class Data implements Serializable {
    private int n;
    public Data(int n) { this.n = n; }
    public String toString() { return Integer.toString(n); }
}
class Worm implements Serializable {
    private static Random rand = new Random(47);
    private Data[] d = {
            new Data(rand.nextInt(10)),
            new Data(rand.nextInt(10)),
            new Data(rand.nextInt(10))
    };
    private Worm next;
    private char c;
    public Worm(int i, char x) {
        System.out.println("Worm constructor:" + i);
        c = x;
        if (--i > 0) {
            next = new Worm(i, (char) (x + 1));
        }
    }
    public Worm() {
        System.out.println("Default constructor");
    }
    public String toString() {
        StringBuilder result = new StringBuilder(":");
        result.append(c);
        result.append("(");
        for (Data dat : d) {
            result.append(dat);
        }
        result.append(")");
        if (next != null)
            result.append(next);
        return result.toString();
    }
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Worm w = new Worm(6, 'a');
        //输出对象链表的字符串值
        System.out.println("w =" + w);
       	//使用ObjectOutputStream对对象进行序列化到文件中
        ObjectOutputStream out = new ObjectOutputStream(
                new FileOutputStream("worm.out"));
        out.writeObject("Worm storage\n");
        out.writeObject(w);
        out.close();
	    //使用ObjectInputStream对文件中的值进行反序列化为对象
        ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("worm.out"));
        String s = (String) in.readObject();
        Worm w2 = (Worm) in.readObject();
        System.out.println(s + "w2 = " + w2);
        //将对象序列化至字节数组中
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        ObjectOutputStream out2 = new ObjectOutputStream(bout);
        out2.writeObject("Worm storage\n");
        out2.writeObject(w);
        out2.flush();
	    //从字节数组中反序列化为对象
        ObjectInputStream in2 = new ObjectInputStream(
                new ByteArrayInputStream(bout.toByteArray()));
        String s1 = (String) in2.readObject();
        Worm w3 = (Worm) in2.readObject();
        System.out.println(s1 + "w3 = " + w3);
    }
}
//output:
Worm constructor:6
Worm constructor:5
Worm constructor:4
Worm constructor:3
Worm constructor:2
Worm constructor:1
w =:a(853):b(119):c(802):d(788):e(199):f(881)
Worm storage
w2 = :a(853):b(119):c(802):d(788):e(199):f(881)
Worm storage
w3 = :a(853):b(119):c(802):d(788):e(199):f(881)
```

1. 自己写得对象的序列化

    ```java
    public class SerialTestDemo1 implements Serializable {
        private SerialTestDemo2[] test = {
                new SerialTestDemo2("zhangsan1",11),
                new SerialTestDemo2("zhangsan2",11),
                new SerialTestDemo2("zhangsan3",11)};
        public SerialTestDemo1() {
            System.out.println("SerialTestDemo1 constructor");
        }
        public String toString() {
            StringBuilder sb = new StringBuilder("SerialTestDemo1:");
            for (SerialTestDemo2 temp : test)
                sb.append(temp);
            return sb.toString();
        }
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            SerialTestDemo1 std = new SerialTestDemo1();
            ObjectOutputStream out = new ObjectOutputStream(
                    new FileOutputStream("output.txt"));
            out.writeObject(std);
            ObjectInputStream in = new ObjectInputStream(
                    new FileInputStream("output.txt"));
            SerialTestDemo1 s = (SerialTestDemo1) in.readObject();
            System.out.println(s);
        }
    }
    
    class SerialTestDemo2 implements Serializable {
        private String name;
        private int age;
        public SerialTestDemo2(String name,int age) {
            this.name = name;
            this.age = age;
        }
        @Override
        public String toString() {
            final StringBuilder sb = new StringBuilder("{");
            sb.append("\"name\":\"")
                    .append(name).append('\"');
            sb.append(",\"age\":")
                    .append(age);
            sb.append('}');
            return sb.toString();
        }
    }
    
    //output:
    SerialTestDemo1 constructor
    SerialTestDemo1:{"name":"zhangsan1","age":11}{"name":"zhangsan2","age":11}{"name":"zhangsan3","age":11}
    ```

2. static字段也无法序列化

### 1.7.11 寻找类

1. 将一个对象从它的序列化状态中恢复出来，除了序列化之后的内容之外，还需要序列化对象的class文件，java虚拟机必须可以通过类路径找到该class文件。

### 1.7.12 序列化的控制

1. 实现Externalizable接口可以控制序列化生成时的信息

    ```java
    public class Blip3 implements Externalizable {
        private int i;
        private String s;
        public Blip3() {
            System.out.println("Blip3 Constructor");
            System.out.println(s);
            System.out.println(i);
        }
        public Blip3(String x, int a) {
            System.out.println("Blip3(String x,int a)");
            s = x;
            i = a;
        }
        public String toString() {
            return s + i;
        }
        @Override
        public void writeExternal(ObjectOutput out) throws IOException {
            System.out.println("Blip3.writeExternal");
            out.writeObject(s);
            out.writeInt(i);
        }
        @Override
        public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
            System.out.println("Blip3.readExternal");
            s = (String) in.readObject();
            i = in.readInt();
        }
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            Blip3 b3 = new Blip3("A String ", 47);
            System.out.println(b3);
            ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream(new File("Blip3.out")));
            o.writeObject(b3);
            o.close();
            ObjectInputStream in = new ObjectInputStream(new FileInputStream(new File("Blip3.out")));
            //使用Externalizable反序列化是，会调用对象的默认构造函数
            b3 =(Blip3) in.readObject();
            System.out.println(b3);
        }
    }
    
    //output:
    Blip3(String x,int a)
    A String 47
    Blip3.writeExternal
    Blip3 Constructor
    null
    0
    Blip3.readExternal
    A String 47
    ```

2. 使用Externalizable序列化对象过程中，需要实现writeExternal和readExternal两个函数，序列化对象信息的存储和恢复操作需要通过这两个函数实现。可以显式的序列化自己想要的信息

3. transient(瞬时关键字)

    - 如果对象中有些敏感部分不想被序列化，例如密码，可以将类实现Externalizable，显式的序列化
    - 如果使用的Serializable实现序列化，可以使用transient关闭某个字段的序列化
    - 因为Externalizable默认不保存任何字段，所以transient关键字只能和Serializable对象一起使用

    ```java
    public class Login implements Serializable {
        private Date date = new Date();
        private String username;
        //密码使用transient修饰，不会被序列化
        private transient String password;
        public Login(String username, String password) {
            this.username = username;
            this.password = password;
        }
        @Override
        public String toString() {
            final StringBuilder sb = new StringBuilder("{");
            sb.append("\"date\":\"")
                    .append(date).append('\"');
            sb.append(",\"username\":\"")
                    .append(username).append('\"');
            sb.append(",\"password\":\"")
                    .append(password).append('\"');
            sb.append('}');
            return sb.toString();
        }
    
        public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
            Login a = new Login("root", "123456");
            System.out.println("Login a = " + a);
            ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream(new File("Login.out")));
            o.writeObject(a);
            o.close();
            TimeUnit.SECONDS.sleep(1);
            ObjectInputStream in = new ObjectInputStream(new FileInputStream(new File("Login.out")));
            System.out.println("Recovering object at " + new Date());
            a = (Login) in.readObject();
            System.out.println("Login a = " + a);
        }
    }
    
    //output:
    Login a = {"date":"Fri Nov 15 11:12:17 CST 2019","username":"root","password":"123456"}
    Recovering object at Fri Nov 15 11:12:18 CST 2019
    Login a = {"date":"Fri Nov 15 11:12:17 CST 2019","username":"root","password":"null"}
    ```

### 1.7.13 XML

1. 使用XOM jar包中的方法进行对象和XML之间的转换

    ```java
    public class Person {
        private String first, last;
        public Person(String first, String last) {
            this.first = first;
            this.last = last;
        }
    	//将Person对象转换为XML文件
        public Element getXML() {
            Element person = new Element("person");
            Element firstName = new Element("first");
            firstName.appendChild(first);
            person.appendChild(firstName);
            Element lastName = new Element("last");
            lastName.appendChild(last);
            person.appendChild(lastName);
            return person;
        }
    	//将XML中的ELement转换为Person对象
        public Person(Element person) {
            first = person.getFirstChildElement("first").getValue();
            last = person.getFirstChildElement("last").getValue();
        }
        public String toString() {
            return first + " " + last;
        }
    	//使用Serializer对象可以使转换的XML更具有可读性
        public static void format(OutputStream os, Document doc) throws IOException {
            Serializer serializer = new Serializer(os, "ISO-8859-1");
            //设置缩进为4个字符
            serializer.setIndent(4);
            serializer.setMaxLength(60);
            serializer.write(doc);
            serializer.flush();
        }
        public static void main(String[] args) throws IOException {
            List<Person> people = Arrays.asList(
                    new Person("Dr.Bunsen", "Honeydew"),
                    new Person("Gonzo", "The Great"),
                    new Person("Phillip J.", "Fry"));
            System.out.println(people);
            Element root = new Element("person");
            for (Person person : people) {
                root.appendChild(person.getXML());
            }
            Document doc = new Document(root);
            format(System.out,doc);
            format(new BufferedOutputStream(new FileOutputStream("People.xml")),doc);
        }
    }
    
    //把People.xml文件转换为Person对象
    public class People extends ArrayList<Person> {
        public People(File fileName) throws ParsingException, IOException {
            Document doc = new Builder().build(fileName);
            Elements elements = doc.getRootElement().getChildElements();
            for (Element element : elements) {
                add(new Person(element));
            }
        }
    
        public static void main(String[] args) throws ParsingException, IOException {
            File file = new File("People.xml");
            People p = new People(file);
            System.out.println(p);
        }
    }
    //output：
    [Dr.Bunsen Honeydew, Gonzo The Great, Phillip J. Fry]
    ```

### 1.7.14 preferences API

1. 使用preferences将数据保存一次，保存的数据资源在注册表中存在

 ```java
//Preferences在windows上使用注册表保存数据资源
public class PreferencesDemo {
    public static void main(String[] args) throws BackingStoreException {
        Preferences prefs = Preferences.userNodeForPackage(Preferences.class);
        prefs.put("Location","0z");
        prefs.put("Footwear","Ruby Slippers");
        prefs.putInt("Companions",4);
        prefs.putBoolean("Are there witches?",true);
        int usageCount = prefs.getInt("UsageCount", 0);
        usageCount++;
        prefs.putInt("UsageCount",usageCount);
        for (String key : prefs.keys()) {
            System.out.println(key + ": " + prefs.get(key,null));
        }
        System.out.println("How many companions does Dorothy have? " + prefs.getInt("COmpanions", 0));
    }
}
//output:
Location: 0z
Footwear: Ruby Slippers
Companions: 4
Are there witches?: true
UsageCount: 2
How many companions does Dorothy have? 4
 ```



## 1.8 接口与类型信息

1. 使用接口的一种重要目标就是允许程序员隔离构件，进而降低耦合性，但是通过类型信息，这种耦合性还是会传播出去。

    - 下面的问题是客户端程序员通过RTTI(Run-Time Type Identification)，可以发现a是被当作B实现的，通过将其转型为B，可以调用不在A中的方法
    - 一种解决方案是直接声明
    - 对实现使用包访问权限
    - 将接口的实现封装在私有内部类
    - 使用匿名类实现接口

    ```java
    public class InterfaceViolation {
        public static void main(String[] args) {
            A a = new B();
            a.f();
            System.out.println(a.getClass().getName());
            //通过对类型的判断可以调用不在A中的方法
            if (a instanceof B) {
                B b = (B) a;
                b.g();
            }
        }
    }
    class B implements A{
        public void f() {}
        public void g() {}
    }
    //output:
    com.wl.B
        
    //实现使用包访问权限
    class C implements A {
        public void f() {
            System.out.println("public HiddenC.f()");
        }
        public void g() {
            System.out.println("public HiddenC.g()");
        }
        void u() {
            System.out.println("package HiddenC.u()");
        }
        protected void v(){
            System.out.println("protected HiddenC.v()");
        }
        private void w(){
            System.out.println("private HiddenC.w()");
        }
    }
    public class HiddenC {
        public static A makeA() {
            return new C();
        }
    }
    
    public class HiddenImplementation {
        public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
            A a = HiddenC.makeA();
            a.f();
            System.out.println(a.getClass().getName());
            //无法找到类型C，因为该类和类C不在一个包
            /*if (a instanceof C) {
                C c = (C) a;
                c.g();
            }*/
            callHiddenMethod(a,"g");
            callHiddenMethod(a,"u");
            callHiddenMethod(a,"v");
            callHiddenMethod(a,"w");
        }
        //使用反射的方法依然可以访问到类C中的其他私有函数，即使两个类不在一个包里
        static void callHiddenMethod(Object a, String methodName) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
            Method d = a.getClass().getDeclaredMethod(methodName);
            d.setAccessible(true);
            d.invoke(a);
        }
    }
    ```

    2. 不管使用什么方法，都无法阻止通过反射调用哪些非公共访问权限的方法，即便是private域

        - final域实际上在遭遇修改时是安全的，运行时系统会在不抛异常的情况下接受任何修改尝试，但是实际上不会发生任何修改

            ```java
            //使用final修饰的变量无法修改(可以修改，但是值不会变)
            class WithPrivateFinalField {
                private int i = 1;
                private final String s = "I'm totally safe";
                private String s2 = "Am I safe?";
            
                public String toString() {
                    return " i = " + i + ", " + s + ", " + s2;
                }
            }
            public class ModifyingPrivateFields {
                public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
                    WithPrivateFinalField pf = new WithPrivateFinalField();
                    System.out.println(pf);
                    Field f = pf.getClass().getDeclaredField("i");
                    f.setAccessible(true);
                    System.out.println("f.get(pf): "+ f.get(pf));
                    f.setInt(pf,47);
                    System.out.println(pf);
                    f = pf.getClass().getDeclaredField("s");
                    f.setAccessible(true);
                    System.out.println("f.get(pf): " + f.get(pf));
                    f.set(pf,"No, you're not!");
                    System.out.println(pf);
                    f = pf.getClass().getDeclaredField("s2");
                    f.setAccessible(true);
                    System.out.println("f.get(pf): " + f.get(pf));
                    f.set(pf,"No, you're not!");
                    System.out.println(pf);
                }
            }
            ```

            