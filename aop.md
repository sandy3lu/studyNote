# 简介
AOP全称Aspect Oriented Programming，面向切面

# 常用注解说明
@Aspect -- 把类标识为一个切面供容器读取
@Pointcut -- (切入点):就是带有通知的连接点，在程序中主要体现为书写切入点表达式

5种通知：
- @Before -- 标识一个前置增强方法，相当于BeforeAdvice的功能
- @AfterReturning -- 后置增强，相当于AfterReturningAdvice，方法退出时执行
- @AfterThrowing -- 异常抛出增强，相当于ThrowsAdvice
- @After -- final增强，不管是抛出异常或者正常退出都会执行
- @Around -- 环绕增强，相当于MethodInterceptor ,这个方法要决定真实的方法是否执行，而且必须有返回值

# 引入包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```


# 例子1
```java
/**
 * 切面 打印请求、返回参数信息
 */
@Aspect // 定义一个切面
public class LogRecordAspect {
    private static final Logger logger = LoggerFactory.getLogger(LogRecordAspect.class);

    // 定义切点Pointcut
    @Pointcut("execution(* com.example.helloSpringBoot.controller.*Controller.*(..))")
    public void excudeService() {
    }

    @Around("excudeService()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        RequestAttributes ra = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) ra;
        HttpServletRequest request = sra.getRequest();

        String method = request.getMethod();
        String uri = request.getRequestURI();
        String paraString = JSON.toJSONString(request.getParameterMap());        
        logger.info("请求开始, URI: {}, method: {}, params: {}", uri, method, paraString);

        // result的值就是被拦截方法的返回值
        Object result = pjp.proceed();
        logger.info("请求结束，controller的返回值是 " + JSON.toJSONString(result));
        return result;
    }
}
```

# 例子2
```java
@Component  
@Aspect  
public class LogAspect {  
  
    /** 
     * 定义Pointcut，Pointcut的名称 就是simplePointcut，此方法不能有返回值，该方法只是一个标示 
     */  
    @Pointcut("execution(public * com.service.impl..*.*(..))")  
    public void recordLog() {  
    }  
  
    @AfterReturning(pointcut = "recordLog()")  
    public void simpleAdvice() {  
        LogUtil.info("AOP后处理成功 ");  
    }  
  
    @Around("recordLog()")  
    public Object aroundLogCalls(ProceedingJoinPoint jp) throws Throwable {  
        LogUtil.info("正常运行");  
        return jp.proceed();  
    }  
  
    @Before("recordLog()")  
    public void before(JoinPoint jp) {  
        String className = jp.getThis().toString();  
        String methodName = jp.getSignature().getName(); // 获得方法名  
        LogUtil.info("位于：" + className + "调用" + methodName + "()方法-开始！");  
        Object[] args = jp.getArgs(); // 获得参数列表  
        if (args.length <= 0) {  
            LogUtil.info("====" + methodName + "方法没有参数");  
        } else {  
            for (int i = 0; i < args.length; i++) {  
                LogUtil.info("====参数  " + (i + 1) + "：" + args[i]);  
            }  
        } 
        
    }  
  
    @AfterThrowing("recordLog()")  
    public void catchInfo() {  
        LogUtil.info("异常信息");  
    }  
  
    @After("recordLog()")  
    public void after(JoinPoint jp) {  
        LogUtil.info("" + jp.getSignature().getName() + "()方法-结束！");         
    }  
}  
```

