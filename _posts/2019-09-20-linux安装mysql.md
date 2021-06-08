---

layout: post

title:  linux安装mysql

tag: 数据库

---
#linux安装mysql

## 1、前言

​	MYSQL 是最流行的关系型数据库系统，在WEB应用方面MYSQL是最好的RDBMS，数据库就是按照数据结构来组织。存储和管理数据的仓库。

​	关系型数据库的特点：

1. 数据以表格的形式出现
2. 每行为各种记录名称
3. 每列为记录名称所对应的的数据域
4. 许多的行和列组成了一张表单
5. 若干的表单组成database。

好了，恢复主题，如何安装MYSQL，接下来，将使用俩种方式安装MYSQL，一种是传统的虚拟机上直接安装MYSQL的二进制文件，另一种就是现在比较火的容器技术，docker安装MYSQL。

## 2、虚拟机安装MYSQL

1. 下载mysql安装包

   ```
   wget https://cdn.mysql.com//archives/mysql-5.6/mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
   ```

2. 解压mysql到固定路径

   ```
   tar zxvf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz -C /usr/local/mysql
   mv mysql-5.6.33-linux-glibc2.5-x86_64 ../mysql1
   cd ..
   mv mysql1 mysql
   ```

3. 赋权

   ```
   cd mysql
   chmod -R 777 *
   ```

4. 创建mysql的用户组和用户

   ```
   groupadd mysql
   useradd -r -g mysql mysql
   ```

5. 给新建的用户授权

   ```
   cd mysql
   chmod -R mysql:mysql .
   ```

6. 开始安装mysql

   ```
   #安装指令
    ./scripts/mysql_install_db --user=mysql
    # 这时候一般会报错 差perl依赖 下载依赖
    yum install perl*
    yum install perl perl-devel
    yum install -y libaio
    再次执行安装 则会出现以下情况
    ./scripts/mysql_install_db --user=mysql
   Filling help tables...2019-09-20 14:09:18 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
   2019-09-20 14:09:18 0 [Note] ./bin/mysqld (mysqld 5.6.33) starting as process 8230 ...
   2019-09-20 14:09:18 8230 [Note] InnoDB: Using atomics to ref count buffer pool pages
   2019-09-20 14:09:18 8230 [Note] InnoDB: The InnoDB memory heap is disabled
   2019-09-20 14:09:18 8230 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
   2019-09-20 14:09:18 8230 [Note] InnoDB: Memory barrier is not used
   2019-09-20 14:09:18 8230 [Note] InnoDB: Compressed tables use zlib 1.2.3
   2019-09-20 14:09:18 8230 [Note] InnoDB: Using Linux native AIO
   2019-09-20 14:09:18 8230 [Note] InnoDB: Using CPU crc32 instructions
   2019-09-20 14:09:18 8230 [Note] InnoDB: Initializing buffer pool, size = 128.0M
   2019-09-20 14:09:18 8230 [Note] InnoDB: Completed initialization of buffer pool
   2019-09-20 14:09:18 8230 [Note] InnoDB: Highest supported file format is Barracuda.
   2019-09-20 14:09:18 8230 [Note] InnoDB: 128 rollback segment(s) are active.
   2019-09-20 14:09:18 8230 [Note] InnoDB: Waiting for purge to start
   2019-09-20 14:09:18 8230 [Note] InnoDB: 5.6.33 started; log sequence number 1625977
   2019-09-20 14:09:18 8230 [Note] Binlog end
   2019-09-20 14:09:18 8230 [Note] InnoDB: FTS optimize thread exiting.
   2019-09-20 14:09:18 8230 [Note] InnoDB: Starting shutdown...
   2019-09-20 14:09:20 8230 [Note] InnoDB: Shutdown completed; log sequence number 1625987
   OK
   To start mysqld at boot time you have to copy
   support-files/mysql.server to the right place for your system
   
   PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
   To do so, start the server, then issue the following commands:
     ./bin/mysqladmin -u root password 'new-password'
     ./bin/mysqladmin -u root -h instance-cj6k1fma password 'new-password'
   Alternatively you can run:
     ./bin/mysql_secure_installation
   which will also give you the option of removing the test
   databases and anonymous user created by default.  This is
   strongly recommended for production servers.
   See the manual for more instructions.
   You can start the MySQL daemon with:
     cd . ; ./bin/mysqld_safe &
   You can test the MySQL daemon with mysql-test-run.pl
     cd mysql-test ; perl mysql-test-run.pl
   Please report any problems at http://bugs.mysql.com/
   The latest information about MySQL is available on the web at
     http://www.mysql.com
   Support MySQL by buying support/licenses at http://shop.mysql.com
   New default config file was created as ./my.cnf and
   will be used by default by the server when you start it.
   You may edit this file to change server settings
   WARNING: Default config file /etc/my.cnf exists on the system
   This file will be read by default by the MySQL server
   If you do not want to use this, either remove it, or use the
   --defaults-file argument to mysqld_safe when starting the server
   #启动mysql
   [root@instance-cj6k1fma mysql]# ./support-files/mysql.server start
   Starting MySQL.. SUCCESS! 
   #更改密码
   ./bin/mysqladmin -u root -h localhost.localdomain password 'root'
   Warning: Using a password on the command line interface can be insecure.
   #操作数据库 这时候会出现以下错误
   [root@instance-cj6k1fma mysql]# ./bin/mysql -uroot -p
   Enter password: 
   ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
   # 修改配置文件
   [root@instance-cj6k1fma mysql]# vim my.cnf 
   #####################内容#######################
   character-set-server=utf8
   lower_case_table_names=1
   max_allowed_packet=100M
   socket=/var/lib/mysql/mysql.sock
   #################结束###########################
   #建立软连接
   ln -s  /var/lib/mysql/mysql.sock /tmp/mysql.sock 
   # 重启mysql服务
   [root@instance-cj6k1fma mysql]# ./support-files/mysql.server restart
   Shutting down MySQL.. SUCCESS! 
   Starting MySQL. SUCCESS! 
   #此时登录即可成功
   [root@instance-cj6k1fma mysql]# ./bin/mysql -uroot -p
   Enter password: 
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   ```

7. 操作数据库

   ```
   #登录数据库
   [root@instance-cj6k1fma mysql]# ./bin/mysql -uroot -p
   Enter password: 
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   #使用数据库
   [root@instance-cj6k1fma mysql]# ./bin/mysql -uroot -p
   Enter password: 
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 2
   Server version: 5.6.33 MySQL Community Server (GPL)
   Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   mysql> use mysql;
   Reading table information for completion of table and column names
   You can turn off this feature to get a quicker startup with -A
   Database changed
   mysql> select host,user,password from user;
   +-------------------+------+-------------------------------------------+
   | host              | user | password                                  |
   +-------------------+------+-------------------------------------------+
   | localhost         | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
   | instance-cj6k1fma | root |                                           |
   | 127.0.0.1         | root |                                           |
   | ::1               | root |                                           |
   | %                 | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
   +-------------------+------+-------------------------------------------+
   5 rows in set (0.00 sec)
   mysql> 
   
   ```

8. 增加远程登录权限

   ```
   grant all privileges on *.* to root@'%' identified by 'root';
   flush privileges;
   ```

9. 将mysql加入到service系统服务

   ```
   [root@instance-cj6k1fma mysql]# cp ./support-files/mysql.server /etc/init.d/mysqld
   [root@instance-cj6k1fma mysql]#     chkconfig --add mysqld
   #即可使用service命令进行服务管理
   [root@instance-cj6k1fma mysql]# service mysqld restart
   Shutting down MySQL.. SUCCESS! 
   Starting MySQL. SUCCESS! 
   [root@instance-cj6k1fma mysql]# service mysqld status
   SUCCESS! MySQL running (10159)
   ```

10. 使用DG访问mysql数据库，可以正常连接，这块就不再粘贴图片了

## 3、使用docker安装mysql

​       使用docker安装mysql的前提是必须都虚拟机上必须有docker服务，安装docker的过程不再累述，咱们直奔主题，如何使用docker启动一个mysql服务。

1. 查找mysql镜像

   ```
   # 使用docker命令查找mysql的镜像 ，这里面我使用的是百度云的服务器，docker服务是我自己安装的，连接就是公网的dockerhub
   root@instance-cj6k1fma mysql]# docker search mysql
   ... 列出了大概几十个mysql的镜像，在这里我们选择    mysql:5.6这个镜像                           
   ```

2. 拉取镜像

   ```
   [root@instance-cj6k1fma mysql]# docker pull mysql:5.6
   5.6: Pulling from library/mysql
   8f91359f1fff: Pull complete 
   6bbb1c853362: Pull complete 
   e6e554c0af6f: Pull complete 
   f391c1a77330: Pull complete 
   414a8a88eabc: Pull complete 
   ced966b42b0e: Pull complete 
   509b7d623b06: Pull complete 
   94a03dec9143: Pull complete 
   d40a4af51b78: Pull complete 
   e01756c8536e: Pull complete 
   327b60c1a0e5: Pull complete 
   Digest: sha256:07ebe49dc810444e172c2b5a72ae1a23ad9f4942bfe70a7f0a578590da610579
   Status: Downloaded newer image for mysql:5.6
   ```

3. 运行镜像

   ```
   #创建映射路径
   [root@instance-cj6k1fma mysql]#  mkdir -p ~/mysql/data ~/mysql/logs ~/mysql/conf
   [root@instance-cj6k1fma mysql]# cd ~/mysql
   root@instance-cj6k1fma mysql]# docker run -p 3307:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
   a428cf049a01e9382e07d2c7fe2b56c333f44146585fc663b23d0057da83ad9c
   [root@instance-cj6k1fma mysql]# docker ps
   CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
   a428cf049a01        mysql:5.6                   "docker-entrypoint.s…"   51 seconds ago      Up 47 seconds       0.0.0.0:3307->3306/tcp   mymysql
   eb13cf4818d9        squidfunk/mkdocs-material   "mkdocs serve -a 0.0…"   2 months ago        Up 2 months         0.0.0.0:80->8000/tcp     mkdocs
   ```

4. 使用DG进行连接 成功。

##     4、总结

​		到此为止，mysql的两种安装方式已经全部讲解完成，mysql作为现在比较火的一个关系型数据库，被很多项目所使用，对于开发者的技术栈来讲，mysql必不可少，学习mysql的第一步就是安装。

​    

