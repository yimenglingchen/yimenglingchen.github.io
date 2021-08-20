---

layout: post

title:  maven项目迁移到gradle流程

tag: 设计模式

---

# maven项目迁移到gradle流程

---

## 1 背景

现在已经存在一个可以正常运行的maven+springBoot项目，需要将该项目转换成gradle项目，并能够正常打包部署。

---

## 2 gradle准备工作

### 2.1 下载gradle

[gradle官网地址](https://gradle.org/)

[gradle7.2版本下载地址](https://gradle.org/next-steps/?version=7.2&format=all)

### 2.2 gradle环境变量配置

将下载好的包解压后，复制所在路径，添加如下环境变量配置：

![alt GRADLE_HOME配置](/images/posts/maven项目迁移到gradle流程/GRADLE_HOME配置.png)

![alt path配置](/images/posts/maven项目迁移到gradle流程/path配置.png)

其中GRADLE_HOME配置的路径就是gradle所在路径。

---

## 3 迁移

### 3.1 maven转为gradle

gradle提供了转化的语句。首先打开命令行窗口（cmd.exe），进入项目所在根目录，然后输入：

```
gradle init --type pom
```

然后就可以静等项目的gradle相关文件生成。

### 3.2 修改gradle配置

在命令窗口提示成功后，便可以通过Idea打开此项目，此时右下角会提示通过gradle加载本项目，点击加载，即可开始加载gradle。

注：加载后，pom.xml文件依旧存在，后续只需要重新打开项目，右下角就会有通过maven加载本项目的提醒，右侧边栏同样也有maven。

此时项目概率启动失败，因为pom.xml在迁移过来之后可能会导致部分包版本失效，例如引入本地jar包，需要手动更改build.gradle中如下样式：

```groovy
implementation fileTree(dir:'lib',includes:['*jar'])
```

dir为项目根目录中的lib文件夹，在该文件夹中的jar包都会被引入进来。

其余情况也可根据具体问题进行配置。

此时，通过gradle的Tasks->build->jar打出的jar包是不包含引入的jar包，只会将项目中的文件编译，因此还需要配置springBoot插件来进行打包。

配置方式：

在build.gradle文件的最上面添加如下插件配置

```groovy
buildscript {
    repositories {
        maven { url 'https://repo.spring.io/libs-milestone' }
    }

    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:2.5.4'
    }
}
```

在plugins括号里添加如下一行（不确定是否必须）：

```groovy
id 'idea'
```

最后在下面添加一行：

```groovy
apply plugin: 'org.springframework.boot'
```

需注意，buildscript、plugins、apply plugin三个配置项必须按照顺序进行配置，否则会失败。

配置完成后，执行该配置文件，刷新配置信息。

刷新后，gradle执行时会出现对应的执行事务，执行bootJar即可打包成功。

![alt boot](/images/posts/maven项目迁移到gradle流程/boot.png)

---

## 4 小结

因为刚刚开始接触gradle，还不清楚gradle和maven相比有哪些优势，目前能够确定的只有配置中，不需要大量的xml，看起来更加清爽简洁外，其余的优势暂未发现。在后续的学习过程里，我还会继续研究它和maven的优劣。