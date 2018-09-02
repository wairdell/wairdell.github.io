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

写 UI 自动化测试用例，归结起来就是3步:
* 定位 View 控件
* 操作 View 控件
* 校验 View 控件的状态

对应 Espresso，就是以下3个方法的调用：onView(ViewMatcher)、perform(ViewAction)、check(ViewAssertion)
4. ### 运行测试类
连接上真机或虚拟机，在编写完成的测试类中右键 run，或则点击测试方法左边的绿色三角形。运行的结果可以在底部查看，如下图。
{% asset_img QQ20180819-183113.png %}

## 相关 API 
### 视图匹配
利用Espresso.onView()方法，您可以访问目标应用中的 UI 组件并与之交互。此方法接受Matcher参数并搜索视图层次结构，以找到符合给定条件的相应View实例。您可以通过指定以下条件来优化搜索：
- 视图的类名称 onView(withClassName());
- 视图的内容描述 onView(withContentDescription());
- 视图的ID onView(withId());
- 在视图中显示的文本 onView(withText());

更多的可以查看 [ViewMatchers](https://developer.android.google.cn/reference/android/support/test/espresso/matcher/ViewMatchers)。如果搜索成功，onView()方法将返回一个引用，让您可以执行用户操作并基于目标视图对断言进行测试。
### 适配器匹配
在 AdapterView 布局中，布局在运行时由子视图动态填充。如果目标视图位于某个布局内部，而该布局是从AdapterView（例如 ListView 或 GridView）派生出的子类，则 onView() 方法可能无法工作，因为只有布局视图的子集会加载到当前视图层次结构中。因此，需要使用 Espresso.onData() 方法访问目标视图元素。Espresso.onData() 方法将返回一个引用，让您可以执行用户操作并根据 AdapterView 中的元素对断言进行测试。
```java
//点击spinner
onView(withId(R.id.spinner)).perform(click());
//点击adpaterviewer中类型为String 并且内容为test的文本，
onData(allOf(is(instanceOf(String.class)),is("test"))).perform(click());
```
### 操作API
- `click()`：返回一个点击动作，Espresso利用这个方法执行一次点击操作,就和我们自己手动点击按钮一样。
- `clearText()`：返回一个清除指定view中的文本action，在测试EditText时用的比较多。
- `swipeLeft()`：返回一个从右往左滑动的action,这个在测试ViewPager时特别有用。
- `swipeRight()`：返回一个从左往右滑动的action,这个在测试ViewPager时特别有用。
- `swipeDown()`：返回一个从上往下滑动的action。
- `swipeUp()`：返回一个从下往上滑动的action。
- `closeSoftKeyboard()`：返回一个关闭输入键盘的action。
- `pressBack()`：返回一个点击手机上返回键的action。
- `doubleClick()`：返回一个双击action
- `longClick()`：返回一个长按action
### 校验结果
- `doesNotExist`：断言某一个view不存在。
- `matches`：断言某个view存在，且符合一列的匹配。
- `selectedDescendentsMatch`：断言指定的子元素存在，且他们的状态符合一些列的匹配。



## 所有的包说明
- `espresso-core` 核心的包，对 View 的匹配，操作以及断言。 
- [`espresso-web`](https://developer.android.com/training/testing/espresso/web.html) 提供对 WebView 测试的相关支持
- [`espresso-idling-resource`](https://developer.android.com/training/testing/espresso/idling-resource.html) 与后台作业同步的机制
- `espresso-contrib` 提供 DatePicker、RecyclerView 和 Drawer 操作、辅助功能检查 和 CountingIdlingResource 的支持
- [`espresso-intents`](https://developer.android.com/training/testing/espresso/intents.html) 校验多app测试中intent的正确性
- `espresso-remote` 不同进程功能

> Espesso 的运行是基于 Android SDK 的，要想运行一条用例必须在 Android 模拟器（或真机）上安装 App，启动 App，然后基于 UI 的操作来运行测试用例。

	