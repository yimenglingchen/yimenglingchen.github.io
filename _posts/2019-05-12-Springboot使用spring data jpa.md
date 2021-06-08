---

layout: post

title: Springboot使用spring Data jpa

tag: 框架

---
# Springboot使用spring Data jpa

## 1、spring jpa简介

​	JPA(Java Persistence API)是Sun官方提出的Java持久化规范. 为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据. 它的出现是为了简化现有的持久化开发工作和整合ORM技术. 结束各个ORM框架各自为营的局面.

   JPA仅仅是一套规范,不是一套产品, 也就是说Hibernate, TopLink等是实现了JPA规范的一套产品。

   Spring Data JPA是Spring基于ORM框架、JPA规范的基础上封装的一套JPA应用框架,是基于Hibernate之上构建的JPA使用解决方案,用极简的代码实现了对数据库的访问和操作,包括了增、删、改、查等在内的常用功能.

## 2、如何使用

​	maven坐标

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

  常用配置

```
#通用数据源配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://xxx06/xxx?charset=utf8mb4&useSSL=false
spring.datasource.username=springboot
spring.datasource.password=springboot
# Hikari 数据源专用配置
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
# JPA 相关配置
spring.jpa.show-sql=true
```

配置解释

1. spring.jpa.show-sql=true 配置在日志中打印出执行的 SQL 语句信息。

## 3、程序体现

​	   定义一个实体类

```
package com.example.springbootjpa.entity;
import lombok.Data;
import org.hibernate.annotations.GenericGenerator;
import javax.persistence.*;
@Entity
@Table(name = "tb_user")
@Data
public class User {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;
    @Column(name = "username", unique = true, nullable = false, length = 64)
    private String username;
    @Column(name = "password", nullable = false, length = 64)
    private String password;
    @Column(name = "email", length = 64)
    private String email;
}
```

主键采用UUID策略
`@GenericGenerator`是Hibernate提供的主键生成策略注解，注意下面的`@GeneratedValue`（JPA注解）使用generator = "idGenerator"引用了上面的name = "idGenerator"主键生成策略。

JPA的主键策略

```
TABLE： 使用一个特定的数据库表格来保存主键
SEQUENCE： 根据底层数据库的序列来生成主键，条件是数据库支持序列。这个值要与generator一起使用，generator 指定生成主键使用的生成器（可能是orcale中自己编写的序列）
IDENTITY： 主键由数据库自动生成（主要是支持自动增长的数据库，如mysql）
AUTO： 主键由程序控制，也是GenerationType的默认值
```

​    dao层

```
package com.example.springbootjpa.repository;
import com.example.springbootjpa.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
public interface UserRepository extends JpaRepository<User, String> {
}
```

   继承JpaRepository，并且传入实体类和主键类型。

​    看一下Jpa中的类图关系

![](https://github.com/superhxf/superhxf.github.io/blob/master/_posts/images/10458268-9c37138a55393b99.webp)

最后看一下controller中的写法

```
package com.example.springbootjpa.controller;

import com.example.springbootjpa.entity.User;
import com.example.springbootjpa.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;
import java.util.Optional;
@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserRepository userRepository;
    @PostMapping()
    public User saveUser(@RequestBody User user) {
        return userRepository.save(user);
    }
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable("id") String userId) {
        userRepository.deleteById(userId);
    }
    @PutMapping("/{id}")
    public User updateUser(@PathVariable("id") String userId, @RequestBody User user) {
        user.setId(userId);
        return userRepository.saveAndFlush(user);
    }
    @GetMapping("/{id}")
    public User getUserInfo(@PathVariable("id") String userId) {
        Optional<User> optional = userRepository.findById(userId);
        return optional.orElseGet(User::new);
    }
    @GetMapping("/list")
    public Page<User> pageQuery(@RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
                                @RequestParam(value = "pageSize", defaultValue = "10") Integer pageSize) {
        return userRepository.findAll(PageRequest.of(pageNum - 1, pageSize));
    }

}
```

## 4、spring jpa中的查询

###     4.1、方法名解析

​	Spring Data Jpa通过解析方法名创建查询，框架在进行方法名解析时，会先把方法名多余的前缀find…By, read…By, query…By, count…By以及get…By截取掉，然后对剩下部分进行解析，第一个By会被用作分隔符来指示实际查询条件的开始。 我们可以在`实体属性`上定义条件，并将它们与And和Or连接起来，从而创建大量查询：

```
User findByUsername(String username);
List<User> findByUsernameIgnoreCase(String username);
List<User> findByUsernameLike(String username);
User findByUsernameAndPassword(String username, String password);
User findByEmail(String email);
List<User> findByEmailLike(String email);
List<User> findByIdIn(List<String> ids);
List<User> findByIdInOrderByUsername(List<String> ids);
void deleteByIdIn(List<String> ids);
Long countByUsernameLike(String username);
```

支持的关键字、示例及JPQL片段如下表所示：

| Keyword           | Sample                                                  | JPQL snippet                                                 |
| :---------------- | :------------------------------------------------------ | :----------------------------------------------------------- |
| And               | findByLastnameAndFirstname                              | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                | findByLastnameOrFirstname                               | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is,Equals         | findByFirstname,findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                     |
| Between           | findByStartDateBetween                                  | … where x.startDate between ?1 and ?2                        |
| LessThan          | findByAgeLessThan                                       | … where x.age < ?1                                           |
| LessThanEqual     | findByAgeLessThanEqual                                  | … where x.age <= ?1                                          |
| GreaterThan       | findByAgeGreaterThan                                    | … where x.age > ?1                                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual                               | … where x.age >= ?1                                          |
| After             | findByStartDateAfter                                    | … where x.startDate > ?1                                     |
| Before            | findByStartDateBefore                                   | … where x.startDate < ?1                                     |
| IsNull            | findByAgeIsNull                                         | … where x.age is null                                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                                    | … where x.age not null                                       |
| Like              | findByFirstnameLike                                     | … where x.firstname like ?1                                  |
| NotLike           | findByFirstnameNotLike                                  | ... findByFirstnameNotLike                                   |
| StartingWith      | findByFirstnameStartingWith                             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith                               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining                               | … where x.firstname like ?1 (parameter bound wrapped in %)   |
| OrderBy           | findByAgeOrderByLastnameDesc                            | … where x.age = ?1 order by x.lastname desc                  |
| Not               | findByLastnameNot                                       | … where x.lastname <> ?1                                     |
| In                | findByAgeIn(Collection<Age> ages)                       | … where x.age in ?1                                          |
| NotIn             | findByAgeNotIn(Collection<Age> ages)                    | … where x.age not in ?1                                      |
| True              | findByActiveTrue()                                      | … where x.active = true                                      |
| False             | findByActiveFalse()                                     | … where x.active = false                                     |
| IgnoreCase        | findByFirstnameIgnoreCase                               | … where UPPER(x.firstame) = UPPER(?1)                        |

### 4.2、自定义查询

​	@Query 注解的使用非常简单，只需在声明的方法上面标注该注解，同时提供一个 JPQL 查询语句即可。

```
@Query("select u from User u where u.email = ?1")
User getByEmail(String eamil);

@Query("select u from User u where u.username = ?1 and u.password = ?2")
User getByUsernameAndPassword(String username, String password);

@Query("select u from User u where u.username like %?1%")
List<User> getByUsernameLike(String username);
```

 使用命名参数Using Named Parameters

​		默认情况下，Spring Data JPA使用基于位置的参数绑定，如前面所有示例中所述。 这使得查询方法在重构参数位置时容易出错。 要解决此问题，可以使用`@Param`注解为方法参数指定具体名称并在查询中绑定名称，如以下示例所示：

```
@Query("select u from User u where u.id = :id")
User getById(@Param("id") String userId);

@Query("select u from User u where u.username = :username or u.email = :email")
User getByUsernameOrEmail(@Param("username") String username, @Param("email") String email);
```

### 4.3、使用spel

​	从Spring Data JPA release 1.4开始，Spring Data JPA支持名为entityName的变量。 它的用法是`select x from #{#entityName} x`。 entityName的解析方式如下：如果实体类在@Entity注解上设置了name属性，则使用它。 否则，使用实体类的简单类名。为避免在@Query注解使用实际的实体类名，就可以使用`#{#entityName}`进行代替。如以上示例中，@Query注解的查询字符串里的User都可替换为#{#entityName}。

```
@Query("select u from #{#entityName} u where u.email = ?1")
User getByEmail(String eamil);
```

### 4.4、自定义修改 删除

​	单独使用@Query注解只是查询，如涉及到修改、删除则需要再加上`@Modifying`注解，如：

```
@Transactional()
@Modifying
@Query("update User u set u.password = ?2 where u.username = ?1")
int updatePasswordByUsername(String username, String password);

@Transactional()
@Modifying
@Query("delete from User where username = ?1")
void deleteByUsername(String username);
```

### 4.5、表关系描述

​	多对对

```
package com.example.springbootjpa.entity;
import lombok.Data;
import org.hibernate.annotations.GenericGenerator;
import javax.persistence.*;
import java.util.Date;
import java.util.Set;
import java.util.UUID;
@Entity
@Table(name = "tb_user")
@Data
public class User {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;
    @Column(name = "username", unique = true, nullable = false, length = 64)
    private String username;
    @Column(name = "password", nullable = false, length = 64)
    private String password;
    @Column(name = "email", unique = true, length = 64)
    private String email;
    @ManyToMany(targetEntity = Role.class, cascade = {CascadeType.PERSIST, CascadeType.MERGE}, fetch = FetchType.LAZY)
    @JoinTable(name = "tb_user_role", joinColumns = {@JoinColumn(name = "user_id", referencedColumnName = "id")},
            inverseJoinColumns = {@JoinColumn(name = "role_id", referencedColumnName = "id")})
    private Set<Role> roles;
}
```

```
package com.example.springbootjpa.entity;
import lombok.Data;
import org.hibernate.annotations.GenericGenerator;
import javax.persistence.*;
@Entity
@Table(name = "tb_role")
@Data
public class Role {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;
    @Column(name = "role_name", unique = true, nullable = false, length = 64)
    private String roleName;
}
```

```
@Test
public void findByIdTest() {
    Optional<User> optional = userRepository.findById("40289f0c65674a930165674d54940000");
    Set<Role> roles = optional.get().getRoles();
    System.out.println(optional.get());
}
```

这块注意配置懒加载

```
spring:
  jpa:
    open-in-view: true
    properties:
      hibernate:
        enable_lazy_load_no_trans: true
```

一对多

```
package com.example.springbootjpa.entity;
import lombok.Data;
import org.hibernate.annotations.GenericGenerator;
import javax.persistence.*;
import java.util.Set;
@Entity
@Table(name = "tb_dept")
@Setter
@Getter
public class Department {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;
    @Column(name = "dept_name", unique = true, nullable = false, length = 64)
    private String deptName;
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private Set<Employee> employees;
}
```

```
package com.example.springbootjpa.entity;
import lombok.Data;
import org.hibernate.annotations.GenericGenerator;
import javax.persistence.*;
import java.util.UUID;
@Entity
@Table(name = "tb_emp")
@Setter
@Getter
public class Employee {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;
    @Column(name = "emp_name", nullable = false, length = 64)
    private String empName;
    @Column(name = "emp_job", length = 64)
    private String empJob;
    @Column(name = "dept_id", insertable = false, updatable = false)
    private String deptId;
    @ManyToOne(targetEntity = Department.class, cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "dept_id")
    private Department department;
}
```

```
@Test
public void findByIdTest() {
    Optional<Employee> optional = employeeRepository.findById("93fce66c1ef340fa866d5bd389de3d79");
    System.out.println(optional.get());
}
```

注意，使用@Data会出现问题。

## 5、spring data jpa的审计功能

​	一般数据库表在设计时都会添加4个审计字段，Spring Data Jpa同样支持审计功能。Spring Data提供了`@CreatedBy`，`@LastModifiedBy`，`@CreatedDate`，`@LastModifiedDate`4个注解来记录表中记录的创建及修改信息。

```
package com.example.springbootjpa.entity;

import lombok.Data;
import org.hibernate.annotations.GenericGenerator;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.util.Date;
import java.util.Set;

@Entity
@EntityListeners(AuditingEntityListener.class)
@Table(name = "tb_user")
@Data
public class User {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;
    @Column(name = "username", unique = true, nullable = false, length = 64)
    private String username;
    @Column(name = "password", nullable = false, length = 64)
    private String password;
    @Column(name = "email", unique = true, length = 64)
    private String email;
    @ManyToMany(targetEntity = Role.class, cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinTable(name = "tb_user_role", joinColumns = {@JoinColumn(name = "user_id", referencedColumnName = "id")},
            inverseJoinColumns = {@JoinColumn(name = "role_id", referencedColumnName = "id")})
    private Set<Role> roles;
    @CreatedDate
    @Column(name = "created_date", updatable = false)
    private Date createdDate;
    @CreatedBy
    @Column(name = "created_by", updatable = false, length = 64)
    private String createdBy;
    @LastModifiedDate
    @Column(name = "updated_date")
    private Date updatedDate;
    @LastModifiedBy
    @Column(name = "updated_by", length = 64)
    private String updatedBy;
}
```

实体类上还添加了`@EntityListeners(AuditingEntityListener.class)`，而AuditingEntityListener是由Spring Data Jpa提供的。

 实现AuditorAware接口，光添加了4个审计注解还不够，得告诉程序到底是谁在创建和修改表记录

```
package com.example.springbootjpa.auditing;
import org.springframework.data.domain.AuditorAware;
import org.springframework.stereotype.Component;
import java.util.Optional;
@Component
public class AuditorAwareImpl implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.of("admin");
    }
}
```

### 5.1、启动jpa的审计功能

```
package com.example.springbootjpa;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;
@SpringBootApplication
@EnableJpaAuditing
public class SpringBootJpaApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootJpaApplication.class, args);
    }
}
```

## 6、备注

​	spring data jpa官方地址： <https://docs.spring.io/spring-data/jpa/docs/current/reference/html/>

​    更多版本文档可以查阅，<https://docs.spring.io/spring-data/>。



