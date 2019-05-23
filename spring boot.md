

# 依赖

Spring Boot的父级依赖，这样当前的项目就是Spring Boot项目了。
spring-boot-starter-parent 是一个特殊的starter，它用来提供相关的Maven默认依赖。使用它之后，常用的包依赖可以省去version标签

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.0.RELEASE</version>
</parent>

<!-- web应用，可以像下面这样添加spring-boot-starter-web依赖： -->
<dependencies>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
</dependencies>

```
