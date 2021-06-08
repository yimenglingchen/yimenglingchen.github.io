---

layout: post

title: Spring的IOC和DI

tag: 框架

---
# Spring的IOC和DI

## 1.概念

 IOC：Inversion of Control，控制反转，这是一种思想，在java语言中，可以理解为设计好的对象交给容器管理，而不是直接在使用方直接控制，在传统的做法中，我们是直接在使用方直接new出来的对象，是程序主动去创建依赖对象，但是IOC会提供一个容器来创建这些对象，让IOC容器来管理控制对象的创建。

DI： dependency Injection 依赖注入，组件之间的依赖关系是容器在运行时决定的，可以理解为容器动态的将某个依赖管理注入到组件之中，依赖注入提升了组件重用的拼接，为系统搭建了一个灵活，可以扩展的平台。

## 2、Spring中的实现

​	在spring中主要通过spring-beans,spring-cores，spring-context模块实现了IOC和DI，梳理一下关系。

​    core负责了发现，创建并且处理bean之间关系，core把bean的创建，注入的方法都做好了，上次模块只需要调用就可以了。

   beans主要是bean的定义，创建和bean的解析。

​    context来调用，bean其实包装的是一个Object，Object是有数据的，如何给这些数据提供环境则是context包需要做的事情，context使用core发现bean之间的关系，并且维护好这种关系，context就是bean关系的集合，我们称之为IOC容器。
