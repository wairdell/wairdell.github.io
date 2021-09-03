---
title: JNI学习笔记
date: 2021-04-12 14:13:06
tags:
---
## 两种注册方式
1. ### 静态注册
    静态方式就是根据函数名来建立 Java 函数和 JNI 函数之间的关联关系。可以使用 Java 的工具程序 javah 生成
    ```
    javah -o output packageName.className
    ```
    如下的 java 类声明的 native 方法
    ``` java
    package com.wairdell;

    public class Demo {
        public int void sum(int num1, int mum2);
    }
    ```
    对应的 JNI 函数是
    ``` C
    jint Java_com_wairdell_Demo_sum(JNIEnv *env, jobject thiz, jint num1, jint num2) {

    }
    ```
2. ### 动态注册
    Java native 函数和 JNI 函数是一一对应的。在 JNI 技术中，使用 `JNINativeMethod` 结构来记录这种一一对应的关系。
    ``` C
    typedef struct {
        //Java中native函数的名字，不用携带包的路径，例如“native_init”。
        const char* name;
        //Java函数的签名信息，用字符串表示，是参数类型和返回值类型的组合。
        const char* signature;
        //JNI层对应函数的函数指针，注意它是void*类型。
        void* fnPtrl;
    } JNINativeMethod;
    ```
    可是使用下面方法进行注册
    ``` C
    JNINativeMethod gMethods[] {
        {
            "sum",//Java 中 native 函数的函数名。
            "(II)I",//native的签名信息，
            (void*)Java_com_wairdell_Demo_sum //JNI层对应的函数指针。
        }
    }
    AndroidRuntime:register(env, "com/wairdell/Demo", gMethods, NELEM(gMethods));
    ```

    `AndroidRunTime.cpp`
    ``` Cpp
    int AndroidRuntime:registerNativeMethods(JNIEnv* env, const char* className, const JNINativeMethod* gMethods, int numMethods) {
        return jniRegisterNativeMethods(env, className, gMethods);
    }
    ```

    `JNIHelp.c`
    ``` C
    int jniRegisterNativeMethods(JNIEnv *env, const char* className, const JNINativeMethod* gmethods, int numMethods) {
        jclass clazz;
        clazz = (*env)->FindClass(env, className);
        if((*env)->RegisterNatives(env, clazz, gMethods, numMethods) < 0) {
            return -1; 
        } else {
            return 0;
        }
    }
    ```
    注册时机：当 Java 层通过 System.loadLibrary 加载完 JNI 动态库后，紧接着会查找该库中一个叫 JNI_OnLoad 的函数。如果有，就调用它，而动态注册的工作就是在这里完成的。

## 基本数据类型的转换表
|   Java    |   Native类型  |   符号属性    |   字长    |
|   ----    |   ----        |   ----        |   ----    |
|   boolean |   jboolean    |   无符号      |   8位     |
|   byte    |   jbyte       |   无符号      |   8位     |
|    char   |   jchar       |   无符号      |   16位    |
|   short   |   jshort      |   有符号      |   16位    |
|   int     |   jint        |   有符号      |   32位    |
|   long    |   jlong       |   有符号      |   64位    |
|   float   |   jfloat      |   有符号      |   32位    |
|   double  |   jdouble     |   有符号      |   64位    |

## Java 引用数据类型转化关系表
|   java引用类型    |   Native类型  |
|   ----            |   ----        |
|   All objects     |   jobject     |
|   Class 类型      |   jclass      |
|   String 类型     |   jstring     |
|   Object[]        |   jobjectArray    |
|   Throwable 类型  |   jthrowable  |
|   int[]           |   jintArray   |
|   short[]         |   jshortArray |
|   long[]          |   jlongArray  |
|   float[]         |   floatArray  |
|   double[]        |   jdoubleArray    |
|   byte[]          |   jbyteArray  |
|   boolean[]       |   jbooleanArray   |

##  JNIEnv介绍
JNIEnv是一个与线程相关的代表JNI环境的结构体。JNIEnv 在每个线程都是不同实例。调用JavaVM的AttachCurrentThread函数，就可得到这个线程的JNIEnv结构体,在线程退出前，需要调用JavaVM的DetachCurrentThread函数来释放对应的资源。
```
//全进程只有一个JavaVM对象，所以可以保存，并且在任何地方使用都没有问题。
jint JNI_OnLoad（JavaVM*vm,void*reserved）
``` 
使用 JNIEnv 可以让 native 调用 java 的代码。大概的使用方式先用`jfieldID GetFieldID(jclass clazz, char* name, char* sig)` 和 `jmethodID GetMethodID(jclass clazz, const char* name, const char* sig)` 来获取类的成员函数和成员变量信息。使用类似 `CallVoidMethod(Call＜type＞Method)` 的函数调用 java 方法，使用类似 `GetIntField(Get＜type＞Field)` 的函数访问 java 的字段。

#### Java String 对象转成本地字符串
```C
env->GetStringChars()
env->GetStringUTFChars()
```
#### 本地字符串转成 Java String 对象
```C
env->NewString()
env->NewStringUTF()
```
> Note:  记得调用 `ReleaseStringChars` 和 `ReleaseStringUTFChars` 函数来释放资源


## 类型标识示意表
|   类型标识    |   Java类型    |
|   ----        |   ----        |
|   V           |   void        |
|   Z           |   boolean     |
|   B           |   byte        |
|   S           |   short       |
|   I           |   int         |
|   J           |   long        |
|   F           |   float       |
|   D           |   double      |
|   L/java/lang/String;  |   String  |
|   [I          |   int[]       |
|   [L/java/lang/Object; |  Object[]    |


### 引用方式
- `Local Reference`:本地引用。在JNI层函数中使用的非全局引用对象都是Local Reference，它包括函数调用时传入的jobject和在JNI层函数中创建的jobject。Local Reference最大的特点就是，一旦JNI层函数返回，这些jobject就可能被垃圾回收。
- `Global Reference`:全局引用，这种对象如不主动释放，它永远不会被垃圾回收。
- `Weak Global Reference`:弱全局引用，一种特殊的Global Reference，在运行过程中可能会被垃圾回收。所以在使用它之前，需要调用 JNIEnv 的 IsSameObject 判断它是否被回收了。
> Note: `DeleteGlobalRef` 和 `DeleteLocalRef` 释放对应对象

### JNI层函数可以在代码中截获和修改这些异常，JNIEnv提供了三个函数给予帮助：
- `ExceptionOccured` 函数，用来判断是否发生异常。
- `ExceptionClear` 函数，用来清理当前JNI层中发生的异常。
- `ThrowNew` 函数，用来向Java层抛出异常。