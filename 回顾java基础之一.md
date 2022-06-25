---
title: 回顾java基础之一(多态,接口,异常,常用类,集合)
date: 2022年2月16日16:05:18
tags: []
categories: 学习笔记
---

<!--more-->

5.
pubic  谁都可以
protected 同包和子类 可以访问
默认 同包可以访问 
private  只有类内访问 

6.**封装**目的 隐藏实现细节  对数据进行验证安全 不然用户自己赋值,而是让\调用方法实现赋值
- 实现 1.属性私有化,只能自己操作    2.对外提供只getset方法操作
- 在构造器中用set 可以防止通过 构造器直接赋值不安全数据

7.直接打set 自动完成set方法

8.**继承**  实现代码复用  提高拓展性和可维护性放
 - 多个类存在相同属性和方法,可以抽象出父类
- 子类不能用父类的私有属性和方法,非私有的都能用

- **子类构造器必须直接或间接调用父类构造器,完成对父类属性的初始化**,分工明确
 (间接调用:构造器中用this()调用其他参数的构造器,而后者调用super() ,同一构造器this()和super()不能同时用 )
- 默认自动隐式调用无参构造器,相当于自动super()
- 可以在第一行用super(参数)可以指定父类某一个构造器,如果父类没有无参构造器.则必须指定
- 如果子类有和父类重名的属性和方法,用super. 区分
如果没有重名,可以自己用,也可以super. this, 都一样 
-程序怎么识别某个方法呢?现在本类找有没有定义  再到父类一层一层找
9.override   重写覆盖父类  名字和参数严格一致的方法 
- override 方法返回类型必须是原方法返回类型的子类
- override方法访问权限 只能大 不能小 
- overload 只是本类同名方法的不同参数形式

10.**多态** 提高代码复用性  便于维护拓展
- 方法多态   父类方法中参数Animal  子类重写时参数可以是Dog类
  11.上转型对象  Animal animal =new Cat()
  animal对象只能用她的编译类型的成员 ,只能用父类成员
  Cat cat =(Cat) animal   强制先下转型  
  12.方法的动态绑定  调用对象方法,方法与实际运行类型绑定
  多态数组,父类定义的数组可以放子类,  
  13.==判断基本类型时,值是否相等 
          判断其他类型,判断地址
   equal 默认用==判断地址,子类重写以判断内容
  所以判断内容时,用equal  (基本类型没有equal用==)
        判断地址时用==
  14.hashcode()方法返回哈希码值  用于提高哈希表性能  
  不同对象的值不一样
  15.toString() 默认返回全类名@十六进制的hashcode
  直接输出对象默认调用toString()
  16.finalize()在对象没有任何引用,需要被回收时自动调用,重写业务逻辑代码(释放数据库,打开的文件)
  17.main方法 
    java虚拟机静态调用main,无需创建对象. 接受编译命令,用作String数组参数
  18.static 代码块 只会在类加载时执行一次
      普通代码块每创建一次对象在构造器就执行一次

类什么时候被加载?(new  子类加载时,父类也加载  使用类静态成员  )
静态代码块只能调用静态成员
构造器隐含了 1.super()  2.调用本普通代码块
19.子类创建对象时  调用顺序
    1.父类静态代码块和静态属性
    2.子类静态代码块和静态属性
    3.父类构造器(隐含执行普通代码块)
     4. 子类构造器(隐含执行普通代码块)
20.单例 
 1.构造private防止直接new
2.类内声明一个private 静态对象
3.只向外暴露一个静态公共方法创建返回该对象
21.final 
  1.不希望类被继承
 2.不希望方法可以被子类重写
3.不希望属性变量被修改,也叫产量,用大写下划线隔开
final+static 同时修饰,在调用时不会导致类被加载(底层优化了)
22.抽象类  是 设计者设计好之后 让子类继承并实现
抽象方法不能有方法体
一个非抽象类继承了抽象类,必须实现父类的所有抽象方法
抽象方法可以让不同类   同名方法的多态
23.接口 
java8后接口里的默认实现方法和静态方法可以有方法体   
接口就是 规范 实现的类 必须有的方法,且不同类的方法名 统一
接口中方法都是抽象方法,且public
接口中属性都是 public static fiinal 
接口是对java单继承的补充
小猴子继承猴子的爬树方法,但是想要飞怎么办?实现飞翔的接口

接口的多态:多态参数 ,多态数组 ,多态传递(实现了A接口,也就实现了A父接口)
24.内部类:一般在方法中,可以直接访问外部类的所有成员,包括私有
内部类就像局部变量,不能加访问修饰,但是可以final
作用域仅在定义的方法中 其他方法中要访问内部类成员要通过创建其对象
匿名内部类,只用一次,可以当实参用
静态内部类:只能访问静态成员,外部类要使用内部类的成员要通过创建对象    如果成员重名,默认里面就近
25.枚举类:有限的几个值且不需要修改
1.enum 代替class
2.第一行   A("agree"),B("disagree"); 
26.
错误:资源不足,堆溢出  内存不足 错误
异常:
运行时异常(检查不出来) :空指针 数组越界  类型转换  数字格式 数学运算   异常
编译时异常(不处理就没法运行)  网络数据库 文件

子类重写的方法抛出的异常 应该为 父方法抛出异常的子类
方法 throws 异常类
方法体中 throw  手动生成异常对象

27.Integer a =45; 自动装箱调用valueof()时,数如果在-128到127内 不
创建新的对象

28.String a=b+c; String对象相加 ,底层实现:
创建一个StringBuilder, 添加b,c   再转成String

String s="aaa" s指向常量池
String ss=new String("aaa")   ss指向堆

常用方法:String.format(format,多个变量)
format 字符串中有占位符  %s  %d  %.2f  %c

29.StringBuffer()     无参构造器 默认容量为16
      StringBuffer( int capacity)  指定容量
      StringBuffer(String str)  指定内容    容量再次基础上+16
30.StringBuilder不具有线程安全,用作StringBuffer的简易替换
只在单个线程使用中,但比Buffer要快,优先使用
31.字符串很少修改---String
     大量修改,单线程----StringBuilder
     大量修改,多线程----StringBUffer
 32.Arrays.sort()自定义排序
System.out.println(Arrays.binarySearch(a, 8));
 Integer[] newArr = Arrays.copyOf(a, 2);
        Arrays.fill(newArr, 88);
Arrays.equals(a, newArr);
        List list = Arrays.asList(arr);
33.property  get set 的内容
    区别于field  
#34.单列集合(实现Collection接口) :]]
###List (ArrayList  LinkedList Vector)   有序 允许重复  支持索引


######ArrayList  由数组实现 允许null 基本等同Vector 
线程不安全  增删效率低(数组)  查改效率高(索引)  
默认容量为0 第一次扩容为10  此后1.5倍扩容
指定容量时,1.5倍扩容  Arrays.copy()方法扩容

LinkedList 双向链表  双端队列    可以存null
线程不安全 
添加和删除不是通过数组,增删效率高,查改效率低(无索引)

查找较多用ArrayList  增删较多用LinkedList

Vector 由可变数组实现,线程安全但效率不高  
  默认容量10 也可以指定大小    2倍扩容


Set(  HashSet TreeSet)   无序(添与取顺序不一致) 不允许重复
 无索引
hashset 底层是hashmap ,而hashmap底层是数组+链表+红黑树
1.添加一个元素,得到hash值,再转成索引值
2.查table,没有加入 有就equals   equals为真,添加 equals为假,不添加
扩容:
1.底层hashmap 第一次添加 扩到16  临界值为16*加载因子=12
2. 到临界值 两倍扩容到32 新的临界值 32*0.75=24 以此类推
3.如果单个链表个数>=8且table大小>=64 转成红黑树

LinkeHashSet是HashSet子类  底层是数组+双向链表 不能重复
有序!(存与取顺序一致)

TreeSet 底层用TreeMap实现,可以在构造器传入Compareble对元素进行排序


   #双列集合:Map 接口类 key不能重复,value可以重复  多个id可以有相同值     map.keySet()  map.values()  map.entrySet()
              TreeMap 可以在构造器传入Comparable进行元素排序
               HashMap(线程不安全) 和其子类LinkedHashMap
               HashTable  和其子类 Properties  
hashtable (kv都不能为空,线程安全 初始化为11 临界系数0.75           
  2倍+1扩容) 

#集合的选择
  单列(Collection接口)
      有重复List 
          查改多:ArrayList (数组有索引,查找快)
          增删多:LinkedList  双向链表
      无重复:Set
          无序:hashset  排序treeset
          存取顺序一致:Linkedhashset
  双列:Map
      键无序:HashMap   键排序TreeMap  
      键存取顺序一致:LinkedHashMap 
      读取文件:Properties
#Collections工具类  操作集合
reverse()   shuffle()  sort(list,Comparable )  swap()  max(list Comparable)
![image.png](https://s2.loli.net/2022/02/16/RQ2rqUhz6aDm9jS.png)

#treeSet 添加的对象  必须有CompareTo,要么自己类就有,要么自定义,否则类型转换异常,
因为treeSet去重是通过compareto()方法的,返回0,就认为相同元素不添加  
1.构造器传入Comparable匿名,自定义compareTo  2.添加对象自己实现的compareTo

![image.png](https://s2.loli.net/2022/02/16/l3ZsOVxRmtNuB5Q.png)
#泛型:某一个类型
好处:1.约束数据类型 2.遍历时不需类型转化
静态成员不能使用泛型  
泛型类  泛型方法  使用泛型前必须尖括号声明]
<? extends A>  表示 A或者其子类的泛型
<? super A> 表示 A或者其父类的泛型
#JUnit
方便测试某个方法