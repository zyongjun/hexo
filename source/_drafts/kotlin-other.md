---
title: kotlin-语法补充
date: 2017-01-05 00:21:19
categories:kotlin
keywords:kotlin,Android
tags:[kotlin]
---

#### 多重声明
1. 实现了componentN() 方法，该类的对象就可以实现多重构造
注意:数据类自动声明 componentN() 函数
```
class Person(val name: String, val age: Int){

      operator fun component1():String{
        return name
      }

      operator fun component2():Int{
          return age
      }
}

调用

>val person = Person("joe",28)

>val (name,age) = person

>val pname = person.component1()

>println("name:$name  age:$age  pname:$pname")

```

2. 多重声明可以在for循环中使用，标准库中提供了componentN的扩展
```

fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
fun <K, V> Map.Entry<K, V>.component1() = getKey()
fun <K, V> Map.Entry<K, V>.component2() = getValue()
```
因此你可以用 for 循环方便的读取 map (或者其它数据集合)
```
val m = hashMapOf(Pair(1,2), Pair(3,4))
   for ((a, b) in m) {
       println("a:$a  b:$b")
   }
```

3. 函数返回值,由于数据类自动声明 componentN() 函数，可以返回数据类，用多重声明接收
