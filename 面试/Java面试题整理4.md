# 1.JDK和JRE的区别
* JDK是Java Development Kit的缩写，他包含了JRE和Java编译器Javac以及相关的调试工具
* JRE表示Java Runtime Environment，它为Java运行提供了环境
* 如果需要编写Java程序，则需要安装JDK；如果只是运行Java程序，可以只安装JRE

--------------------------------------------------------------------------------
# 2.普通类和抽象类的区别
* 抽象类可以有抽象方法和非抽象方法，普通类中只能有普通方法
* 普通类可以被实例化，抽象类不可以。

--------------------------------------------------------------------------------
# 3.抽象类可以被 final修饰么
* 不可以，抽象类需要子类的继承才能够使用；如果加了final关键字，则无法被继承

--------------------------------------------------------------------------------

# 4.Java中的IO流分类
* 按照用途分类：可以分为输入流和输出流
* 按照传输单位分类：字节流，一次性传输8位；字符流按照16位字符传输

--------------------------------------------------------------------------------
# 5.Collection和Collections的区别
* Collection是Java中集合的接口，里面提供了一些对于集合的通用操作
* Collections是一个集合工具类，它通过设置构造方法为private实现不可以被实例化，里面提供了很多静态方法，比如sort

--------------------------------------------------------------------------------

# 6.HashMap 和 Hashtable 有什么区别？
* Map是线程不安全的，table是线程安全的
* Map允许key和value为空值，table不允许
* table一般不再使用；在单线程环境下，使用map，多线程下使用concurrentMap
--------------------------------------------------------------------------------

# 7.如何决定使用HashMap还是TreeMap
*  Treeset除非需要对集合进行排序，否则都是用Map
*  Map不支持排序
--------------------------------------------------------------------------------

# 8.ArrayList和Vector的区别

![在这里插入图片描述](https://img-blog.csdnimg.cn/7d7a144979204480b5b9e3e08094471a.png)


--------------------------------------------------------------------------------

# 9.Array和ArrayList的区别
* 存储的长度不同；Array是定长的，不能自动扩容；ArrayList是变长的，会自动根据存储元素的个数进行扩容
* Array可以存储基本数据类型，也可存储引用数据类型；ArrayList只能存储引用数据类型，如果非要存储基本数据类型，则只能存储基本数据类型的包装类
* Array只能存储同一种类型的数据；ArrayList可以存储多种数据，底层都属转化为Object类型
* Array在初始化时需要指定长度，ArrayList不需要

```java
public static void main(String[] args) {
    // 如果在创建ArrayList的时候不指定类型，那么其存储的都是Object类型
    // 因为Object是所有类的父类，因此该list就可以存放任何类型的元素
    ArrayList list = new ArrayList();
    list.add("abc");
    list.add(123);
    // 在获取元素时，得到的是Object类型，需要根据自己的需要强转
    Object o = list.get(1);
    
    // 如果指定存储的类型，那么list集合中只能存储该类对象以及其子类的对象
    ArrayList<String> list1 = new ArrayList<>();
    list1.add(12); // 报错
}
```

--------------------------------------------------------------------------------

# 10.Java中的队列
* Java中的队列分为了阻塞队列和非阻塞队列，有界和无界，单链表和双链表
* 阻塞队列当队列为空的时候阻塞，队列不为空则解除阻塞；当队列满的时候也阻塞，不满则解除；它是线程安全的；* 非阻塞队列在队列为空时仍可以出队，满时仍可以入队，但会产生异常。阻塞队列是线程安全的
* 有界队列的长度不变，无界长度动态调整
* 单链表和双链表两种方式
--------------------------------------------------------------------------------

# 11.集合遍历方式
* for+size()，最基础的遍历方式
* 使用迭代器iterator

```java
// 通过地迭代器遍历
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()) {
    System.out.println(iterator.next());
}
// 通过索引遍历
for(int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

通过增强for语句

```java
List<String> list = new ArrayList<String>();
list.add("Hello");
list.add("World");
for(String s : list) {
    System.out.println(s);
}
```

并发修改异常
![在这里插入图片描述](https://img-blog.csdnimg.cn/bfb1f60b875044dbba8835b9e40c6442.png)

```java
public class Test {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        list.add("java");
        ListIterator it = list.Iterator();
        while(it.hasNext()){
            if(it.next().equals("world")){
                list.add("JavaEE"); // 这里会报错
            }
        }
        System.out.println(list);
    }
}
```


```java
/*
方案一：通过列表迭代器解决
        必须使用列表迭代器中的添加元素的方法，即：ListIterator#add()
        这种方式添加的元素，是跟在刚才迭代的元素后面的
 */
public class Test {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        list.add("java");
        ListIterator it = list.listIterator();
        while(it.hasNext()){
            if(it.next().equals("world")){
                it.add("JavaEE");
            }
        }
        System.out.println(list);
    }
}
/*
方案二：通过for循环+size()解决
        这种方式添加的元素，是在集合的最后位置添加的
 */
public class Test {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        list.add("java");
        for (int i=0;i<list.size();i++){
            String temp = list.get(i);
            if (temp.equals("java")){
                list.add("JavaEE");
            }
        }
        System.out.println(list);
    }
}
```

**iterator和listiterator的区别**
* iterator可以遍历list和set；list iterator只能遍历list
* iiterator遍历只能从前往后单向；list iterator可以向前也可以向后
--------------------------------------------------------------------------------

# 12.并发和并行
* 并行：多个处理器或多核处理器同时处理多个任务。
* 并发：多个任务在同一个 CPU 核上，按细分的时间片轮流(交替)执行，从逻辑上来看那些任务是同时执行
--------------------------------------------------------------------------------

# 13.Java中的线程调度
* i线程调度的方式有分时调度，所有线程轮流使用CPU时间片；抢占式，让优先级高的线程先使用CPU，如果优先级相同，则随机选择一个执行；
* Java中使用抢占式，使用setPriority方法设置优先级。默认为5，最大为10，最小为1
* 优先级低只是意味着获得调度的概率低，并不是优先级低就不会被调用了，这都是看CPU的调度。
--------------------------------------------------------------------------------

# 14.什么是守护线程
* 守护线程是一个特殊的线程，它的生命周期无法由自己决定，当所有的线程都是守护线程时，JVM将会退出；
* 一个线程将自己设置成守护线程需要在start方法前执行setDaemon(true)
* 最典型的例子是Java中的垃圾回收进程，当所有当进程都执行完毕后，垃圾回收因为是守护线程，所以JVM退出执行。
--------------------------------------------------------------------------------

# 15.创建线程的方法和这几种方法的区别
* i继承Thread，由于Java是单继承，这会导致局限性
* i实现Runable接口；上面这两种都是线程执行是没有返回值的
* i实现Callable，可以实现线程执行带有返回值
--------------------------------------------------------------------------------

# 16.进程的5种执行状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b141b1ae9334bb2a5974aa063c7a78f.png)

--------------------------------------------------------------------------------

# 17.Sleep wait notify notifyAll start run
* Sleep是Thread的静态方法，调用sleep方法可以让当前正在运行的线程进入睡眠状态，即暂时停止运行指定的单位时间。并且该线程在睡眠期间不会释放对象锁。
* Wait是Object类的一个方法，调用wait方法可以让当前线程进入等待唤醒状态，并释放锁。该线程会处于等待唤醒状态直到另一个线程调用了object对象的notify方法或者notifyAll方法
* Nofity：唤醒一个线程，其他线程依然处于wait的等待唤醒状态，如果被唤醒的线程结束时没调用notify，其他线程就永远没人去唤醒，只能等待超时，或者被中断
* NotifyAll：所有线程退出wait的状态，开始竞争锁，但只有一个线程能抢到，这个线程执行完后，其他线程又会有一个幸运儿脱颖而出得到锁
* Start只是启动线程，为其分配除了CPU以外的所有资源，线程进入就绪状态
* Run是当线程得到CPU资源以后执行的方法
--------------------------------------------------------------------------------

# 18.Java中的对象序列化
* 序列化就是将Java内存中的对象保存到文件到形式，把保存后到对象再读出来的过程是反序列化
* 用途：将对象保存到数据库，对象进行网络传输等
--------------------------------------------------------------------------------

# 19.代理模式
* 代理模式分为了静态代理和动态代理两种
* 代理即代理类和被代理类实现同一个接口，我们在调用时，不直接调用代理类的方法，而是通过被代理进行调用；在被代理类中调用代理类的方法，还可能会在方法执行前后添加一些其他操作
* 静态代理需要我们手动编写，在程序运行前就已经写好，会生成class文件。一个对象就需要一个代理类，导致代码复杂
* 动态代理在程序运行时使用反射的方式动态生成代理类，实现方式实现InvocationHandler和利用Proxy中的newInstance方法生成。动态代理实现主要有Java自带和cglib
* 动态代理主要应用AOP 
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

--------------------------------------------------------------------------------

# 20.对象克隆
* 克隆的对象可能包含一些已经修改过的属性，而 new 出来的对象的属性都还是初始化时候的值，所以当需要一个新的对象来保存当前对象的“状态”就靠克隆方法了。

https://www.cnblogs.com/Qian123/p/5710533.html