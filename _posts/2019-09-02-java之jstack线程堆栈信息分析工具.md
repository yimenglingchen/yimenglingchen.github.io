---

layout: post

title: java之jstack线程堆栈信息分析工具

tag: java语言

---
# java之jstack线程堆栈信息分析工具

### 1、工具说明

 Jstack是Jdk自带的线程跟踪工具，用于打印指定Java进程的线程堆栈信息。

### 2、常用命令

```
jstack -l pid
jstack pid >> thread.dump
```

命令说明：

1. -F当’jstack [-l] pid’没有相应的时候强制打印栈信息,如果直接jstack无响应时，用于强制jstack），一般情况不需要使用
2. -l长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表，会使得JVM停顿得长久得多（可能会差很多倍，比如普通的jstack可能几毫秒和一次GC没区别，加了-l 就是近一秒的时间），-l 建议不要用。一般情况不需要使用
3. -m打印java和native c/c++框架的所有栈信息.可以打印JVM的堆栈,显示上Native的栈帧，一般应用排查不需要使用
4. `-h | -help`打印帮助信息
5. pid 需要被打印配置信息的java进程id,可以用jps查询.

### 3、线程相关内容

​	线程的状态说明

```
NEW：未启动的。不会出现在Dump中。
RUNNABLE：在虚拟机内执行的。运行中状态，可能里面还能看到locked字样，表明它获得了某把锁。
BLOCKED：受阻塞并等待监视器锁。被某个锁(synchronizers)給block住了。
WATING：无限期等待另一个线程执行特定操作。等待某个condition或monitor发生，一般停留在park(), wait(), sleep(),join() 等语句里。
TIMED_WATING：有时限的等待另一个线程的特定操作。和WAITING的区别是wait() 等语句加上了时间限制 wait(timeout)。
TERMINATED：已退出的。
```

Monitor

​	在多线程的 JAVA程序中，实现线程之间的同步，就要说说 Monitor。 Monitor是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。

```
进入区(Entrt Set)：表示线程通过synchronized要求获取对象的锁。如果对象未被锁住,则迚入拥有者;否则则在进入区等待。一旦对象锁被其他线程释放,立即参与竞争。
拥有者(The Owner)：表示某一线程成功竞争到对象锁。
等待区(Wait Set) ：表示线程通过对象的wait方法,释放对象的锁,并在等待区等待被唤醒。
```

​		一个 Monitor在某个时刻，只能被一个线程拥有，该线程就是 “Active Thread”，而其它线程都是 “Waiting Thread”，分别在两个队列 “ Entry Set”和 “Wait Set”里面等候。在 “Entry Set”中等待的线程状态是 “Waiting for monitor entry”，而在“Wait Set”中等待的线程状态是 “in Object.wait()”。

### 4、线程dump文件分析

​	使用工具，IBM的jca工具，我本地使用的是jca443版本，导入线程dump文件之后，就可以分析线程冲突，锁，等待等问题，优化程序。

​	

