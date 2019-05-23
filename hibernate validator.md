
# 简介
在开发中经常需要写一些字段校验的代码，比如字段非空，字段长度限制，邮箱格式验证等等，写这些与业务逻辑关系不大的代码有两个麻烦：
- 验证代码繁琐，重复劳动
- 方法内代码显得冗长
- 每次要看哪些参数验证是否完整，需要去翻阅验证逻辑代码
hibernate validator提供了一套比较完善、便捷的验证实现方式。
spring-boot-starter-web包里面有hibernate-validator包，不需要引用hibernate validator依赖

# 引入包
```xml
<dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>6.0.9.Final</version>
</dependency>
```

# 例子:post
```java
@Getter
@Setter
@NoArgsConstructor
public class DemoModel {
    @NotBlank(message="用户名不能为空")
    private String userName;

    @NotBlank(message="年龄不能为空")
    @Pattern(regexp="^[0-9]{1,2}$",message="年龄不正确")
    private String age;

    @AssertFalse(message = "必须为false")
    private Boolean isFalse;
    /**
     * 如果是空，则不校验，如果不为空，则校验
     */
    @Pattern(regexp="^[0-9]{4}-[0-9]{2}-[0-9]{2}$",message="出生日期格式不正确")
    private String birthday;
}

//POST接口验证，参数加valid注解， BindingResult是验证不通过的结果集合
@PostMapping("/demo2")
public void demo2(@RequestBody @Valid DemoModel demo, BindingResult result){
    if(result.hasErrors()){
        for (ObjectError error : result.getAllErrors()) {
            System.out.println(error.getDefaultMessage());
        }
    }
}
```


# 例子:Get
使用校验bean的方式，没有办法校验RequestParam的内容,需要使用@Validated注解

```java
@Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        MethodValidationPostProcessor postProcessor = new MethodValidationPostProcessor();
　　　　　/**设置validator模式为快速失败返回*/
        postProcessor.setValidator(validator());
        return postProcessor;
    }


@RequestMapping("/validation")
@RestController
@Validated
public class ValidationController {
    /**如果只有少数对象，直接把参数写到Controller层，然后在Controller层进行验证就可以了。*/
    @RequestMapping(value = "/demo3", method = RequestMethod.GET)
    public void demo3(@Range(min = 1, max = 9, message = "年级只能从1-9")
                      @RequestParam(name = "grade", required = true)
                      int grade,
                      @Min(value = 1, message = "班级最小只能1")
                      @Max(value = 99, message = "班级最大只能99")
                      @RequestParam(name = "classroom", required = true)
                      int classroom) {
        System.out.println(grade + "," + classroom);
    }
}

```



# hibernate的校验模式
1. 普通模式（默认是这个模式）
　　普通模式(会校验完所有的属性，然后返回所有的验证失败信息)
2. 快速失败返回模式
　　快速失败返回模式(只要有一个验证失败，则返回)

```java
//配置hibernate Validator为快速失败返回模式
@Configuration
public class ValidatorConfiguration {
    @Bean
    public Validator validator(){
        ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
                .configure()
                .addProperty( "hibernate.validator.fail_fast", "true" )
                .buildValidatorFactory();
        Validator validator = validatorFactory.getValidator();

        return validator;
    }
}
```