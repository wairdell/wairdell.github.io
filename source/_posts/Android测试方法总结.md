---
title: Android测试方法总结
date: 2018-08-26 15:34:48
tags:
- Android
- 测试
categories:
- 技术
---
## 白盒测试和黑盒测试
- 白盒测试
	是通过程序的源代码进行测试而不使用用户界面。这种类型的测试需要从代码句法发现内部代码在算法，溢出，路径，条件等等中的缺点或者错误，进而加以修正。

- 黑盒测试
	在测试时，把程序看作一个不能打开的黑盆子，在完全不考虑程序内部结构和内部特性的情况下，测试者在程序接口进行测试，它只检查程序功能是否按照需求规格说明书的规定正常使用，程序是否能适当地接收和正确的输出。

## 测试的大致流程
#### 写单元测试用例，归结起来是以下3步:
1. 编写测试的输入参数
2. 调用被测试类的方法，并传入输入参数
3. 用测试库相关的 assert 方法来判断方法的返回值是否与预计结果一致

####  写 UI 自动化测试用例，归结起来以下3步:
1. 定位 View 控件
2. 操作 View 控件
3. 校验 View 控件的状态

### Android 常用自动化测试
- #### Monkey 
	用作简单的自动化测试工作，不依赖源码。{% post_link Android测试-Monkey Monkey %} 测试就是让设备随机的乱点，事件都是随机产生的。
- #### MokeyRunner 
	[MonkeyRunner](http://www.android-doc.com/tools/help/monkeyrunner_concepts.html) 工具是使用 Jython(使用 Java 编程语言实现的 Python)写出来的，它提供了多个 API，通过 monkeyrunner API 可以写一个 Python 的程序来模拟操作控制 Android 设备 app,测试其稳定性并通过截屏可以方便地记录出现的问题。依靠控件坐标进行定位控件。
- #### Instrumentation 
	Instrumentation 是 Android 主推的白盒测试框架。依靠控件的ID来进行定位的, 依赖于源码。
- #### UI Automator
鉴于 Instrumentation 框架需要读懂项目源码、脚本开发难度较高并且不支持多应用交互，Android 官网亮出了自动化测试的王牌——UIAutomator，并主推这个自动化测试框架。该框架无需项目源码，脚本开发效率高且难度低，并且支持多应用的交互。UIAutomator 支持多应用的交互，弥补了 Instrumentation 工具的不足。但 UIAutomator 难以捕捉到控件的颜色、字体粗细、字号等信息，要验证该类信息的话，需要通过截图的方式进行半自动验证。同时，UIAutomator 的调试相比 Instrumentation 要困难。
- #### Robotium
Robotium 是一款国外的自动化测试框架，是一款免费的 Android UI 测试工具，主要针对 Android 平台的应用进行黑盒自动化测试，它提供了模拟各种手势操作（点击、长按、滑动等）、查找和断言机制的 API，能够对各种控件进行操作。 
Robotium 测试脚本是用 java 写的，该框架使用简便，支持性良好，与 Monkeyrunner 相比测试也更为高级，不必为每个设备编写脚本。但没有十全十美的框架，Robotium 的短板在于它不适合与系统软件的交互。

## AndroidStudio 的测试方法
- #### 本地单元测试
源码位于 `module-name/src/test/java/`，这些测试在计算机的本地 Java 虚拟机 (JVM) 上运行。常用的测试库有 {% post_link 单元测试之JUnit JUnit %} 和 {% post_link 单元测试之Mockito Mockito %}。
- #### 仪器测试(Instrumentation Test)
	1. 源码位于 `module-name/src/androidTest/java/`，这些测试在硬件设备或模拟器上运行。常用的测试库有 [Android JUnit Runner](https://developer.android.com/reference/android/support/test/runner/AndroidJUnitRunner)、[Instrumentation Test Runner](https://developer.android.com/reference/android/test/InstrumentationTestRunner)。
	2. 仪器测试会在你的设备上多安装一个测试包，包名为 "被测试应用包名.test"。 再通过 [Instrumentation](https://developer.android.com/reference/android/app/Instrumentation) 把测试包和目标测试应用加载到同一个进程中运行。
	3. Android JUnit Runner 和 Instrumentation Test Runner 都是基于 Instrumentation 实现的。Instrumentation Test Runner 已被谷歌标记为过时，不建议使用，Android JUnit Runner 支持 Instrumentation Test Runner 所有特性。


### Mockito
被测试的类一般都会依赖其他的类，有些类的对象不好实例化，可以使用 {% post_link 单元测试之JUnit Mockito %} 这个库帮你模拟出这个对象。

{% asset_img android_test_orchestrator_flow.png %}
## 参考
- [测试应用](https://developer.android.com/studio/test/)
- [Test apps on Android](https://developer.android.com/training/testing/)
- [Android标准App的四大自动化测试](https://blog.csdn.net/n8765/article/details/53992820)