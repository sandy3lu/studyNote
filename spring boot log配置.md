
# 日志配置
## 默认logback
Spring Boot默认使用LogBack日志系统，如果不需要更改为其他日志系统如Log4j2等，则无需多余的配置，LogBack默认将日志打印到控制台上。

因为新建的Spring Boot项目一般都会引用spring-boot-starter或者spring-boot-starter-web，而这两个起步依赖中都已经包含了对于spring-boot-starter-logging的依赖，所以，无需额外添加依赖

Spring Boot默认的日志级别为INFO

## 改变日志设定
在文件系统的任意一个位置提供自己的logback.xml配置文件，然后通过logging.config配置项指向这个配置文件然后引用它，例如在application.properties中指定如下的配置：
```yml
logging.config=/{some.path.you.defined}/any-logfile-name-I-like.log}
```

## 打印日志到文件
- logging.path 该属性用来配置日志文件的路径
- logging.file 该属性用来配置日志文件名，如果该属性不配置，默认文件名为spring.log
```.yml
logging.path=/Users/jackie/workspace/rome/ 
logging.file=springbootdemo.log
```

## 设置日志级别
日志级别总共有TARCE < DEBUG < INFO < WARN < ERROR < FATAL ，且级别是逐渐提供，如果日志级别设置为INFO，则意味TRACE和DEBUG级别的日志都看不到。
```yml
# root级别，即项目的所有日志
logging.level.root=INFO
# 将指定包下的日志级别设置为WARN
logging.level.com.jackie.springbootdemo.config=WARN
```

## 定制自己的日志格式
%d{HH:mm:ss.SSS}——日志输出时间

%thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用

%-5level——日志级别，并且使用5个字符靠左对齐

%logger- ——日志输出者的名字

%msg——日志消息

%n——平台的换行符

```yml
logging.pattern.console=%d{yyyy/MM/dd-HH:mm:ss} [%thread] %-5level %logger- %msg%n 
logging.pattern.file=%d{yyyy/MM/dd-HH:mm} [%thread] %-5level %logger- %msg%n
```

## 使用log4j2

1. Spring Boot默认使用LogBack，但是我们没有看到显示依赖的jar包，其实是因为所在的jar包spring-boot-starter-logging都是作为spring-boot-starter-web或者spring-boot-starter依赖的一部分。
如果这里要使用Log4j2，需要从spring-boot-starter-web中去掉spring-boot-starter-logging依赖，同时显示声明使用Log4j2的依赖jar包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

2. 在resources目录下新建一个log4j2.xml文件
但是这样还不够，Spring Boot并不知道log4j2.xml是干嘛的，需要通过在application.properties文件中显示声明才行
logging.config= classpath:log4j2.xml


注意，这里有个“潜规则”。如果想在application.properties中注释掉和配置文件的关系前提下仍然能读取到配置文件的信息，可以这样做
将log4j2.xml重命名为`log4j2-spring.xml`

## 自定义日志配置
根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：
Logback： logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy
Log4j： log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml
Log4j2： log4j2-spring.xml, log4j2.xml
JDK (Java Util Logging)： logging.properties


# 配置文件
spring默认使用yml中的配置，但有时候要用传统的xml或properties配置，就需要使用spring-boot-configuration-processor了
```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
 </dependency>
```
再在你的配置类开头加上@PropertySource("classpath:your.properties")，其余用法与加载yml的配置一样