---

layout: post

title:  Springboot中的actuator监控说明

tag: 框架

---
# Springboot中的actuator监控说明

## 1、前言

​	SpringBoot Actuator是SpringBoot四大神器之一，提供了全方位的监控和指标数据，通过HTTP或者JMX的形式访问，有了这些端点，就可以对应用程序进行全方便的指标分析甚至是关闭控制。

​	SpringBoot中的源生端点大概分为三类

1. 应用配置类
2. 度量指标类
3. 操作控制类

## 2、使用actuator

​	使用actuator步骤可以分为两步

1. 引入pom依赖

   ```
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
   ```

2. 配置相关属性，由于springboot从2.0开始对外只开发两个源生端点，所以，想要开发其他的端点需要配置

   ```
   management:
     endpoints:
       web:
         exposure:
           include: "*"
         base-path: /metric
     endpoint:
       health:
         show-details: always
   info:
     app: actuator-demo1
     age: 23
     name: testddd
   ```

   解释，我上面的这个配置文件有以下一些操作

   1. 展示所有的源生端点
   2. 基本路径由actuator 替换成metric
   3. 展示health的细节内容，否则只有一个应用是UP或者DOWN
   4. 配置info端点需要展示的内容
   5. 更多的可以借助idea的提醒功能查看配置内容。

3. 启动服务

   启动服务之后，就可以通过/metric 来访问，这个url会列出所有的可以访问的端点内容。

## 3、actuator端点的一些配置说明

1. 启用或者禁用端点 (一旦禁用，在应用的上下文中也会完全删除，不被容器管理)

   ```
   #启用某个端点
   management.endpoint.shutdown.enabled=true
   #修改全局端口默认配置
   management.endpoints.enabled-by-default=false
   management.endpoint.info.enable=true
   ```

2. 端点暴露

   ​	很多端点会显示出来敏感信息，比如beans会暴露出整个项目中被spring容器管理的bean，所以就需要应用的开发管理者来决定是否需要公开某些端点，或者何时公开这些端点。公开端点的做法如下所示

   ```
   # jmx暴露端点设置
   management.endpoints.jmx.exposure.exclude="*"
   management.endpoints.jmx.exposure.include="*"
   #http形式暴露端点
   management.endpoints.web.exposure.exclude="*"
   management.endpoints.web.exposure.include="*"
   #比如jmx公开health和info
   management.endpoints.jmx.exposure.include=health,info
   ```

   解释，include代表列出所有公开的端点，exclude代表的是列出所有的不公开的端点，exclude优先级要高于include。

3. 端点保护

   一般我们会对http访问的端点进行保护，不管是使用spring security的内容协商策略保护端点还是使用自定义的安全策略。

4. 端点缓存时间设置

   对于没有带参数的读取操作，端点会自动缓存相应，防止出现请求过大，对应用程序操作影响， 这个缓存时间是可以自定义设置的

   ```
   management.endpoint.beans.cache.time-to-live=10s
   ```

5. 端点的发现页

   发现页会指向所有的端点的链接，比如上文说的是`/metric`，默认的使用`/actuator`，但是，如果当基础路径设置为`/`，则会出现问题，系统会禁用发现页面以防止和其他映射出现冲突。

6. 端点的映射路径

   默认的话，在发现页可以找到所有的端点的访问地址，比如beans，可以通过`/actuator/beans`来访问，如果希望将这个端点映射到其他的路径，可以使用以下属性设置

   ```
   management.endpoints.web.base-path=base-path
   management.endpoints.web.path-mapping.health=healthcheck
   ```

7. 跨域支持

   跨域资源共享，spring mvc或者spring webFlux，可以配置actuator的web端点支持哪些场景。

   在默认的情况下， CORS会处于禁用的状态，看一个配置实例

   ```
   management.endpoints.web.cors.allowed-origins=http://actuator-demo.com
   management.endpoints.web.cors.allowed-methods=GET,POST
   ```

## 4、actuator中的端点内容详细介绍和配置

​	actuator源生端点大概有十几个，接下来会对常用的几个端点进行详细解释

### 4.1、beans端点

​	beans端点用于获取spring中上下文中创建的所有beans

  内容是这个样子的

```
{
	"contexts": {
		"actuator-demo1": {
			"beans": {
				"endpointCachingOperationInvokerAdvisor": {
					"aliases": [],
					"scope": "singleton",
					"type": "org.springframework.boot.actuate.endpoint.invoker.cache.CachingOperationInvokerAdvisor",
					"resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/EndpointAutoConfiguration.class]",
					"dependencies": [
						"environment"
					]
				},
				"defaultServletHandlerMapping": {
					"aliases": [],
					"scope": "singleton",
					"type": "org.springframework.web.servlet.HandlerMapping",
					"resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
					"dependencies": []
				}
```

   一些key的说明

1. contenxs：上下文
2. beans：beans列表
3. aliases：别名
4. scope：spring bean的生命周期 默认是singleton
5. type：类型 会打印出路径
6. resource：资源文件
7. dependencies： 依赖

通过这个端点，我们可以很清晰的看到当前应用程序中被spring管理的bean的详细信息。

### 4.2、health端点

​	health端点展示的是正在运行的应用程序的状态，health的展示内容取决于如下配置

```
mamagement.endpoint.health.show-details=never(永远不会展示)，when-authorized(授权展示),always(永远展示) 
#注意，默认的是never。
```

​    除了应用本身，health还对以下应用依赖中间件进行状态判断

| Name                         | Description                          |
| ---------------------------- | ------------------------------------ |
| CassandraHealthIndicator     | 检查Cassandra数据库是否启动          |
| DiskSpaceHealthIndicator     | 检查磁盘空间是否不足。               |
| DataSourceHealthIndicator    | 检查是否可以获得与DataSource的连接。 |
| ElasticsearchHealthIndicator | 检查Elasticsearch集群是否启动。      |
| InfluxDbHealthIndicator      | 检查InfluxDB服务器是否启动。         |
| JmsHealthIndicator           | 检查JMS代理是否启动。                |
| MailHealthIndicator          | 检查邮件服务器是否启动。             |
| MongoHealthIndicator         | 检查Mongo数据库是否启动。            |
| Neo4jHealthIndicator         | 检查Neo4j服务器是否启动。            |
| RabbitHealthIndicator        | 检查Rabbit服务器是否启动。           |
| RedisHealthIndicator         | 检查Redis服务器是否启动。            |
| SolrHealthIndicator          | 检查Solr服务器是否已启动。           |

健康端点能够提供很多的应用以及连接的中间件的状态信息，方便对外监控系统的使用，比如springboot-admin，再比如使用prometheus和grafana展示。

### 4.3、env端点

​	env端点显示出了从Spring的ConfigurableEnvironment中公开属性，获取的是应用所有可用的环境属性报告，像环境变量，JVM属性，应用的配置信息，命令行中的参数等。

​	通过env端点可以看到很多配置的内容，属于程序的极高的私密信息，建立不暴露，或者安全访问。

​    本身这个端点也提供了一些加密的方式，会对password，secret等敏感关键词的值使用*加密。

### 4.4、conditions端点

​	conditions端点在1.x版本叫做autoconfig，这个端点是用来获取应用的自动化配置报告，内容涵盖所有自动化配置的候选项，列出每个候选项自动化配置的条件是否满足，这个报告大概分为三个内容

1. positiveMatches：返回的就是条件匹配成功的自动化配置。
2. negativeMatches: 返回的是条件匹配不成功的自动化配置。
3. unconditionalClasses：

通过这个端点，可以很清晰的看到项目中哪些类加载是自动化配置通过的，哪些没有通过自动化配置，配置项没有满足或者覆盖再或者没有引入相关的依赖等。

### 4.5、configprops端点

​	这个端点用来获取应用中配置的属性信息报告，prefix代表了属性的配置前缀，properties代表了各个属性的名称和值。比如：

```
"management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties": {
	"prefix": "management.endpoints.web",
	"properties": {
		"pathMapping": {},
		"exposure": {
			"include": [
				"*"
			],
			"exclude": []
		},
		"basePath": "/metric"
	}
}
```

### 4.6、mappings端点

 这个端点会显示所有@RequestMapping的路径，比如

```
{
	"handler": "Actuator web endpoint 'auditevents'",
	"predicate": "{GET /metric/auditevents, produces [application/vnd.spring-boot.actuator.v2+json || application/json]}",
	"details": {
		"handlerMethod": {
			"className": "org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping.OperationHandler",
			"name": "handle",
			"descriptor": "(Ljavax/servlet/http/HttpServletRequest;Ljava/util/Map;)Ljava/lang/Object;"
		},
		"requestMappingConditions": {
			"consumes": [],
			"headers": [],
			"methods": [
				"GET"
			],
			"params": [],
			"patterns": [
				"/metric/auditevents"
			],
			"produces": [{
					"mediaType": "application/vnd.spring-boot.actuator.v2+json",
					"negated": false
				},
				{
					"mediaType": "application/json",
					"negated": false
				}
			]
		}
	}
},
```

会列出来接口的详细信息，包括实现类，访问地址，访问方式，请求类型等。

### 4.7、info端点

   info端点返回的是应用自定义的信息，比如应用版本，应用文档站点，应用其他的信息，或者应用作者等等，这些信息配置到application.yml中就可以。

```
info:
  app: actuator-demo1
  age: 23
  name: testddd
```

### 4.8、shutdown端点

​	在原生的端点中，提供了用来关闭应用的端点，/shutdown，默认是关闭的，如果需要的话可以通过

```
management.endpoint.shutdown.enable=true来开启。
```

这样的话，只要访问这个端点就可以关闭应用。

### 4.9、metric端点

​	这个端点会返回当前应用的各类重要的度量指标，jvm的，中间件的（tomcat或者jetty），操作系统的，日志，乃至系统启动时间等都有记录。

​	接下来，会对一直的常用的暴露出来的监控指标数据一一解释。

1.   jvm.memory.max  这个指标指的是可用作内存管理的最大内存，内容大概如下

   ```
   {
   	"name": "jvm.memory.max",
   	"description": "The maximum amount of memory in bytes that can be used for memory management",
   	"baseUnit": "bytes",
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 3439329279
   	}],
   	"availableTags": [{
   			"tag": "area",
   			"values": [
   				"heap",
   				"nonheap"
   			]
   		},
   		{
   			"tag": "id",
   			"values": [
   				"Compressed Class Space",
   				"PS Survivor Space",
   				"PS Old Gen",
   				"Metaspace",
   				"PS Eden Space",
   				"Code Cache"
   			]
   		}
   	]
   }
   ```

2. jvm.threads.states 展示的是线程数和状态 内容如下所示

   ```
   {
   	"name": "jvm.threads.states",
   	"description": "The current number of threads having TERMINATED state",
   	"baseUnit": "threads",
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 28
   	}],
   	"availableTags": [{
   				"tag": "state",
   				"values": [
   					"timed-waiting",
   					"new",
   					"runnable",
   					"blocked",
   					"waiting",
   					"terminated"
   				]
   ```

3. jvm.memory.used jvm使用内存，统计信息如下所示

   ```
   {
   	"name": "jvm.memory.used",
   	"description": "The amount of used memory",
   	"baseUnit": "bytes",
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 296142128
   	}],
   	"availableTags": [{
   			"tag": "area",
   			"values": [
   				"heap",
   				"nonheap"
   			]
   		},
   		{
   			"tag": "id",
   			"values": [
   				"Compressed Class Space",
   				"PS Survivor Space",
   				"PS Old Gen",
   				"Metaspace",
   				"PS Eden Space",
   				"Code Cache"
   			]
   		}
   	]
   }
   ```

4. jvm.gc.max.data.size  老年代内存最大值 ，展示内容如下所示

   ```
   {
   	"name": "jvm.gc.max.data.size",
   	"description": "Max size of old generation memory pool",
   	"baseUnit": "bytes",
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 1417674752
   	}],
   	"availableTags": []
   }
   ```

5. jvm.gc.pause jvm gc暂停时间统计 统计jvm进行垃圾回收的时候暂停系统多少次，总共时间多少次，如下所示

   ```
   {
   	"name": "jvm.gc.pause",
   	"description": "Time spent in GC pause",
   	"baseUnit": "seconds",
   	"measurements": [{
   			"statistic": "COUNT",
   			"value": 16
   		},
   		{
   			"statistic": "TOTAL_TIME",
   			"value": 0.313
   		},
   		{
   			"statistic": "MAX",
   			"value": 0
   		}
   	],
   	"availableTags": [{
   			"tag": "cause",
   			"values": [
   				"Metadata GC Threshold",
   				"Allocation Failure"
   			]
   		},
   		{
   			"tag": "action",
   			"values": [
   				"end of minor GC",
   				"end of major GC"
   			]
   		}
   	]
   }
   ```

6. system.cpu.count 操作系统的cpu个数，统计服务所在操作系统的cpu个数

   ```
   {
   	"name": "system.cpu.count",
   	"description": "The number of processors available to the Java virtual machine",
   	"baseUnit": null,
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 4
   	}],
   	"availableTags": []
   }
   ```

7. jvm.buffer.memory.used jvm缓冲区使用内存统计

   ```
   {
   	"name": "jvm.buffer.memory.used",
   	"description": "An estimate of the memory that the Java virtual machine is using for this buffer pool",
   	"baseUnit": "bytes",
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 90112
   	}],
   	"availableTags": [{
   		"tag": "id",
   		"values": [
   			"direct",
   			"mapped"
   		]
   	}]
   }
   ```

8. system.cpu.usage 对于操作系统最近cpu使用

   ```
   {
   	"name": "system.cpu.usage",
   	"description": "The \"recent cpu usage\" for the whole system",
   	"baseUnit": null,
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 0
   	}],
   	"availableTags": []
   }
   ```

9. process.uptime 应用已运行时间

   ```
   {
   	"name": "process.uptime",
   	"description": "The uptime of the Java virtual machine",
   	"baseUnit": "seconds",
   	"measurements": [{
   		"statistic": "VALUE",
   		"value": 67238.06
   	}],
   	"availableTags": []
   }
   ```

10. process.start.time 应用启动的时间点

    ```
    {
    	"name": "process.start.time",
    	"description": "Start time of the process since unix epoch.",
    	"baseUnit": "seconds",
    	"measurements": [{
    		"statistic": "VALUE",
    		"value": 1569315024.285
    	}],
    	"availableTags": []
    }
    ```

11. jvm.gc.memory.allocated gc时候，年轻代分配的内存空间

    ```
    {
    	"name": "jvm.gc.memory.allocated",
    	"description": "Incremented for an increase in the size of the young generation memory pool after one GC to before the next",
    	"baseUnit": "bytes",
    	"measurements": [{
    		"statistic": "COUNT",
    		"value": 5118203400
    	}],
    	"availableTags": []
    }
    ```

12. jvm.gc.memory.promoted  GC时，老年代分配的内存空间

    ```
    {
    	"name": "jvm.gc.memory.promoted",
    	"description": "Count of positive increases in the size of the old generation memory pool before GC to after GC",
    	"baseUnit": "bytes",
    	"measurements": [{
    		"statistic": "COUNT",
    		"value": 10834648
    	}],
    	"availableTags": []
    }
    ```

    还有很多其他的信息，这里就不再列出了。

    ### 4.10、dump端点

    ​	这个端点会生成当前进程活动的快照，报告中会包含应用程序的每一个线程，包含了很多线程特点信息，线程阻塞，锁状态等。

    