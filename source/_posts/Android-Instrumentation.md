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

翻译成中文的意思是：实现检测应用程序的基类，当 instrumentation 运行打开时，这个类将会在任何应用程序代码之前实例化，从而允许您监控系统与应用程序的所有交互。一个 Instrumentation 的实现需要在 `AndroidMainfest.xml` 文件的 `<instrumentation>` 标签中定义。

## Instrumentation 能做些什么
- 进行测试
Instrumentation 可以把测试包和目标测试应用加载到同一个进程中运行。既然各个控件和测试代码都运行在同一个进程中了，测试代码当然就可以调用这些控件的方法了，同时修改和验证这些控件的一些数据。已知的基于 Instrumentation 实现的测试库有 [Instrumentation Test Runner](https://developer.android.google.cn/reference/android/test/InstrumentationTestRunner) 和 [Android JUnit Runner](https://developer.android.google.cn/training/testing/junit-runner)
- 进行 Hook
Android 的应用中有许多操作（newActivity() ，startActivity()，callActivityOnCreate() 等）是通过 Instrumentation 实现的，我们可以通过反射技术替换应用中的 Instrumentation 变成自己的实现，来进行一些 hook 操作。如绕过 Activity 需要在清单文件注册的校验：
```java
public class HookInstrumentation extends Instrumentation {

    private static final String PACKAGE_NAME = "package_name";
    private static final String CLASS_NAME = "class_name";

    Instrumentation mBase;

    public HookInstrumentation(Instrumentation base) {
        this.mBase = base;
    }

    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {

        if (intent.getComponent() != null) {
            intent.putExtra(PACKAGE_NAME, intent.getComponent().getPackageName());
            // 因为下面会把要启动的activity变为已注册的activity，这里保存本来要启动的acitivity
            intent.putExtra(CLASS_NAME, intent.getComponent().getClassName());
            // 使用已经注册的文件绕过校验
            intent.setClassName(intent.getComponent().getPackageName(),
                    target.getClass().getName());
        }

        ActivityResult result = null;
        try {
            Class[] parameterTypes = {Context.class, IBinder.class, IBinder.class, Activity.class, Intent.class,
                    int.class, Bundle.class};
            result = (ActivityResult) invoke(Instrumentation.class, mBase,
                    "execStartActivity", parameterTypes,
                    who, contextThread, token, target, intent, requestCode, options);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }


    @Override
    public Activity newActivity(ClassLoader cl, String className, Intent intent) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        // 得到保存的实际要启动的activity
        String targetClassName = intent.getStringExtra(CLASS_NAME);
        if (!TextUtils.isEmpty(targetClassName)) {
            Activity activity = mBase.newActivity(cl, targetClassName, intent);
            activity.setIntent(intent);
            return activity;
        }
        return mBase.newActivity(cl, className, intent);
    }

    public static Object invoke(Class clazz, Object target, String name, Class[] parameterTypes, Object... args)
            throws Exception {
        Method method = clazz.getDeclaredMethod(name, parameterTypes);
        method.setAccessible(true);
        return method.invoke(target, args);
    }

}
```