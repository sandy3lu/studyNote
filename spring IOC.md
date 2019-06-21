

# Spring IOC （控制反转，依赖注入）
Spring支持三种依赖注入方式，分别是属性（Setter方法）注入，构造注入和接口注入。

在Spring中，那些组成应用的主体及由Spring IOC容器所管理的对象被称之为Bean。
Spring的IOC容器通过反射的机制实例化Bean并建立Bean之间的依赖关系。
简单地讲，Bean就是由Spring IOC容器初始化、装配及被管理的对象。

获取Bean对象的过程，首先通过Resource加载配置文件并启动IOC容器，然后通过getBean方法获取bean对象，就可以调用他的方法。

Spring Bean的作用域：
- Singleton：Spring IOC容器中只有一个共享的Bean实例，一般都是Singleton作用域。
- Prototype：每一个请求，会产生一个新的Bean实例。
- Request：每一次http请求会产生一个新的Bean实例