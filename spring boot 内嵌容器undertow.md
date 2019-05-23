
# 简介
默认情况下,Spring Boot使用Tomcat来作为内嵌的Servlet容器。
Undertow是一个Java开发的灵活的高性能Web服务器，提供包括阻塞和基于NIO的非阻塞机制。Undertow是红帽公司的开源产品，是Wildfly默认的Web服务器

# 配置
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 从依赖信息里移除 Tomcat配置 -->
    <exclusions> 
        <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions> 
</dependency>

 <!-- 添加 Undertow依赖 -->
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency> 
```

```yml

# 设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接, 默认设置每个CPU核心一个线程
# 不要设置过大，如果过大，启动项目会报错：打开文件数过多
server.undertow.io-threads=16

# 阻塞任务线程池, 当执行类似servlet请求阻塞IO操作, undertow会从这个线程池中取得线程
# 它的值设置取决于系统线程执行任务的阻塞系数，默认值是IO线程数*8
server.undertow.worker-threads=256

# 以下的配置会影响buffer,这些buffer会用于服务器连接的IO操作,有点类似netty的池化内存管理
# 每块buffer的空间大小,越小的空间被利用越充分，不要设置太大，以免影响其他应用，合适即可
server.undertow.buffer-size=1024

# 每个区分配的buffer数量 , 所以pool的大小是buffer-size * buffers-per-region
server.undertow.buffers-per-region=1024

# 是否分配的直接内存(NIO直接分配的堆外内存)
server.undertow.direct-buffers=true

```