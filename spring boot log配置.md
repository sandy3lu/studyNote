
# 日志配置
## 默认
Spring Boot默认使用LogBack日志系统，如果不需要更改为其他日志系统如Log4j2等，则无需多余的配置，LogBack默认将日志打印到控制台上。

因为新建的Spring Boot项目一般都会引用spring-boot-starter或者spring-boot-starter-web，而这两个起步依赖中都已经包含了对于spring-boot-starter-logging的依赖，所以，无需额外添加依赖

Spring Boot默认的日志级别为INFO

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