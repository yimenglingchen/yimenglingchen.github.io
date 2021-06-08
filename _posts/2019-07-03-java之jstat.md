---

layout: post

title: java工具之jstat

tag: java语言

---
# java之jstat

​	Jstat是JDK自带的一个轻量级小工具。全称“Java Virtual Machine statistics monitoring tool”，它位于java的bin目录下，主要利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。

直接看用法

```
jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

参数说明

```
option： 参数选项
-t： 可以在打印的列加上Timestamp列，用于显示系统运行的时间
-h： 可以在周期性数据数据的时候，可以在指定输出多少行以后输出一次表头
vmid： Virtual Machine ID（ 进程的 pid）
interval： 执行每次的间隔时间，单位为毫秒
count： 用于指定输出多少次记录，缺省则会一直打印
```

options列表

```
-class                 显示ClassLoad的相关信息；
-compiler           显示JIT编译的相关信息；
-gc                     显示和gc相关的堆信息；
-gccapacity 　　  显示各个代的容量以及使用情况；
-gcmetacapacity 显示metaspace的大小
-gcnew               显示新生代信息；
-gcnewcapacity  显示新生代大小和使用情况；
-gcold                 显示老年代和永久代的信息；
-gcoldcapacity    显示老年代的大小；
-gcutil　　           显示垃圾收集信息；
-gccause             显示垃圾回收的相关信息（通-gcutil）,同时显示最后一次或当前正在发生的垃圾回收的诱因；
-printcompilation 输出JIT编译的方法信息；
```

## 1、compiler

   查看编辑情况

```
jstat -compiler 15172
Compiled Failed Invalid   Time   FailedType FailedMethod
    1187      0       0     2.12          0
```

  列表数据解释

```
Compiled：编译任务执行数量
Failed：编译任务执行失败数量
Invalid ：编译任务执行失效数量
Time ：编译任务消耗时间
FailedType：最后一个编译失败任务的类型
FailedMethod：最后一个编译失败任务所在的类及方法
```

## 2、class

​	显示加载的class的信息、

```
jstat -class 15172
Loaded  Bytes  Unloaded  Bytes     Time
  3207  6036.7        0     0.0       1.92
```

列表数据解释

```
Loaded : 已经装载的类的数量
Bytes : 装载类所占用的字节数
Unloaded：已经卸载类的数量
Bytes：卸载类的字节数
Time：装载和卸载类所花费的时间
```

## 3、gc

​	查看垃圾回收信息

```
jstat -gc 15172
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
5120.0 5120.0 4538.2  0.0   33280.0  18091.9   87552.0      99.1    18688.0 18108.3 2304.0 2145.8      2    0.024   0      0.000    0.024
```

列表数据说明

```
S0C：年轻代中第一个survivor（幸存区）的容量 （字节）
S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
EC：年轻代中Eden（伊甸园）的容量 (字节)
EU：年轻代中Eden（伊甸园）目前已使用空间 (字节)
OC：Old代的容量 (字节)
OU：Old代目前已使用空间 (字节)
MC：metaspace(元空间)的容量 (字节)
MU：metaspace(元空间)目前已使用空间 (字节)
YGC：从应用程序启动到采样时年轻代中gc次数
YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
FGC：从应用程序启动到采样时old代(全gc)gc次数
FGC：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)
```

## 4、gccapacity

​	VM内存中三代（young,old,perm）对象的使用和占用大小。

```
jstat -gccapacity 15172
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
 43520.0 238592.0  43520.0 5120.0 5120.0  33280.0    87552.0   478208.0    87552.0    87552.0      0.0 1064960.0  18688.0      0.0 1048576.0   2304.0      2     0
```

列表数据说明

```
NGCMN：年轻代(young)中初始化(最小)的大小(字节)
NGCMX：年轻代(young)的最大容量 (字节)
NGC：年轻代(young)中当前的容量 (字节)
S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
EC：年轻代中Eden（伊甸园）的容量 (字节)
OGCMN：old代中初始化(最小)的大小 (字节)
OGCMX：old代的最大容量(字节)
OGC：old代当前新生成的容量 (字节)
OC：Old代的容量 (字节)
MCMN：metaspace(元空间)中初始化(最小)的大小 (字节)
MCMX ：metaspace(元空间)的最大容量 (字节)
MC ：metaspace(元空间)当前新生成的容量 (字节)
CCSMN：最小压缩类空间大小
CCSMX：最大压缩类空间大小
CCSC：当前压缩类空间大小
YGC：从应用程序启动到采样时年轻代中gc次数
FGC：从应用程序启动到采样时old代(全gc)gc次数
```

## 5、gcnew

年轻代信息

```
jstat -gcnew 15172
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
5120.0 5120.0 4538.2    0.0  7  15 5120.0  33280.0  18091.9      2    0.024
```

数据列表解释

```
S0C   ：年轻代中第一个survivor（幸存区）的容量 (字节)
S1C   ：年轻代中第二个survivor（幸存区）的容量 (字节)
S0U   ：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
S1U  ：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
TT：持有次数限制
MTT：最大持有次数限制
DSS：期望的幸存区大小
EC：年轻代中Eden（伊甸园）的容量 (字节)
EU ：年轻代中Eden（伊甸园）目前已使用空间 (字节)
YGC ：从应用程序启动到采样时年轻代中gc次数
YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
```

## 6、gcnewcapacity

​	年轻代对象信息及其占用量

```
jstat -gcnewcapacity 15172
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
   43520.0   238592.0    43520.0  79360.0   5120.0  79360.0   5120.0   237568.0    33280.0     2     0
```

数据列表解释

```
NGCMN     ：年轻代(young)中初始化(最小)的大小(字节)
NGCMX      ：年轻代(young)的最大容量 (字节)
NGC     ：年轻代(young)中当前的容量 (字节)
S0CMX    ：年轻代中第一个survivor（幸存区）的最大容量 (字节)
S0C    ：年轻代中第一个survivor（幸存区）的容量 (字节)
S1CMX ：年轻代中第二个survivor（幸存区）的最大容量 (字节)
S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
ECMX：年轻代中Eden（伊甸园）的最大容量 (字节)
EC：年轻代中Eden（伊甸园）的容量 (字节)
YGC：从应用程序启动到采样时年轻代中gc次数
FGC：从应用程序启动到采样时old代(全gc)gc次数
```

## 7、gcold

​	老年代 持久代信息

```
jstat -gcold 15172
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
 18688.0  18108.3   2304.0   2145.8     87552.0        99.1      2     0    0.000    0.024
```

列表数据解释

```
MC ：metaspace(元空间)的容量 (字节)
MU：metaspace(元空间)目前已使用空间 (字节)
CCSC:压缩类空间大小
CCSU:压缩类空间使用大小
OC：Old代的容量 (字节)
OU：Old代目前已使用空间 (字节)
YGC：从应用程序启动到采样时年轻代中gc次数
FGC：从应用程序启动到采样时old代(全gc)gc次数
FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)
```

## 8、gcoldcapacity

​	老年代gc信息详情

```
jstat -gcoldcapacity 15172
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
    87552.0    478208.0     87552.0     87552.0     2     0    0.000    0.024
```

列表数据解释

```
OGCMN      ：old代中初始化(最小)的大小 (字节)
OGCMX       ：old代的最大容量(字节)
OGC        ：old代当前新生成的容量 (字节)
OC      ：Old代的容量 (字节)
YGC  ：从应用程序启动到采样时年轻代中gc次数
FGC   ：从应用程序启动到采样时old代(全gc)gc次数
FGCT    ：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)
```

### 9、gcutil

整体gc情况

```
jstat -gcutil 15172
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
 88.64   0.00  54.36   0.11  96.90  93.13      2    0.024     0    0.000    0.024
```

列表数据说明

```
S0    ：年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
S1    ：年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
E     ：年轻代中Eden（伊甸园）已使用的占当前容量百分比
O     ：old代已使用的占当前容量百分比
P    ：perm代已使用的占当前容量百分比
YGC  ：从应用程序启动到采样时年轻代中gc次数
YGCT   ：从应用程序启动到采样时年轻代中gc所用时间(s)
FGC   ：从应用程序启动到采样时old代(全gc)gc次数
FGCT    ：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)
```

## 10、gccause

  垃圾回收的信息，最后一次垃圾回收的信息，原因等

```
jstat -gccause 15172
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
 88.64   0.00  54.36   0.11  96.90  93.13      2    0.024     0    0.000    0.024 Allocation Failure   No GC
```

列表解释

```
LGCC：最后一次GC原因
GCC：当前GC原因（No GC 为当前没有执行GC）
```

## 11、printcompilation

​	vm执行信息

```
jstat -printcompilation 15172
Compiled  Size  Type Method
    1191     81    1 io/netty/util/concurrent/SingleThreadEventExecutor hasTasks
```

列表数据解释

```
Compiled ：编译任务的数目
Size ：方法生成的字节码的大小
Type：编译类型
Method：类名和方法名用来标识编译的方法。类名使用/做为一个命名空间分隔符。方法名是给定类中的方法。上述格式是由-XX:+PrintComplation选项进行设置的
```

