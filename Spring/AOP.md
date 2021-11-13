AOP(Aspect Oriented Programming)的背景，官方文档的总结很精炼：

*面向切面编程*（AOP）通过提供另外一种思考程序结构的途经来弥补面向对象编程（OOP）的不足。在OOP中模块化的关键单元是类（classes），而在AOP中模块化的单元则是*切面*。切面能对关注点进行模块化，例如横切多个类型和对象的事务管理。

除核心业务代码块外，还有其他外围业务逻辑，可以抽象出通用代码，例如权限校验，日志，事务，性能监测等。如果使用OOP，在代码中需要注入通用业务逻辑，容易造成代码混乱，分散冗余，可扩展性低的问题。所以希望这些模块可以实现热插拔特性而且无需把外围的代码入侵到核心模块中。

![img](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/21257086/1626248438823-9c8c29c7-38b4-4c0c-a5c8-23967ad1bf5c.png)

**将外围业务（日志，权限等） 抽象成切面，对关注点进行模块化，“低入侵”式地注入到其他业务模块中，即AOP（面向切面编程）。**

大概了解AOP是什么后，可以找个demo简单感受一下。下面主要简单介绍AspectJ实现AOP的原理，详细介绍Spring实现AOP的原理。

## AspectJ

AspectJ是Java实现的AOP框架，能对Java代码进行AOP编译（一般在编译期进行），让Java代码具有AspectJ的AOP功能（需要特殊的编译器，ajc编译器），AspectJ是最早的，目前实现AOP框架中最成熟，功能最丰富的，且与Java程序完全兼容。

将切面逻辑应用到Java代码的过程（weave织入），可以分为静态织入和动态织入，AspectJ采用静态织入，Spring AOP采用动态织入。

AspectJ的织入方式是：编译期织入。将Aspect类通过ajc编译器编译成class字节码，在Java目标代码编译时织入，最终的class文件包含切面代码和目标代码。

## Spring Aspect

Spring AOP 的焦点不在提供完整的AOP功能，更注重的是与Spring IOC容器的结合，并结合该优势来解决横切业务的问题。AspectJ在AOP实现上依赖ajc编译器，Spring AOP使用Java实现，采用动态代理。

代理模可以总结为**使用代理对象来代替对真实对象的访问，不修改原目标对象，提供额外的功能操作，扩展目标对象的功能。**可以采用静态代理或动态代理实现代理模式。动态代理又可分为基于JDK的动态代理和CGLIB。

## 静态代理

静态代理是基于手动完成的，我们称被代理的为委托类，代理为代理类，他们都实现同一个接口。**这里有个关键问题是，为什么实现同一个接口？是为了尽可能保证代理对象内部结构和被代理对象一致，这样对代理对象的操作最终都可以转移到被代理对象。**

**静态代理的实现步骤是：**

1. 定义一个接口及其实现类（委托类）

2. 创建一个代理类同样实现这个接口

3. 将目标对象注入代理类，然后在代理类的对应方法调用目标类中的对应方法。这样，我们就可以通过代理类屏蔽对被代理对象的访问，并且可以在目标方法执行前后进行功能增强。

![img](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/21257086/1626348225172-50debc13-f77f-46cc-b70a-f7eb10e9fc6b.png)静态代理的缺陷也显而易见，有一个委托类就要写一个代理类，**创建代理类工作量大，且有的代理增强可以更抽象。**

**动态代理主要解决两个问题：**

1. 如何动态的创建代理类，减少手动创建代理类的工作量？
2. 如何抽象代理类，使其更精简？

## JDK动态代理

一个class类javac编译成一个.class字节码文件，字节码文件中存储了类信息，类加载器将字节码文件加载到内存中，JVM解析字节码文件，生成class对象。如果可以在运行时期，生成字节码，加载转换成相应的类，即可完成在运行时动态的创建一个类。实现动态代理第一个问题，动态创建代理类，思路done，这通过反射实现。

第二个问题，静态代理中，不同委托类，代理类做的增强不同，那如何抽象代理类呢？我们可以把调用委托类这个动作抽象出来，这是所有代理类的通用方法。在JDK动态代理中，创建了`InvocationHandler`调用处理器类，解决这个问题。

**JDK动态代理方案如图：**

![img](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/21257086/1626405498002-86b5787b-e34e-45ca-aaa7-56110ab63629.png)

**JDK动态代理实现步骤：**

1. 定义一个接口（Subject）

2. 创建委托类（Real Subject）实现接口（Subject）

3. 创建调用处理器类实现`InvocationHandler`接口，重写`invoke`方法，在 `invoke` 方法中利用反射机制调用委托类的方法，并自定义一些处理逻辑），并将委托类注入调用处理器类。

```java
/**
* proxy: 代理类对象
* method: 调用委托类的方法
* args: 传给委托类方法的参数列表
**/
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

4. 创建代理对象，通过`Proxy.newProxyInstance()`创建。

```java
/**
* loader: 类加载器
* interface: 委托类实现的接口数组，至少传一个接口
* InvocationHandler的实现类
**/
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
```

**日志动态代理，写个demo：**

1. 定义接口`LogService`：

```java
public interface LogService {
    void log(String content);
}
```

2. 创建接口实现类（委托类）`LogSrviceImpl`：

```java
public class LogServiceImpl implements LogService{
    @Override
    public void log(String content) {
        System.out.println("JDK proxy log service: "+ content);
    }
}
```

3. 创建Handler实现`InvocationHandler`接口：

```java
public class LogInvocationHandler implements InvocationHandler {
    // 委托类对象
    private final Object target;

    public LogInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before...");
        // 通过反射调用委托类中的方法
        Object result = method.invoke(target, args);
        System.out.println("after...");
        return result;
    }
}
```

4. 创建代理对象，注入委托对象和调用处理器，执行委托类方法时通过代理对象：

```java
public class Main {
    public static void main(String[] args) {
        LogService proxy = (LogService) getProxy(new LogServiceImpl()); //注入委托对象，获取代理对象
        proxy.log("hello");
    }
    private static Object getProxy(Object target){
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                new LogInvocationHandler(target));
    }
}
```

## CGLIB

JDK动态代理的实现方案中，更像是“代理接口“，只能代理接口的实现类和实现类中重写接口的方法，无法代理委托类的私有方法，感觉并不向是对委托类的直接代理。

CGLIB(Code Generation Library)是基于ASM的Java字节码生成框架，提供在运行时动态修改或生成字节码的功能。CGLIB原理简单说是，通过字节码生成委托类的子类，在子类中拦截父类（委托类）的方法调用，织入增强逻辑。不同于JDK动态代理，CGLIB通过继承的方式聚焦于代理委托类，引入方法拦截器接口`MethodInterceptor`，和`Enhancer`可以动态的创建给定类的子类，并设置拦截器。

![img](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/21257086/1626417495221-150ecfad-b936-4108-8147-151d9a5cc623.png)

**CGLIB的实现步骤：**

1. 创建委托类（Real Subject）

2. 创建方法拦截器实现`MethodInterceptor`，重写`intercept`方法。对`intercept`接口参数的解释参照官方文档[Interface MethodInterceptor](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodInterceptor.html)，其中MethdProxy参数尤其重要，指路官方接口文档[Class MethodProxy](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodProxy.html)。

```java
/**
* var1: 增强后的对象（"this", the enhanced object)
* var2: 被拦截的方法
* var3: 方法入参
* var4: 方法执行的代理
**/
public interface MethodInterceptor extends Callback {
    Object intercept(Object var1, Method var2, Object[] var3, MethodProxy var4) throws Throwable;
}
```

实现时注意到MethodProxy有两个接口`invoke`和`invokeSuper`，很容易踩坑。仔细看，官方文档中对参数的描述已经足够让我们使用接口，简单说`invoke`中入参为被代理对象，`invokeSuper`中入参为被代理后的对象，具体原理简单展开分析可以参考这篇博客[聊聊cglib动态代理遇到的坑](https://juejin.cn/post/6844903565996081160)，再有更详细的问题可以参考CGLIB源码，这里不展开。

3. 创建代理对象，在CGLIB中通过`Enhancer.create()`创建。

**还是日志服务，demo time：**

1. 创建委托类：

```java
public class LogService {
    public void log(String content){
        System.out.println("CGLIB proxy log service: " + content);
    }
}
```

2. 创建拦截器：

```java
public class LogServiceInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before...");
        Object obj = methodProxy.invokeSuper(o, objects);
        System.out.println("after...");
        return obj;
    }
}
```

3. 创建代理对象：

```java
public class Main {
    public static void main(String[] args) {
        LogService logService = (LogService) getProxy(LogService.class);
        logService.log("hello");
    }

    public static Object getProxy(Class<?> targetClass){
        Enhancer enhancer = new Enhancer();
        // 设置类加载器
        enhancer.setClassLoader(targetClass.getClassLoader());
        // 设置委托类
        enhancer.setSuperclass(targetClass);
        // 设置方法拦截器
        enhancer.setCallback(new LogServiceInterceptor());
        return enhancer.create();
    }
}
```

**最后再回顾一下动态代理如何解决静态代理缺陷的问题：**

1. **动态创建代理类？**JDK采用Proxy.newProxyInstance()根据接口实现动态创建代理类，CGLIB采用Enhancer基于继承，创建委托类的子类（代理类）。

2. **如何抽象代理类中增强逻辑？**JDK中采用`InvocationHandler`，CGLIB采用`MethodInterceptor`在子类调用目标方法时，织入增强逻辑。

这里挖个坑，根据接口和继承两种方式有什么不同？`InvocationHandler`和`MethodInterceptor`的区别？

## 总结

简单介绍了AOP背景，准备从代理模式开始了解Spring AOP原理，大概了解了JDK和CGLIB两种方式实现动态代理的思路，过程中发现还有更多底层原理值得了解。

## 参考

[Chapter 6. 使用Spring进行面向切面编程（AOP）](http://shouce.jb51.net/spring/aop.html)

[Interface MethodInterceptor](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodInterceptor.html)

[Class MethodProxy](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodProxy.html)

[从头捋了一遍 Java 代理机制，收获颇丰](https://www.cnblogs.com/cswiki/p/14461856.html)

[聊聊cglib动态代理遇到的坑](https://juejin.cn/post/6844903565996081160)

[cglib动态代理中invokeSuper和invoke的区别](https://blog.csdn.net/z69183787/article/details/106878203/?spm=1001.2101.3001.4242)

[解析JDK动态代理实现原理](https://www.cnblogs.com/yeyang/p/10087293.html)