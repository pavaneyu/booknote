---

title: JUC笔记
date: 2022年2月14日17:45:36
tags: [思考]
categories: 读书笔记

---



[toc]

# 什么是线程上下文切换

用户态切换到内核态    同时 程序计数器和寄存器也会切换过来

![image-20220619094425643](https://s2.loli.net/2022/06/19/7Neq536FVX2kT9y.png)



![image-20220401170004154](https://s2.loli.net/2022/04/01/sxPhCjEHZQqcOzm.png)

# 原子引用

```
AtomicReference<String> re = new AtomicReference<>();
```

# 原子时间戳引用  解决   ABA 问题 

```
AtomicStampedReference<String> stampedReference = new AtomicStampedReference<>("a", 1);
```

# 集合线程不安全问题

+ 第一代集合类  vector
+ collections.syncList 
+ 第三代  concurrentHashmap     copyonwriteArrayList

# 可重入锁  (递归锁)         可以防止死锁

同一线程在外层方法获取锁  时候,,进入内层方法 自动获取该锁 

即 线程可以进入 锁内  任意代码块 

# 自旋锁   见 CAS自旋.java

```
//不会立即阻塞,而是循环的方式获取锁
//避免线程切换的上下文开销,但是会消耗CPU
```

# 可重入读写锁    

相比于可重入锁    锁的粒度更细

#  阻塞队列  满了后再添加会阻塞 ,直到移除一个

一直阻塞   还是阻塞一段时间后 自动 退出

![image-20220619152026744](../../../../AppData/Roaming/Typora/typora-user-images/image-20220619152026744.png)

```
offer 和take 失败  不会阻塞   ,但如果加上时间参数,就会在给定的时间内阻塞
put 和take   失败会一直阻塞  

可以阻塞的,都会抛出打断异常 





BlockingQueue<String> strings = new ArrayBlockingQueue<String>(3);
//基于数组的阻塞队列  满了还添加会阻塞,也可以设定一定时间后自动消亡
BlockingQueue<String> strings1 = new LinkedBlockingQueue<>(3);
BlockingQueue<String> blocking = new SynchronousQueue<>();
//同步队列 ,put和take 同步,只存在一个元素
```
