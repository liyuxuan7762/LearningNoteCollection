## **1.MyBatis是否支持懒加载；原理？**

**什么是懒加载**

如果查询订单并且关联查询用户信息。如果先查询订单信息即可满足要求，当我们需要查询用户信息。把对用户信息的按需去查询就是延迟加载。 所以延迟加载即先从单表查询、需要时再从关联表去关联查询，大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。

MyBatis支持懒加载，仅仅可以在使用Collection和Association中使用

**原理**

**底层通过过滤器来实现**

比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

https://blog.csdn.net/eson_15/article/details/51668523

------

## **2.Mybatis中的缓存**

**一级缓存**

- 一级缓存是sqlsession级别的，当执行某一条SQL操作时，会先查询缓存中是否已经有该结果，如果有，则直接从缓存中取，否则就查询数据库。
- 当有增删改操作commit执行后，则会清空缓冲区来避免脏读问题。
- 一级缓存不需要手动开启
- Mybatis内部存储缓存使用的是一个HashMap对象，key为 hashCode + sqlId + sql 语句。而value值就是从查询出来映射生成的**java对象**。

![image-20230206184629792](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206184629792.png)

**二级缓存**

- 二级缓存是Mapper级别的，二级缓存与一级缓存区别在于二级缓存的范围更大，多个sqlSession可以共享一个mapper中的二级缓存区域。mybatis是如何区分不同mapper的二级缓存区域呢？它是按照不同mapper有不同的namespace来区分的，也就是说，如果两个mapper的namespace相同，即使是两个mapper，那么这两个mapper中执行sql查询到的数据也将存在相同的二级缓存区域中。
- 开启方式
- 多表操作一定不要使用缓存。不管多表操作写到那个namespace下，都会存在某个表不在这个namespace下的情况。例如两个表：role和user_role，如果我想查询出某个用户的全部角色role，就一定会涉及到多表的操作。不管是写到RoleMapper.xml还是UserRoleMapper.xml，或者是一个独立的XxxMapper.xml中。如果使用了二级缓存，都会导致上面这个查询结果可能不正确。
- 使用二级缓存的所有POJO类都需要实现序列化接口

```
mybatis-config.xml
<setting name="cacheEnabled" value="true">
mapper.xml
<cache></cache>
```

> **我们应该尽量避免使用二级缓存**
>
> - **在实际使用中，我们很难保证对于一个表的操作都在同一个Mapper中，比如对于一个user表的操作可能在UserMapper，也可能在XXXMapper中，因此如果我们在UserMapper中对表格已经做了修改，那么UserMapper中的缓存会被清理，查询不会出错；但是在XXXMapper中的缓存并没有被清理，当XXXMapper中某一个查询访问缓存内容时，这个缓存可能已经和User表中不同了。导致脏读**
> - **在多表查询中一定不要使用二级缓存。假设我们员工表和部门表格联表查询(写在EmpMapper中)，查询第一次直接从数据库取，没有什么问题。查询第二次时，刚好DeptMapper中执行了更新操作，那么此时由于EmpMapper中并没有执行增删改，所以不会更新，第二次查询直接从缓存中取，但是实际上此时缓存和数据库中的数据已经不一致了。导致脏读的发生。**

![image-20230206184731547](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206184731547.png)

参考文章

https://blog.csdn.net/eson_15/article/details/51669021

https://blog.csdn.net/eson_15/article/details/51669608

https://blog.csdn.net/qq_40277163/article/details/124752274

## **3.Mybatis中的执行器**

- SimpleExecutor：简单执行器，每一次执行都会创建一个新的Statement对象，每一条SQL语句都会编译。执行完毕后关闭Statement
- ResueExecutor：每次读或者写的时候，会以sql作为key来查询对应的Map中是否已经存在 Statement，如果存在则直接使用，否则创建新的。使用完成后，不关闭statement，而是存放到Map里面
- BatchExecutor：将所有的增删改SQL通过AddBatch方法添加到一起，然后统一执行。

------

## **4.Mybatis的执行过程**

- sqlsession接受用户发来的sql语句，委托给Exector执行器执行。
- Executor调用StatementHandler来实现参数和结果的处理
- 它首先调用ParameterHandler将SQL语句中的参数进行绑定，然后使用java.sql.statement对象执行sql语句并返回结果集
- 最后通过ResultSetHandler对结果集进行处理，将结果返回

https://zhuanlan.zhihu.com/p/509061195

------

## **5.Mybatis分页插件实现的原理**

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 SQL，然后重写 SQL，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

------

## **6.数据库第三个范式**

- 第一范式：字段具有原子性，字段不可以再被划分。

- 第二范式：非主键字段必须完全依赖主键。比如一个成绩表中有(学号，课程号，成绩，课程名称)，成绩需要由学号和课程号组成的主键决定，满足2NF，但是课程名称只依赖于课程号，不满足2NF

  - 数据冗余：如果在学生选课表中，有n个学生选择了某一门课，那么这个课程在这个表中的学分字段就重复了n-1次
	- 更新冗余：如果修改一门课的名称，那么需要修改n次
	- 插入异常：如果新开一门课，由于还没有人选，那么学号，姓名等字段为空
	- 如果一批学生这门课结课了，需要删除这些学生，那么课程信息也被删除了

- 第三范式：不存在依赖传递。sex_desc不直接依赖于主键，需要通过id->sex_code->sex_desc才能建立依赖，存在依赖传递，不满足3NF

![image-20230206184801149](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230206184801149.png)

https://blog.csdn.net/qq15035899256/article/details/126315832

------

## **7.char和varchar的区别**

- char类型是固定长度字符串char(10)，在声明是确认长度

- - 好处：查询效率高
	- 缺点：占用空间大；适合一些固定长度的值存放，比如MD5值

- varchar类型是变长字符串。会根据字段值进行调整。占用空间为存储的值所占字节数在加上存储所占长度数据所占的字节。

------

## **8.内链接，外链接，全链接**

- 内链接仅展示符合关联关系的记录
- 左外链接是将左边表里面所有的信息都显示出来。显示驱动表所有记录加上满足关联关系的从表记录，没有数据的字段使用null
- 全外连接：展示A B表中所有数据 数据为空的使用null代替
- 全链接：全链接要求左右两个查询的结果返回一样的colum。union会自动将两个链接的结果中完全相同的记录合并为一条记录。union all则会保留两个查询的所有记录。

------