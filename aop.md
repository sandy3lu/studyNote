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
# Spring Pointcut 切面表达式
## Wildcard
- *： 匹配任意数量的字符
- +：匹配制定数量的类及其子类
- ..：一般用于匹配任意数量的子包或参数
## Operators
- &&：与操作符
- ||：或操作符
- !：非操作符

## Designators 指示符
### within
```java
//匹配productService类中的所有方法
@pointcut("within(com.sample.service.productService)")
public void matchType()

//匹配sample包及其子包下所有类的方法
@pointcut("within(com.sample..*)")
public void matchPackage()
```
### 匹配对象（this, target, bean）
- this(Atype)意思是连接所有this.instanceof(AType) == true的点，所以这意味着，this.instanceof(Service) 为真的Service实例中的，所有方法被调用时。
- target(AType)意思是连接所有anObject.instanceof(AType)。如果你调用一个Object中的一个方法，且这个object是Service的实例，则这是一个合法的切点。
>总结：this(AType)接受者方面描述，target(AType)则从调用者方面描述。

```java
@pointcut("this(com.sample.demoDao)")
public void thisDemo()

@pointcut("target(com.sample.demoDao)")
public void targetDemo()


//匹配所有以Service结尾的bean里面的方法
@pointcut("bean(*Service)")
public void beanDemo
```
### 参数匹配
```java
//匹配所有以find开头，且只有一个Long类型参数的方法
@pointcut("execution(* *..find*(Long))")
public void argDemo1()

//匹配所有以find开头，且第一个参数类型为Long的方法
@pointcut("execution(* *..find*(Long, ..))")
public void argDemo2()

@pointcut("arg(Long)")
public void argDemo3()

@pointcut("arg(Long, ..)")
public void argDemo4()

```
### 匹配注解
```java
//匹配注解有AdminOnly注解的方法
@pointcut("@annotation(com.sample.security.AdminOnly)")
public void demo1()

//匹配标注有admin的类中的方法，要求RetentionPolicy级别为CLASS
@pointcut("@within(com.sample.annotation.admin)")
public void demo2()

//注解标注有Repository的类中的方法，要求RetentionPolicy级别为RUNTIME
@pointcut("target(org.springframework.stereotype.Repository)")
public void demo3()

//匹配传入参数的类标注有Repository注解的方法
@pointcut("args(org.springframework.stereotype.Repository)")
public void demo3()
```

### execution()
execution(<修饰符模式>? <返回类型模式> <方法名模式>(<参数模式>) <异常模式>?),
标注?的项目表示着可以省略

```java
execution(
    modifier-pattern?  //修饰符
    ret-type-pattern  //返回类型
    declaring-type-pattern?  //方法模式
    name-pattern(param-pattern)  //参数模式
    throws-pattern?  //异常模式
)

/*
整个表达式可以分为五个部分：
 1、execution(): 表达式主体。
 2、第一个*号：表示返回类型，*号表示所有的类型。
 3、包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，com.sample.service.impl包、子孙包下所有类的方法。
 4、第二个*号：表示类名，*号表示所有的类。
 5、*(..):最后这个星号表示方法名，*号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数。
*/
@pointcut("execution(* com.sample.service.impl..*.*(..))")
```


# 例子1
```java

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
     * 定义Pointcut，此方法不能有返回值，该方法只是一个标示 
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

