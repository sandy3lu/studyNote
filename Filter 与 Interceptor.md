
# Servlet和Filter的区别
Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。

Filter有如下几个用处：
- Filter可以进行对特定的url请求和相应做预处理和后处理。
- 在HttpServletRequest到达Servlet之前，拦截客户的HttpServletRequest。
根据需要检查HttpServletRequest，也可以修改HttpServletRequest头和数据。
- 在HttpServletResponse到达客户端之前，拦截HttpServletResponse。
根据需要检查HttpServletResponse，也可以修改HttpServletResponse头和数据。

实际上Filter和Servlet极其相似，区别只是`Filter不能直接对用户生成响应`。实际上Filter里doFilter()方法里的代码就是从多个Servlet的service()方法里抽取的通用代码，通过使用Filter可以实现更好的复用。

# Filter和Servlet的生命周期
1. Filter在web服务器启动时初始化
2. 如果某个Servlet配置了 ，该Servlet也是在Tomcat（Servlet容器）启动时初始化。
3. 如果Servlet没有配置 ，该Servlet不会在Tomcat启动时初始化，而是在请求到来时初始化。
4. 每次请求， Request都会被初始化，响应请求后，请求被销毁。
5. Servlet初始化后，将不会随着请求的结束而注销。
6. 关闭Tomcat时，Servlet、Filter依次被注销。


# Filter

Servlet中的过滤器Filter是实现了javax.servlet.Filter接口的服务器端程序。

使用Filter完整的流程是：Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。

Filter有如下几个用处：
- 在HttpServletRequest到达Servlet之前，拦截客户的HttpServletRequest。
- 根据需要检查HttpServletRequest，也可以修改HttpServletRequest头和数据。
- 在HttpServletResponse到达客户端之前，拦截HttpServletResponse。
- 根据需要检查HttpServletResponse，也可以修改HttpServletResponse头和数据。
     
Filter有如下几个种类。
- 用户授权的Filter：Filter负责检查用户请求，根据请求过滤用户非法请求。
- 日志Filter：详细记录某些特殊的用户请求。
- 负责解码的Filter:包括对非标准编码的请求解码。
- 能改变XML内容的XSLT Filter等。
- Filter可以负责拦截多个请求或响应；一个请求或响应也可以被多个Filter拦截。




创建Filter必须实现`javax.servlet.Filter`接口，在该接口中定义了如下三个方法。
- void init(FilterConfig config):用于完成Filter的初始化。
- void destory():用于Filter销毁前，完成某些资源的回收。
- void doFilter(ServletRequest request,ServletResponse response,FilterChain chain):实现过滤功能，该方法就是对每个请求及响应增加的额外处理。该方法可以实现对用户请求进行预处理(ServletRequest request)，也可实现对服务器响应进行后处理(ServletResponse response) — 它们的分界线为是否调用了`chain.doFilter()`,执行该方法之前，即对用户请求进行预处理；执行该方法之后，即对服务器响应进行后处理


# 拦截器 Interceptor
拦截器是在面向切面编程中应用的，在AOP(Aspect-Oriented Programming)中用于在某个方法或字段被访问之前，进行拦截，然后在之前或之后加入某些操作。`拦截是AOP的一种实现策略`。

## spring Interceptor
在SpringMVC 中定义一个Interceptor 非常简单，主要有两种方式，第一种方式是要定义的Interceptor类要实现了Spring 的`HandlerInterceptor` 接口，或者是这个类继承实现了HandlerInterceptor 接口的类，比如Spring 已经提供的实现了HandlerInterceptor 接口的抽象类HandlerInterceptorAdapter ；第二种方式是实现Spring的`WebRequestInterceptor`接口，或者是继承实现了WebRequestInterceptor的类。

1. preHandle (HttpServletRequest request, HttpServletResponse response, Object handle) 方法，该方法将在请求处理之前进行调用。SpringMVC 中的Interceptor 是链式的调用的，在一个应用中或者说是在一个请求中可以同时存在多个Interceptor 。每个Interceptor 的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor 中的preHandle 方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值是布尔值Boolean类型的，`当它返回为false 时，表示请求结束，后续的Interceptor 和Controller 都不会再执行`；当返回值为true 时就会继续调用下一个Interceptor 的preHandle 方法，如果已经是最后一个Interceptor 的时候就会是调用当前请求的Controller 方法。

2. postHandle (HttpServletRequest request, HttpServletResponse response, Object handle, ModelAndView modelAndView) 方法，`这个方法包括后面要说到的afterCompletion 方法都只能是在当前所属的Interceptor 的preHandle 方法的返回值为true 时才能被调用`。postHandle 方法，是在当前请求进行处理之后，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。postHandle 方法被调用的方向跟preHandle 是相反的，也就是说先声明的Interceptor 的postHandle 方法反而会后执行。

3. afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex) 方法，该方法也是需要当前对应的Interceptor 的preHandle 方法的返回值为true 时才会执行。该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。


>在Spring构架的程序中，要优先使用拦截器。几乎所有 Filter 能够做的事情， interceptor 都能够轻松的实现。拦截器是一个Spring的组件，归Spring管理，配置在Spring文件中，因此能使用Spring里的任何资源、对象，例如 Service对象、数据源、事务管理等，通过IoC注入到拦截器即可

## 例子
```java
//统计请求耗时
public class ExecuteTimeInterceptor extends HandlerInterceptorAdapter{

    private static final Logger logger = Logger.getLogger(ExecuteTimeInterceptor.class);

    //before the actual handler will be executed
    public boolean preHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler)
        throws Exception {

        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        return true;
    }

    //after the handler is executed
    public void postHandle(
        HttpServletRequest request, HttpServletResponse response,
        Object handler, ModelAndView modelAndView)
        throws Exception {

        long startTime = (Long)request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        long executeTime = endTime - startTime;

        //modified the exisitng modelAndView
        modelAndView.addObject("executeTime",executeTime);

        //log it
        if(logger.isDebugEnabled()){
           logger.debug("[" + handler + "] executeTime : " + executeTime + "ms");
        }
    }
} 
```



# 拦截器（Interceptor）和过滤器（Filter）的执行顺序
过滤前-拦截前-Action处理-拦截后-过滤后


