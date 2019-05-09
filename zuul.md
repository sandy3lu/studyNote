
## Filter
1. pre:可以在请求被路由之前调用
2. routing:在请求时被调用
3. post:在routing和error过滤之后调用。
4. error：处理请求时发生错误是被调用。

`com.netflix.zuul.http.ZuulServlet`的`service`方法实现也可以看出，Zuul处理外部请求过程时，各个类型过滤器的执行逻辑：
```java
try {
    preRoute();
} catch (ZuulException e) {
    error(e);
    postRoute();
    return;
}
try {
    route();
} catch (ZuulException e) {
    error(e);
    postRoute();
    return;
}
try {
    postRoute();
} catch (ZuulException e) {
    error(e);
    return;
}
```
从代码中可以看到三个try-catch块，它们依次分别代表了pre、route、post三个阶段的过滤器调用，在catch的异常处理中都会被error类型的过滤器进行处理。error类型的过滤器处理完毕之后，除了来自post阶段的异常之外，都会再被post过滤器进行处理。


### 自定义Zuul过滤器
定义一个简单的Zuul过滤器，它实现了在请求被路由之前检查HttpServletRequest中是否有accessToken参数，若有就进行路由，若没有就拒绝访问，返回401 Unauthorized错误。
```java
@Component
public class AccessFilter extends ZuulFilter {

    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);

    @Override
    public String filterType() {
        return "pre";
    }

    //过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行
    @Override
    public int filterOrder() {
        return 0;
    }

    //判断该过滤器是否需要被执行。这里直接返回了true，因此该过滤器对所有请求都会生效。实际运用中可以利用该函数来指定过滤器的有效范围
    @Override
    public boolean shouldFilter() {
        return true;
    }

    //过滤器的具体逻辑。这里通过ctx.setSendZuulResponse(false)令zuul过滤该请求，不对其进行路由
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        log.info("send {} request to {}", request.getMethod(), request.getRequestURL().toString());

        Object accessToken = request.getParameter("accessToken");
        if (accessToken == null) {
            log.warn("access token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.setResponseBody("miss token");
            return null;
        }
        log.info("access token ok");
        return null;
    }
}
```


## route filters

### 1. RibbonRoutingFilter
uses Ribbon, Hystrix and pluggable http clients to send requests.
```java
public boolean shouldFilter() {
		RequestContext ctx = RequestContext.getCurrentContext();
		return (ctx.getRouteHost() == null && ctx.get(SERVICE_ID_KEY) != null
				&& ctx.sendZuulResponse());
	}
```
![](pic/zuulcontext.JPG)


获取到地址
![](pic/zuulcontext-1.JPG)



### 2. SimpleHostRoutingFilter

sends requests to predetermined URLs

```java
public boolean shouldFilter() {
		return RequestContext.getCurrentContext().getRouteHost() != null
				&& RequestContext.getCurrentContext().sendZuulResponse();
	}
```


不满足条件，routeHost为空，'SKIPPED'


### 3. SendForwardFilter
forwards requests using the RequestDispatcher Forwarding location is located in the RequestContext attribute

```java
 public boolean shouldFilter() {
		RequestContext ctx = RequestContext.getCurrentContext();
		return ctx.containsKey(FORWARD_TO_KEY)
				&& !ctx.getBoolean(SEND_FORWARD_FILTER_RAN, false);
	}
```

不满足条件，routeHost为空，'SKIPPED'



## post route filters
### 1. TracePostZuulFilter

```java
public boolean shouldFilter() {
    //执行失败了才调用
		return !httpStatusSuccessful(RequestContext.getCurrentContext().getResponse());
	}
```






