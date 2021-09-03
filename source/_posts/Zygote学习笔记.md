---
title: Zygote学习笔记
date: 2021-04-12 18:34:00
tags:
---
Zygote 进程启动后会先调用 App_main.cpp 的 Main 方法，这个方法会调用 AppRuntime 的 start 方法来完成重要功能。AppRuntime 是 AndroidRuntime 的子类，主要完成了三个大功能
- startVm 创建虚拟机

    这个函数绝大部分都是设置虚拟机的参数

- startReg 注册JNI函数

    将 Android Framework 下 Java Class 的 native 方法进行动态注册(关联对应的 JNI 函数)。

- ZygoteInit.main 调用 Java Class 的 main

    调用了 Java 层的入口方法

然后余下的初始操做大多都是由 Java 层实现。ZygoteInit 的 main 函数有如下关键的步骤

- registerZygoteSocket 建立IPC通信服务端

- preloadClasses 和 preloadResources 预加载类和资源

- startSystemServer 启动 system_server
    ``` java
    private static boolean startSystemServer() throws MethodAndArgsCaller,RuntimeException {
        //设置参数
        String args[]={..., "com.android.server.SystemServer"};
        ZygoteConnection.Arguments parsedArgs = null；
        int pid；
        try {
            parsedArgs = new ZygoteConnection.Arguments(args);
            pid = Zygote.forkSystemServer(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids, debugFlags, parsedArgs.debugFlags, null);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }
        if(pid == 0) {
            //根据 Fork 函数的然后值，这里处理子进程，也就是 system_server 的进程
            handleSystemServerProcess(parsedArgs);
        }
        return true;
    ```
- runSelectLoopMode 处理客户连接和客户请求
