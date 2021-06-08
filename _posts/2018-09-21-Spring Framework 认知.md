---

layout: post

title: Spring Framework 认知

tag: 框架

---
# Spring Framework 认知

Spring Framework 是一个开发框架。

The Spring Framework provides a comprehensive programming and configuration model for modern Java-based enterprise applications - on any kind of deployment platform. A key element of Spring is infrastructural support at the application level: Spring focuses on the "plumbing" of enterprise applications so that teams can focus on application-level business logic, without unnecessary ties to specific deployment environments.

​		Spring框架为现代基于Java的企业应用提供了一个全面的编程和配置模型——在任何类型的部署平台上。Spring的一个关键元素是应用程序级别的基础结构支持：Spring关注企业应用程序的“管道”，这样团队就可以专注于应用程序级别的业务逻辑，而不必与特定的部署环境建立不必要的联系。

------

## 1、Spring Framework 主要组件

  官方提供的组件图：

![](https://github.com/superhxf/superhxf.github.io/blob/master/_posts/images/spring-framwwork-components.png)

### 1.1、四大核心容器

#### 1.1.1、Beans

​	Beans组件主要解决了Bean的定义，创建和解析，具体实现对应`org.springframework.beans`包下、

​	Bean的创建对应设计模式中的工厂模式，顶级接口是`BeanFactory`。

​	BeanFactory 有三个子类：ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory。但是从上图中我们可以发现最终的默认实现类是 DefaultListableBeanFactory，他实现了所有的接口。

​	每个接口都有他使用的场合，它主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制。例如 ListableBeanFactory 接口表示这些 Bean 是可列表的，而 HierarchicalBeanFactory 表示的是这些 Bean 是有继承关系的，也就是每个 Bean 有可能有父 Bean。AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则。这四个接口共同定义了 Bean 的集合、Bean 之间的关系、以及 Bean 行为。

#### 1.1.2、Core

​	Core组件是Spring的核心组件，包含了很多的关键类，例如:定义资源的访问方式，将所有的资源抽象成一个接口。

​	Context和Core之间的关系，Context会把资源的加载，解析和描述工作委托给ResourcePatternResolve来完成，Core完成了资源的加载，解析，定义 整合在一起方便其他组件使用，从名称上也可以看出Core组件在框架中扮演的角色。

#### 1.1.3、Context

​	Context 在 Spring 的 org.springframework.context 包下，实际上就是给 Spring 提供一个运行时的环境，用以保存各个对象的状态。**ApplicationContext** 是 Context 的顶级父类，他除了能标识一个应用环境的基本信息外，他还继承了五个接口，这五个接口主要是扩展了 Context 的功能。

ApplicationContext 的子类主要包含两个方面：

​	ConfigurableApplicationContext 表示该 Context 是可修改的，也就是在构建 Context 中用户可以动态添加或修改已有的配置信息，它下面又有多个子类，其中最经常使用的是可更新的 Context，即 AbstractRefreshableApplicationContext 类。

​	WebApplicationContext 顾名思义，就是为 web 准备的 Context 他可以直接访问到 ServletContext，通常情况下，这个接口使用的少。

#### 1.1.4、SpEL

​		Spring Expression Language 是一种强大的表达式语言，在Spring中的各个组件中，是表达式计算的基础，支持在运行时候查询和操作对象图，可以基于XML和基于注解的Spring噢诶之还有bean一起使用，由于运行时动态分配值，可以节省大量的java代码。

### 1.2、AOP和Instrumentation 

#### 1.2.1、AOP

spring-aop模块提供了一个符合aop设计要求的面向切面编程实现，允许定义方法级别的拦截器和切入点，干净的解耦了应该分离的功能实现，（如业务和日志记录）。

#### 1.2.2、spring-aspects

这个模块主要就是描述的spring与aspectJ的整合。

#### 1.2.3、spring-instrument

该模块提供了类植入的支持和类加载器的实现，可以应用在特定的应用服务器中。

### 1.3、消息

#### 1.3.1、messaging

spring messaging模块集成了nessaging api和消息协议提供支持。

### 1.4、数据访问及第三方集成

#### 1.4.1、JDBC

spring-jdbc模块提供了一个JDBC的抽象层，消除了需要的繁琐的JDBC编码和数据库厂商特有的错误代码。

#### 1.4.2、tx

spring-tx模块支持用于实现特殊接口和所有POJO类的编程和声明式事务管理。

#### 1.4.3、spring-orm

spring-orm模块为流行的ORM API提供了集成层，包括JPA和hibernetes，使用spring-orm模块，可以将这些O/R映射框架和spring提供的其他所有功能结合。

spring-orm封装了一个支持对象/XML映射的抽象层，如JAXB，CASTOR等、

#### 1.4.4、spring-jms

该模块包含用于生产和消息消息的功能，从4.1开始何spring-mesaging模块集成。

### 1.5、web层

#### 1.5.1、spring-web

spring-web模块提供了基本的面向web的集成功能，如文件按上传，初始化一个使用了servlet监听器和面向web的应用程序上下文的IOC容器，一个HTTP客户端，和spring远程支持。

#### 1.5.2、spring-web-mvc

spring-webmvc包含用于web应用程序的spring的模型-视图-控制器和REST web Services实现，Spring的MVC框架提供了领域模型代码和We表单之间的清洗分离，并且可以和spring其他所有的功能进行集成。

### 1.6、测试

#### 1.6.1、spring-test

模块支持使用JUnit或者TestNG对spring组件进行单元测试和集成测试，提供了Spring ApplicationContexts的一致加载和这些上下文的换缓存，提供了用于独立测试代码的模仿对象。



















