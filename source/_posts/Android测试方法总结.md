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
	一般通过程序的源代码进行测试而不使用用户界面。这种类型的测试需要从代码句法发现内部代码在算法，溢出，路径，条件等等中的缺点或者错误，进而加以修正。

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
- #### Espresso
	Espresso 测试框架提供了一组 API 来构建 UI 测试，用于测试应用中的用户流。利用这些 API，您可以编写简洁、运行可靠的自动化 UI 测试。Espresso 非常适合编写白盒自动化测试，其中测试代码将利用所测试应用的实现代码详情。
- #### UI Automator
	UI Automator 测试框架提供了一组 API 来构建 UI 测试，用于在用户应用和系统应用中执行交互。利用 UI Automator API，您可以执行在测试设备中打开“设置”菜单或应用启动器等操作。UI Automator 测试框架非常适合编写黑盒自动化测试，其中的测试代码不依赖于目标应用的内部实现详情。


## AndroidStudio 的测试方法
- #### 本地单元测试
源码位于 `module-name/src/test/java/`，这些测试在计算机的本地 Java 虚拟机 (JVM) 上运行。常用的测试库有 {% post_link 单元测试之JUnit JUnit %} 和 {% post_link 单元测试之Mockito Mockito %}。
- #### 仪器测试(Instrumentation Test)
	1. 源码位于 `module-name/src/androidTest/java/`，这些测试在硬件设备或模拟器上运行。常用的测试库有 [Android JUnit Runner](https://developer.android.com/reference/android/support/test/runner/AndroidJUnitRunner)、[Instrumentation Test Runner](https://developer.android.com/reference/android/test/InstrumentationTestRunner)。
	2. 仪器测试会在你的设备上多安装一个测试包，包名为 "被测试应用包名.test"。 再通过 [Instrumentation](https://developer.android.com/reference/android/app/Instrumentation) 把测试包和目标测试应用加载到同一个进程中运行。
	3. Android JUnit Runner 和 Instrumentation Test Runner 都是基于 Instrumentation 实现的。Instrumentation Test Runner 已被谷歌标记为过时，不建议使用，Android JUnit Runner 支持 Instrumentation Test Runner 所有特性。
	4. AndroidJUnitRunner 类是一个 JUnit 测试运行器，可让您在 Android 设备上运行 JUnit 3 或 JUnit 4 样式测试类，包括使用 Espresso 和 UI Automator 测试框架的设备。


### Mockito
被测试的类一般都会依赖其他的类，有些类的对象不好实例化，可以使用 {% post_link 单元测试之Mockito Mockito %} 这个库帮你模拟出这个对象。

## 参考
- [测试支持库](https://developer.android.com/topic/libraries/testing-support-library/?hl=zh-cn)
- [Test apps on Android](https://developer.android.com/training/testing/)
- [Android标准App的四大自动化测试](https://blog.csdn.net/n8765/article/details/53992820)