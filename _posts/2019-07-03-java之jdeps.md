---

layout: post

title: java工具之jdeps工具

tag: java语言

---
# java之jdeps工具

​	jdeps(java dependencies)是java8推出一个命令行工具，此工具主要是帮助开发者们列出类依赖及层级关系的信息，如下所示，看一个spring-core包的类依赖关系。

```
$ jdeps spring-core-2.5.6.SEC03.jar
spring-core-2.5.6.SEC03.jar -> E:\program files\Java\jdk1.8\jre\lib\rt.jar
spring-core-2.5.6.SEC03.jar -> 找不到
   org.springframework.asm (spring-core-2.5.6.SEC03.jar)
      -> java.io
      -> java.lang
      -> java.lang.reflect
   org.springframework.asm.commons (spring-core-2.5.6.SEC03.jar)
      -> java.io
      -> java.lang
      -> java.security
      -> java.util
      -> org.springframework.asm                            spring-core-2.5.6.SEC03.jar
   org.springframework.asm.signature (spring-core-2.5.6.SEC03.jar)
      -> java.lang
   org.springframework.core (spring-core-2.5.6.SEC03.jar)
      -> edu.emory.mathcs.backport.java.util.concurrent     找不到
      -> java.io
      -> java.lang
      -> java.lang.ref
      -> java.lang.reflect
      -> java.util
      -> java.util.concurrent
      ....
```

具体使用一下看看它的参数

```
jdeps -h
用法: jdeps <options> <classes...>
其中 <classes> 可以是 .class 文件, 目录, JAR 文件的路径名,
也可以是全限定类名。可能的选项包括:
  -dotoutput <dir>                   DOT 文件输出的目标目录
  -s           -summary              仅输出被依赖对象概要
  -v           -verbose              输出所有类级别被依赖对象
                                     等同于 -verbose:class -filter:none。
  -verbose:package                   默认情况下输出程序包级别被依赖对象,
                                     不包括同一程序包中的被依赖对象
  -verbose:class                     默认情况下输出类级别被依赖对象,
                                     不包括同一程序包中的被依赖对象
  -cp <path>   -classpath <path>     指定查找类文件的位置
  -p <pkgname> -package <pkgname>    查找与给定程序包名称匹配的被依赖对象
                                     (可多次指定)
  -e <regex>   -regex <regex>        查找与指定模式匹配的被依赖对象
                                     (-p 和 -e 互相排斥)
  -f <regex>   -filter <regex>       筛选与指定模式匹配的被依赖对象
                                     如果多次指定, 则将使用最后一个被依赖对象。
  -filter:package                    筛选位于同一程序包内的被依赖对象 (默认)
  -filter:archive                    筛选位于同一档案内的被依赖对象
  -filter:none                       不使用 -filter:package 和 -filter:archive 筛选
                                     通过 -filter 选项指定的筛选仍旧适用。
  -include <regex>                   将分析限制为与模式匹配的类
                                     此选项筛选要分析的类的列表。
                                     它可以与向被依赖对象应用模式的
                                     -p 和 -e 结合使用
  -P           -profile              显示配置文件或包含程序包的文件
  -apionly                           通过公共类 (包括字段类型, 方法参数
                                     类型, 返回类型, 受控异常错误类型
                                     等) 的公共和受保护成员的签名
                                     限制对 API (即被依赖对象)
                                     进行分析
  -R           -recursive            递归遍历所有被依赖对象。
                                     -R 选项表示 -filter:none。如果指定了 -p, -e, -f
                                     选项, 则只分析匹配的
                                     被依赖对象。
  -jdkinternals                      在 JDK 内部 API 上查找类级别的被依赖对象。
                                     默认情况下, 它分析 -classpath 上的所有类
                                     和输入文件, 除非指定了 -include 选项。
                                     此选项不能与 -p, -e 和 -s 选项一起使用。
                                     警告: 在下一个发行版中可能无法访问
                                     JDK 内部 API。
  -version                           版本信息
```

核心参数：

1. jdeps -s  仅打印依赖性摘要。
2. jdeps -v （jdeps -verbose:class）打印所有类级别的依赖项。
3. jdeps -verbose:package 打印包级别依赖项
4. jdeps -cp (-classpath) 在何处查找类文件去打印依赖信息
5. jdeps -p （-package） 在何处查找类文件（包下）
6. jdeps -R （-recursive） 递归遍历所有的依赖项



