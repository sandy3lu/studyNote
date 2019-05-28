
# 简介
实体对象间属性映射

http://modelmapper.org/user-manual/configuration/#matching-strategies

# pom
```xml
<!-- https://mvnrepository.com/artifact/org.modelmapper/modelmapper -->
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.3.4</version>
</dependency>
```

# 例子
```java
public class AppleVo {
    private String name ;
    private String id ;
    
}
public class Apple {
    private String id ;
    private String name ;
    private String createAge ;
}
public class AppleDTO {
    private String name ;
    private String create_age ;
}


/**
     *
     *  属性名保持一致，采用默认规则
     */
public  void test1() {
    apple = new Apple();
    apple.setName("black");
    apple.setCreateAge("22");
    apple.setId("1");

    ModelMapper modelMapper = new ModelMapper();
    AppleDTO appleDto = modelMapper.map(apple, AppleDTO.class);
    System.out.println(appleDto.toString());
}

```


