---

layout: post

title:  springboot中的条件注解

tag: 框架

---
# springboot中的条件注解

## 1、前言

​	@Conditional注解可以根据是否满足某一个特定的条件来决定要不要创建某个特定的Bean，比如一些条件，是否依赖，某个bean被注册之后再进行创建等，这个注解在spring framework 4.x版本之后开始提供。

​	springboot也是基础这个注解来实现的自动配置，并且，springboot在此基础上作了相应的扩展，形成了组合注解如下所示，

1. @ConditionalOnBean ： 当容器中有指定bean的条件下
2. @ConditionalOnClass：当classpath类路径下有指定类的条件下
3. @ConditionalPOnCloudPlatform： 当指定的云平台处于active状态下。
4. @ConditionalOnExpression：基于spel表达式的条件判断。
5. @ConditionalOnJava:基于jvm的版本作为判断条件。
6. @ConditionalOnMissingBean：当容器中没有指定bean 的条件。
7. @ConditionalOnMissingClass：当类路径下面没有指定类的条件下。
8. @ConditionalOnSingleCandidate：当指定的bean在容器中只有一个，或者有多个但是制定了首选的bean。
9. @ConditionalOnWebApplication：当项目是一个web项目、

## 2、Conditional是怎么实现的

​	先看一下Conditional的源码

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

对于springboot扩展出来的注解，随便拿一个来举例如下所示：

```
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnJavaCondition.class)
public @interface ConditionalOnJava {
	Range range() default Range.EQUAL_OR_NEWER;
	JavaVersion value();
	enum Range {
		EQUAL_OR_NEWER,
		OLDER_THAN
	}
}
```

 这样大概能看出了点东西，这个扩展的注解使用了@Conditional注解，并且要求要满足OnJavaCondition这个类中的条件。对于OnJavaCondition.java这个类的源码就不再展示，只讲一下具体实现的过程。

1. 继承了SpringBootCondition类，并且实现了它的getMatchOutcome方法。

2. 这个方法主要就是执行了一下操作

   1. 获取当前使用的jdk的版本
   2. 获取注解属性range范围和valuejdk版本
   3. 判断当前版本是否在指定范围内
   4. 返回比对结果

   