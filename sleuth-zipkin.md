# sleuth
pom.xml依赖管理中增加spring-cloud-starter-sleuth依赖即可
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

在方法中记录日志我们会发现在日志的最前面对了一部分内容，这部分内容就是Sleuth提供的链路信息
INFO [trace-1,f410ab57afd5c145,a9f2118fa2019684,false] 25028 --- [nio-9101-exec-1] 

关键字：[appname,traceId,spanId,exportable]， 具体含义如下：
- appname：服务的名称，也就是spring.application.name的值
- traceId：整个请求的唯一ID，它标识整个整个请求的链路
- spanId：基本的工作单元，发起一起远程调用就是一个span
- exportable：是否导入数据到Zipkin中

分析：[trace-1,f410ab57afd5c145,a9f2118fa2019684,false]
第一个值：trace-1，它记录了应用的名称，也就是application.properties中spring.application.name参数配置的属性。
第二个值：f410ab57afd5c145，Spring Cloud Sleuth生成的一个ID，称为Trace ID，它用来标识一条请求链路。一条请求链路中包含一个Trace ID，多个Span ID。
第三个值：a9f2118fa2019684，Spring Cloud Sleuth生成的另外一个ID，称为Span ID，它表示一个基本的工作单元，比如：发送一个HTTP请求。
第四个值：false，表示是否要将该信息输出到Zipkin等服务中来收集和展示。

上面四个值中的Trace ID和Span ID是Spring Cloud Sleuth实现分布式服务跟踪的核心。在一次服务请求链路的调用过程中，会保持并传递同一个Trace ID，从而将整个分布于不同微服务进程中的请求跟踪信息串联起来，以上面输出内容为例，trace-1和trace-2同属于一个前端服务请求来源，所以他们的Trace ID是相同的，处于同一条请求链路中。


# zipkin & sleuth 配置
```yml
  zipkin:
    sender:
      type: web
    locator:
      discovery:
        enabled: true
    base-url: http://192.168.20.11:9411
  sleuth:
    sampler:
      probability: 1.0
```



