---
title: mybatis-plus使用笔记
date: 2022年2月16日16:01:15
tags: []
categories: 学习笔记
---

# mapper接口对应一张表,关联entity

```
public interface UserMapper extends BaseMapper<User> {
}
```
<!--more-->

# 使用mapper  操作一张表

```
    @Autowired
    private UserMapper userMapper;

    @org.junit.jupiter.api.Test
    public void test() {
        List<User> users = userMapper.selectList(null);
        users.forEach(System.out::println);
```
# activeRecord  操作主题是某条记录,代替了mapper , entity继承model<>

```
@Data
@TableName("user")//excludeProperty
@AllArgsConstructor
public class User extends Model<User> {
    @TableId(type = IdType.ASSIGN_UUID)//id==primary key
    private Integer id;

    @TableField("name") //默认根据名称自动匹配属性跟列名，默认支持下划线映射驼峰,
    private String name;

    private String pwd;
}
```
# 支持原始自定义mapper和对应xml写SQL
但要注册xml到系统`mybatis-plus.mapper-locations=classpath*:/mapper/**/*.xml
`
# 查询queryWrapper 条件  及分组
```kotlin
@SpringBootTest
public class Test {
    @Autowired
    private UserMapper userMapper;

    @org.junit.jupiter.api.Test
    public void test() {
        List<User> users = userMapper.selectList(null);
        users.forEach(System.out::println);
        //  int insert = userMapper.insert(new User(8, "ff", "dff"));
        //   System.out.println("insert = " + insert);
        HashMap<String, Object> selectMap = new HashMap<>();
        selectMap.put("name", "b");
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //默认为true，表示map的查询条件值为null也强制查询，false表忽略null的查询条件

        queryWrapper
                .exists("select name from user where id>10")
                //只有在exists存在记录的情况下才查询
                .allEq(selectMap, false)//不查询map中值为null的条件
                .eq("pwd", "bbb")//equalQuery
                .ne("id", 5)//notEqual
                .or()//连接两个查询条件,默认是and
                .gt("id", 7)//greaterThan
                .ge("id", 4)//greaterOrEqual
                .lt("id", 1)//lessThan
                .le("id", 1)//lessOrEqual
                .between("id", 1, 7)
                .notBetween("id", 1, 3)
                .like("id", 1)
                .notLike("id", 1)
                .likeLeft("id", 1)//左边模糊查询
                .likeRight("id", 1)//
                .isNull("id")//查询字段值为null的所有记录
                .isNotNull("id")
                .in("id", 1, 2, 4, 5)//id等于给定值的都返回
                .notIn("id", 4, 5, 5, 6)
                .inSql("id", "select id from user where name=b")//相当于in查询,不过后面的列表是查询出来的
                .notInSql("id", "...")

                .orderByAsc("id")
                .orderByDesc("id")

                .last("limit 3");//补充的sql放在末尾执行


        List<User> list = userMapper.selectList(queryWrapper);
        list.forEach(System.out::println);


    }

    @org.junit.jupiter.api.Test
    public void groupBy() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.select("name,count(*) ");
        queryWrapper.groupBy("name");//name相同的作为一条记录
        List<User> users1 = userMapper.selectList(queryWrapper);

        System.out.println("users1 = " + users1);
        users1.forEach(System.out::println);
    }
}

```
# 默认内存分页,可以配置分页拦截器实现物理分页
```
@Configuration
public class Config {

    @Bean
    public PaginationInnerInterceptor paginationInnerInterceptor() {
        return new PaginationInnerInterceptor();
    }
}
```
```java
    @Test
    public void page() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.gt("id", 1);

        Page<User> userPage = userMapper.selectPage(new Page<>(1,3), queryWrapper);
        List<User> records = userPage.getRecords();
        records.forEach(System.out::println);
        System.out.println("userPage.getTotal() 总记录数= " + userPage.getTotal());
        System.out.println("userPage.getPages() 分了几页= " + userPage.getPages());
        System.out.println("userPage.getSize() 每页大小= " + userPage.getSize());
        System.out.println("userPage.getCurrent() 当前页数= " + userPage.getCurrent());
    }

```
# 自动生成entity类  service controller  和原生mapper的xml代码
```java
 FastAutoGenerator
                .create("url", "username", "password")
                .globalConfig(builder -> {
                    builder.author("ouyang") // 设置作者
                         //   .enableSwagger() // 开启 swagger 模式
                          //  .fileOverride() // 覆盖已生成文件
                            .outputDir("D://"); // 指定输出目录
                })
                .packageConfig(builder -> {
                    builder.parent("com.ouyang.mpdemo") // 设置父包名
                           // .moduleName("system") // 设置父包模块名
                            .pathInfo(Collections.singletonMap(OutputFile.mapperXml, "com/ouyang/mpdemo/mapper")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> {
                    builder.addInclude("t_simple") // 设置需要生成的表名
                            .addTablePrefix("t_", "c_"); // 设置过滤表前缀
                })
                //.templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
                .execute();
```