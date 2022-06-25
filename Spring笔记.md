---
title: Spring笔记	
date: 2022年2月16日15:54:04
tags: []
categories: 学习笔记

---

# 概述

IOC  控制反转  :创建对象的过程交给Spring

AOP面向切面:不修改源代码进行内容增强 

<!--more-->

# 原始XML注册对象

xmlBean里用setter方法设置属性值
```
      <property name="name" value="Ouyang">
      </property>

        <property name="name">
            <null></null>   设为null
        </property>

         <property name="name">
            <value><![CDATA[<<Hello>>]]></value>  设置特殊值 或者用转义
        </property>
      property内部可以再嵌套一个bean  表示 属性是个类




    <property name="likes">
        <array>                                       设置数组属性
            <value>吃饭</value>
            <value>睡觉</value>
        </array>
    </property>
       
        <property name="map">         设置map属性
            <map>
                <entry key="A" value="B"></entry>
                <entry key="C" value="D"> </entry>
            </map>
        </property>

       <property name="list">    
            <list>                  设置list<User>属性,先在外层新建好几个Userbean对象
                <ref bean="user1"/>
                <ref bean="user2"/>
            </list>
        </property>
    <bean id="user2" class="com.ouyang.User">   
<!--        对象名            对象类型-->
        <property name="username" value="lwj"/>  
<!--        属性名              属性值-->
  
  不在list里面对元素赋值 先赋值好再直接引用行不行?
        <property name="list" ref="userList"/>

    <util:list id="userList">
        <bean id="user3" class="com.ouyang.User">
            <property name="username" value="aa"/>
        </bean>
        <bean id="user4" class="com.ouyang.User">
            <property name="username" value="bb"/>
        </bean>
    </util:list>
```
## xml里用有参构造器设置属性值

```
        <constructor-arg name="name" value="Liu"></constructor-arg>
```
## xml注入集合属性

## 工厂bean对象: 

xml定义的factorybean接口实现类型(真正的返回类型是方法里面)

 可以与java里返回的对象类型不一样 
在java代码里,从文件找,竟然找到一个工厂bean ,那就找工厂的产品 !

```
<bean id="myFactoryBean" class="com.ouyang.myFactoryBean">
</bean>
```
 ```
public class myFactoryBean implements FactoryBean<User> {

    @Override
    public User getObject() throws Exception {
        User user = new User();
        user.setUsername("oii");
        return user;
    }
 ```
```
        ApplicationContext context = new ClassPathXmlApplicationContext("com/ouyang/factoryBean.xml");
        User user = context.getBean("myFactoryBean", User.class);
```
## **默认bean是singleton,单例,加载xml时自动创建singleton**

 bean属性     scope="prototype"  设置bean为原型  可以有多个对象 ,getBean()才会创建对象

## **bean生命周期**

1.xml构造器创建实例
2.xmlsetter设置属性(包括引用其他bean的属性)
3.配置初始化方法  bean属性 init-method=""
4.getBean获取并使用bean
5.配置销毁bean方法  bean属性 destroy-method="getObject"  context.close()

## **xml中为所有bean添加后置处理器**

```<bean id="myBeanPost" class="com.ouyang.myBeanPost"/>```

```
实现类
public class myBeanPost implements BeanPostProcessor {
    @Nullable
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之前调用方法");
        return bean;
    }

    @Nullable
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之后调用方法");
        return bean;
    }
}
```
## **xml自动根据外部bean配置类对象属性** (很少用   ,往往基于注解注入属性)

bea  n 属性 autowire="byName"    根据外部bean id 配置这个bean中同名属性
bean 属性 autowire="byType  "    根据外部bean 类 配置这个bean中同类属性
第二种只能有一种外部bean

## **xml引入外部文件**

```
    <context:property-placeholder location="classpath:com/ouyang/jdbc.properties"/>   src目录下
    <bean id="dataSource" class="com.ouyang.User">
        <property name="username" value="${userName}"/>   key值
    </bean>
```


# **注解创建Bean对象**

## 配置类开启需要组件扫描 

```
@Configuration
@ComponentScan(basePackages ="com.ouyang")
```
## 需要创建对象的类上写注解         来创建对象

```
@Component(value = "user") //value(变量名)可以不写,默认为类名(首字母转换为小写)
在service类上写@Service   
在Dao类 写@Repository
```
## 手动获取对象

 context.getBean(变量名,bean类型.class)

补充:组件扫描自定义规则 ,比如只扫描Component注解
```
    <context:component-scan base-package="com.ouyang" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
    </context:component-scan>
```

## 自动获取对象(自动注入)

在需要设置属性的对象上,写注解
@Autowired  根据类型自动注入,如果有多个同类型,需要配合  @Qualifier精准注入特定名字的bean

@Resource   注入特定名称的bean
@Value          注入普通类型   

**例如 MyService类中有个Dao类型的属性 ,用注解创建Bean对象,并设置该属性的值**

```
@Configuration
@ComponentScan(basePackages = {"com.ouyang"})
public class config {
}
```
```
@Repository  //
public class Dao  {
    public void add() {
        System.out.println("dao is adding");
    }
}
```

```
@Service(value = "myy")
public class MyService {
//    @Autowired  //按类型注入Bean对象,
//    @Qualifier(value = "dao")//如果有多个,需要写明对象名称,需要跟 @Autowired一起使用
    @Resource(name = "dao")
    private Dao basicBao;

    @Value(value = "32")
    public int anInt;

    public void add() {
        System.out.println("service add");
        basicBao.add();
    }
}
```
```
public class AnnotationTest {
    public static void main(String[] args) {

      //  ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("com/ouyang/Annotation/bean2.xml");
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(config.class);

        MyService myService = context.getBean("myy", MyService.class);
        myService.add();
        System.out.println(myService.anInt);
    }
}
```

# AOP
连接点   可以被增强的方法
切入点   实际被增强的方法
通知(增强)  增强的逻辑部分
   前置 
 后置通知 
环绕(前后置)  
异常通知 
 finally通知

## 1.配置文件里开启切面自动代理

```
@Configuration
@ComponentScan(basePackages = {"com.ouyang.AOP"}) //如果父目录下有相关同名类,一定要细化扫描范围
@EnableAspectJAutoProxy //有Aspect注解的类自动生成代理
public class Config {

}
```
## 2.需要增强的类和方法  

```
@Component(value = "user")
public class User {
    public void use() {
        System.out.println("using");
    }
}
```
## 3. @Aspect 提供增强的类    增强方法注解@before @After @Around 表示怎样增强

```
@Component
@Aspect
@Order(1) //如果有多个增强类对同一个方法进行插入增强,设置优先级
public class UserProxy {
    @Pointcut(value = "execution(* com.ouyang.AOP.User.use(..))")
    public void pointCut() {
    }

    @Before(value = "pointCut()")
    //切入点表达式 (权限修饰+被增强类路径+被增强方法+方法参数列表)
    public void before() {
        System.out.println("before using");
    }

    @AfterReturning(value = "pointCut()")
    public void 返回通知() {
        System.out.println("前一个执行成功");
    }

    @After(value = "pointCut()")
    public void 最终通知() {//不管被插入的地方 有无异常都执行
        System.out.println("finally");
    }

    @AfterThrowing(value = "pointCut()")
    public void exception() {
        System.out.println("an exception!");
    }

    @Around(value = "pointCut()")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {

        System.out.println("环绕之前");
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后 ");
    }

}
```
## 4.使用及结果

```
public class TestAOP {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        User user = context.getBean("user", User.class);  //得到被增强的对象
        user.use();

    }
}
```
```kotlin
环绕之前
before using
using
前一个执行成功
finally
环绕之后 


```

5.AOP底层原理  

jdk的一个动态代理和这个cglib的一个动态代理。
被代理类如果有实现接口，就是jdk代理，如果有继承父类，就是cglib代理，默认是jdk代理

# 结构分层

 **应用层  调用逻辑层方法
  逻辑指挥层 进行业务操作  只调用实现层方法
  具体实现层(包括数据库实现和其他实现 ) 调用API   不涉及业务逻辑 
  抽象层规定脉络**

# 事务@Transactional

## ACID特性

原子性 :不可分割  atomicity
一致性:事务前后总量一致consistency
隔离性:多个事务不会互相影响 isolation  
持久性:事务提交后,数据库里永久保存 durability

## 事务参数

1.propagation  传播   多事务方法直接调用,如何管理  默认required
![image.png](https://upload-images.jianshu.io/upload_images/19490681-8a509372829d34ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.ioslation 隔离   避免并发时多事务之间  产生的问题
    脏读 一个未提交的事务A读到了另一个未提交的事务B的数据
    不可重复读:一个未提交的事务A读到了   已经提交的事务B的修改的数据
    幻读:一个未提交的事务A读到了另一个未提交的事务B的提交的数据
![image.png](https://upload-images.jianshu.io/upload_images/19490681-d1e4b11529ba2b3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.timeout 默认-1 不计算超时   以秒为单位
4.readOnly  默认false
5.rollBackFor  哪些异常需要回滚

# **完全注解步骤**

## 1.config类







 几个Bean DataSource  jdbcTemplate   TransactionManager

```
/**
 * 2022-02-05 16:28
 */
@Configuration
@ComponentScan(basePackages = "com.ouyang.JDBCTemplate使用")
@EnableTransactionManagement
public class TxConfig {
    @Bean
    public DruidDataSource getDataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        druidDataSource.setUrl("jdbc:mysql://localhost:3306/db01");
        druidDataSource.setPassword("");
        druidDataSource.setUsername("root");
        return druidDataSource;
    }

    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }
}
```
## 2.业务逻辑层  开启事务

```
@Service(value = "accService")
@Transactional( readOnly = false,propagation = Propagation.REQUIRED,isolation = Isolation.REPEATABLE_READ)

public class AccService {

    @Autowired
    private AccDao accDao;

    public void transMoney(String from, String to, int money) {
        accDao.addMoney(money, from);
        //  int i = 10 / 0;
        accDao.reduceMoney(money, to);
    }

}

```

## 3.抽象层

```
public interface AccDao {
    void addMoney(int money,String name);

    void reduceMoney(int money,String name);
}
```
## 4.数据操作层

```
@Repository
public class AccDaoImpl implements AccDao{
    @Autowired
    private JdbcTemplate jdbcTemplate;


    @Override
    public void addMoney(int money, String name) {
        int update = jdbcTemplate.update("update table_account set money=money-? where name=? ", money, name);
        System.out.println(update);
    }

    @Override
    public void reduceMoney(int money, String name) {
        int update = jdbcTemplate.update("update table_account set money=money+? where name=? ", money, name);
        System.out.println(update);
    }
}
```
## 5.应用层

 ```
public class AccTest {
    public static void main(String[] args) {
//        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("com/ouyang/JDBCTemplate使用/config/Empconfig.xml");
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(TxConfig.class);

        AccService accService = context.getBean("accService", AccService.class);

        accService.transMoney("oyy","liu",100);
    }
}

 ```

# 在Spring中注册自己new出来的对象

```
        GenericApplicationContext context = new GenericApplicationContext();
        context.refresh();

        //第三个参数是一个Supplier函数
        context.registerBean("varUser", Account.class, Account::new);
        Account account = context.getBean("varUser", Account.class);
        System.out.println(account);
```

整合JUnit5 的注解

```
//@ExtendWith(SpringExtension.class)
//@ContextConfiguration(classes = TxConfig.class)
@SpringJUnitConfig(classes = TxConfig.class)//相当于上面两行注解
public class 整合JUnit5 {
    @Autowired
    private AccService accService;

    @Test 
    public void tr() {
        accService.transMoney("oyy", "liu", 300);
    }
}
```

# WebFlux  响应式编程(观察者模式)  异步非阻塞
在Spring里面分别用注解和函数式编程实现
### 背景介绍
![image.png](https://s2.loli.net/2022/02/22/wBa4XkLhxCeKruJ.png)
![image.png](https://s2.loli.net/2022/02/22/t7gW92EwpNLoMru.png)


![image.png](https://s2.loli.net/2022/02/22/KzqF4HSx2NTevlM.png)
![image.png](https://s2.loli.net/2022/02/22/9PJXMxtBeD1qohk.png)
###  结构分析
1.核心控制器DispatchHandler 实现了WebHandler接口的handle方法![image.png](https://s2.loli.net/2022/02/22/s8tYhkrpuqKdTRe.png)
![image.png](https://s2.loli.net/2022/02/22/yOUBlT7w1uakjWV.png)
![image.png](https://s2.loli.net/2022/02/22/axQEWXJfTdoshB9.png)



## 注解 方式实现webflux,从浏览器查询数据库
### 1.依赖

```
 <dependencies>

        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>

        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>

        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>

        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>

        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.21</version>
        </dependency>
    </dependencies>
```

### 2.启动端口号   server.port=8081

### 3.数据库相关配置类

```

@Configuration
@ComponentScan(basePackages = "com.ouyang.reactordemo")
public class Config {
    /**
     * 数据源  绑定数据库参数
     * @return
     */

    @Bean
    public DruidDataSource getDataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        druidDataSource.setUrl("jdbc:mysql://localhost:3306/db01");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("");
        return druidDataSource;
    }

    /**
     * {@link JdbcTemplate} 绑定数据源
     * @param dataSource
     * @return
     */
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    /**
     * 事务管理器绑定 数据源
     * @param dataSource
     * @return
     */
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```
### 4.数据操作类Dao  注入jdbctemplate

```

@Repository(value = "bookDaoImpl")
public class EmpDaoImpl implements EmpDaoIn {
    @Autowired //注入的bean在config里创建好了
    private JdbcTemplate jdbcTemplate;

    @Override
    public void add(Employee employee) {
        int update = jdbcTemplate.update(
                "insert into employee values (?,?,?)",
                employee.getId(),
                employee.getName(),
                employee.getResume()
        );
        System.out.println(update+" rows added");

    }

    @Override
    public void deleteByID(int i) {
        int update = jdbcTemplate.update("delete from employee where id=?", i);
        System.out.println(update +" row deleted");
    }

    @Override
    public void update(Employee employee) {

    }

    @Override
    public void select(Employee employee) {

    }

    @Override
    public int selectCount() {
        return jdbcTemplate.queryForObject("select count(*) from employee", Integer.class);
    }

    @Override
    public Employee queryOneByID(int id) {
        Employee employee = jdbcTemplate.queryForObject(
                "select * from employee where id=?",
                new BeanPropertyRowMapper<>(Employee.class),
                id
        );
        return employee;
    }

    public List<Employee> querySomeByName(String name) {
        return jdbcTemplate.query(
                "select *from employee where name =?",
                new BeanPropertyRowMapper<>(Employee.class),
                name
        );
    }

    public List<Employee> queryAll() {
        return jdbcTemplate.query("select * from employee", new BeanPropertyRowMapper<>(Employee.class));
    }

    @Override
    public void addBatch(List<Object[]> batchArgs) {
        int[] ints = jdbcTemplate.batchUpdate("insert into employee values (?,?,?)", batchArgs);
        System.out.println(Arrays.toString(ints));
    }
}

```
### 5.业务逻辑层Service   注入DAO

```

@Repository
public class MyService implements EmpService {
    @Autowired
    private EmpDaoImpl empDao;

    @Override
    public Mono<Employee> getEmpByID(int id) {
        return Mono.just(empDao.queryOneByID(id));
    }

    @Override
    public Flux<Employee> getAllEmps() {
        return Flux.fromIterable(empDao.queryAll());
    }

    @Override
    public Mono<Void> addEmp(Mono<Employee> empMono) {
        return empMono.doOnNext(employee -> empDao.add(employee)).thenEmpty(Mono.empty());
    }

    @Override
    public Flux<Employee> getEmpsByName(String name) {
        return Flux.fromIterable(empDao.querySomeByName(name));
    }


}

```

### 6.controller 层  

@RestController 告诉spring 这个是导航类
 注入service   实现 根据url导航到service具体方法 

```

@RestController
public class EmpController {

    //    @Resource(name = "my")
    @Autowired
    public MyService myService;

    @GetMapping("/emp/id={id}")  //浏览器查询匹配到就会进入这个方法
    public Mono<Employee> getEmpByID(@PathVariable int id) { //表示id从地址栏路径获取
        return myService.getEmpByID(id);  //
    }

    @GetMapping("/emps")
    public Flux<Employee> getAllEmps() {
        return myService.getAllEmps();
    }

    @GetMapping("emp/name={name}")
    public Flux<Employee> getEmpsByName(@PathVariable String name) {
        return myService.getEmpsByName(name);
    }

    @PostMapping("/add")
    public Mono<Void> saveEmp(@RequestBody Employee employee) {
        return myService.addEmp(Mono.just(employee));
    }


}
```
## 用函数式编程而非注解实现WebFlux(使用模拟数据)

0.测试API

```java
public class Test {
    public static void main(String[] args) {
        /**
         * Flux容器注入对应属性方法
         */
        Flux.just(1, 2, 3).subscribe(System.out::println);
        Mono.just(8).subscribe(System.out::println);
        Flux.fromArray(new Integer[]{5, 6, 2}).subscribe(System.out::println);
        Flux.fromIterable(Arrays.asList(5, 6, 9)).subscribe(System.out::println);
        Flux.fromStream(Stream.of(5, 8, 8)).subscribe(System.out::println);
    }
}
```

1.服务器绑定adapter,adapter绑定路由 
2.路由绑定handler   ,通过Url类型  导航到 handler 具体方法 ,返回一个RouterFunction<ServerResponse>
3.handler 绑定 一个service ,根据Url里面参数,用service获取一个Flux
再把Flux统一转成Mono<ServerResponse>

```

public class Server {

    public static void main(String[] args) throws IOException {
        new Server().createReactorServer();
        System.out.println("enter to exit");
        System.in.read();
    }


    /**
     * 路由 指明由地址栏解析的请求具体 去执行那个方法  bind handler  ,
     * @return
     */
    public RouterFunction<ServerResponse> routerFunction() {
        EmpHandler handler = new EmpHandler(new MyService());

        return RouterFunctions.route(
                RequestPredicates.GET("/emps/id={id}").and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
                handler::getEmpByID
        ).andRoute(
                RequestPredicates.GET("/emps").and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
                handler::getAllEmps
        );
    }

    /**
     * 适配器 binds with router
     */
    public void createReactorServer() {

        HttpHandler httpHandler = RouterFunctions.toHttpHandler(routerFunction());
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);

        //server binds adapter
        HttpServer.create().handle(adapter).bindNow();
    }

}
```
```

public class EmpHandler {
    private final EmpService empService;  //handler binds with service

    public EmpHandler(EmpService empService) {//这里的形参是接口,传入要传入一个具体实现类
        this.empService = empService;
    }

    public Mono<ServerResponse> getEmpByID(ServerRequest request) {
        int id = Integer.parseInt(request.pathVariable("id"));
        Mono<Employee> employeeMono = empService.getEmpByID(id);
//        employeeMono.flatMap( employee -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(employee,employee.getClass()))
        return employeeMono.flatMap( employee -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(fromObject(employee)))
                .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> getAllEmps(ServerRequest request) {
        Flux<Employee> allEmps = empService.getAllEmps();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(allEmps, Employee.class);
    }

    public Mono<ServerResponse> addEmp(ServerRequest request) {
        Mono<Employee> employeeMono = request.bodyToMono(Employee.class);
        return ServerResponse.ok().build(empService.addEmp(employeeMono));
    }


}
```
```
@Repository
public class MyService implements EmpService {

    private List<Employee> list = new ArrayList<>();
    
    {
        list.add(new Employee(1, "a", "A"));
        list.add(new Employee(2, "b", "B"));
        //list.add(new Employee(3, "c", "C"));
    }
    
    public List<Employee> getList() {
        return list;
    }


    @Override
    public Mono<Employee> getEmpByID(int id) {
        for (Employee employee : list) {
            if (employee.getId() == id) {
                return Mono.justOrEmpty(employee);
            }
        }
        return null;
    }
    
    @Override
    public Flux<Employee> getAllEmps() {
        return Flux.fromIterable(list);
    }
    
    @Override
    public Mono<Void> addEmp(Mono<Employee> empMono) {
        return empMono.doOnNext(employee -> list.add(employee)).thenEmpty(Mono.empty());
    }
```
