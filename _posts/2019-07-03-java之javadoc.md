# java之javadoc

在编写完成java程序中的文档注释后，可以使用javadoc工具提取程序中的文档注释来生成API文档。javadoc命令可对源文件、包生成API文档，java源文件可以支持通配符。

```
用法: javadoc [options] [packagenames] [sourcefiles] [@files]
  -overview <file>                 从 HTML 文件读取概览文档
  -public                          仅显示 public 类和成员
  -protected                       显示 protected/public 类和成员 (默认值)
  -package                         显示 package/protected/public 类和成员
  -private                         显示所有类和成员
  -help                            显示命令行选项并退出
  -doclet <class>                  通过替代 doclet 生成输出
  -docletpath <path>               指定查找 doclet 类文件的位置
  -sourcepath <pathlist>           指定查找源文件的位置
  -classpath <pathlist>            指定查找用户类文件的位置
  -cp <pathlist>                   指定查找用户类文件的位置
  -exclude <pkglist>               指定要排除的程序包列表
  -subpackages <subpkglist>        指定要递归加载的子程序包
  -breakiterator                   计算带有 BreakIterator 的第一个语句
  -bootclasspath <pathlist>        覆盖由引导类加载器所加载的
                                   类文件的位置
  -source <release>                提供与指定发行版的源兼容性
  -extdirs <dirlist>               覆盖所安装扩展的位置
  -verbose                         输出有关 Javadoc 正在执行的操作的信息
  -locale <name>                   要使用的区域设置, 例如 en_US 或 en_US_WIN
  -encoding <name>                 源文件编码名称
  -quiet                           不显示状态消息
  -J<flag>                         直接将 <flag> 传递到运行时系统
  -X                               输出非标准选项的提要

通过标准 doclet 提供:
  -d <directory>                   输出文件的目标目录
  -use                             创建类和程序包用法页面
  -version                         包含 @version 段
  -author                          包含 @author 段
  -docfilessubdirs                 递归复制文档文件子目录
  -splitindex                      将索引分为每个字母对应一个文件
  -windowtitle <text>              文档的浏览器窗口标题
  -doctitle <html-code>            包含概览页面的标题
  -header <html-code>              包含每个页面的页眉文本
  -footer <html-code>              包含每个页面的页脚文本
  -top    <html-code>              包含每个页面的顶部文本
  -bottom <html-code>              包含每个页面的底部文本
  -link <url>                      创建指向位于 <url> 的 javadoc 输出的链接
  -linkoffline <url> <url2>        利用位于 <url2> 的程序包列表链接至位于 <url> 的文档
  -excludedocfilessubdir <name1>:.. 排除具有给定名称的所有文档文件子目录。
  -group <name> <p1>:<p2>..        在概览页面中, 将指定的程序包分组
  -nocomment                       不生成说明和标记, 只生成声明。
  -nodeprecated                    不包含 @deprecated 信息
  -noqualifier <name1>:<name2>:... 输出中不包括指定限定符的列表。
  -nosince                         不包含 @since 信息
  -notimestamp                     不包含隐藏时间戳
  -nodeprecatedlist                不生成已过时的列表
  -notree                          不生成类分层结构
  -noindex                         不生成索引
  -nohelp                          不生成帮助链接
  -nonavbar                        不生成导航栏
  -serialwarn                      生成有关 @serial 标记的警告
  -tag <name>:<locations>:<header> 指定单个参数定制标记
  -taglet                          要注册的 Taglet 的全限定名称
  -tagletpath                      Taglet 的路径
  -charset <charset>               用于跨平台查看生成的文档的字符集。
  -helpfile <file>                 包含帮助链接所链接到的文件
  -linksource                      以 HTML 格式生成源文件
  -sourcetab <tab length>          指定源中每个制表符占据的空格数
  -keywords                        使程序包, 类和成员信息附带 HTML 元标记
  -stylesheetfile <path>           用于更改生成文档的样式的文件
  -docencoding <name>              指定输出的字符编码
```

生成api文档的操作比较简单，重点在于java源码中的javadoc标记。常用的标记有如下集中：

1. @author: 指定java程序的作者。
2. @version： 指定源文件的版本
3. @deprecated: 不推荐使用的方法。
4. @param：方法的参数说明信息。
5. @return ： 方法的返回值说明信息。
6. @see： 用于指定交叉参考的内容
7. @exception：抛出异常的类型
8. @throws： 抛出的异常 。



​	

​	