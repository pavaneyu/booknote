---
title: Mybatis笔记	
date: 2022年2月16日15:46:07
tags: []
categories: 学习笔记

---

<!--more-->

# 流程

## 1.maven依赖 和build配置  :解析java目录下的xml

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
```

## 2.entity   dao接口  和对应的mapper

+ `<mapper namespace="com.ouyang.dao.UserDao">`对应接口
+ `<select id="getUserBy" parameterType="map" resultType="User"> `id对应方法名,resultType返回的entity

```java
public interface UserDao {

    @Select("select * from user ")
    List<User> getAllUsers();

    /**
     * from 不要写错  form
     * @param id
     * @return
     */
    @Select("select * from user  where id=#{id}")
    User getUserById(@Param("id") int id); //多个参数时要加@Param ,一个可以不加


    @Insert("insert into user values(#{id},#{name},#{pwd})")
    int addUser(User user);

    @Update("update user\n" +
            "        set name = #{name},pwd=#{pwd}\n" +
            "        where id=#{id};")
    int updateUser(User user);

    @Delete("delete from user where id=#{id}")
    int deleteById(int id);

    /**
    动态SQL,根据传入的动态参数,返回动态结果
     */
    List<User> getUserBy(Map<String, String> map);

}
```

```java
<!--id为接口的方法名,不要写错,返回类型为User的一个或多个对象-->
<mapper namespace="com.ouyang.dao.UserDao">
<!--    <select id="getAllUsers" resultType="com.ouyang.entity.User">-->
<!--        select * from user-->
<!--    </select>-->
<!--    <select id="getUserById" resultType="com.ouyang.entity.User" parameterType="int">-->
<!--        select * from user u where id=#{id}-->
<!--    </select>-->
<!--    <insert id="addUser" parameterType="com.ouyang.entity.User" >-->
<!--        insert into user values(#{id},#{name},#{pwd})-->
<!--    </insert>-->
<!--    <insert id="updateUser" parameterType="com.ouyang.entity.User">-->
<!--        update user-->
<!--        set name = #{name},pwd=#{pwd}-->
<!--        where id=#{id};-->
<!--    </insert>-->

<!--    <delete id="deleteById" parameterType="int">-->
<!--        delete from user where id=#{id}-->
<!--    </delete>-->

    <select id="getUserBy" parameterType="map" resultType="User">

        select * from user
#    传入一个参数map,括号里的形参是map的key,实参是map的value
        <where>
            <if test="id!=null">
                and id=#{id}
            </if>
            <if test="name!=null">
                and name=#{name}
            </if>
        </where>
    </select>

    <cache eviction="LRU" flushInterval="6000" size="512" readOnly="true"></cache>
</mapper>
```



## 3.配置文件

- 引入外部文件(可选)

- 设置(日志实现)

- 类型别名

- 数据库连接信息

- 注册每个entity对应mapper

  

  

  ```xml
  <configuration>
  <!--    <settings>-->
  <!--        <setting name="logImpl" value="LOG4J"/>-->
  <!--    </settings>-->
      <typeAliases>
          <typeAlias type="com.ouyang.entity.User" alias="User"/>
  <!--        扫描包里面的类,自动将类名作为别名,user也能识别-->
  <!--        也可以直接在entity上面加@Alias注解起别名-->
  <!--        有默认的别名,int ,map-->
          <package name="com.ouyang.entity"/>
      </typeAliases>
      <environments default="development">
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/db01"/>
                  <property name="username" value="root"/>
                  <property name="password" value=""/>
              </dataSource>
          </environment>
      </environments>
      <mappers>
          <mapper resource="com/ouyang/dao/UserMapper.xml"/>
  <!--        <mapper resource="com/ouyang/dao/StudentMapper.xml"/>-->
          <!--        用接口映射mapper.xml时候必须同名-->
          <!--        <mapper class="com.ouyang.dao.UserDao"/>-->
  <!--        扫描包下所有接口,从而映射xml,同样要求全部同名-->
  <!--        <package name="com.ouyang.dao"/> -->
      </mappers>
  </configuration>
  ```

  ## 4.工具类getSession
```
    static SqlSession getSession() {
        try {
            return new SqlSessionFactoryBuilder()
                    .build(Resources.getResourceAsStream("mybatis-config.xml"))
                    .openSession();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
```
## 5.使用

```java
       SqlSession session = MybatisUtils.getSession();


        assert session != null;
        UserDao userDao = session.getMapper(UserDao.class);
        List<User> allUsers = userDao.getAllUsers();


        for (User user : allUsers) {
            System.out.println(user);
        }
        System.out.println(userDao.getUserById(1));



        int b = userDao.updateUser(new User(2, "b", "122310"));
        System.out.println(b>0?"updated!!":"fail");

        int i = userDao.deleteById(3);
        System.out.println(i+" row deleted");



        Map<String, String> map = new HashMap<>();
        map.put("id", "2");
        map.put("name", "b");
        List<User> userBy = userDao.getUserBy(map);
        System.out.println(userBy);


        session.commit();
        session.close();

    }

    @Test
    public void testAdd() {
        SqlSession session = MybatisUtils.getSession();
        UserDao mapper = session.getMapper(UserDao.class);
        int i = mapper.addUser(new User(7, "asd", "asd"));
        System.out.println(i+" row infected");

        session.commit();
        session.close();
    }
```

注意:
1.万一数据库字段名  跟entity字段名不一致  使用resultmap ,将返回的不匹配的数据库字段映射成对应的entity字段名,从而返回一个entity
2.接口对应的mapper.xml 可以由在接口上加注解代替,注意注册接口mapper
反射 实现的动态代理  
```
    @Select("select * from user ")
    List<User> getAllUsers();
```


# 多对一 表  查询  1.子查询  2.连表查询  

![image.png](https://s2.loli.net/2022/02/16/a2vLsiE8SwNFP6X.png)

![image.png](https://s2.loli.net/2022/02/16/Fg3ioN4zXJVbYPB.png)



## sql片段 应用
![image.png](https://s2.loli.net/2022/02/16/XO7TBpDRdmh5UjF.png)


## 缓存
一级缓存: session 级别  DML会刷新缓存,回话关闭,一级缓存才会保存到二级缓存中
二级缓存 mapper级别  对应entity需要序列化   缓存保存在map里  

    <cache eviction="LRU" flushInterval="6000" size="512" readOnly="true"

缓存策略 :
LRU 默认,最近最少使用   移除长时间不被使用的
FIFO,  先进先出,  移除先进来的
![image.png](https://s2.loli.net/2022/02/16/S5W6AugR1op3Ced.png)

  