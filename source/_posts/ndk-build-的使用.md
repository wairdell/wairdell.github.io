---
title: ndk-build 的使用
date: 2019-05-23 20:53:52
tags:
- Android
- ndk 
categories:
- 技术
---
### NDK（Native Development Kit），是用于在 Android 应用中嵌套本地代码的工具集，现在 NDK 开发的方式有两种: ndk-build 和 cmake，本章介绍使用 ndk-build 的方式进行 ndk 开发
### 1、首先安装配置 NDK
在 Android 中依次点击 `Tools -> SDK Manager -> SDK Tools`，找到并选中 NDK 进行下载。
{% asset_img QQ截图20190523211348.png %}

### 2、编写包含 *native* 方法的 java 类
```java
package com.example.ndkbuilddemo;

public class Demo {

    public native int sum(int num1, int num2);

}

```

### 3、使用 javah 命令生成 .h 文件
在 AndroidStudio 的 Terminal 中 cd 到上面类对应的 module 目录下，在使用下面的命令，`-d jni` 是指定生成 *.h* 文件到的目录，`-classth path com.example.ndkbuilddemo.Demo` 是上面编写的包含 native 方法的 java 类
```
javah -d jni -classpath java com.example.ndkbuilddemo.Demo
```

### 4、新建 *.c* 文件 inlude 上面生成的 *.h* 文件并编写 C 代码，实现 *native* 方法
```
#include "com_example_ndkbuilddemo_Demo.h"
JNIEXPORT jint JNICALL Java_com_example_ndkbuilddemo_Demo_sum
  (JNIEnv *env, jobject thiz, jint num1, jint num2) {
    return num1 + num2;
  }
```
### 5、在 jni 路径下新建 `Android.mk` 文件
将下面的内容拷入进去，*LOCAL_MODULE* 是要编译的模块的名称，*LOCAL_SRC_FILES* 是 C 的源码，相关配置具体的内容可查阅 [Google 的文档](https://developer.android.google.cn/ndk/guides/android_mk)
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE := NdkBuildDemo
LOCAL_SRC_FILES := demo.c
include $(BUILD_SHARED_LIBRARY)
```

### 6、在 build.gradle 中进行 ndk-build 配置
右键 module 选择 Link C++ Project with Gradle， Build System 选择 ndk-build, Project Path 选择上面编写的 `Android.mk` 文件，也可自己在 build.gradle 加入以下内容
```
externalNativeBuild {
        ndkBuild {
            path file('src/main/jni/Android.mk')
        }
    }
```

### 7、在 java 代码中加载 .so
这里 loadLibarary 中的内容就是第5步 *LOCAL_MODULE* 的值
```
package com.example.ndkbuilddemo;

public class Demo {

    static {
        System.loadLibrary("NdkBuildDemo");
    }


    public native int sum(int num1, int num2);

}
```