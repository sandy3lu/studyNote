

## mysql 5 vs 6的驱动
com.mysql.jdbc.Driver 是 mysql-connector-java 5中的，
com.mysql.cj.jdbc.Driver 是 mysql-connector-java 6中的

```yml
# JDBC连接Mysql5 com.mysql.jdbc.Driver
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=


# JDBC连接Mysql6 com.mysql.cj.jdbc.Driver， 需要指定时区serverTimezone:如果在中国，可以选择Asia/Shanghai或者Asia/Hongkong
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
```
# mybatis

每一个Mybatis的应用程序都以一个SqlSessionFactory对象的实例为核心。
- 首先用字节流通过Resource将配置文件读入，
- 然后通过SqlSessionFactoryBuilder().build方法创建SqlSessionFactory，
- 然后再通过SqlSessionFactory.openSession()方法创建一个SqlSession为每一个数据库事务服务。
- 经历了Mybatis初始化 –>创建SqlSession –>运行SQL语句，返回结果三个过程




# myBatis plus
全新的 MyBatis-Plus 3.0 版本基于 JDK8，提供了 lambda 形式的调用

## 引入包
```xml
<!-- Spring boot -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.1.1</version>
</dependency>
```
> 引入 MyBatis-Plus 之后请不要再次引入 MyBatis 以及 MyBatis-Spring，以避免因版本差异导致的问题。


## 配置MapperScan
```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }

}
```

# 代码生成器
AutoGenerator 是 MyBatis-Plus 的代码生成器，通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率
## 依赖
MyBatis-Plus 从 3.0.3 之后移除了代码生成器与模板引擎的默认依赖，需要手动添加相关依赖
```xml
<!-- 添加 代码生成器 依赖 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.1.1</version>
</dependency>
```

## 模板引擎
MyBatis-Plus 支持 Velocity（默认）、Freemarker、Beetl，用户可以选择自己熟悉的模板引擎，如果都不满足您的要求，可以采用自定义模板引擎
```xml
<!-- Freemarker模板 -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.28</version>
</dependency>
```

如果您选择了非默认引擎，需要在 AutoGenerator 中 设置模板引擎。
```java
AutoGenerator generator = new AutoGenerator();
// set freemarker engine
generator.setTemplateEngine(new FreemarkerTemplateEngine());
// set beetl engine
generator.setTemplateEngine(new BeetlTemplateEngine());
// set custom engine (reference class is your custom engine class)
generator.setTemplateEngine(new CustomTemplateEngine());
```

## 编写配置
```java


```


