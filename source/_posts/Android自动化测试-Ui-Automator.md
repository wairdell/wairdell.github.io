---
title: Android自动化测试-Ui Automator
date: 2018-08-26 21:24:47
tags:
- Ui Automator
- Android
- 自动化测试
categories:
- 技术
---
## UI Automator 介绍
UI Automator 是一个 UI 测试框架，适用于跨系统和已安装应用程序的跨应用程序功能 UI 测试。该框架无需项目源码，脚本开发效率高且难度低，并且支持多应用的交互。UIAutomator 支持多应用的交互，弥补了 Instrumentation 工具的不足。但 UIAutomator 难以捕捉到控件的颜色、字体粗细、字号等信息，要验证该类信息的话，需要通过截图的方式进行半自动验证。

UI Automator测试框架的主要功能包括：
- 检查布局层次结构的查看器。 
- 用于检索状态信息并在目标设备上执行操作的API
- 支持跨应用程序UI测试的API

## 在 Android Studio 中使用
首先我们需要先倒入 Android JUnit Runner 相关依赖，如果不知道如何导入，可以查看我之前的文章 {% post_link Android测试-Android-JUnit-Runner %}，然后在 build.gradle 添加以下代码
```
androidTestImplementation 'com.android.support.test.uiautomator:uiautomator-v18:2.1.3'
```
编写测试用例,以下是官网提供的测试 demo,我稍微改了下
```java
@RunWith(AndroidJUnit4.class)
@SdkSuppress(minSdkVersion = 18)
public class ChangeTextBehaviorTest {

    private static final String BASIC_SAMPLE_PACKAGE
            = "com.yeahka.android.pospay.mobile";
    private static final int LAUNCH_TIMEOUT = 5000;
    private static final String STRING_TO_BE_TYPED = "UiAutomator";
    private UiDevice mDevice;

    @Before
    public void startMainActivityFromHomeScreen() {
        // 初始化一个 UiDevice 对象
        mDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());

        // 模拟 home 按键的点击
        mDevice.pressHome();

        // 得到 launcher 的包名
        final String launcherPackage = mDevice.getLauncherPackageName();
        // 断言 launcher 的包名是否为空
        assertThat(launcherPackage, notNullValue());
        // 等待 launcher 的启动
        mDevice.wait(Until.hasObject(By.pkg(launcherPackage).depth(0)),
                LAUNCH_TIMEOUT);

        // 启动应用 （BASIC_SAMPLE_PACKAGE 填写自己要测试应用的包名）
        Context context = InstrumentationRegistry.getContext();

        // 通过包名得到测试应用入口的 intent
        final Intent intent = context.getPackageManager()
                .getLaunchIntentForPackage(BASIC_SAMPLE_PACKAGE);
        // Clear out any previous instances
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
        context.startActivity(intent);

        // 等待测试应用的启动
        mDevice.wait(Until.hasObject(By.pkg(BASIC_SAMPLE_PACKAGE).depth(0)),
                LAUNCH_TIMEOUT);
    }

    @Test
    public void clickButton() throws UiObjectNotFoundException {
    	// 应用的入口一版是启动页，这里再次等待从启动页进入 MainActiivty
        mDevice.wait(Until.hasObject(By.clazz(com.yeahka.android.pospay.MainActivity.class).depth(0)), LAUNCH_TIMEOUT);
        // 找到文本内容为“1”的TextView
        UiObject oneText = mDevice.findObject(new UiSelector()
                .text("1")
                .className("android.widget.TextView"));
        // 找到文本内容为“2”的TextView
        UiObject twoText = mDevice.findObject(new UiSelector()
                .text("2")
                .className("android.widget.TextView"));

        // 存在就点击
        if(oneText.exists()) {
            oneText.click();
        }
        // 存在就点击
        if(twoText.exists()) {
            twoText.click();
        }
        // 找到文本内容为“收款”的Button
        UiObject confirmBtn = mDevice.findObject(new UiSelector().text("收款").className("android.widget.Button"));
        if(confirmBtn.exists()) {
            confirmBtn.click();
        }
        //我的有点击事件的监听 跳转到了下一个Activity
        mDevice.wait(Until.hasObject(By.clazz(ScanCashActivity.class).depth(0)), LAUNCH_TIMEOUT);
    }
}

```

[UiDevice](https://developer.android.com/reference/android/support/test/uiautomator/UiDevice) 是 Ui Automator 十分重要的对象。提供对设备状态信息的访问。 您还可以使用此类来模拟设备上的用户操作，例如按 d-pad 或按 Home 和 Menu 按钮。

### 常用方法
UiDevice 类
- `click(int x, int y)` 在指定的任意坐标上执行单击。
- `drag(int startX, int startY, int endX, int endY, int steps)` 从一个坐标到另一个坐标的滑动。
- `findObject(UiSelector selector)` 根据一个 UiSelector 匹配到一个 Ui 试图。
- `getDisplayHeight()` 获取以像素为单位显示器的高度
- `getDisplayWidth()` 获取以像素为单位显示器的宽度
- `getDisplayRotation()` 
- `pressSearch()` 模拟点击搜索按钮
- `pressMenu()` 模拟点击菜单按钮
- `pressKeyCode(int keyCode)` 模拟键盘事件的点击
- `pressHome()` 模拟点击 home 按钮
- `pressDelete()` 模拟点击确认按钮
- `pressEnter()` 模拟点击确认按钮
- `pressBack()` 模拟点击后退按钮
- `hasObject(BySelector selector)` 返回给定选择器标准是否匹配
- `wait(SearchCondition<R> condition, long timeout)` 等待条件得到满足
## 参考
- [Test UI for multiple apps](https://developer.android.com/training/testing/ui-testing/uiautomator-testing#java)