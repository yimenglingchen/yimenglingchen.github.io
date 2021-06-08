# spring中的spring-data介绍和使用



## 	1、前文

​	Spring data是spring中的一个子项目，旨在简化对数据库的访问，支持NoSql和关系型数据库，主要的目的是让数据库的访问变得方便，快捷，规范。

​	看一下官方站点对于spring-data的介绍

```
   spring data的任务是为数据访问提供一个熟悉的、一致的、基于spring的编程模型，同时仍然保留底层数据存储的特殊特性。
    它使使用数据访问技术、关系数据库和非关系数据库、map-reduce框架以及基于云的数据服务变得容易。这是一个伞形项目，包含许多特定于给定数据库的子项目。这些项目是通过与这些令人兴奋的技术背后的许多公司和开发人员合作开发的。
```

​	翻译过来的意思就是，spring-data做的抽象，让我们可以基于这些抽象去做使用类似乃至相同的操作去对底层的各个存储进行操作，对我们的好处就是更加方便和稳定的集成了存储，而不用每一个项目都去单独写这些和存储连接操作的中间工具代码。

## 2、支持

从官网上看大概支持这么多

1. Spring Data Commons
2. Spring Data JPA
3. Spring Data KeyValue
4. Spring Data LDAP
5. Spring Data MongoDB
6. Spring Data Redis
7. Spring Data REST
8. Spring Data for Apache Cassandra
9. Spring Data for Apache Geode
10. Spring Data for Apache Solr
11. Spring Data for Pivotal GemFire
12. Spring Data Couchbase (community module)
13. Spring Data Elasticsearch (community module)
14. Spring Data Neo4j (community module)

这块只列出项目中常用的几种

​	NOSQL：

1. MongoDB（文档数据库）
2. Redis（键值对存储）
3. HBase（列族数据库）

关系型数据库

1. JDBC
2. JPA

### 2.1、spring data对mongoDB的支持





​	

