---

layout: post

title: springboot中的actuator
tag: 框架

---
# springboot中的actuator

​      `actuator`是`spring boot`项目中非常强大一个功能，有助于对应用程序进行监视和管理，通过`restful api`请求来监管、审计、收集应用的运行情况，针对微服务而言它是必不可少的一个环节…

## 1、Endpoints介绍

​      actuator的核心部分，它用来监视应用程序及交互，spring-boot-actuator中已经内置了非常多的Endpoints（health、info、beans、httptrace、shutdown等等），同时也允许我们自己扩展自己的端点

​        Spring Boot 2.0中的端点和之前的版本有较大不同,使用时需注意。另外端点的监控机制也有很大不同，启用了不代表可以直接访问，还需要将其暴露出来，传统的management.security管理已被标记为不推荐。

内置的Endpoints列表

| id                 | desc                                                         | Sensitive |
| ------------------ | ------------------------------------------------------------ | --------- |
| **auditevents**    | 显示当前应用程序的审计事件信息                               | Yes       |
| **beans**          | 显示应用Spring Beans的完整列表                               | Yes       |
| **caches**         | 显示可用缓存信息                                             | Yes       |
| **conditions**     | 显示自动装配类的状态及及应用信息                             | Yes       |
| **configprops**    | 显示所有 @ConfigurationProperties 列表                       | Yes       |
| **env**            | 显示 ConfigurableEnvironment 中的属性                        | Yes       |
| **flyway**         | 显示 Flyway 数据库迁移信息                                   | Yes       |
| **health**         | 显示应用的健康信息（未认证只显示`status`，认证显示全部信息详情） | No        |
| **info**           | 显示任意的应用信息（在资源文件写info.xxx即可）               | No        |
| **liquibase**      | 展示Liquibase 数据库迁移                                     | Yes       |
| **metrics**        | 展示当前应用的 metrics 信息                                  | Yes       |
| **mappings**       | 显示所有 @RequestMapping 路径集列表                          | Yes       |
| **scheduledtasks** | 显示应用程序中的计划任务                                     | Yes       |
| **sessions**       | 允许从Spring会话支持的会话存储中检索和删除用户会话。         | Yes       |
| **shutdown**       | 允许应用以优雅的方式关闭（默认情况下不启用）                 | Yes       |
| **threaddump**     | 执行一个线程dump                                             | Yes       |
| **httptrace**      | 显示HTTP跟踪信息（默认显示最后100个HTTP请求 - 响应交换）     | Yes       |

## 2、如何使用

1、导入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2、`info`接口想获取`maven`中的属性内容请记得添加如下内容

```
<build>
      <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

3、配置

​		在`application.properties`文件中配置`actuator`的相关配置，其中`info`开头的属性，就是访问`info`端点中显示的相关内容，值得注意的是`Spring Boot2.x`中，默认只开放了`info、health`两个端点，剩余的需要自己通过配置`management.endpoints.web.exposure.include`属性来加载（有`include`自然就有`exclude`，不做详细概述了）。如果想单独操作某个端点可以使用`management.endpoint.端点.enabled`属性进行启用或禁用

```
# 描述信息
info.blog-url=http://winterchen.com
info.author=Luis
info.version=@project.version@
# 加载所有的端点/默认只加载了 info / health
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
# 可以关闭制定的端点
management.endpoint.shutdown.enabled=false
# 路径映射，将 health 路径映射成 rest_health 那么在访问 health 路径将为404，因为原路径已经变成 rest_health 了，一般情况下不建议使用
# management.endpoints.web.path-mapping.health=rest_health
```

## 3、自定义

### 3.1、默认装配 HealthIndicators

​	下列是依赖`spring-boot-xxx-starter`后相关`HealthIndicator`的实现（通过`management.health.defaults.enabled`属性可以禁用它们），但想要获取一些额外的信息时，自定义的作用就体现出来了…

| 名称                             | 描述                               |
| -------------------------------- | ---------------------------------- |
| **CassandraHealthIndicator**     | 检查`Cassandra`数据库是否启动。    |
| **DiskSpaceHealthIndicator**     | 检查磁盘空间不足。                 |
| **DataSourceHealthIndicator**    | 检查是否可以获得连接`DataSource`。 |
| **ElasticsearchHealthIndicator** | 检查`Elasticsearch`集群是否启动。  |
| **InfluxDbHealthIndicator**      | 检查`InfluxDB`服务器是否启动。     |
| **JmsHealthIndicator**           | 检查`JMS`代理是否启动。            |
| **MailHealthIndicator**          | 检查邮件服务器是否启动。           |
| **MongoHealthIndicator**         | 检查`Mongo`数据库是否启动。        |
| **Neo4jHealthIndicator**         | 检查`Neo4j`服务器是否启动。        |
| **RabbitHealthIndicator**        | 检查`Rabbit`服务器是否启动。       |
| **RedisHealthIndicator**         | 检查`Redis`服务器是否启动。        |
| **SolrHealthIndicator**          | 检查`Solr`服务器是否已启动。       |

### 3.2、自定义健康端点

1. 方式1

```
package com.winterchen.health;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;
/**
 * 自定义健康端点
 */
@Component("my1")
public class MyHealthIndicator implements HealthIndicator {
    private static final String VERSION = "v1.0.0";
    @Override
    public Health health() {
        int code = check();
        if (code != 0) {
            Health.down().withDetail("code", code).withDetail("version", VERSION).build();
        }
        return Health.up().withDetail("code", code)
                .withDetail("version", VERSION).up().build();
    }
    private int check() {
        return 0;
    }
}
```

2、方式2

​	继承AbstractHealthIndicator抽象类，重写doHealthCheck方法，功能比第一种要强大一点点，默认的DataSourceHealthIndicator 、 RedisHealthIndicator都是这种写法，内容回调中还做了异常的处理。

```
package com.winterchen.health;
import org.springframework.boot.actuate.health.AbstractHealthIndicator;
import org.springframework.boot.actuate.health.Health;
import org.springframework.stereotype.Component;
/**
 * 自定义健康端点
 * <p>功能更加强大一点，DataSourceHealthIndicator / RedisHealthIndicator 都是这种写法</p>
 */
@Component("my2")
public class MyAbstractHealthIndicator extends AbstractHealthIndicator {
    private static final String VERSION = "v1.0.0";
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        int code = check();
        if (code != 0) {
            builder.down().withDetail("code", code).withDetail("version", VERSION).build();
        }
        builder.withDetail("code", code)
                .withDetail("version", VERSION).up().build();
    }
    private int check() {
        return 0;
    }
}
```

## 4、定义应用自己扩展访问点

​	上面介绍的**info、health**都是`spring-boot-actuator`内置的，真正要实现自己的端点还得通过`@Endpoint、 @ReadOperation、@WriteOperation、@DeleteOperation`。

不同请求的操作，调用时缺少必需参数，或者使用无法转换为所需类型的参数，则不会调用操作方法，响应状态将为400（错误请求）

1. @Endpoint构建 rest api 的唯一路径
2. **`@ReadOperation`**GET请求，响应状态为 200 如果没有返回值响应 404（资源未找到）
3. @WriteOperationPOST请求，响应状态为 200 如果没有返回值响应 204（无响应内容）
4. @DeleteOperationDELETE请求，响应状态为 200 如果没有返回值响应 204（无响应内容）

```
package com.winterchen.endpoint;
import org.springframework.boot.actuate.endpoint.annotation.Endpoint;
import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;
import java.util.HashMap;
import java.util.Map;
/**
 * * <p>@Endpoint 是构建 rest 的唯一路径 </p>
 * 不同请求的操作，调用时缺少必需参数，或者使用无法转换为所需类型的参数，则不会调用操作方法，响应状态将为400（错误请求）
 * <P>@ReadOperation = GET 响应状态为 200 如果没有返回值响应 404（资源未找到） </P>
 * <P>@WriteOperation = POST 响应状态为 200 如果没有返回值响应 204（无响应内容） </P>
 * <P>@DeleteOperation = DELETE 响应状态为 200 如果没有返回值响应 204（无响应内容） </P>
 */
@Endpoint(id = "demopoints")
@Component
public class MyEndPoint {
    @ReadOperation
    public Map<String, String> hello() {
        Map<String, String> result = new HashMap<>();
        result.put("author", "hxf");
        result.put("age", "25");
        result.put("email", "xxx.gmail.com");
        return result;
    }
}
```

