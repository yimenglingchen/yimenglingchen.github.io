---

layout: post

title: springboot中的流行实践解读

tag: 框架

---
# springboot中的流行实践解读

## 1、使用自定义的pom来维护第三方依赖

​	SpringBoot本身使用和集成了大量的开源项目，非常便捷的帮我们维护好了这些项目，比如数据源hikalio，但是也有一些在实际项目开发中并没有包括，或者用不上，这就需要在项目中自行维护，借鉴一下spring IO platform的思想，spring io platform是springboot的子项目，它维护了其他第三方的开源库，同理，我们可以维护自己的platform-bom，所有的业务模块项目使用BOM的方式引入，这样，升级第三方依赖的时候，只需要升级这一个依赖的BOM版本就可以了。

## 2、使用自动配置

​	springboot一个很核心的特性就是自动配置，它可以简化很多代码，当在类路径中检测到特定的jar的时候，自动配置就会被激活，比如，我们使用"spring-boot-starter-data-redis"，就可以和redis集成，我们还可以使用

```
@EnableAutoConfiguration（exclude={ClassNotToAutoConfig.class})
```

排除某些类的自动注入。

自动配置的官方文档地址：<https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html>

## 3、使用spring Initializr来创建一个新的springboot项目

​	地址是：<https://start.spring.io/>

   这个站点创建springboot项目很方便，内容涵盖springboot项目的版本管理方式，语言，springboot的版本，项目元数据，依赖（很赞，关键字补全），之后就会生成项目的zip，下载，解压，导入就可以了。

## 4、考虑为常见的组织问题创建自己的自动配置

​	如果你在一个严重依赖Spring Boot的公司或团队中工作，并且有共同的问题需要解决，那么你可以创建自己的自动配置。这项任务涉及较多工作，因此你需要考虑何时获益是值得投入的。与多个略有不同的定制配置相比，维护单个自动配置更容易。如果将这个提供Spring Boot配置以开源库的形式发布出去，那么将极大地简化数千个用户的配置工作。

## 5、正确设计代码目录结构

​	尽管允许你有很大的自由，但是有一些基本规则值得遵守来设计你的源代码结构。避免使用默认包。确保所有内容（包括你的入口点）都位于一个名称很好的包中，这样就可以避免与装配和组件扫描相关的意外情况；

```
    将Application.java（应用的入口类）保留在顶级源代码目录中；
```

 我建议将控制器和服务放在以功能为导向的模块中，但这是可选的。一些非常好的开发人员建议将所有控制器放在一起。不论怎样，坚持一种风格！

## 6、保持@Controller的简介和专注

​	Controller应该非常简单。你可以在此处阅读有关GRASP中有关控制器模式部分的说明。你希望控制器作为协调和委派的角色，而不是执行实际的业务逻辑。以下是主要做法：

1. 控制器应该是无状态的！默认情况下，控制器是单例，并且任何状态都可能导致大量问题；
2. 控制器不应该执行业务逻辑，而是依赖委托；
3. 控制器应该处理应用程序的HTTP层，这不应该传递给服务；
4. 控制器应该围绕用例/业务能力来设计。

## 7、围绕业务功能构建@Service

​	Service是Spring Boot的另一个核心概念。我发现最好围绕业务功能/领域/用例（无论你怎么称呼都行）来构建服务。

在应用中设计名称类似`AccountService`, `UserService`, `PaymentService`这样的服务，比起像`DatabaseService`、`ValidationService`、`CalculationService`这样的会更合适一些。

​	你可以决定使用Controler和Service之间的一对一映射，那将是理想的情况。但这并不意味着，Service之间不能互相调用！

## 8、让数据库独立于核心业务逻辑之外

​	你希望你的数据库逻辑于服务分离出来。理想情况下，你不希望服务知道它正在与哪个数据库通信，这需要一些抽象来封装对象的持久性。

## 9、保持业务逻辑不受Spring Boot代码的影响

​	考虑到“Clear Architecture”的教训，你还应该保护你的业务逻辑。将各种Spring Boot代码混合在一起是非常诱人的……不要这样做。如果你能抵制诱惑，你将保持你的业务逻辑可重用。部分服务通常成为库。如果不从代码中删除大量Spring注解，则更容易创建。

## 10、推荐使用构造函数注入

​	保持业务逻辑免受Spring Boot代码侵入的一种方法是使用构造函数注入。不仅是因为`@Autowired`注解在构造函数上是可选的，而且还可以在没有Spring的情况下轻松实例化bean。

## 11、熟悉并发模型

​	在Spring Boot中，Controller和Service是默认是单例。如果你不小心，这会引入可能的并发问题。你通常也在处理有限的线程池。请熟悉这些概念。

## 12、加强配置管理的外部化

​	这一点超出了Spring Boot，虽然这是人们开始创建多个类似服务时常见的问题……

你可以手动处理Spring应用程序的配置。如果你正在处理多个Spring Boot应用程序，则需要使配置管理能力更加强大。

我推荐两种主要方法：

1. 使用配置服务器，例如Spring Cloud Config；
2. 将所有配置存储在环境变量中（可以基于git仓库进行配置）。

这些选项中的任何一个（第二个选项多一些）都要求你在DevOps更少工作量，但这在微服务领域是很常见的。

## 13、提供全局异常处理

​	你真的需要一种处理异常的一致方法。Spring Boot提供了两种主要方法：

1. 你应该使用HandlerExceptionResolver定义全局异常处理策略；
2. 你也可以在控制器上添加@ExceptionHandler注解，这在某些特定场景下使用可能会很有用。

## 14、使用日志框架

​	你可能已经意识到这一点，但你应该使用Logger进行日志记录，而不是使用System.out.println()手动执行。这很容易在Spring Boot中完成，几乎没有配置。只需获取该类的记录器实例：

```
Logger logger = LoggerFactory.getLogger(MyClass.class);
```

这很重要，因为它可以让你根据需要设置不同的日志记录级别。

