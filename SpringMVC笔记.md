---

---

[toc]

# maven依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.1</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <!--不打包进去,由执行服务器provide-->
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring5</artifactId>
            <version>3.0.12.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.1</version>
        </dependency>

    </dependencies>
```



# web.xml  配置

1.注册编码过滤器

```
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <!--    为所有请求设置过滤器解决乱码问题-->
    </filter-mapping>
```

2.注册Servlet

```
    <servlet>

        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--    spring config.xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:MVCConfig.xml</param-value>
        </init-param>
        <!--    servlet  初始化提前到服务器启动时,而不是第一次收到请求-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <url-pattern>/*</url-pattern>
        <!--      为所有servlet设置前端控制器(包含配置文件),里面有解析器和前后缀,编码格式等信息-->
    </servlet-mapping>
```

# java代码代替web.xml

```
/**
 * 代替web.xml
 * 2022-02-09 19:46
 * @author ouyan
 */
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }


    @Override
    protected Filter[] getServletFilters() {
        return new Filter[]{
                new CharacterEncodingFilter("UTF-8",true),
                new HiddenHttpMethodFilter()
        };
    }
}
```



# MVCConfig.xml

```
    <context:component-scan base-package="com.ouyang.demo1"/>
    <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
    <mvc:annotation-driven/>
<!--    注解驱动-->
    <mvc:default-servlet-handler/>
<!--    servlet处理静态资源-->
    <mvc:interceptors>
<!--        <bean class="com.ouyang.demo1.interceptor.MyIntercept"/>-->
<!--        <ref bean="myIntercept"/>    -->
        <mvc:interceptor>
            <mvc:mapping path="/*"/>
            <mvc:exclude-mapping path="/target"/>
            <ref bean="myIntercept"/>
        </mvc:interceptor>
    </mvc:interceptors>

<!--    视图分发器-->
    <bean id="ViewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="order" value="1"/>
        <property name="characterEncoding" value="UTF-8"/>
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                        <property name="prefix" value="/WEB-INF/templates/"/>
                        <property name="suffix" value=".html"></property>
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8"></property>
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
```



# java代码代替MVCConfig.xml

```java
/**
 * 代替SpringMVC的配置文件
 * 2022-02-09 19:47
 * @author ouyan
 */
@Configuration
@ComponentScan("com.ouyang.demo1")
@EnableWebMvc

public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyIntercept())
                .addPathPatterns("/**")
                .excludePathPatterns("/");
    }


    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/index").setViewName("index");
    }

    /**
     * 异常解析器
     * @param resolvers
     */
    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties properties = new Properties();
        properties.setProperty("java.lang.ArithmeticException", "error");
        exceptionResolver.setExceptionMappings(properties);
        exceptionResolver.setExceptionAttribute("exception");
        resolvers.add(exceptionResolver);
    }

    /**
     * 文件上传解析器
     */
    @Bean
    public MultipartResolver multipartResolver() {
        return new CommonsMultipartResolver();
    }

    /**
     * 模板解析器
     *
     * @return
     */
    @Bean
    public ITemplateResolver templateResolver() {
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(ContextLoader.getCurrentWebApplicationContext().getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    /**
     * 模板引擎
     *
     * @param templateResolver 自动注入
     * @return
     */
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    /**
     * 视图解析器
     *
     * @param templateEngine 主动注入
     * @return
     */
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }
}
```

# Controller类的方法返回对应页面的String

## RequestMapping的参数

## 方法的形参

```java
@Controller
/**
 * //@RequestMapping("/my")//类上加mapping 注解 表示里面的所有方法url都是类url的子目录
 * //例如     /my/target才能对应target文件
 */
public class MyController {
    /*
     RequestMapping的参数
      1.value 可以有多个url的集合 都会map到这个方法 ,必要  value = {"/","//"}  失败返回404
        ?模糊匹配单个字符  * 模糊匹配n个字符  [/]**[/]模糊匹配n层目录
      2.method = {RequestMethod.GET,RequestMethod.POST} 也可以用派生注解@GetMapping  拒绝返回405
      3.params = "name"  表示url之后必须有?name=XX   也可以有多个必须要求的参数
        params = "!name"  表示后面不能有name   失败返回400
      4.headers = "AA=BB"   表示请求头里必须包含AA且值必须为BB,否则拒绝访问,并返回404
     */

    @RequestMapping(value = {"/"})
    @GetMapping
    public String index() {   //根据url 返回 网 页文件名 ,之后会自动加上前缀和后缀找到文件地址
        return "index";

    }


    @RequestMapping("/target")
    public String tar(
            String user,
            @RequestParam(value = "passW", required = true, defaultValue = "123456") String psw,
            //解析通过get或者post里面的参数,若形参名跟url里面不一样,要通过注解映射
            //也可以required = false设为非必须, 如果传入null或"",设置默认值
            @RequestHeader("Host") String host, //获取请求头里面的Host信息,这个参数不是url里的
            @CookieValue("JSESSIONID") String jasonID//获取请求头里面的JSESSIONID信息,这个不是参数url里的
            /*
            如果参数列表是entity的字段,形参甚至可以写成 entity对象
             */
    ) {

        System.out.println("user = " + user);
        System.out.println("psw = " + psw);
        System.out.println("host = " + host);
        System.out.println("jasonID = " + jasonID);

        return "target";
    }

    @RequestMapping("/modelAndView")
    public ModelAndView testModelAndView() {
        return new ModelAndView("index", "model", "bbb");
        //在index.html中用 <p th:text="${model}"></p> 就可以得到传进来的 model值  //整个request都有效
        //index.addObject("a", "b");
    }

    @RequestMapping("/session")
    public String session(HttpSession session) {
        session.setAttribute("aaa", "bbb");
        //在index.html中可以用${session.aaa} 获取值  //整个session都有效
        return "index";
    }

    @RequestMapping("application")
    public String app(HttpSession session) {
        session.getServletContext().setAttribute("aaa", "bbb");
        //在index.html中用${application.aaa} 获取值 //整个项目应用都有效
        return "index";
    }

    @RequestMapping("/transmit")
    public String transmit() {
        return "forward:/target";
        //由服务器转发,委托其他页面处理,浏览器不需要重新请求,地址栏不变
    }

    @RequestMapping("/redirect")
    public String redirect() {
        return "redirect:/target";
        //重定向,由浏览器重新请求,地址栏跳转
    }

    @RequestMapping("/requestBody")
    public ModelAndView requestBody(@RequestBody String requestBody) {
        System.out.println("requestBody = " + requestBody);
        return new ModelAndView("index", "model", requestBody);
    }

    @RequestMapping("requestEntity")
    public ModelAndView requestEntity(RequestEntity<String> requestEntity) {
        System.out.println("requestEntity.getHeaders() = " + requestEntity.getHeaders());
        System.out.println("requestEntity.getBody() = " + requestEntity.getBody());

        return new ModelAndView("index", "model", requestEntity);
    }

    @RequestMapping("responseEntity")
    public ModelAndView responseBody(ResponseEntity<String> responseEntity) {
        return new ModelAndView("index", "model", "responseEntity.getHeaders().toString()");
    }

    @RequestMapping("/response")
    @ResponseBody
    public String response() {
        return "response you!!!!! ";
        //返回响应体String ,也可以返回一个对象,解析成json返回
    }
}
```

# html中thymeleaf解析参数

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>nmsl</title>
</head>
<body>
<h1>首页</h1>

<a th:href="@{/emps}"> 所有员工信息  </a>
<!--/*@thymesVar id="model" type="Model"*/-->
<p th:text="${model}"></p>

</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>emps info</title>
</head>
<body>
<table border="1" cellspacing="0" cellpadding="0" style="text-align: center">
    <tr>
        <th colspan="5">emps info</th>
    </tr>
    <tr>
<!--        一行四个th表头-->
        <th>id</th>
        <th>name</th>
        <th>resume</th>
        <th>option</th>
    </tr>
    <!--/*@thymesVar id="emps" type="List"*/-->
    <tr th:each="emp:${emps}">
        <!--allEmps是model携带过来的map的keyName对应的value -->
        <!--/*@thymesVar id="emp" type="com.ouyang.demo1.entity.Employee"*/-->

        <td th:text="${emp.id}"/>
        <td th:text="${emp.name}"/>
        <td th:text="${emp.resume}"/>
        <td>
            <a href="@{/emps/}+${emp.id}">delete</a>
            <a href="@{'/emps/'+${emp.id}}">delete</a>
            <a href="">save</a>
        </td>
<!--        一行四个th元素-->
    </tr>
</table>
</body>
</html>
```

