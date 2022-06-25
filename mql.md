1.约定convention    粗体和小写都是命令  命令参数用中括号  attribute[Weight]  

 ![image.png](https://s2.loli.net/2022/03/28/gwkX19pDUVtGR3K.png)
![image.png](https://s2.loli.net/2022/03/28/7xMzCDHuAGjswWa.png)
![image.png](https://s2.loli.net/2022/03/28/DzOLqP9resSp5kV.png)


上下文环境用户  
  set context person creator   [password xxx]  vault Manufacturing 
帮助   help 
常用缩写  bus 是businessObject 缩写    rel 是relationship   缩写  

mql文件   脚本模式
文件执行   run +mql文件地址
命令大小写不敏感     但是 属性名称  特有的名字   大小写是不一样的   

支持基于命令行的交互式模式 
To build a complete data model, however, the script mode would be preferred to the interactive mode.
然而，为了构建完整的数据模型，脚本模式比交互模式更可取。
简单地定义和改变    复制粘贴   数据模型的轻松测试

TCL  工具命令语言  作为 MQL的拓展 可以被MQL调用
他提供了 很多编程特性  比如 变量 循环  数组  和方法

dump 使用分割符输出 
output 将结果输出到文件



# 查询 从句

对象的所有字段都可以被查询,如果没有指定就查询所有字段

print bus  业务类型  名称  版本  (业务对象三要素) select  policy attribute[ xxx ] (字段) 
print bus  业务对象id
print role/type/format/person/relationship   名字  select  字段

list 字段(policy )

# 建立永久查询  :

add query  查询名字 
  bus  业务对象类型   业务对象名字  业务对象版本
  where attribute[]==XXX
  valut  Engineering
  owner  Dave;

执行
evaluate   query  查询名

# 临时查询
temp query bus  类型  名称  版本
  where attribute[]==XXX
  limit 5
  select policy current  /attribute[Weight]  (字段)
  dump |
  output  文件路径  

# expand 查看从起始对象 连接到业务对象

expand bus  类型  名字  版本 
  from   [type/relationship   名字]             recurse to all / tarse (表示简洁的)
  rel  名字
  select bus  attribute[某个属性] ...
 dump |
  output 文件地址
  limit 5

# 计算表达式 Evaluate Expression 




每条MQL  都隐藏一个事务   什么是事务  ACID   同一事务下的所有操作 

也可以显式声明事务  start transaction      commit   transaction

# 业务管理命令     在数据库中管理对象  并指定可选择项
操作包括 add modify connect   这些操作也可以在business软件用GUI来操作
对象包括 attribute /type/relationship/program/format  /vault/store /gruop/role /person/policy


add  bus 类型  名称 版本
policy XXX
vault XXX
属性名  属性值
-------------------

connect  bus 类型  名字  版本
to 类型  名字  版本
relationship  名字
-------------------------
approve  bus 类型名字 版本  
signature Ready
--------------------
promote  bus  类型  名字  版本
-------------------
demote bus 类型 名字 版本



![image.png](https://s2.loli.net/2022/03/28/ZyK62zVWLYAHqIT.png)

![image.png](https://s2.loli.net/2022/03/28/pKHfSLJr7FeBGvE.png)
![image.png](https://s2.loli.net/2022/03/28/FZleBbLKEmcAkjz.png)



  