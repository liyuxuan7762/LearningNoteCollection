### 1.Java中实现代理的几种方式
**什么是代理**

代理是一种设计模式，在Java中当我们要访问目标类时，不是直接的访问目标类，而是访问目标类的代理类来访问。从直接访问转换为了间接访问。

**代理的好处**

这样做可以方面的在调用目标类之前和之后进行一些操作，比如说添加日志和开启事务。

**实现方式**

静态代理和动态代理

**静态代理**
在程序运行前由我们自己编写的，运行时会生成字节码文件。实现方式为代理类和被代理类同时实现同一个接口。在代理类中实现对被代理类的方法的调用。

```java
public interface Person {
    void eating();
    void drinking();
}

public class Student implements Person{
    @Override
    public void eating() {
        System.out.println("学生吃饭");
    }

    @Override
    public void drinking() {
        System.out.println("学生喝水");
    }
}
public class StudentProxy implements Person {
    private Student s;

    public StudentProxy(Student s) {
        this.s = s;
    }
    @Override
    public void eating() {
        System.out.println("饭前洗手");
        s.eating();
    }
    @Override
    public void drinking() {
        System.out.println("去打水6");
        s.drinking();
    }
}
```

**动态代理 实现方法Java自带和CGLib**
方法：在程序运行中自动创建的代理类，底层通过反射实现。具体方法是实现InvocationHandler接口并重写invoke方法。在invoke中调用代理类方法，并实现方法的增强
产生：使用Proxy的newInstance方法生成代理类。

```java
public class MyHandler implements InvocationHandler {
    private Object obj;
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("打印日志");
        Object invoke = method.invoke(obj, args);
        return invoke;
    }
    public Object getNewInstance(Object obj) {
        this.obj = obj;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
    }
}
public class MyTest {
    @Test
    public void test() {
        Student s = new Student();
        Person proxy = (Person) new MyHandler().getNewInstance(s);
        proxy.drinking();
    }
}
```
> 动态代理最常用的就是AOP。Spring中的AOP就是通过动态代理的方式，对原有方法进行增强。
>
> 此外，Spring中的声明式事务也是通过动态代理的方式实现的。
---


### 2.hashcode和equals如何使用
hashcode是用来获取一个对象一个哈希散列码。该方法通过以key作为参数，输出对应的散列码。根据散列码来计算出一个整数值，这个值就是存储数组的下标。
当同时存在多个对象指向同一个数组元素时，就会产生冲突，那么就需要使用equals方法来比较对应的数组元素中单链表中所有元素，判断是否该对象已经存在，如果存在，则不再添加。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3d3397d86b941229003875956078657.png)

---


### 3.为什么重写equal方法同时也要重写hashCode方法
需要满足的前提：两个对象的equals()方法返回TRUE，那么要求两对象的hashcode()必须相等
如果用不到基于哈希码的容器，实际上不重写也没什么问题
方法返回对象的哈希码值，默认情况下该方法会根据对象的地址来计算

```java
String s1 = new String("abc");
String s2 = new String("abc");
// s1和s2的地址是不一样的，因此从理论上来说，直接计算的哈希值
// 应该是不一样的
// 但是由于String类重写了equal方法，因此使用equal方法得到的结果是
// s1.equals(s2) 返回值为true
// 因此为了满足这个要求，也必须重写hashCode方法，使得
// hashCode(s1) == hasoCode(s2)返回值为true
```

* 假设只重写hashcode方法，那么将s1和s2加入到HashSet中时，会因为哈希值相同，而进入equal方法比较，一比较，不相同，则插入两次
* 假设值重写equal方法，因为s1 s2哈希值不同，则直接插入到HashSet中，导致插入两次

**不重写的情况下，哈希码相同对象不一定是一个（哈希冲突）**


Java两个对象的HashCode相等但这两个对象不一定相同------hash算法 hash算法是一种思想，例如最简单的哈希函数：求余运算。 例如有不同的整数通过对指定数（如2）的求余运算得到余数就那么几个。不同的整数可可以得到相同的余数即hash值 HashCode就是将对象的内存地址转化为整数后进行hash运算后得到的hash值。 所以两个对象的HashCode相同但内存地址不一定相同。


> 插入HashMap的key也需要保持唯一性，因此需要对HashMap<Object1,
> Object2>中Object1类中的hashCode和equals方法，不然会出现key重复的问题。我们之前没有重写是因为key普遍为String类型，但是实际上String类重写了这两个方法

---

### 4.HashMap和HashTable的区别和底层实现
**主要区别：**
* HashTable为线程安全的，HashMap为线程不安全的；在多线程环境下，如果HashMap作为共享变量执行后的结果可能和单线程环境下不同。
* HashMap允许Key和Value为null

**底层实现**
* 底层实现为数组+链表的实现方式，元素以Node节点的形式存储。

**存储过程**
* 首先根据key计算出哈希值，然后二次计算哈希值，得到的结果和数组长度取余得到一个整数，这个整数就是该元素应该存储在数组中的下标。
* 如果该下标中没有出现哈希冲突，那么直接创建节点，将元素存入。
* 如果产生冲突，则需要调用equals方法进行比较，如果相同，则不插入；否则，将其元素插入链表中，当链表长度大于8，则转化为红黑树；长度低于6，则从红黑树在转化成链表。
* 数组扩容

**HashMap实现线程安全**
SyncMap 和  ConcurrentMap

--------------------------------------------------------------------------------

### 5.Java中异常的处理方式  
**Throws 声明异常**
throws出现在方法签名处用来声明这个方法可能出现的异常；运行时异常不可以被抛出。声明一个异常，这个异常可能抛出，也可能不抛出
**Throw 抛出异常**
出现在方法内部，将某一异常跑抛出
**try catch 捕获异常**
如果出现的异常可以处理，那么就使用try catch处理异常
![在这里插入图片描述](https://img-blog.csdnimg.cn/c51f2899dfde4f38b492f7264716cf73.png)


--------------------------------------------------------------------------------


### 6.String是基本数据类型吗？
基本数据类型包括：byte(1)、short(2)、char(2)、int(4)、float(4)、long(8)、double(8)、boolean。因此String不是基本数据类型；

--------------------------------------------------------------------------------

### 7.String s = new String("xyx")创建了几个对象？
![在这里插入图片描述](https://img-blog.csdnimg.cn/8c407e3a5f7f4311b12a6303594dd16f.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/3b8e844ac4ed43999790f37cecd6cf8e.png)


一个或者两个。如果常量池中已经有xyz了，则仅在堆内存中创建s，用来指向xyz。如果常量池中没有，则还需要在常量池中创建xyz。
* 所以在平时用String s1=“abcd”这种方式更好，因为new一定会创建一个新的对象，一定会占用新的空间。
* 如果在循环里面要拼接字符串的话，不适合用+拼接，因为每用一次+号就会新生成一个String对象，就很浪费。可以在循环外生成一个StringBuilder对象（或StringBuffer），然后在循环里直接调用append方法来拼接。

原文链接：https://blog.csdn.net/weixin_44847031/article/details/110249936

----------------------------------------

### 8.Integer和int的区别
**区别**
* int是基本数据类型，Integer是包装数据类型
* int默认值是0，Integer默认是是null
* int在内存中存储的是值，Integer存储的是内存地址
* int不实例化就可以使用，Integer必须实例化
**比较**

```java
// 情况1 Integer和int比较
// Integer在和int比较时会自动拆箱，相当于两个int比较
Integer i = new Integer(100);
int j = 100;
sout(i==j); true
```

```java
// 情况2 两个new出来的Integer变量进行比较时，永远都是false。因为integer变量实际上是对一个Integer对象的引用，
// 所以两个new出来的Integer变量的内存地址一定是不相同的，即为false
Integer a = new Integer(100);
Integer b = new Integer(100);
System.out.println(a == b);//false
```

```java
//情况3 一个new出来的Integer变量和一个非new的Integer变量比较时，也为false
Integer x = new Integer(100);
Integer y = 100;
System.out.println(x == y);//false
```

```java
//两个非new生成的对象进行比较时，如果两个变量的值在区间-128~127之内，且两个的值相等，那么则输出为true（这是因为此时两个变量指向的是常量池中的同一对象）；而如果不在这个区间内，则输出为false。代码示例如下：
Integer x = -128;
Integer y = -128;
System.out.println(x == y);//true

Integer x = 128;
Integer y = 128;
System.out.println(x == y);//false
```

```java
//对于以非new形式生成的Integer对象，即Integer i = 100，实际上在编译时会将其翻译为 Integer i = Integer.valueOf(100),而查阅javaAPI可知Integer.valueOf(）方法是这么定义的：
public static Integer valueOf(int i){
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high){
        return IntegerCache.cache[i + (-IntegerCache.low)];
    }
    return new Integer(i);
}
// 所以当定义的Integer变量处于-128~127这个区间时，则会指向java常量池中已有的对象；反之则会指向重新new出的Integer对象
```

https://blog.csdn.net/m0_51358164/article/details/122536750


--------------------------------------------------------------------------------
### 9.对于static关键字的理解
**static方法**
静态方法中不能访问非静态成员方法和非静态成员变量，但是在非静态成员方法中是可以访问静态成员方法/变量的
**static变量**
static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。
**static代码块**
在类被加载时加载一次，可以给静态变量赋值

--------------------------------------------------------------------------------
### 10.Java的值传递和引用传递
Java中只有值传递
**对于基本数据类型**

```java
public void test() {
    int a = 20;
    fun(a);
    System.out.println("实际参数的值为" + a);
}

public void fun(int a) {
    a = 10;
    System.out.println("形式参数的值为" + a);
}
形式参数的值为10
实际参数的值为20
基本数据类型通过形参无法改变实参
```

**对于对象类型**

```java
@Test
public void test() {
    Student s = new Student();
    s.age = 20;
    fun(s);
    System.out.println("实参" + s.age);
}
public void fun(Student student) {
   student.age = 10;
    System.out.println("形式参数" + student.age);
}
形式参数10
实参10
```

对于引用来说，看似改变了实参的值，但是实际上还是值传递。只不过这次传递的值是引用，形参和实参同时指向堆内存中的student对象，然后形参修改，实际上是修改的堆内存中的对象，并没有修改实参的值。这个原理和C语言中的指针传递相同。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d136610431244ab8da8a3c98a7cd572.png)

---

### 11.Java中的常见异常
* Java中所有的异常都是Throwable类，这个类包含Error和Exception
* Error是一些严重的错误，程序无法处理的错误，比如内存溢出
* Exception是一些异常，程序本身可以处理。其又分为运行时异常，和非运行时异常
* 运行时异常在编译时不会检查，比如空指针，数组越界
* 非运行时异常在编译时会检测，需要使用try catch处理或者使用throws向上抛出 比如IO异常，找不到指定文件异常

--------------------------------------------------------------------------------

### 12.Java中的final finally和finalize
* final是访问修饰符，可以修饰变量，方法和类。被修饰的变量的值必须初始化且不能再次改变，通常为常量，被final修饰的方法一定要初始化；被修饰的方法为最终方法，不能被重写；被修饰的类为最终类，不能被继承。
* finally位于异常处理try catch中，被它包围的代码无论有没有异常都会被执行
* finalize为Object类中的一个方法，在Java垃圾回收机制要收回该对象时，这个方法会被执行。
--------------------------------------------------------------------------------
### 13.对反射的理解
**定义**：JAVA的反射机制就是指JAVA在运行过程中，通过获取任意一个类的字节码文件对象，都可以获取这个类的相对应的信息，包括成员变量和成员方法。通过反射可以创建该类的对象，调用这个类的方法和变量，私有方法也可以通过暴力破解进行调用。这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
**使用**：Class.forName("全类名")
**方法**：getDeclaredConstructors()，获取类声明的所有构造方法，getFields()：获得类的所有公共字段，getDeclaredMethods()：获得类声明的所有方法
**应用**：Spring IOC容器，动态代理， DispatcherServlet

```java
public class Demo {
    public static void main(String[] args) throws ClassNotFoundException {
        // 三种方式获取类对应的字节码
        // 1 通过类名.class获取
        Class<Actor> actorClass = Actor.class;
        System.out.println(actorClass);
        // 2 通过类的实例化对象.getclass()获取
        Actor a = new Actor();
        Class<? extends Actor> aClass = a.getClass();
        // 一个类在内存中只能存在一个 所以 aClass==actorClass
        System.out.println(aClass == actorClass);
        // 3 通过Class的静态方法forname(String 类的全路径)
        Class<?> aClass1 = Class.forName("com.itheima.Actor");
        System.out.println(actorClass == aClass1);
    }
}
```

--------------------------------------------------------------------------------

### 14.深拷贝 浅拷贝
![在这里插入图片描述](https://img-blog.csdnimg.cn/f95d31730b964fa08ac551773f04f729.png)


```java
// 引用拷贝
People p1 = new People();
People p2 = p1;
```

* 引用拷贝只是拷贝的是一个地址，实际上p1和p2都指向同一个对象，因此修改p2实际上p1的值也会变
* 浅拷贝：浅拷贝仅仅复制所考虑的对象，而不复制它所引用的对象。浅拷贝在重写的clone()中，只是拷贝了主对象。如果要拷贝的对象中有引用对象，那么引用对象是不会被拷贝的
* 深拷贝：会将该对象和里面所有的引用对象都重新拷贝一遍。
--------------------------------------------------------------------------------
### 15.创建对象的几种方式
* 使用New关键字
* 使用反射获取Constructor，然后调用newInstance方法

```java
public class Demo {
    public static void main(String[] args) throws Exception {
        // 通过反射找到Student类的字节码文件
        Class<?> student = Class.forName("com.itheima.Student");
        // 获取对应的构造方法
        Constructor<?> constructor = student.getConstructor(String.class, int.class);
        // 调用构造方法
        Object stu = constructor.newInstance("jack", 12);
        System.out.println(stu);
    }
}
```

* 通过Clone方法
* 通过反序列化创建对象