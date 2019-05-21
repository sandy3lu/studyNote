

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


