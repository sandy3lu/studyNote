


## feign的组件
Spring Cloud Netflix provides the following beans by default for feign (BeanType beanName: ClassName):
- Decoder feignDecoder: `ResponseEntityDecoder` (which wraps a SpringDecoder)
- Encoder feignEncoder: `SpringEncoder`
- Logger feignLogger: `Slf4jLogger`
- Contract feignContract: `SpringMvcContract`
- Feign.Builder feignBuilder: `HystrixFeign.Builder`
- Client feignClient: if Ribbon is enabled it is a `LoadBalancerFeignClient`, otherwise the default feign client is used.


The OkHttpClient and ApacheHttpClient feign clients can be used by setting `feign.okhttp.enabled` or `feign.httpclient.enabled` to true, respectively, and having them on the classpath. You can customize the HTTP client used by providing a bean of either `ClosableHttpClient` when using Apache or `OkHttpClient` when using OK HTTP.

Spring Cloud Netflix does not provide the following beans by default for feign, but still looks up beans of these types from the application context to create the feign client:
- Logger.Level
- Retryer
- ErrorDecoder
- Request.Options
- Collection<RequestInterceptor>
- SetterFactory

## 配置

### 自定义配置
Creating a bean of one of those type and placing it in a @FeignClient configuration allows you to override each one of the beans described

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}

// This replaces the SpringMvcContract with feign.Contract.Default and adds a RequestInterceptor to the collection of RequestInterceptor
@Configuration
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}

```
the client is composed from the components already in FeignClientsConfiguration together with any in FooConfiguration (where the latter will override the former).

If we create both `@Configuration` bean and configuration properties, configuration properties will win. It will override @Configuration values. But if you want to change the priority to @Configuration, you can change `feign.client.default-to-properties` to `false`.
配置的优先级顺序(org.springframework.cloud.openfeign.FeignClientFactoryBean的configureFeign方法):

```java
if (properties.isDefaultToProperties()) {
    configureUsingConfiguration(context, builder);
    configureUsingProperties(properties.getConfig().get(properties.getDefaultConfig()), builder);
    configureUsingProperties(properties.getConfig().get(this.contextId), builder);
} else {
    configureUsingProperties(properties.getConfig().get(properties.getDefaultConfig()), builder);
    configureUsingProperties(properties.getConfig().get(this.contextId), builder);
    configureUsingConfiguration(context, builder);
}
```

在使用FeignClient时，Spring会按name创建不同的ApplicationContext，通过不同的Context来隔离FeignClient的配置信息，在使用配置类时，不能把配置类放到Spring App Component scan的路径下，否则，配置类会对所有FeignClient生效

> FooConfiguration does not need to be annotated with `@Configuration`. However, if it is, then take care to exclude it from any `@ComponentScan `that would otherwise include this configuration as it will become the default source for feign.Decoder, feign.Encoder, feign.Contract, etc., when specified. This can be avoided by putting it in a separate, non-overlapping package from any `@ComponentScan` or `@SpringBootApplication`, or it can be explicitly excluded in `@ComponentScan. `

### 用yml配置

```yml
feign:
  client:
    config:
      feignName: #指定feign
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
        errorDecoder: com.example.SimpleErrorDecoder
        retryer: com.example.SimpleRetryer
        requestInterceptors:
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
        decode404: false # 当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException
        encoder: com.example.SimpleEncoder
        decoder: com.example.SimpleDecoder
        contract: com.example.SimpleContract

feign:
  client:
    config:
      default: #对所有feign都生效
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```


## ThreadLocal


If you need to use ThreadLocal bound variables in your RequestInterceptor\`s you will need to either set the thread isolation strategy for Hystrix to `SEMAPHORE` or disable Hystrix in Feign. 
```yml
# To disable Hystrix in Feign
feign:
  hystrix:
    enabled: false

# To set thread isolation to SEMAPHORE
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
```

## 多个feign client 访问同一个server
If we want to create multiple feign clients with the same name or url so that they would point to the same server but each with a different custom configuration then we have to use `contextId` attribute of the @FeignClient in order to avoid name collision of these configuration beans.
默认的contextId=name

```java
@FeignClient(contextId = "fooClient", name = "stores", configuration = FooConfiguration.class)
public interface FooClient {
    //..
}

@FeignClient(contextId = "barClient", name = "stores", configuration = BarConfiguration.class)
public interface BarClient {
    //..
}
```

## 手动创建feign clients


```java
//FeignClientsConfiguration.class is the default configuration provided by Spring Cloud Netflix
@Import(FeignClientsConfiguration.class)
class FooController {

	private FooClient fooClient;

	private FooClient adminClient;

    	@Autowired
	public FooController(Decoder decoder, Encoder encoder, Client client, Contract contract) {
        //PROD-SVC is the name of the service the Clients will be making requests to. 
		this.fooClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.contract(contract)
				.requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
				.target(FooClient.class, "https://PROD-SVC");

		this.adminClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.contract(contract)
				.requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
				.target(FooClient.class, "https://PROD-SVC");
    }
}
```

## Feign Hystrix Support

If Hystrix is on the classpath and `feign.hystrix.enabled=true`, Feign will wrap all methods with a circuit breaker. Returning a `com.netflix.hystrix.HystrixCommand` is also available. This lets you use reactive patterns (with a call to `.toObservable()` or `.observe()` or asynchronous use (with a call to `.queue()`).

org.springframework.cloud.openfeign.FeignAutoConfiguration类
```java
@Configuration
@ConditionalOnClass(name = "feign.hystrix.HystrixFeign")
protected static class HystrixFeignTargeterConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public Targeter feignTargeter() {
        return new HystrixTargeter();
    }
}

@Configuration
@ConditionalOnMissingClass("feign.hystrix.HystrixFeign")
protected static class DefaultFeignTargeterConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public Targeter feignTargeter() {
        return new DefaultTargeter();
    }
}
```

### Feign Hystrix Fallbacks

```java
// fallback指定的类必须实现@FeignClient标记的接口
@FeignClient(name = "hello", fallback = HystrixClientFallback.class)
protected interface HystrixClient {
    @RequestMapping(method = RequestMethod.GET, value = "/hello")
    Hello iFailSometimes();
}

static class HystrixClientFallback implements HystrixClient {
    @Override
    public Hello iFailSometimes() {
        return new Hello("fallback");
    }
}
```
If one needs access to the cause that made the fallback trigger, one can use the `fallbackFactory` attribute inside @FeignClient.
```java
@FeignClient(name = "hello", fallbackFactory = HystrixClientFallbackFactory.class)
protected interface HystrixClient {
	@RequestMapping(method = RequestMethod.GET, value = "/hello")
	Hello iFailSometimes();
}

@Component
static class HystrixClientFallbackFactory implements FallbackFactory<HystrixClient> {
	@Override
	public HystrixClient create(Throwable cause) {
		return new HystrixClient() {
			@Override
			public Hello iFailSometimes() {
				return new Hello("fallback; reason was: " + cause.getMessage());
			}
		};
	}
}
```

### Feign and @Primary
When using Feign with Hystrix fallbacks, there are multiple beans in the ApplicationContext of the same type. This will cause @Autowired to not work because there isn’t exactly one bean, or one marked as primary. To work around this, Spring Cloud Netflix marks all Feign instances as `@Primary`, so Spring Framework will know which bean to inject
```java
// To turn off this behavior set the primary attribute of @FeignClient to false.
@FeignClient(name = "hello", primary = false)
public interface HelloClient {
	// methods here
}
```

## Feign request/response compression
```yml
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

## Feign logging
A logger is created for each Feign client created.
1. 先通过配置文件，设置feign的debug级别
```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        //set the Logger.Level to FULL
        return Logger.Level.FULL;
    }
}
```
The `Logger.Level` object that you may configure per client, tells Feign how much to log. Choices are:
- NONE, No logging (DEFAULT).
- BASIC, Log only the request method and URL and the response status code and execution time.
- HEADERS, Log the basic information along with request and response headers.
- FULL, Log the headers, body, and metadata for both requests and responses.

2. 再通过yml文件，指定哪一个feign client开启debug模式
```yml
logging：
  level： 
    project.user.UserClient: DEBUG # feign client的类名
```


## Feign @QueryMap support
The OpenFeign `@QueryMap` annotation provides support for POJOs to be used as GET parameter maps. Unfortunately, the default OpenFeign QueryMap annotation is incompatible with Spring because it lacks a value property.

Spring Cloud OpenFeign provides an equivalent `@SpringQueryMap` annotation, which is used to annotate a POJO or Map parameter as a query parameter map.

```java
// Params.java
public class Params {
    private String param1;
    private String param2;

    // [Getters and setters omitted for brevity]
}

@FeignClient("demo")
public class DemoTemplate {

    @GetMapping(path = "/demo")
    String demoEndpoint(@SpringQueryMap Params params);
}
```

## feign with ribbon
配置文件在org.springframework.cloud.openfeign.ribbon路径下
```xml
<!-- 如果只有这个配置，使用DefaultFeignLoadBalancedConfiguration.java -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!-- 如果有这个，使用HttpClientFeignLoadBalancedConfiguration.java -->
<dependency>
    <groupId>io.github.openfeign</groupId>-->
    <artifactId>feign-httpclient</artifactId>-->
</dependency>
<!-- 如果有这个，并且配置feign.okhttp.enable=true，使用OkHttpFeignLoadBalancedConfiguration.java -->
<!--        <dependency>
<!--            <groupId>io.github.openfeign</groupId>-->
<!--            <artifactId>feign-okhttp</artifactId>-->
<!--        </dependency>-->
```
