---
title: 单元测试之JUnit
date: 2018-03-20 11:25:39
tags:
- JUnit
- 单元测试
categories:
- 技术
---
## JUnit 介绍
[JUnit](https://junit.org/junit5/) 是 Java 社区中知名度最高的单元测试工具。它诞生于 1997 年，由 Erich Gamma 和 Kent Beck 共同开发完成。其中 Erich Gamma 是经典著作《设计模式：可复用面向对象软件的基础》一书的作者之一，并在 Eclipse 中有很大的贡献；Kent Beck 则是一位极限编程（XP）方面的专家和先驱。
麻雀虽小，五脏俱全。JUnit 设计的非常小巧，但是功能却非常强大。Martin Fowler 如此评价 JUnit：在软件开发领域，从来就没有如此少的代码起到了如此重要的作用。它大大简化了开发人员执行单元测试的难度，特别是 JUnit 4 使用 Java 5 中的注解（annotation）使测试变得更加简单。
## 如何在 Android Studio 开发中使用
1. 需要先在 build.gradle 添加依赖。
   ```   
   testCompile 'junit:junit:4.12'
   ```
2. 打开你要测试的类右键，选择 Go To > Test，或者在 src/test/java 中新建类。再新建一个 public void 的方法，并使用注解（annotation）*@Test* 修饰。
	 ```java
import org.junit.Test;

import static org.junit.Assert.*;

/**
 * Example local unit test, which will execute on the development machine (host).
 *
 * @see <a href="http://d.android.com/tools/testing">Testing documentation</a>
 */
public class ExampleUnitTest {

    //用 @Test 修饰的方法代表这是个测试方法
    @Test
    public void addition_isCorrect() throws Exception {
        //使用 org.junit.Assert 各种方法判断结果是否与预期一致，不一致则表示测试不通过
        assertEquals(4, 2 + 3);
    }
}
	```
3. 右键新建的类，再弹出的菜单栏中选择 run。

> JUnit 的测试环境是没有 android.jar，所以如果用到其中相关的类会报错。

## JUnit 基本使用
从 JUnit4 开始使用 Java5 中的注解（annotation），以下是 JUnit4 常用的几个 annotation： 
- @Before：初始化方法   对于每一个测试方法都要执行一次（注意与 BeforeClass 区别，后者是对于所有方法执行一次）
- @After：释放资源  对于每一个测试方法都要执行一次（注意与 AfterClass 区别，后者是对于所有方法执行一次）
- @Test：测试方法，在这里可以测试期望异常和超时时间 
- @Test (expected=ArithmeticException.class) 检查被测方法是否抛出 ArithmeticException 异常 
- @Ignore：忽略的测试方法 
- @BeforeClass：针对所有测试，只执行一次
- @AfterClass：针对所有测试，只执行一次

> Before、After、Test、Ignore 所注解的方法必须为 *public void*, BeforeClass、而 AfterClass 所注解的方法必须为 *public static void*
### 一个JUnit4的单元测试用例执行顺序为： 
@BeforeClass -> @Before -> @Test -> @After -> @AfterClass; 