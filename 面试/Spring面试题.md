---
title: Spring和Boot面试题
date: 2022年2月18日14:21:30
tags: []
categories: 学习笔记 
---
[toc]

# Springboot帮你配置了什么?

+ webmvcautoconfuguration   读取webmvcproperties文件中的默认信息 



# boot配置文件加载顺序

+ 当前项目下  config目录有没有配置文件

+ 当前项目下有没有配置文件 

+ resources下的config目录有没有配置文件

+ resources目录下有没有配置文件

  # boot读取配置文件

  @value  @ConfigurationProperties


# BeanFactory 与ApplicationContext

+ ApplicationContext  是BeanFactory的子接口
+ ApplicationContext  启动的时候就实例化bean,不会等到要用时候 
+ 优先选择ApplicationContext 

# Spring   和MVC  和boot  区别

+ Spring 是轻量级java框架,核心是IOC与AOP  还有其他的Spring全家桶
+ MVC是基于Spring的MVC框架,解决web开发问题
+ boot是为了简化开发 可以快速创建应用,  convention  over configuration 原则  
  + 内嵌web容器 摒弃了繁琐的xml配置    starter方便 整合其他框架


# MVC流程 ?   以dispatchServlet为核心   7步

![SpringMVC.jpg](https://s2.loli.net/2022/02/27/trOmcIeqFvKywXJ.png)



# spring中@component和@Bean的区别

1. @Component是在类上面的注解,便于这个组件被扫描到,
2. @Bean是标在配置类的方法上面,表示将方法的返回类型作为一个Bean,默认组件名字为方法名
3. 在将第三方类注册到Spring中,由于不能修改源代码,只能在配置类里面@Bean注册到Spring



# 单例Bean不是线程安全的

+ Bean默认单例,是线程不安全的  ,不要在bean定义成员变量        要定义使用ThreadLocal

+ 如果 @scope作用域设为原型,则每次都创建新的对象,则没有线程安全问题

  

# Spring事务隔离级别 Isolation枚举类的五个值 ?

+ 默认  ,也就是采用后端数据库的隔离级别    

+ 读取未提交内容  --产生**脏读**  幻读  不可重复读 很少用
+ 读取已提交内容 -- 避免了脏读  但是有幻读和不可重复读的问题
+ 可重复读  -默认隔离级别 ,可能产生**幻读**  InnoDB引擎通过多版本并发控制来解决幻读问题
+ 串行化--最高级别的隔离,通过**强制事务排序**,在每个读的数据行加锁,但可能有超时和锁竞争

# Spring事务传播propogation机制

+ 需要 required     默认传播机制    有就加入 没有新建
+ 需要新的  required_new              必须新建,旧的挂起
+ 支持  support          有就加入 没有算了  
+ 不支持  notSupport   不应该运行在事务中  ,有就挂起
+ 强制 mandatory       必须运行在事务中,没有就抛出异常
+ 从不  never                不应该运行在事务中,有就抛出异常
+ 嵌套  nested            嵌套运行





# Spring设计模式

1.工厂模式   BeanFactory   Applicatio  nContext 

2..代理模式   AOP 用的就是动态代理

3.单例  bean 默认就是单例

4.观察者模式  ApplicationEvent   ApplicationListoner

5.适配器模式   

6.装饰者模式   wrapper   decorator

# Spring声明式事务实现原理

+ 通过aop动态代理

# transactional 失效的场景

+ propagation 或者 rollbackfor 属性设置错误

+ 非public方法  因为cglib动态代理 是重写 父类方法的   必须保证能够被重写

+ 被同类方法调用

+ 异常被捕获, 导致无法因为异常而回滚 , 需要再catch里面  手动 回滚

  ![image-20220624180003715](../../../../../AppData/Roaming/Typora/typora-user-images/image-20220624180003715.png)

# Bean的作用域

+ @scope
+ 单例
+ 原型
+ request
+ session
+ globle session

# Bean的生命周期(Spring怎么创建bean)

 doCreateBean() 方法中有四个步骤

+ 如果bean为空,createBeanInstance()**实例化**
+ populateBean给bena**赋值**
+ initializeBean()  **初始化**bean   
  + Aware接口依赖注入
  + beanPostProcesso r 前后处理
  + init-method  执行初始化方法
+ registerDisposableBeanIfNecessary  **销毁**   









# 说说aop

面向切面编程,通过动态代理实现   ,包括切面,切点,前后置通知,增强 组成,

可以在不改变原有类的情况下，能动态的添加某些功能

实际中日志,  监控 ,和声明式事务 都是aop实现的



# AOP动态代理中  jdk与cglib区别

+ jdk:  前提有接口  生成动态代理类proxy   实现目标类的接口  通过反射增强方法
+ cglib:生成的代理类 是  目标类的子类  重写父类方法   



# Spring中类的属性注入

+ 如果属性是必须的,使用构造器注入可以防止NPE
+ 如果属性是可选的,使用setter方法上加@Autowired注入
+ filed 注入不推荐使用

# spring怎么解决循环依赖的

Spring能解决setter注入的循环依赖了，因为实例化和属性赋值是分开的

+ **属性赋值**，过程，Spring 通过**三级缓存**
  + ​	一级缓存 是 单例池  保存          完全初始化完成的bean
  +  二级缓存 是早期曝光对象 保存    仅实例化后的bean
  + 三级缓存是  bean创建工厂  以便以后有机会创建代理对象

# Springboot常用注解

+ 启动类的SpringBootApplication  这个是一个复合注解 他包含   Configuration  EnableAutoConfiguration    ComponentScan
  
+ web相关
  + cotroller  ,RestController 用在控制层,其中后者将返回对象直接作为响应体
  
  + @RequestMapping 请求映射  ,分得更细一点GetMapping  PostMapping
  + 在方法参数里面   @pathvariable     @RequestParam获取请求头里面参数  
  
+ 容器相关  
  + @Repository  用在dao层  @Service用在服务层   通用的话就是@Component 
  + @Autowired 按类型注入  @resouces按名称导入
  + @bean 标在配置类的方法上面,表示将方法的返回类型作为一个组件
  
+ 声明式事务管理  @Transactional

# @Autowired的实现原理

Bean后置处理器   **AutowiredAnnotationBeanPostProcessor**

# SpringBoot自动配置原理

`@SpringBootApplication`等同于下面三个注解：

- `@SpringBootConfiguration`

- `@EnableAutoConfiguration`

  `@ComponentScan`

其中`@EnableAutoConfiguration`是关键内部实际上就去加载`META-INF/spring.factories`文件的信息，然后筛选出以`EnableAutoConfiguration`为key的数据，加载到IOC容器中，实现自动配置功能！

# 自定义starter

+ 包含两个信息   依赖  和 配置
+ @ConfigurationProperties定义配置类  
+ 编写自动配置类   @Configuration   @EnableConfigurationProperties  开启配置属性
+ spring.factories 添加自动配置类的路径
+ 
+ 在maven中intall 到本地仓库  或者deploy到远程仓库



# Servlet 生命周期

+ init方法 初始化
+ service方法  自动派遣与请求对应的（doGet，doPost）方法
+ destroy方法



