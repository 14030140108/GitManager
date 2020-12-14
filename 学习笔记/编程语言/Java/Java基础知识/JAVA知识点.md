## 1. 命令行编译Java程序

1. javac编译生成class文件

    - 示例程序的包结构如图所示：

    <img src="D:\软件\typora\笔记\Java学习总结\img\包结构.png" style="zoom:60%;" />

    - 编译如图的包结构时

        ```java
        //可以在src/main/java目录下编译(包所在目录的上级目录)
        javac ./com/wl/App.java
        java com.wl.App  或者   java -classpath . com.wl.App
            
        //也可以随便在任何一个目录下编译(下面举例在项目的根目录下)
        //下面的语句中-classpath的作用是设置其他编译所需要的第三方类的位置(不包含包名，相当于设置了一个根地址),下面的语句中可以找到App.java,但是如果不设置-classpath,编译会报错无法找到DirFilter类文件，
        javac -classpath ./src/main/java ./src/main/java/com/wl/App.java
        //下面语句中classpath设置了一个找寻class文件的根地址，去指定的路径下找com.wl.App这个class文件
        java -classpath ./src/main/java com.wl.App
        ```

        

    - 如果程序中带有包名，则javac命令去 当前路径+包路径 所组成的路径中寻找源文件编译，所以使用javac命令编译的时候不要进入最内层，只需要在包名上层即可

    - javac命令在编译源文件时，会去寻找相关的import的类，他根据我们的-classpath路径去寻找相关的类，默认是先在当前路径下搜寻import导入类对应的包名

    - 如果当前编译的源文件需要其他的源文件，则可能需要设置下面两个参数

        - -classpath: 设定要搜索的类路径(不要包含类文件包所在的路径，只需要指定至包所在目录的上级目录即可，默认为当前路径)，该参数是设置编译源文件依赖的其他第三方类库的默认搜寻根路径
        - -sourcepath：设定要搜索的java文件路径(不要包含源文件包所在的路径，只需要指定至包所在目录的上级目录即可，默认为当前路径)，该参数是设置编译源文件依赖的其他源文件的默认搜寻跟路径

2. java运行class文件

    - java命令中用-classpath指定搜寻位置，在classpath指定的位置找包名，进而找到class文件
    - -classpath：设定要搜索的类的路径

## 2. Integer类型

1. jvm会默认对-128至127的Integer数据进行cache
    - 所以在这个范围内的数据使用” ==“ 进行判断时为true

## 3. JAVA初始化类时调用顺序

1. 先静态在普通
2. 先父类在子类
3. 父类的静态变量和静态代码块  -> 子类的静态变量和静态代码块 -> 父类的成员初始化和实例初始化 -> 父类的构造器 -> 子类的成员初始化和实例初始化 -> 子类的构造器 

## 4. JVM内存参数

- -Xms：JVM启动时申请的初始Heap值，默认当空余堆内存大于70%，JVM会减小Heap的大小到-Xms指定的值
- -Xmx：JVM可申请的最大Heap值，默认当空余堆内存小于40%时，JVM会增大Heap的大小到-Xmx指定的值
- -Xmn：java heap young区的大小，整个堆大小=年轻代大小+年老代大小+持久代大小（64M）
- -Xss：java中每个线程的栈内存大小，JDK1.5之后每个线程堆栈大小为1M

## 5. Java使用JNI接口调用C程序

- 使用Java编写函数，并用native修饰

    ```java
    public class JavaUseCpp {
        public JavaUseCpp() {
            System.out.println("JNI Test Start");
        }
        static {
            System.loadLibrary("JNIDemo");
        }
        public native void printStr(String str);
    
        public static void main(String[] args) {
            String cname = "lss";
            new JavaUseCpp().printStr(cname);
        }
    }
    ```

- 使用C/C++编写对应的函数实现，并编译成DLL动态库

    ```C++
    JNIEXPORT void JNICALL Java_com_wl_JavaUseCpp_printStr(JNIEnv *env, jobject obj, jstring str)
    {
    	const char* pTempStr = (char*)(*env)->GetStringUTFChars(env,str, NULL);  
    	printf("Hello %s",pTempStr);
    }
    ```

- 重新运行Java项目，调用DLL库文件

## 6. JavaWeb项目查找资源与路径问题

- JavaWeb项目中有两种项目发布形式

    - exploded：以文件夹形式发布在指定的路径下

        - 在此种方式下，getRealPath获取的路径为指定的发布路径
            - getRealPath函数并不是用来查找项目资源的，而是可以用来生成项目路径

        ```java
        System.out.println(this.getServletContext().getRealPath("BZKDownFile"));
        //上述语句将会直接生成一个字符串
        output: D:\software\idea\workspace\DownloadServer\out\artifacts\DownloadServer_war_exploded\BZKDownFile
        ```

        - 查找资源使用getResourceAsStream函数，参数为指定的资源路径

        ```java
        System.out.println(this.getServletContext().getResourceAsStream("/WEB-INF/Config/config.json"));
        //在项目的根路径 + 指定的路径下查找，如果找不到报错
        ```

    - archive：以war包形式发布，然后将其部署在tomcat上，

        - 在此种方式下，getRealPath获取的路径为相对于tomcat的路径。

            `D:\software\Tomcat\apache-tomcat-8.5.30\webapps\DwonloadServer\`

        - 其他的与文件夹发布的一致

        

