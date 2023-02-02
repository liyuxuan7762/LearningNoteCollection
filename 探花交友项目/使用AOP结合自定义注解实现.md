## 使用AOP结合自定义注解实现

在项目中，需要记录用户的登录，发送动态，点赞等行为，将这些行为记录下来发送到消息队列中供其他模块对数据进行处理。之前采用的方法是在原有代码上修改。但是这样很不优雅。

之前学习Spring的时候就听说过AOP，我还记得作用主要是在不修改原有代码的情况下实现对方法对增强。于是我想尝试使用AOP解决上述的问题。奈何我对AOP的理解仅仅停留在理论方面，脑子里面也只有动态代理，对于他们的实际用法，我确实一次没用过，搜索资料发现网上讲的有一些过于笼统，并不适合小白入门。经过大量的翻阅博客，我基本掌握了AOP+自定义注解的简单实用，为方便后人学习，写下这篇文章。

### 1. AOP的原理

关于AOP的原理就不在赘述，其主要的原理就是利用了Java中的动态代理机制和反射机制，在真正的被代理的方法执行前后去添加一些代码，从而实现对原有方法对增强。这一点相信大家在刚刚接触Spring的时候就已经接触到了。

### 2. 自定义注解

要实现自定义注解实现AOP，我们就必须先了解一下自定义注解是如何定义的。下面是一个基本的自定义注解的定义方式：在IDEA中我们可以直接创建一个自定义注解：

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LogConfig {

    //动态获取方法参数，支持SpringEL
    String objId() default "";

    //路由的key
    String key();

    //日志类型
    String type();
}
```

在定义自定义注解的时候涉及到了另外几个核心注解，他们的作用如下：

* `@Target`：Target翻译中文为目标，即该注解可以声明在哪些目标元素之前，比如有一些注解只能写在方法上，有一些可以写在类上。这个就是通过@Target注解来实现的。我定义的这个数据就是只能作用于方法上的。还有以下的一些位置：
  * `ElementType.PACKAGE`：该注解只能声明在一个包名前。
  * `ElementType.ANNOTATION_TYPE`：该注解只能声明在一个注解类型前。
  * `ElementType.TYPE`：该注解只能声明在一个类前。
  * `ElementType.CONSTRUCTOR`：该注解只能声明在一个类的构造方法前。
  * `ElementType.LOCAL_VARIABLE`：该注解只能声明在一个局部变量前。
  * `ElementType.METHOD`：该注解只能声明在一个类的方法前。
  * `ElementType.PARAMETER`：该注解只能声明在一个方法参数前。
  * `ElementType.FIELD`：该注解只能声明在一个类的字段前。
* `@Retention`：Retention 翻译成中文为保留，可以理解为如何保留，即告诉编译程序如何处理，也可理解为注解类的生命周期。这个注解的具体含义我并不太明白，但是一般都是写RUNTIME。除了RUNTIME，还有以下的一些：
  * `RetentionPolicy.SOURCE` : 注解只保留在源
  * `RetentionPolicy.CLASS` : 注解保留在class文件中，在加载到JVM虚拟机时丢弃。

* @Documented：这个注解只是用来标注生成javadoc的时候是否会被记录。在自定义注解的时候可以使用@Documented来进行标注，如果@Documented标注了，在生成javadoc的时候就会把@Documented注解给显示出来。@Documented注解只是用来做标识，没什么实际作用，了解就好。

> 此外还需要注意定义注解前面的关键字是`@interface`，不要和定义接口混了。

除了上面介绍的注解以外，在注解定义的内部，我们可以定义注解接收的一些变量的值，这样我们在编写切面的时候可以获取到。可以通过default关键字来制定这些变量的默认值。

```java
@LogConfig(type = "0202",key = "movement",objId = "#movementId")
```

我们在使用自定义注解的时候，可以向我们在自定义注解中定义的变量中传递值。

到此为止，自定义注解的基本只是就讲解完毕了。接下来要定义的就是切面了。

### 3. 切面

切面这个词也不陌生了，在学习Spring的时候就经常提起，但是这个词语过于的抽象。我的理解是切面类就是**定义要增强哪些方法，以及如何增强的这么一个类**。具体来说：

* 通过execution表达式可以指定要增强方法
* 通过在切面类中编写方法，可以实现对方法的增强

下面是我为了完成之前说的功能编写的一个切面类，代码如下：

```java
@Component
@Aspect
public class LogAspect {

    @Autowired
    private AmqpTemplate amqpTemplate;

    @PointCut(value="@annotation(com.tanhua.server.aop.anno.LogConfig)")
    public void pointcut() {}
  
    @Before("pointcut()")
    public void checkUserState(JoinPoint pjp , LogConfig config) throws Throwable {
        //解析SpringEL获取动态参数
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        String objId = parse(config.objId(), signature.getParameterNames(), pjp.getArgs());
        //构造Map集合
        Map<String, Object> msg = new HashMap<>();
        msg.put("userId", UserHolder.getUserId());
        msg.put("date", new SimpleDateFormat("yyyy-MM-dd").format(new Date()));
        msg.put("objId", objId);
        msg.put("type", config.type());
        String message = JSON.toJSONString(msg);
        //发送消息
        try {
            amqpTemplate.convertSendAndReceive("tanhua.log.exchange",
                    "log."+config.key(),message);
        }catch (Exception e) {
            e.printStackTrace();
        }
    }

    public String parse(String expression, String[] paraNames,Object [] paras) {
        if(StringUtils.isEmpty(expression)) return "";
        StandardEvaluationContext context = new StandardEvaluationContext();
        for(int i=0;i<paraNames.length;i++) {
            context.setVariable(paraNames[i], paras[i]);
        }
        Expression exp = new SpelExpressionParser().parseExpression(expression);
        Object value = exp.getValue(context);
        return value == null ? "" : value.toString();
    }
}
```

下面来具体的解释一下：

* 首先，切面就是一个类，所以要先定义一个切面类，名字最好符合命名规范，然后在类上添加`@Component`和`@Aspect`两个注解，交由IOC容器管理，并指明这个类是一个切面类。

* 接下来要定义切点，也就是定义要增强哪些方法。定义的方式主要有execution表达式，这个应该大家都很熟悉，在学习Spring的时候就学习的这种方式。还有一种方式是`@annotation`方式。

### 3.1 execution表达式

  execution表达式方式：具体用法如下图所示：![image-20230103204002189](https://img-blog.csdnimg.cn/img_convert/22789d4b80380c9bd4a56d7213f422ed.png)

还有另一种表达：

![image-20230103204308100](https://img-blog.csdnimg.cn/img_convert/29361aeb043a4d8b6bcb5939445c0284.png)

这一个指的就是对com.zhang.service包下面所有结尾是impl的所有方法进行增强。

> 这种方式是我们最初学习Spring AOP的方式，这种方式的缺点显而易见，表达式写起来过于麻烦，而且对于某一些方法，我们可能仅仅在某一些方法上才想让AOP进行增强，所以采用这种表达式的方式并不是很方便。因此衍生出下面的`@annotation`的方法，通过这种方式可以将AOP和我们自定义注解结合起来使用。我在定义自己的切面类的时候就采用的下面的这种方式

### 3.2 @Annotation表达式

`@annotation`：作用：凡是带有被这个`@annotation`修饰的注解，然后这个注解所修饰的方法或是接口都会被拦截！请看代码：

```java
@PointCut(value="@annotation(com.tanhua.server.aop.anno.LogConfig)")
public void pointcut() {

}
```

`@LogConfig`是我自定义的注解，他现在被`@annotation`修饰。所以在代码中，只要被`@LogConfig`修饰过的方法，都会进行增强，这样灵活性是不是比使用execution表达式提高了不少。

### 3.3 关于解析Spring EL表达式

```java
@LogConfig(type = "0202",key = "movement",objId = "#movementId")
```

可以看到，在使用自定义注解的时候，我还使用了Spring EL表达式传递了一个变量的值。那么这一个是如何进行解析的呢？

```java
//解析SpringEL获取动态参数
MethodSignature signature = (MethodSignature) pjp.getSignature();
String objId = parse(config.objId(), signature.getParameterNames(), pjp.getArgs());

public String parse(String expression, String[] paraNames,Object [] paras) {
  if(StringUtils.isEmpty(expression)) return "";
  StandardEvaluationContext context = new StandardEvaluationContext();
  for(int i=0;i<paraNames.length;i++) {
    context.setVariable(paraNames[i], paras[i]);
  }
  Expression exp = new SpelExpressionParser().parseExpression(expression);
  Object value = exp.getValue(context);
  return value == null ? "" : value.toString();
}
```

从上面的代码可以看出来解析过程如下：

* 首先，获取到这个EL表达式。`config.objId()`
* 因为EL表达式的值都保存在方法签名中，因此还需要获取方法签名的参数列表。获取参数的名字和参数值。`signature.getParameterNames(), pjp.getArgs()`
* 将获取到的参数列表封装到`StandardEvaluationContext`类的对象中。
* 由于传递过来的EL表达式是字符串，所以使用`SpelExpressionParser`转换成EL表达式对象
* 最后调用EL表达式对象的`getValue(context)`方法完成对EL表达式的解析。

### 4. 使用自定义注解

到此为止，自定义注解结合AOP的相关代码都应编写完毕。下面看一下如何使用自定义注解：

以查询动态详情为例：

```java
//根据id查询
@LogConfig(type = "0202",key = "movement",objId = "#movementId")
public MovementsVo findById(String movementId) {
    //1、调用api根据id查询动态详情
    Movement movement = movementApi.findById(movementId);
    //2、转化vo对象
    if(movement != null) {
        UserInfo userInfo = userInfoApi.findById(movement.getUserId());
        return MovementsVo.init(userInfo,movement);
    }else {
        return null;
    }
}
```

只需要在要增强的方法前添加我们自定义的注解即可。比之前在每一个方法中修改代码的方式要优雅不少，而且方便维护。
