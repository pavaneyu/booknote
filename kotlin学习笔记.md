---
title: kotlin学习笔记
date: 2022年2月16日15:50:19
tags: []
categories: 学习笔记

---

<!--more-->

```
/**
 * kotlin中类和方法的参数,可以在创建类的时候指定
 * 不需要setXXX方法传入对象,如果不需要设定,采用默认值
 * 不需要java里面设置null和重载方法(或者构造器)来应对多种参数组合
 * 这也说明kotlin里面方法和类  类似 ,之所以去掉new,因为构造器也是一种方法,返回一个对象
 * 只不过类名一般首字母大写,当然你小写也没关系,只不过这样看起来构造器跟方法没区别了
 */
 
class Test(val int: Int, val str: String, private val char: Char='c'){
    //类的参数,定义的时候要指明val,可以加访问修饰符
    fun pl(){
        println(int+str.toInt()+char.toInt())
    }
}

fun funny(int: Int, str: String, char: Char='1') {
    println(int+str.toInt()+char.toInt())

}

fun main() {

    Test(int = 2,str = "345").pl()//默认参数可以不指定
    Test( str = "345", char = '2',int = 3).pl() //指定参数名时候,顺序无所谓
    Test(3,"23").pl()
    Test(4,"222",'2').pl()

    funny(int = 3,char = '5',str = "342")
}
```

1.返回值为函数类型的函数   是高阶函数

1.kotlin 类和方法默认public    默认final 不能继承,除非open
2.属性引用与函数应用要 类名+::
> ![image.png](https://s2.loli.net/2022/02/16/YvIi96UJCMedDmB.png)

3.labmda表达式最后一行为返回值类型
4.
> <img src="https://s2.loli.net/2022/02/16/oXebA75JCVQfjvP.png" alt="image.png" style="zoom:50%;" />

5.
```
import java.io.File


//region 注释

//endregion

/**
 * count what time right show in a String
 */
operator fun String.div(subString: String): Int {
    return this.windowed(subString.length,1){
        it==subString
    }.count()
}
fun main() {
    println("asd"/"a") //运算符重载
    File("src/SS.java").inputStream().reader().buffered()
            .use {
                println(it.readLine())
            }
    /**
     * use函数自动关闭closeable和自动捕获异常
     */

    val student = Student("asd", "das", "sd")
    /**
     *  also是作为it参数
     *  apply是作为调用对象receiver
     *  返回类型都是调用者的类型
     *
     */
    val stu1=student.also {
        it.name="asd"
    }
    val stu2=student.apply {
        name = "asd"
    }
    /**
     * let用it作为调用对象
     * run用this,可省略
     * 返回类型都是里面函数(lambda表达式)的返回类型
     */
    student.let {
        println(it)
    }

    //student.let(::println)
    student.run {
        println()
    }
    val list = listOf(1, 2, 3)
    /**
     * filter 接受一个boolean
     * map
     * 不改变原来集合
     */
    val filter = list.filter {
        it % 2 == 0
    }

    val map = list.map {
        it * 2
    }
    /**
     *flatMap将list每一个元素变成集合,再拼接起来
     */
    val flatMap = list.flatMap {
        1 .. it
    }
    println(flatMap.sum())
    println(flatMap)//[1, 1, 2, 1, 2, 3]
    println(list)
    println(filter)
    println(map)
    /**
     * 统计文件里每个字符出现的次数
     */
    File("src/SS.java")
            .readText()
            .toCharArray()
            .filter {
                !it.isWhitespace()
            }.groupBy {
                it
            }.map {
                it.key to it.value.size
            }.let {
                println(it)
            }


    

}

```
6. > 

7.密封类(都为抽象类)的子类是有限的 ,枚举类的值是有限的
8. > <img src="https://s2.loli.net/2022/02/16/Hx8Wun5IRKVcwyZ.png" alt="image.png" style="zoom:50%;" />
9. ><img src="https://s2.loli.net/2022/02/16/ZgAOUT2ldDokKqP.png" alt="image.png" style="zoom:50%;" />
>数据类为final,不可继承  ty