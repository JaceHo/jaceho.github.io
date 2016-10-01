---
layout: post
title: "android内存优化之hprof文件的使用"
description: "A small leak will sink a great ship."
tags: [hprof, android, 内存]
---

“A small leak will sink a great ship.” - Benjamin Franklin

在android开发中，由于程序员的疏忽，对于android framework处理组件对象的声明周期不够了解，导致了一些内存没有释放。

目前Android上处理处理内存问题的方法均是基于强大的[hprof](http://docs.oracle.com/javase/7/docs/technotes/samples/hprof.html)文件的分析，包括[android studio memory dump](https://developer.android.com/studio/profile/investigate-ram.html),  [leakcanary](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0509/2854.html)和[eclipse mat](http://www.eclipse.org/mat/downloads.php)(standalone)工具，本文主要针对通用的hporf分析，这也会适用于leakcanry`无法检测`的情况.

下面将通过eclipse mat2，adb等工具的配合使用来找到示例工程中内存泄漏所出现问题的地方

####内存对象实时监测
- 安装mat示例程序后，为了观察内存中各个对象的使用情况，运行一下命令:
   ```shell
       while true;  do adb shell dumpsys com.example.jefforeilly.mat ; sleep ;        
   ```
   * 打开应用后，内存使用情况
![刚使用时内存使用情况](http://blog.futureme.info/assets/img/928566-0dd4f76cc1cddd0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   * 然后我们正常使用应用（包含2次登录登出，打开主页面，打开fragment，关闭fragment）使用后
![应用使用后内存使用情况](http://blog.futureme.info/assets/img/928566-1af9f4b59d1217a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####内存泄漏分析
#####android studio memory dump
- 遇到这种内存泄漏问题，我们可以先用android studio2的memoery dump功能查看reference tree,选择打开[dominator_tree](https://developer.android.com/studio/profile/am-memory.html)  
![C93EE648-089B-449C-9248-A4EAE8D8F2D8.png](http://blog.futureme.info/assets/img/928566-6285d94d0b3aa871.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中`activity`是我们最关心的组件之一，应为它们占用的内存较大，也更容易发现，按图所示步骤，第图中四步，我们发现`viewpropertyanimatorcompat listener`没有被释放，这个是我们自己产生的对象，显然有问题。


#####eclipse mat分析

- 不过，大多数情况，我们创建的对象可能更多，他们的ref 深度也超出我们可以随意查找的范围，这就需要使用更加请更加专业的分析工具Eclipse Memory Analyzer了，
在刚才应用的路径*Myapplication/captures*下 首先转换android hporf 格式
   ```shell
       hprof-conv com.example.jefforeilly.myapplication_2016.07.22_09.18.hprof converted.hprof 
   ```
- 转换android vm的hprof格式之后，就可以使用eclipse mat做分析了,
    
   * 首先搜索任意我们认为应该被回收的对象名称，如这个例子中的MainActivity, 这里发现了5个入口,  为简单处理，我们全选所有对象， 右键选择 `merge shorttest paths to gc roots`(exclude weak references),  得到距离gc root 更近的节点. 
![EA5A3190-E96B-46C2-943C-F43B81EAD47A.png](http://blog.futureme.info/assets/img/928566-4de3962089671d61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   * 依次展开这些节点， 发现了当前线程函数栈中有animator，也是我们new出的对象，这个Thread就是MainThread, 没有手动释放的animator还在运行, 这就很明确告诉我们对它处理的相关代码有问题了
![EA7CBD37-F2F3-4E95-A18A-D180F4C78686.png](http://blog.futureme.info/assets/img/928566-7c9c533279454729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**上面寻找内存泄漏的方法也同样使用于内存优化的过程，如果所有对象和内存能够合理释放，在`dominator tree`中找到dominator node，看它是否可以释放，如果可以就release掉它，这样由他支配的其他节点也会被回收，如此这样内存便可以持续优化了。**
