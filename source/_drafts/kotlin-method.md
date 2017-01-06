---
title: kotlin函数
date: 2017-01-04 11:23:15
categories: kotlin
keywords: kotlin,函数
tags: [kotlin]
---
#### 函数声明
```
fun sum(x:Int,y:Int):Int{
  return x+y
}
使用  val s = sum(1,2)
```
#### 参数
* 默认参数
```
fun sum(x:Int,y:Int=0):Int{
  return x+y
}
```
* 命名参数

 1. 调用时候可以根据命名指定sum(x=1,y=2)

 2. 顺序可以改变sum(y=2,x=1)
* 可变参数
 1. vararg修饰最后一个参数
 ```
 fun sum(vararg n: Int):Int{
       var s = 0
       for (x in n) {
           s += x
       }
       return s
 }
 调用：
 sum(1,2,3)
 可以使用*前缀传入array
 val arr = intArrayOf(4,5,6)
 println(sum(*arr,1,2,3))
 ```

 2. vararg修饰的参数不是最后一个可以有两种情况：一调用的时候后面的参数使用命名参数传值，二后面的参数是函数时候使用lamada

#### 局部函数（函数支持顶级函数、局部函数或扩展函数）
1. 局部函数可以访问外部函数的局部变量
2. 局部函数可以直接返回到外部函数
```
fun reachable(from: Vertex, to: Vertex): Boolean {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (current == to) return@reachable true
        if (!visited.add(current)) return
        for (v  in current.neighbors)
            dfs(v)
    }
    dfs(from)
    return false
}
```
#### 泛型函数
```
fun <T> tranInt(a: T):T {
    println("====$a")
    return a
}
```

#### 内联函数

#### 扩展函数

#### 高阶函数和lamada表达式

#### 尾递归函数
