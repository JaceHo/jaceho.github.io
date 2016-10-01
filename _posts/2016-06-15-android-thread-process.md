---
layout: post
title: "android中的线程与进程模型"
description: "了解Android的多线程与多进程模型有助于我们对Android app的整个开发模型有更深的理解"
tags: [线程, 进程模型, android, 内存]
---
>了解Android的多线程与多进程模型有助于我们对Android app的整个开发模型有更深的理解,这里记录下一些理解与思考.

###Android中的线程
对于初学java的人都知道`thread`的概念，而到了android thread的概念变味了，出现了譬如service, asynctask, intent service, handler thread, async query loader, threadpool executor等纷繁复杂的概念， android对于线程的处理机制如下。

* 主线程
   * 所有Android基础组件均在**指定进程的主线程中初始化**
   * 所有系统对于基础组件的回调事件均在主线程中**分发**
       * 组件中的系统回调方法在指定进程的主线程中执行
       * 所有的基础组件不应该在系统回调代码中执行耗时操作    
   * 所有不能立即完成的操作都应该被放在非UI线程中执行
       * 线程对象使用标准的java代码来创建
   * Android提供一些方便的类来管理线程:
       * Looper作为`ThreadLocal`的单例变量用来循环处理消息队列(`MessageQueue`)
       * Handler提供消息/Runable的发送以及回调操作
       * HandlerThread提供启动拥有Looper线程的简易方法

针对不同的使用场景，在线程，线程池，handler等基础上我们拓展了一些易用的线程处理组件：
![android_multitasking.png](http://upload-images.jianshu.io/upload_images/928566-c40e4e439581e563.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###Android 中的进程

* 当应用的第一个组件需要被执行时，Android Framework会为此启动一个有单线程(主线程,即UI线程)运行环境的Linux进程
* 当Android系统资源紧缺时会终止一些进程的执行
* 对于特定的Android组件<activity> <service> <receiver> <provider>，我们可以通过android:process来指定
   它运行的进程
   * 每个组件可以运行在它所指定的进程中
   * 一些组件会共享一个进程， 而其他的不在其中
   * 不同application中的组件可以在同一个进程中执行
* 我们可以通过设置<application>的process属性指定应用所有组件的默认运行进程

####Android中的rpc
Android进程间通讯实际上可以看做2个进程中的线程消息传递，作为cs架构，其实现依赖binder机制
#####binder机制解析:

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/928566-0f3b05068e7e3243.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####分层示例
![实例分析](http://upload-images.jianshu.io/upload_images/928566-fe9808d43e9b6400.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####进程间通讯binder机制client,server实现代码示例：
```shell
git clone https://github.com/marakana/FibonacciBinderDemo.git
```
参考链接：
[http://events.linuxfoundation.org/images/stories/slides/abs2013_gargentas.pdf](http://events.linuxfoundation.org/images/stories/slides/abs2013_gargentas.pdf)
[http://www.cubrid.org/blog/dev-platform/binder-communication-mechanism-of-android-processes/](http://www.cubrid.org/blog/dev-platform/binder-communication-mechanism-of-android-processes/)