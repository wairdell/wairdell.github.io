---
title: Android测试-Monkey
date: 2018-08-21 19:26:23
tags:
- Monkey
- Android
- 自动化测试
categories:
- 技术
---
## 什么是 Monkey
Monkey 是 Android 中的一个命令行工具，可以运行在模拟器里或者现实设备中，向系统发送伪随机的用户事件流（点击、滑动、Application 切换、横竖屏、应用关闭）实现对正在开发的应用程序进行压力测试。monkey 测试是一种为了测试软件的稳定性，健壮性的快速有效的方法。

## Monkey 命令参数介绍
1. 启动手机里面的APP,随机操作 N 次
```
	adb shell monkey 1000//不指定包名，启动所有的APP，1000是次数
	adb shell monkey -p com.yeahka.android.pospay -p com.wairdell.demo 1000//-p后面跟包名，启动指定包名的APP，这里指定了两个
```
2. `-v` : 操作日志记录
命令行的每一个 -v 将增加反馈信息的级别。
```
	adb shell monkey -p com.yeahka.android.pospay -v -v -v 100
```
	- -v : Level 0 (缺省值)除 启动提示、测试完成和最终结果之外，提供较少信息。
	- -v -v ：Level 1 提供较为详细的测试信息，如逐个发送到 Activity 的事件。
	- -v -v -v ：Level 2 提供更加详细的设置信息，如测试中被选中的或未被选中的 Activity。
	
3. `--throttle` ： 插入固定延迟
```
	adb shell monkey --throttle 500 -v 500
```
4. 调整事件的百分比
	- `--pct-touch <percent>`
	调整触摸事件的百分比(触摸事件是一个 down-up 事件，它发生在屏幕上的某单一位置)。
	- `--pct-motion <percent>`
	调整动作事件的百分比(动作事件由屏幕上某处的一个 down 事件、一系列的伪随机事 件和一个 up 事件组成)。
	- `--pct-trackball <percent>`
	调整轨迹事件的百分比(轨迹事件由一个或几个随机的移动组成，有时还伴随有点击)。
	- `--pct-nav <percent>`
	调整“基本”导航事件的百分比(导航事件由来自方向输入 设备的 up/down/left/right 组成)。
	- `--pct-majornav <percent>`
	调整“主要”导航事件的百分比(这些导航事件通常引发图 形界面中的动作，如：键盘的中间按键、回退按键、菜单按键)
	- `--pct-syskeys <percent>`
	调整“系统”按键事件的百分比(这些按键通常被保留，由 系统使用，如 Home、Back、Start Call、End Call 及音量控制键)。
	- `--pct-appswitch <percent>`
	调整启动 Activity 的百分比。在随机间隔里，Monkey 将执行一个 startActivity() 调 用，作为最大程度覆盖包中全部 Activity 的一种方法。
	- `--pct-anyevent <percent>`
	调整其它类型事件的百分比。它包罗了所有其它类型的事件，如：按键、其它不常用的设备按钮、等等。
5. -s <seed> : 指定伪随机数生成器的seed值。
如果用相同的seed值再次运行Monkey，它将生成相同的事件序列。
```
	monkey -p com.yeahka.android.pospay -s 35 -v 1000
```
6. 调试选项
	- `--dbg-no-events`
	设置此选项，Monkey 将执行初始启动，进入到一个测试 Activity，然后不 会再进一步生成事件。为了得到最佳结果，把它与-v、一个或几个包约束、以及一个保持 Monkey 运 行30秒或更长时间的非零值联合起来，从而提供一个环境，可以监视应用程序所调用的包之间的转换。
	- `--hprof`
	设置此选项，将在 Monkey 事件序列之前和之后立即生成 profiling 报告。 这将会在data/misc中生成大文件(~5Mb)，所以要小心使用它。
	- `--ignore-crashes`
	通常，当应用程序崩溃或发生任何失控异常时，Monkey 将停止运行。如果设置此选项，Monkey 将继续向系统发送事件，直到计数完成。
	- `--ignore-timeouts`
	通常，当应用程序发生任何超时错误(如“pplication Not Responding”对 话框)时，Monkey 将停止运行。如果设置此选项，Monkey 将继 续向系统发送事件，直到计数完成。
	- `--ignore-security-exceptions`
	通常，当应用程序发生许可错误(如启动一个需要某些许可的 Activity )时，Monkey 将 停止运行。如果设置了此选项，Monkey 将继续向系统发送事件，直到计数完成。
	- `--kill-process-after-error`
	通常，当Monkey由于一个错误而停止时，出错的应用程序将继续处于运行状态。当设置了此选项时，将会通知系 统停止发生错误的进程。注意，正常的(成功的)结束，并没有停止启动的进程，设备只是在结束事件之 后，简单地保持在最后的状态。
	- `--monitor-native-crashes`
	监视并报告 Android 系统中本地代码的崩溃事件。如果设置了--kill-process-after-error， 系统将停止运行。
	- `--wait-dbg`
	停止执行中的 Monkey，直到有调试器和它相连接。
	
7. 将log输出到文件
```
	monkey -p com.yeahka.android.pospay -s 35 -v 1000 > monkey.log
```

## 参考
- [Android的monkey用法](https://blog.csdn.net/wangayabin/article/details/52448930)
- [Android自动化测试--monkey详细到炸的总结](https://blog.csdn.net/zjnuwsf/article/details/52669764)