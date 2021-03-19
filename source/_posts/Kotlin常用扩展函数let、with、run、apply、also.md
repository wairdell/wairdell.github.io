---
title: Kotlin常用扩展函数let、with、run、apply、also
date: 2020-04-22 11:32:40
tags:
- kotlin
categories:
- 技术
---
## 简介
let、with、run、apply、also 这些函数都是方便对同一对象操作编码优化。例如我有一个数据结构如下的对象
```
class Student {

    var sno: String? = null
    var name: String? = null
    var age: Int? = null

    fun eat() {
        print("$name 在吃饭")
    }

    fun run() {
        print("$name 在跑步")
    }
}
```
如果我们想操作这里对象去完成下面这些操作，通常需要如下面这么写，而 kotlin 的自带的函数能让我们更优雅的完成下面的操作。
```
    @Test
    fun normalTest() {
        student?.name = "张三"
        student?.age = 16
        student?.sno = "202004220311"
        student?.run()
        student?.eat()
        student2?.name = "李四"
        student2?.age = 17
    }
```
- ### let 函数
上面的例子我们用 let 可以如下这么写

```
    @Test
    fun letTest() {
        student.let {
            it?.name = "张三"
            it?.age = 16
            it?.sno = "202004220311"
            it?.run()
            it?.eat()
        }
        student2.let {
            it?.name = "李四"
            it?.age = 17
        }
    }
```

这么写的好处有啥呢?我个人认为会比较直观，一眼就能看出来是对同一对象的一系列操作，还可以避免一些bug，如某两个变量名十分相识，当你写一泻千里写着代码的时候很可能就造成 `student2?.name = "李四"` 写成 `student?.name = "李四"` 的 bug。使用 let 函数还有一个好处就是可以对非空判断，如上面的还可以这么写

```
    @Test
    fun letTest() {
        student?.let {
            it.name = "张三"
            it.age = 16
            it.sno = "202004220311"
            it.run()
            it.eat()
        }
    }
```

- ### with 函数
with 与 left 相比优点是可以在函数内部就像对象内部(`this`)一样去访问属性,缺点是无法做非空判断
```
    @Test
    fun withTest() {
        var result = with(student) {
            this?.name = "with"
            this?.age = 16
            "withResult"
        }
        with(student!!) {
            name = "李四"
            age = 17
        }
        println("student = $student, result = $result")
    }
```

- ### run 函数
run 结合 let 和 with 两者的优点，即通过 `this` 访问对象，又能做非空判断
```
    @Test
    fun runTest() {
        student?.run {
            name = "张三"
            age = 17
            runTest()
        }
    }
```

- ### apply 函数
apply 与 run 的行为几乎一模一样了，唯一的区别就是 run 函数返回值是最后一行代码的值，而 apply 返回的是传入的对象本身, 如下面的代码打印是 "return = result"
```
    var runResult = student?.run {
            name = "张三"
            age = 17
            "result"
        }
    print("return = $runResult")
```

而使用 apply 将打印的是传入对象本身 "return = [name = 张三, age = 17, sno = null]"
```
    var runResult = student?.run {
            name = "张三"
            age = 17
            "result"
        }
    print("return = $runResult")
```
- ### also 函数
also 与 let 的行为几乎一模一样，唯一的区别也是返回值的不同，also 返回是传入对象本身，而 let 是最后一行代码
```
    @Test
    fun alsoTest() {
        var alsoResult = student?.also {
            it.name = "张三"
            it.age = 17
            "aaaa"
        }
        println("return = $alsoResult") //打印"return = [name = 张三, age = 17, sno = null]"

        var letResult = student?.let {
            it.name = "张三"
            it.age = 17
            "aaaa"
        }
        println("return = $letResult") //打印"return = aaaa"
    }
```

## 总结
| 操作符       | 对象访问方式        |  返回值       | 扩展函数 |
| --------    | :-----:            | :----:        | :----:  |
| left        | it指代当前对象      | 函数本身返回值 |  是      |
| with        | this访问           | 函数本身返回值 |  否      |
| run         | this访问           | 函数本身返回值 |  是      |
| apply       | this访问           | 传入对象       |  是      |
| also        | it指代当前对象      | 传入对象       | 否       |