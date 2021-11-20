# Spring常见问题

#### Q：Spring重要模块？

- **Spring Core：** 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IOC 依赖注入功能。
- **Spring AOP** ：提供了面向方面的编程实现。
- **Spring JDBC** : Java数据库连接。
- **Spring Web** : 为创建Web应用程序提供支持。
- **Spring Test** : 提供了对 JUnit 和 TestNG 测试的支持。

#### Q：Spring IoC和依赖注入(DI)？

控制反转，将控制权力（bean的创建、管理）交给Spring容器，由之前的自己主动创建对象，变成现在被动接收别人给我们的对象的过程。依赖是程序需要的资源（对象、数据、文件），将其注入到程序中。DI是IoC的实现方式，这实现了对象之间的解耦。

#### Q：Spring中bean的作用域有哪些？

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session：全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

#### Q：Spring单例bean的线程安全问题？

默认是单例Bean，多个线程操作会出问题，采用ThreadLocal变量。

#### Q：Spring中Bean的生命周期？

![image-20211120185854871](C:\Users\libch\AppData\Roaming\Typora\typora-user-images\image-20211120185854871.png)

**1、实例化：**ApplicationContext容器，当容器启动结束后，便实例化所有的bean，有点像new。容器通过BeanDefination对象中的信息进行实例化，实例化后的对象被包在BeanWrapper对象中。

**2、填充属性（依赖注入）：**按照Spring上下文对实例化的Bean进行依赖注入。

**3、注入Aware接口：**

检查BeanNameAware接口，调用setBeanName(String beanId)设置bean的ID

检查BeanFactoryAware接口，调用setBeanFactory()设置BeanFactory

检查ApplicationContextAware接口，调用setApplicationContext()，传入Spring上下文

**4、BeanPostProcessor postProcessBeforeInitialzation：**对象使用前的自定义处理，当前正在初始化的bean对象会被传递进来，这个函数会先于InitialzationBean执行，因此称为**前置处理**。 所有Aware接口的注入就是在这一步完成的。

**5、InitializingBean：**BeanPostProcessor的前置处理完成后就会进入本阶段。

调用afterPropertiesSet()，如果bean设置了init-method，会在这个阶段调用。

**6、BeanPostProcessor postProcessAfterInitialzation：**当前正在初始化的bean对象会被传递进来，这个函数会在InitialzationBean完成后执行，因此称为**后置处理**。

此时bean已经就绪可以使用，一直驻留在Spring Context中，直到Context被销毁。

**7、DisposableBean和destroy-method：**bean销毁前的执行逻辑。

#### Q：SpringMVC工作原理？

![图片](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TxiaKwgUUHQX0aVpNnuopm5wvNYJUuopN5R9BDcW2wxDdnEVyzke9a2paria5AcekHmVaFxicu6qV4kA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler`来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

#### Q：@Component和@Bean的区别？

1、作用对象不同：`@Component` 注解作用于类，告诉Spring为这个类创建bean，而`@Bean`注解作用于方法，告诉Spring这个方法会返回一个对象，将其注册为Spring上下文中的bean。

2、`@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中，`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean。

3、`@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用**第三方库中的类需要装配到 `Spring`容器**时，则只能通过 `@Bean`来实现。

#### Q：Spring容器的启动流程？

