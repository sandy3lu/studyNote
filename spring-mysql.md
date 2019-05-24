

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