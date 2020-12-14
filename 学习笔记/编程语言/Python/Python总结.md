# Python总结
1. 执行python有两种方式：命令行模式和交互模式
    1. 命令行模式：cmd -> python hello.py（一次性执行py文件的源代码）
    2. 交互模式：cmd -> python -> 执行每一行代码（输入python相当于启动了python解释器，每输入一行就执行一行代码）
2. python为动态语言，是鸭子类型
    - 当参数为对象某个函数时，只要对象有这个方法就能调用成功，不必非要时这个对象类型
***

#### 一、数据类型和变量
1. python可以处理各种数值
2. 字符串
    1. 字符串用''或""括起来
    2. 特殊字符用\标识
    3. r''表示内部的字符串不转义
    ```
    print(r'hello\n world')
    ```
    4. '''...'''可以表示多行内容
    ```
    //...是交互模式下的提示符，不是代码的一部分
    print('''line1
    ...line2
    ...line3''')
    //在py文件中写法如下：
    print('''line1
    line2
    line3''')
    ```
3. 变量
    1. //表示地板除，/是浮点数运算，%是取余运算
    2. b = a 表示b变量指向a变量指向的数据，对变量a的赋值不影响变量b的指向
    3. 常量一般用全部大写的变量名表示
4. 编码
    1. python中字符串默认是unicode编码
    2. chr()函数将编码转换为对应字符，ord()函数将字符转换为对应编码
    3. b'Str'将str转换为bytes字节数据
    4. encode()函数可以将unicode表示的str转换为指定编码的bytes数据
    5. decode()函数将bytes数据转换为str
5. 格式化
    1. 使用%s,%d,%x,%f表示占位符，使用%替换
    ```
    'Hello,%s' % 'World!'
    ```
    2. 用%%表示一个普通的%字符
6. list和tuple
    1. list是可变的
    2. tuple的指向不可变

***

#### 二、函数调用
1. python中的函数名称实质上是指向对象的引用，所以可以将函数名称赋值给变量
    ```
    a = abs
    a(-1)   // output:1
    ```
2. 函数定义
        1. def 函数名(x):
3. 函数参数
    1. 位置参数
    2. 默认参数
    3. 可变参数(参数前面加个*)
    4. 关键字参数(参数前面增加**)
    5. 命名关键字参数(没有可变参数时使用*,)

***

#### 三、高级特性
1. 切片
    1. 字符串的切片包含三个索引值:  s[a:b:c]
    ```
    s = 'abcdefg'
    s[x : y : z]   
    // x表示起始索引值，y表示结束索引值(输出结果不包含该值)，z表示索引间隔(由于python字符串支持负向索引，所以z正可负，不能为0)
    // 当z > 0时，表示从左往右获取，此时x对应的字符应该在y对应字符的左侧(如果在右侧，则结果为空字符串)
    // 当z < 0时，表示采用负向索引从右往左获取字符串，此时x对应的字符应该在y对应字符的右侧(如果在左侧，则结果为空字符串)
    
    //当x和y省略不写时，x此时默认为起始位置，y默认为终止位置+1(这里的起始位置和终止位置是不确定的，正向索引和负向索引时取值不一样)，z如果不写默认为1
    ```
2. 迭代
3. 列表生成式
4. 生成器
```
//函数中如果带有yield关键字，那么这个函数就是generator函数
//generator函数的执行是当调用该函数的next()函数时执行，执行到yield时结束，当再次调用next()函数时，从上次yield的地方继续执行，e.g
    //定义generator函数
    def odd():
        print('step 1')
        yield 1
        print('step 2')
        yield(3)
        print('step 3')
        yield(5)
    //调用generator函数
    o = odd()
    next(o)    //output: step 1   1
    next(o)    //output: step 2   3
    next(o)    //output: step 3   5
    
```
5. 迭代器
    1. 可迭代对象iterable(凡是可作用与for循环的对象都是iterable类型)
        1. 集合数据类型 如list tuple dict set str
        2. generator和带yield的generator function
    2. 迭代器(凡是可作用与next函数的都是iterator类型，是一个惰性计算序列)
        1. 集合数据类型虽然不是iterator类型，但是可以通过调用iter()获得一个iterator对象

***

#### 四、函数式编程
1. 高阶函数：
    高阶函数是指把函数作为变量传入另一个函数中，这样的函数称为高阶函数
    1. map/reduce
        1. map(f,iterable): map函数接收两个参数，第一个参数是函数，第二个是iterable对象，将函数作用在序列的每一个元素中，返回值为iterator类型
        ```
        def f(x):
            return x * x
        r = map(f,list(range(1,10)))
        list(r)   //output: [[1, 4, 9, 16, 25, 36, 49, 64, 81]]
        ```
        2. reduce(f,iterable):
        reduce函数中的f函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算
        ```
        reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
        
        //使用reduce实现求和
        form functools import reduce
        def add(x,y):
            return x + y
        reduce(add,list(range(1,11)))
        ```
    2. filter(f,iterable):
    filter函数可以实现一个筛选函数，当f返回结果为True时保留该值，否则过滤
    3. sorted(iterable,key=..,reverse=True):
    sorted函数允许自定义函数
2. 返回函数:
    返回函数可以将函数当作返回值，注意函数内部不要使用循环变量或者之后会更改的变量
3. 匿名函数
    ```
    lambda x: x * x : lambda表示是一个匿名函数，冒号前面为函数的参数，后面为函数的返回值
    ```
4. 装饰器:
    装饰器函数可以动态增加函数的功能，通过python的@功能
    ```
    1. @符后面为装饰器函数，将原函数传入装饰器函数返回代理函数，运行原函数实际是在运行代理函数

    2. @log时，log为装饰器函数，将原函数func传入log得到代理函数wrapper，运行func实际是在运行wrapper

    3. @log(text)时，log(text)的返回结果decorator为装饰器函数，将原函数func传入decorator得到代理函数wrapper，运行func实际是在运行wrapper

    4. 代理函数wrapper无论增加什么功能必须保证返回结果与原函数func返回结果一致
    
    //e.g:
    def log(fn):
        if isinstance(fn, str):
            def decorator(func):
                @functools.wraps(func)
                def wrappers(*args, **kw):
                    print("There is a str : %s" % fn)
                    return func(*args, **kw)
                return wrappers
            return decorator
        else:
            @functools.wraps(fn)
            def wrapper(*args, **kw):
                print("begin call %s" % fn.__name__)
                num = fn(*args, **kw)
                print(num)
                print("end call %s" % fn.__name__)
            return wrapper

    @log('Hello World!')
    def funcs(n):
        return n * n
    ```
5. 偏函数:
    当函数的参数过多时，可以使用functools.partial创建一个新的函数
    ```
    int2 = functools.partial(int,base=2) // 创建一个int2函数,基于2进制进行数值转换
    ```

***

#### 五、模块
1. 使用模块
    1. 一个.py文件就是一个模块
    2. __name__的取值
        1. 当直接运行一个.py模块文件时，模块中的__name__的值为__main__
        2. 当把这个模块文件导入到另一个模块文件时，模块中的__name__的值为模块名
2. 安装第三方模

***

#### 六、面向对象编程
1. 类和实例
2. 访问限制
    1. 在python中，以双下划线开始并且以双下划线结尾的变量时特殊变量，不是private变量
    2. 以双下划线开头的变量为private变量
    3. _name可以被外部访问到，但是不要随意访问
    4. python实现private变量的方式为将__name改名为_student__name,仍然可以通过后者访问到该变量
3. 继承和多态
4. 获取对象信息
    - 通过hasattr、getattr和setattr实现
5. 实例属性和类属性
    - 实例属性如果和类属性的名称相同，会覆盖类属性

***

#### 七、面向对象高级编程
1. 使用__slots__
    
    - __slots__可以限制实例属性的范围，但只对当前类实例有效，继承的子类无效(除非子类也定义该属性，则范围为父类和子类的并集)
2. 使用@property
    - @property相当于将该函数变成了get方法
    - @###.setter相当于将该函数变成了set方法
    - @property和@###.setter修饰的函数不能和函数内部使用的属性变量同名(相同则为无限递归调用)，如下例子：
    ```
    class Screen(object):s
        @property
        def width(self):
            return self.width
    
        @width.setter
        def width(self, value):
            self.width = value
    
        @property
        def height(self):
            return self.height
    
        @height.setter
        def height(self, value):
            self.height = value
    
        @property
        def resolution(self):
            return self.width * self.height
            
    //该测试用例会报错：递归时栈溢出
    //因为s.width相当于调用使用@width.setter注解修饰的函数，但是在该函数内部同样使用了self.width，相当于递归调用该函数，并且会无线循环，所以报错
    //只需要更改该函数内部的width即可
    s = Screen()
    s.width = 1024
    s.height = 768
    print('resolution =', s.resolution)
    if s.resolution == 786432:
        print('测试通过!')
    else:
        print('测试失败!')
    ```
3. 多重继承
4. 定制类
5. 使用枚举类
    - @unique装饰类可以保证没有重复值
    - 访问枚举类型的三种方式
        1. ClassName.name
        2. ClassName[name]
        3. ClassName(int)
6. 使用元类
    - type()函数可以返回一个对象的类型
    - type()函数也可以创建一个新的类型
    - MetaClass元类，是创建类时的模块
    - metaclass的隐式继承的，子类会调用父类的metaclass定义
***

#### 八、错误、调试和测试
1. 错误处理
    - try...except...finally

# python中内置函数
1. range(x)函数生成0--x-1的序列
2. list()将一个序列转化为list集合
3. callable():可以判断一个对象是否可以被调用


# python中特殊函数
1. __len__():len()函数底层调用的函数
2. __str__():print()时调用的函数
3. __repr__():直接显示变量调用的是__repr__()(调试用的)
4. __iter__():如果一个类想要被用于for...in循环，需要实现该函数，其返回一个迭代对象，每次for循环均是调用该对象的__next__()函数
5. __getitem__():类中实现该函数可以实现向list那样直接取第n个元素 fib()[5]
6. __setitem__():把对象视作list或dict来对集合赋值
7. __delitem__():用于删除某个元素
8. __getattr__():调用类的属性时，当属性(函数)不存在，python解释器会试图调用该函数来尝试获得属性(函数)
9. __call__():如果想要直接调用实例本身，需要定义该函数