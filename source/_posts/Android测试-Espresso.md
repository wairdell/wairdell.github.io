---
title: Android测试-Espresso
date: 2018-08-19 17:21:45
tags:
- Espresso
- Android
- 自动化测试
categories:
- 技术
---
## Espresso 介绍
Espresso 是 Google 提供的 Android 自动化测试框架，属于 Android 测试支持库，主要用来进行 Android 界面(UI)相关的自动化测试。使用 Espresso 可以对 Activity 进行测试，还可以对一个 Activity 向另一个 Activity 发送的 Intent 进行测试。我们可以找到一个 UI 组件，对其实施 ViewAction，从而检查 UI 组件的行为是否正确。这一切不需要我们的手动操作，只需要简单的几行代码就能够实现。使用 Espresso 我们可以为应用的 UI 行为添加必要的测试，也可以实施 TDD（测试驱动开发）。

## 如何在 Android Studio 中使用
1. ### 在你要测试的 Module 的 gradle 文件里添加如下两个依赖:
	```
testCompile 'junit:junit:4.12'

androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
}
	```
2. ### 创建测试类
打开你要测试的类，右键 -> Go To -> Test 创建一个测试类（也可自己在 module\src\androidTest\java 目录下创建文件）
3. ### 编写测试类
在类中声明相应的注解，如下代码
```java
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    /**
     * 声明测试哪一个 Activity
     */
    @Rule
    public ActivityTestRule mActivityRule = new ActivityTestRule(LoginActivity.class);

    /**
     * 用 @Test 声明这是个测试方法，一个类中可以声明多个测试方法
     */
    @Test
    public void loginTestCase() throws Exception {

        //根据View的id和hint定位到输入登录账号的Editext，然后往里面输入账号 wairdell
        onView(allOf(withId(R.id.et_input), withHint("请输入登入帐号"))).perform(replaceText("wairdell"));

        //根据View的id和hint定位到输入登录密码的Editext 然后往里面输入账号 aaaaaa
        onView(allOf(withId(R.id.et_input), withHint("请输入登入密码"))).perform(replaceText("aaaaaa"));

        //根据View的id定位到Button,并检查Button的状态
        onView(withId(R.id.btn_login)).check(matches(isEnabled()));

        //根据View的id定位到Button, 模拟点击时间， 进行登陆
        onView(withId(R.id.btn_login)).perform(click());

        Repository repository = Repository.getInstance();
        //判断是否登陆
        assertEquals(repository.isLogin(), true);

    }


}
```

写UI自动化测试用例，归结起来就是3步:
* 定位View控件
* 操作View控件
* 校验View控件的状态

对应Espresso，就是以下3个方法的调用：onView(ViewMatcher)、perform(ViewAction)、check(ViewAssertion)
4. ### 运行测试类
连接上真机或虚拟机，在编写完成的测试类中右键 run，或则点击测试方法左边的绿色三角形。运行的结果可以在底部查看，如下图。
{% asset_img QQ20180819-183113.png %}

> Espesso的运行是基于 SDK 的，要想运行一条用例必须在 Android 模拟器（或真机）上安装 App，启动 App，然后基于 UI 的操作来运行测试用例。

	