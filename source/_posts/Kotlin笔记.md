---
title: Kotlin笔记
date: 2020-05-21 12:39:20
tags:
- kotlin
categories:
- 技术
---
- ### 三目运算
```kotlin
var result = if(a > b) a else b
```

- ### for 循环
```kotlin
var start = 0
var end = 10

//正序遍历
for (i in start..end) {
    print("$i ")
}
//输出结果0 1 2 3 4 5 6 7 8 9 10
println()

//倒序遍历
for (i in end downTo start) {
    print("$i ")
}
//输出结果10 9 8 7 6 5 4 3 2 1 0
println()

//指定步长
for (i in start..end step 2) {
    print("$i ")
}
//输出结果0 2 4 6 8 10
println()

//不包含最有一个区间
for (i in start until end ) {
    print("$i ")
}
//输出结果0 1 2 3 4 5 6 7 8 9 
println()
```
- ### 位运算

| 运算符       | 表示含义        |
| --------    | :-----:         |
| and(bits)   | 按位与          | 
| or(bits)    | 按位或          | 
| inv(bits)   | 按位非          | 
| xor(bits)   | 按位异或         | 
| shl(bits)   | 左移运算符       | 
| shr(bits)   | 右移运算符       | 
| ushr(bits)  | 无符号右移运算符  | 