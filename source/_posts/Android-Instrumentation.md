---
title: Android Instrumentation
date: 2018-08-23 23:11:39
tags:
- Android
- Instrumentation
categories:
- 技术
---
## Instrumentation 是什么
我们先通过谷歌文档对 [Instrumentation](https://developer.android.google.cn/reference/android/app/Instrumentation) 的描述大致的了解下
> Base class for implementing application instrumentation code. When running with instrumentation turned on, this class will be instantiated for you before any of the application code, allowing you to monitor all of the interaction the system has with the application. An Instrumentation implementation is described to the system through an AndroidManifest.xml's <instrumentation\> tag.

翻译成中文的意思是：实现检测应用程序的基类，当 instrumentation 运行打开时，这个类将会在任何应用程序代码之前实例化，从而允许您监控系统与应用程序的所有交互。一个 Instrumentation 的实现需要在 AndroidMainfest.xml 文件的 <instrumentation\> 标签中定义。

## Instrumentation 能做些什么