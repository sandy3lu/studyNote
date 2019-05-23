
# 简介

情景：
我们自己的系统中使用了logback这个日志系统。
我们的系统使用了A.jar，A.jar中使用的日志系统为log4j。
我们的系统又使用了B.jar，B.jar中使用的日志系统为slf4j-simple
 
这样，我们的系统就不得不同时支持并维护logback、log4j、slf4j-simple三种日志框架，非常不便

解决这个问题的方式就是引入一个适配层，由适配层决定使用哪一种日志系统，而调用端只需要做的事情就是打印日志而不需要关心如何打印日志，slf4j或者commons-logging就是这种适配层

`slf4j只是一个日志标准，并不是日志系统的具体实现`。slf4j只做两件事情：
- 提供日志接口
- 提供获取具体日志对象的方法

slf4j-simple、logback都是slf4j的具体实现，log4j并不直接实现slf4j，但是有专门的一层桥接slf4j-log4j12来实现slf4j。

# 实现原理
slf4j的用法就是常年不变的一句”Logger logger = LoggerFactory.getLogger(Object.class);“，通过LoggerFactory去拿slf4j提供的一个Logger接口的具体实现而已，

## 1.8之前
getLogger的时候会去classpath下找org/slf4j/impl/StaticLoggerBinder.class,即所有slf4j的实现，在提供的jar包路径下，一定是有”org/slf4j/impl/StaticLoggerBinder.class”存在的

不同的StaticLoggerBinder其getLoggerFactory实现不同，拿到ILoggerFactory之后调用一下getLogger即拿到了具体的Logger，可以使用Logger进行日志输出。

## 1.8之后
从slf4j 1.8起，slf4j使用SPI的方式寻找日志实现框架。通过SPI的方式寻找SLF4JServiceProvider的实现类。
```java
public interface SLF4JServiceProvider {
    // 返回ILoggerFactory的实现类
    public ILoggerFactory getLoggerFactory();
    
    // 初始化，实现类中一般用于初始化ILoggerFactory
    public void initialize();
}
```

# 例子
```java
@Test
public void testSlf4j() {
     Logger logger = LoggerFactory.getLogger(Object.class);
     logger.error("123");
}
```
```xml
 <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!-- 引入一个实现 -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
<!-- 可以引入的其他实现 -->
<!-- <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.21</version>
</dependency> -->
```

# 使用@Slf4j注解 
首先是在pom中引入:
```xml
<!--可以引入日志 @Slf4j注解-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

然后在类上写上@Slf4j注解 
在方法中直接使用 

如果注解@Slf4j注入后找不到变量log，需要IDEA安装lombok插件

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class LoggerTest {
    /**
     * Slf4j注解方式实现日志
     */
    @Test
    public void test2(){
        log.debug("debug");//默认日志级别为info
        log.info("info");
        log.error("error");
        log.warn("warn");
    }
}
```