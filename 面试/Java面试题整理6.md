## **1.对AOP的理解**

**相关概念**

- 切面：一个特殊的模块化类。
- 通知：切面类中的具体的方法
- 切点：通知要执行的具体位置																

任何一个系统都是由不同的组件组成的，每个组件负责一块特定的功能，当然会存在很多组件是跟业务无关的，例如日志、事务、权限等核心服务组件，这些核心服务组件经常融入到具体的业务逻辑中，如果我们为每一个具体业务逻辑操作都添加这样的代码，很明显代码冗余太多，因此我们需要将这些公共的代码逻辑抽象出来变成一个切面，然后注入到目标对象(具体业务)中去，AOP正是基于这样的一个思路实现的，通过动态代理的方式，将需要注入切面的对象进行代理，在进行调用的时候，将公共的逻辑直接添加进去，而不需要修改原有业务的逻辑代码，只需要在原来的业务逻辑基础之上做一些增强功能即可。

![image-20230206175418080](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206175418080.png)

## **2.Spring常用的注入方式**		

- **通过setter注入** 

**数组注入**

```xml
 <bean id="student" class="com.kuang.pojo.Student">
     <property name="name" value="小明"/>
     <property name="address" ref="addr"/>
     <property name="books">
         <array>
             <value>西游记</value>
             <value>红楼梦</value>
             <value>水浒传</value>
         </array>
     </property>
 </bean>
```

**List注入**

```xml
 <property name="hobbys">
     <list>
         <value>听歌</value>
         <value>看电影</value>
         <value>爬山</value>
     </list>
 </property>
```

**Map注入**

```xml
<property name="card">
     <map>
         <entry key="中国邮政" value="456456456465456"/>
         <entry key="建设" value="1456682255511"/>
     </map>
 </property>
```

**set注入**

```xml
 <property name="games">
     <set>
         <value>LOL</value>
         <value>BOB</value>
         <value>COC</value>
     </set>
 </property>
```

**Null注入**

```xml
<property name="wife"><null/></property>
```

**Properties注入**

```xml
<property name="info">
     <props>
         <prop key="学号">20190604</prop>
         <prop key="性别">男</prop>
         <prop key="姓名">小明</prop>
     </props>
 </property>
```

- **通过构造器创建**

- **自动装配**

- - xml方式，首先需要开启组件扫描。然后在需要注入的bean上写上autowire

	- - 方式有两种：byName和ByType；ByName会根据对应的setXXX为准；ByType会以类型为准，Bytype要求bean容器中不能有两个相同的类对象

	- 注解方式 开启注解支持

	- - AutoWired 先根据类型匹配，如果失败则根据名字匹配
		- `@Resource`和`@Autowired`已经完全一样了，区别就是当名称不一样且类型有多个的时候，`@Autowired`需要配合`@Qualifer`来指定具体的beanid，而`@Resource`只需要写`@Resource(name = beanid)`



##  3.Spring中的bean是线程安全的么？

- 不是。IOC容器本身没有提供线程安全的策略。这个需要结合Scope来讨论
- 对于单例bean，是有可能存在线程不安全的现象的。Spring的单例中分为了无状态和有状态。对于有状态的bean可以设置它的scope为原型，这样每次getBean都会创建一个新的对象。解决线程不安全问题。

------

## **4.Spring中bean的作用域**

- Singleton：单例，仅存在一个对象实例，默认作用域
- prototype：原型，每次调用getBean都会创建一个新的实例 new bean
- request：每一个请求创建一个bean
- session：每一个会话创建一个bean
- golbal-session：全局bean，作用域和application相同

------

## **5.Spring中的自动装配方式**

- ByName
- ByType
- Constructor
- Autodect：先使用构造器装配，如果失败则ByType

------

## **6.Spring中实现事务管理的方式**

- 编程式事务，在代码中手动trycatch 然后调用 rollback commit等
- 声明式事务，基于AOP实现。实现方式可以用xml方式和注解方式。@transactional
- 回到问题上，在正确使用@Transactional注解时，不管@Transactional注解是在类上或实现类的方法上还是在接口上或接口方法上，它的事务功能都是可以实现的，只是选择那种方式更优雅一点而已。

![image-20230206175906918](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206175906918.png)
--
## **7.事务的四大特性**

- 原子性：事务的操作具有原子性，在一个事物中的操作只能都成功或都失败
- 一致性：一个事务在执行前后不能破坏数据库的完整性。比如转账前后总金额应该不变
- 隔离性：在并发环境下，事务之间是不能互相干扰的
- 持久性：事务一旦提交，就会被永久的保存，不可被回滚

--------

## **8.不考虑事务的隔离性会出现的问题**

- 脏读：一个线程读取到另一个线程修改但是还没有提交的数据。张三的工资为5000,事务A中把他的工资改为8000,但事务A尚未提交。与此同时，事务B正在读取张三的工资，读取到张三的工资为8000。随后，事务A发生异常，而回滚了事务。张三的工资又回滚为5000。最后，事务B读取到的张三工资为8000的数据即为脏数据，事务B做了一次脏读。
- 不可重复读：一个事务对同一行数据重复读取两次，但是却得到了不同的结果。
- 幻读：事务在操作过程中进行两次查询，第二次查询的结果包含了第一次查询中未出现的数据或者缺少了第一次查询中出现的数据。目前工资为5000的员工有10人，事务A读取所有工资为5000的人数为10人。事务B插入一条工资也为5000的记录。这是，事务A再次读取工资为5000的员工，记录为11人。
- 幻读并不是说两次读取获取的结果集不同，幻读侧重的方面是某一次的 select 操作得到的结果所表征的数据状态无法支撑后续的业务操作。更为具体一些：A事务select 某记录是否存在，结果为不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。产生这样的原因是因为有另一个事务往表中插入了数据

https://blog.csdn.net/song_myth/article/details/119758995

https://blog.csdn.net/song854601134/article/details/125147625

------

## **9.事务的隔离级别**

![image-20230206183325702](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206183325702.png)

---

## **10.SpringMVC的执行流程**

①用户发送请求至前端控制器DispatcherServlet。

②DispatcherServlet收到请求调用HandlerMapping处理器映射器。

③处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

④DispatcherServlet调用HandlerAdapter处理器适配器。

⑤HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。

⑥Controller执行完成返回ModelAndView。

⑦HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

⑧DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

⑨ViewReslover解析后返回具体View。

⑩DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。DispatcherServlet响应用户。

![image-20230206180050781](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206180050781.png)

------

## **11.SpringMVC里面的组件**

- 前置控制器 DispatcherServlet。

- 映射控制器 HandlerMapping。
- 适配器 HandlerAdapter
- 处理器 Controller。
- 模型和视图 ModelAndView。
- 视图解析器 ViewResolver

------

## **12.@RequestMapping作用**

实现URL请求到Controller方法的映射

------

## **13.MyBatis 中 #{}和 ${}的区别是什么？**

- \#是预编译SQL语句，底层调用PreparedStatement，防止SQL注入
- $采用的是字符串拼接的方式，效果不好，不再使用

------

## **14.Mybatis中两种分页方式**

- 物理分页，使用SQL中的limit关键字进行分页，是真正意义上的分页。但是limit是MySQL特有，不同数据库语法不一样。
- 逻辑分页，使用RowBounds分页。实际上还是从数据库中查询所有数据，在后台再进行分页。

```java
public void getUserByRowBounds(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    //RowBounds实现
    RowBounds rowBounds = new RowBounds(1, 2);
    //通过Java代码层面实现分页
    List<User> userList = sqlSession.selectList("com.kaung.dao.UserMapper.getUserByRowBounds", null, rowBounds);
    for (User user : userList) {
        System.out.println(user);
    }
    sqlSession.close();
}
```

## **15.RowBounds 是一次性查询全部结果吗？为什么？**

表面上来看是一次性查询到所有结果。但是实际上因为mybatis为了防止内存溢出，在底层有一个fetchSize，一次最多取fetchSize条数据。

------

## **16.MyBatis 逻辑分页和物理分页的区别是什么？**

- 逻辑分页是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。
- 物理分页是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。