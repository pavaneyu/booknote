---
title: 面试之Java语言
date: 2022年2月20日15:44:18
tags: []
categories: 学习笔记
---

[toc]

# 1.equals与==

+ ==比较地址,即比较是否为同一个对象     equals用来比较内容,object里默认用==实现,往往需要重写

# 基本类型 字节数

short   2

int      4

long   8



float    4

double  8



byte   1

char   2



# String

+ final  不可变   
  +   多线程安全 
  +    实现字符串常量池   
  + 不可变的话, 保证hashcode唯一 ,所以hashmap 的key 常用string
  + 文件名  路径  url  都是用字符串存储  如果可变,造成安全问题 

+ 有一个常量池 ,没有找到才会新建一个对象   



# 什么是nio  与传统io区别

bio  阻塞io  

nio   非阻塞IO   线程发起io请求后立即返回   直到cpu做好io准备     是多路复用的基础

aio   异步非阻塞io

# 反射优缺点  哪里用到

+ 动态加载类     增加灵活性  提高复用率
+  性能问题   
+ 可以访问私有的属性和方法,存在内部暴露问题
+ sprig加载配置文件   jdbc 驱动类是通过反射加载的  

# 面向对象特征

1.封装   代码复用性

2. 继承代码复用性

3.多态  可移植性,健壮性   变异类型  与运行类型不一样   

# 泛型擦除

+ **泛型信息只存在于代码编译期，在进入 JVM 之前，与泛型相关的信息会被擦除掉**
+ 为了防止过多创建不同的类,造成过度消耗

# 对象内存结构

+ 对象头 包括  markword 和classPointer
+ 实际数据
+ 对齐填充

# 重写了equals()，还要重写hashCode()？

- hashcode()方法返回hash值  ,可以定位索引位置,提高查询效率
-  hash 有个特点,相同对象hash一定一样,不同对象hash值基本不一样
- 当发生hash collision  就要用equals判断两个对象内容是否相等
- 如果两个对象内容相同，地址不同 ,  equals为true的         但是hashCode比较是不相等的
- 这就矛盾了,违反了 equals 为 true ， hashCode 也要相等的原则





往HashMap添加元素的时候，需要hashcode方法定位到在数组的位置

没有重写 hashCode 方法，两个对象equans为true   那么就无法定位到同一个位置，集合还是会插入元素

出现重复元素,重写的equals方法就没意义了



# 3.Collecttions单列集合

- List,线程不安全,有序,可以重复
  - arrylist 底层由数组实现,根据下标来索引,所以查询效率高,增删效率低
  - 扩容:默认容量0,第一次添加扩到10,此后1.5倍扩容  指定容量时,1.5扩容  
    - LinkedList 底层是双向链表,增删效率高,查询效率低


+ Set,HashSet   无序,不可以重复

  + *hashset*底层是hashmap ,而hashmap底层是数组+链表+红黑树

    ​                                                                                        

  + 添加一对key  value的话 

    1.得到  key  hash值, 再转成索引值    (key---hashcode--二次hash--余容量-1-----按位余----索引值(桶下标))

  + 2.查table    没有就添加(创建Node并返回)      有就比较valueObject       

  + 3.相同就覆盖原来的valueObject         不同   判断链表高度插入链表   
  
  + 插入链表时候   1,7是头插法   1.8是尾插法 
  
    ​																						      
  
  + 底层hashmap的扩容:
  
    + 第一次扩到16,到临界值后双倍扩容,
    + 临界值为当前容量*加载因子,默认0.75
    + 如果链表高度超过8,且数组长度大于64,转成红黑树
  
  # 加载因子0.75?
  
  + 在空间占用 与查询时间  取一个平衡
  + 如果快满了才扩容,链表就会比较长 影响查询时间
  + 如果才一半就去扩容 ,那么扩容就会频繁 ,占用空间变多

# 如何线程安全地使用hashmap

+ 直接加锁使用   Collections.synchronizedMap(hashMap)  

+ 使用ConcurrentHashMap 

  

  # ConcurrentHashMap 的实现

  + jdk 1.7 中，由 Segment 数组和 HashEntry 数组结构构成。采取分段锁来保证安全性。

  +  jdk1.8之后使用 Node 数组+链表+红黑树    并发控制使用 Synchronized 和 CAS 实现

# 集合类的线程安全问题

+ 第一代  考虑到了线程安全
vector hashtable 	
 方法全部sync  效率较低

+ 第二代非线程安全
ArrayList  HashMap  但是效率高
要线程安全使用可以用Collecttions的静态方法 sync

+ 第三代并发包下的线程安全集合
ConcurrentHashMap  CopyOnWriteArrayList

# JDK1.8新特性

+ 接口默认方法
+  lambla表达式简化匿名内部类    本质是一个函数式接口     就是只有一个方法的接口,描述了方法的特征  比如Comsumer  Function  当然如果某个方法满足特征,可以直接方法引用
+ Stream流   
+ 新日期API optional 类 



# 双列集合map

+ hashmap继承自AbstractMap
  +   线程不安全,允许空值,底层是数组+链表+红黑树
  + 默认大小16,到临界值  ,2倍扩容

+ concurrentHashMap   线程安全
+ hashtable  继承自Dctionary     
  + 线程安全  不允许有空值
  + 全局锁  所以效率不高
  + 默认大小11,到临界值扩容,2n+1

# 单例

1.构造器私有化

2.私有静态volatile 成员对象

3.双重检查,在第一次判空之后使用同步锁,保证线程安全

4.也可以静态内部类来实现 

5.枚举类也是一种单例模式

# 接口和抽象类

+ 抽象类必须至少有一个抽象方法,子类必须重写所有抽象方法
  
  + 接口中方法默认抽象方法, 接口实现类必须重写抽象方法方法
  
  default可以有默认实现  
  
+ 因为单继承,但是可以实现多个接口

+ 抽象类 用来描述实体**本质概念**,比如猫科动物,

+ 接口用来描述**操作特征**,比如会飞,那么实现接口的都有一个飞的方法,鸟会飞,飞机也会飞

# int和Integer

类型    -> 比较  ->  缓存           使用  (定义时候初始 |使用地方)

+ int是基本类型, 存在栈上        Integer是包装类型,是面向对象的,存在堆里
+ int 比较内容用==   Integer比较内容用equals
+ Integer  在自动装箱的时候      在-128到127内有缓存, 在这中间不新建对象   
+ 未初始化 int默认为0,Integer 默认为null
+ 临时使用时用int, 分配在栈上   定义pojo时用Integer,保证序列化

# String，StringBuffer，StringBuilder的区别

可变   线程安全  效率   场景

+ String          不可变  每次更改都会创建新的String     适合操作少量数据     另外两个是可变的
+ StringBuffer   线程安全       多线程操作大量数据
+ StringBuilder  虽然线程不安全  效率最高       只适合单线程操作大量数据



# 浅拷贝  深拷贝

+ 浅拷贝 属性地址是一样的
+  如果改变a的属性,那么b的属性也会跟着变
+ 深拷贝就不会出现这种问题 ,可以继承cloneable接口,在重写的clone方法里面把属性进行复制





+ # 自定义注解

+ @interface
+ 四个元注解属性   
  + target  注解目标   
  + retention  注解生命周期   到源文件class文件还是到运行期
  + documented  注解时候生成带javadoc文档
  + inherited  是否被继承



# Class对象获取

+ 类名.class
+ 对象.getClass()
+ Class.forName()

# 运行时异常

+ 非法参数

+ 类型转换   数组越界   并发修改 

+ 

  								非法参数 uuid
  		 空指针 obj=null
  		 (类转换)
  		 算数 下标 
  		 并发修改异常



# 多表查询  

# mysql常用函数 

# mysql引擎

innodb  支持行锁 和表锁   默认隔离级别  可重复读

Mylsam  只支持表锁   不支持事务   

![image-20220222211908084](https://s2.loli.net/2022/02/22/VmiCusMxhb4okPD.png)

# 覆盖索引?索引下推?

# redis  底层推送机制

redis事务不支持回滚

# JWT

# TCP四次挥手?







![image-20220223130336737](https://s2.loli.net/2022/02/23/V8nGuJAQWgfo3ae.png)
