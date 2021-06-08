---

layout: post

title: java工具之jConsole可视化工具

tag: java语言

---
# java之jConsole可视化工具

## 1、JConsole简介

​	JConsole图形用户界面是符合Java管理扩展（JMX）规范的监视工具,可以监测有关在Java平台上运行的应用程序的性能和资源消耗的信息。

## 2、JCOnsole使用

​	JConsole是一个图形化界面，使用方式很简单，windows在cmd命令行下直接输入JConsole即可，当然也可以去jdk安装目录下面去找JConsole.exe 双击执行。

## 3、工具介绍

​	首页样式

![](https://github.com/superhxf/superhxf.github.io/blob/master/_posts/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190809140910.png?raw=true)

整个工具大概列出了概览，内存，线程，类，VM概要，MBean六类，平时最关注的是内存和线程。

首页会展示选中java进程的概览，堆内存的使用，线程数量，加载的类，cpu的占用率。

内存这块页面会展示内存的使用情况，有以下几类的展示，堆内存，非堆内存，老年代，伊甸区，su'rviver区，元数据区（1.8之后的称呼，原来是方法区或者说永久代）。

线程重点展示的是线程数，每个线程的堆栈跟踪。

类就是JVM共加载内存的类数量

vm概要有操作系统的信息，虚拟机的信息，堆内存，垃圾收集器，vm参数等。



