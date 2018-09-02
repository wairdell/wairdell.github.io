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
## uiautomatorviewer
uiautomatorviewer 是 Android SDK 自带的工具，提供了一个方便的 GUI，可以扫描和分析 Android 设备上当前显示的 UI 组件。您可以使用此工具检查布局层次结构，并查看在设备前台显示的 UI 组件属性。利用此信息，您可以使用 UI Automator（例如，通过创建与特定可见属性匹配的 UI 选择器）创建控制更加精确的测试。uiautomatorviewer 工具位于 `Android SDK/tools/bin` 目录下，如果你使用 Android Studio 也可通过 Tools -> Android -> Android Device Monitor 来打开。打开界面如下：
{% asset_img QQ20180902-163117.png %} 

整个界面分四个区域:
1.工作栏区（上）
共有4个按钮。从左至右分别用于：打开已保存的布局，获取详细布局，获取简洁布局，保存布局。点击保存，将存储两个文件，一个是图片文件，一个是.uix文件（XML布局结构） 
第二按钮（Device Screenshoot uiautomator dump）与第三按钮（Device Screenshoot with Compressed Hierarchy uiautomator dump –compressed）的区别在于，第二按钮把全部布局呈现出来，而第三按钮只呈现有用的控件布局。比如某一 Frame存在，但只有装饰功能，那么点击第三按钮时，可能不被呈现。

2. 截图区（左），显示当前屏幕显示的布局图片 

3. 布局区（右上）,已XML树的形式，显示控件布局 

4. 控件属性区（右下），当点击某一控件时，将显示控件属性

### UI Automator API
利用 UI Automator API，您可以编写稳健可靠的测试，而无需了解目标应用的实现详情。您可以使用这些 API 在多个应用中捕获和操作 UI 组件：
- UiCollection 枚举容器的 UI 元素以便计算子元素个数，或者通过可见的文本或内容描述属性来指代子元素
- UiObject 表示设备上可见的 UI 元素。
- UiScrollable 为在可滚动 UI 容器中搜索项目提供支持。
- UiSelector 表示在设备上查询一个或多个目标 UI 元素。
- Configurator 允许您设置运行 UI Automator 测试所需的关键参数。


### 常用方法
#### UiDevice 类的相关方法
[UiDevice](https://developer.android.com/reference/android/support/test/uiautomator/UiDevice) 是 Ui Automator 十分重要的对象。提供对设备状态信息的访问。 您还可以使用此类来模拟设备上的用户操作，例如按 d-pad 或按 Home 和 Menu 按钮。
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

#### UiSelector 类的相关方法
- `checkable(boolean val)` 
- `checked(boolean val)` checked 状态匹配的控件（通常用于复选框）。
- `childSelector(UiSelector selector)` 将子UiSelector标准添加到此选择器。
- `className(String className)` 类名相等的控件（例如，“android.widget.Button”）。
- `className(Class<T> type)` 类名相等的控件（例如，“android.widget.Button”）。
- `classNameMatches(String regex)` 类名满足正则表达式的空间
- `clickable(boolean val)` clickable 状态匹配的控件
- `description(String desc)` content-description 属性相等的控件。
- `descriptionContains(String desc)` content-description 属性包含的控件。
- `descriptionMatches(String regex)` content-description 属性满足正则表达式的控件。  
- `descriptionStartsWith(String desc)` content-description 属性开始开始相等的控件。
- `enabled(boolean val)` enable 状态匹配的控件
- `focusable(boolean val)` focusable 状态匹配的控件
- `focused(boolean val)` 设置搜索条件以匹配具有焦点的窗口小部件。
- `fromParent(UiSelector selector)` 将子UiSelector标准添加到此选择器，该选择器用于从父窗口小部件开始搜索。
- `index(int index)` 索引想等的控件
- `longClickable(boolean val)` 是否可长按的控件
- `packageName(String name)` 包名相等的控件
- `packageNameMatches(String regex)` 包名满足正则表达式的控件
- `resourceId(String id)` id 相等的控件
- `resourceIdMatches(String regex)` id 满足正则表达式的控件
- `scrollable(boolean val)` 可以滚动的控件
- `selected(boolean val)` 
- `text(String text)` text 相等的控件
- `textContains(String text)`  text 包含的控件
- `textMatches(String regex)` text 满足正则表达式的控件
- `textStartsWith(String text)` text 开始等于的控件
> UiSelector 可以通过调用一个或 N 个方法进行匹配，来达到最精确找到我们想找到的控件。


## 参考
- [Test UI for multiple apps](https://developer.android.com/training/testing/ui-testing/uiautomator-testing#java)