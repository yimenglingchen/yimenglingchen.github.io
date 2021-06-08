---

layout: post

title:  如何写一个springboot的starter

tag: 框架

---
# 如何写一个springboot的starter

## 1、前言

​	springboot相对于spring来讲，最大的优点就是开箱即用，springboot的starter，内部引入了相关组件的jar，并且封装了默认配置，当我们引用这个starter的时候，就可以直接使用集成的这些框架。比如，redis，只需要将对应的spring-boot-starter-redis通过pom引入就可以，不需要自己在去查找各种依赖，并且，springboot自动完成了一些类的配置，注入到上下文中。

## 2、starter的原理执行步骤

1. springboot在启动的时候在依赖的starter包中寻找`resources/META_INF/spring.factories`文件，根据这个配置文件去扫描项目中需要加载的类，所在的包。
2. 根据spring.factories 文件配置加载AutoConfigure类。
3. 根据@COnditional 族注解，进行自动配置到上下文中。

## 3、starter包的命令规则

​	根据官方的starter命令，官方一般采取spring-boot-starter-｛name｝的命令方式，如 spring-boot-starter-demo1。

​    如果是非官方的，建议命名规则为｛name｝-spring-boot-starter的格式。

## 4、内部实现

1. ​	根据starter的需求开发出相关的功能性代码之后，需要编写一个自动配置类来实现什么时候进行功能类的加载。这时候就是用到了条件加载注解，@Conditional相关的注解，这里就不再列出。

2. 完成了自动配置类之后，需要让spring容器知道这个类，此时，使用spring.factorice文件，这个文件一定要放在`resources/META-INF/`目录下，内容如下

   ```
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=改成自动配置类路径
   ```

​      这里使用了一个spring框架的一个特性，就是在启动的时候，spring会遍历各个jar中的META-INF目录下的spring.factories文件构成一个配置文件链表，进行加载，看过spring boot的启动时，run()方法源码的同学应该看到过这段加载的程序。

## 5、打包发布

   完成一个基本的starter的编写之后，使用maven工具将其发布到公司私服上，命令如下

```
mvn clean deploy
```

   此时，新启项目，引入此starter，使用@Autowired注解，便可以直接使用功能类了，应为符合条件加载，该功能类已经被spring的上下文管理了起来，使用自动注入功能便可获取该类的实例使用其功能。

​	