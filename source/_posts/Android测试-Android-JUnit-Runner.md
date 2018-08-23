---
title: Android测试-Android JUnit Runner
date: 2018-08-23 16:54:55
tags:
- Android JUnit Runner
- Android
- 自动化测试
categories:
- 技术
---
## Android JUnit Runner 介绍
1. 是一个测试运行器，用于运行 Junit3 和 Junit4 的 Android 测试包 
2. 替换 [Instrumentation Test Runner](https://developer.android.google.cn/reference/android/test/InstrumentationTestRunner)（一个比较旧的测试运行器） 
3. 支持 [Instrumentation Test Runner](https://developer.android.google.cn/reference/android/test/InstrumentationTestRunner) 所有特性，但又进行了扩展 
4. 保持了所有 [Instrumentation Test Runner](https://developer.android.google.cn/reference/android/test/InstrumentationTestRunner) 的命令格式

## 如何使用
1. ### 首先需要在被测试 module 下的 `build.gradle` 导入如下依赖
```
	android {
		defaultConfig {
			testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
		}
	}

	dependencies {
		androidTestCompile 'com.android.support:support-annotations:23.1.1'
		androidTestCompile 'com.android.support.test:runner:0.4.1'
		androidTestCompile 'com.android.support.test:rules:0.4.1'
	}
```
2. ### 创建测试类，一下两种方式
	- 打开你要测试的类，右键 -> Go To -> Test -> 选择你要测试的方法 -> 选择测试文件路径(有两个，选择路径包含 androidTest 的那个)
	- 在你的 module\src\androidTest\java 目录下创建测试类
3. ### 给新建的类添加注解 `@RunWith(AndroidJUnit4.class)`，用于指定哪个运行器运行
4. ### 编写测试用例
#### 常用的方法
	- `InstrumentationRegistry.getInstrumentation()` 返回当前正在运行的 Instrumentation
	- `InstrumentationRegistry.getContext()` 返回此 Instrumentation 软件包的上下文。
	- `InstrumentationRegistry.getTargetContext()` 返回目标应用的应用上下文。
	- `InstrumentationRegistry.getArguments()` 返回传递给此 Instrumentation 的参数 Bundle。
#### 常用的注解
	- `@RequiresDevice` 只在物理设备上运行
	- `@SdkSupress` 限定最低SDK版本
	- `@SmallTest`，`@MediumTest` 和 `@LargeTest` 测试分级


> Android JUnit Runner 是运行在 Android 的系统环境下，需要连接 Android 的真机或模拟器才能进行测试
## 参考
[1.Android JUnit Runner（使用AndroidStudio）](https://www.cnblogs.com/JianXu/p/5175945.html)