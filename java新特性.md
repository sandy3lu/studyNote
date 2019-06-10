# 接口默认实现default
```java
/**
    定义接口
*/
interface DAO {
    default int getInt(){
        return 5;
    }
}

/**
 *  接口实现类
 * */
class BasicDAO implements DAO{

    public int getIntDefaultValue(){
        return getInt();
    }
}

/**
 * 测试类
 */
public class Test {

    public static void main(String[] args) {
        BasicDAO basicDAO = new BasicDAO();
        //结果为 5 
        System.out.println(basicDAO.getIntDefaultValue());
    }
}


/**
 *  接口实现类
 * */
class BasicDAO implements DAO{

    public int getIntDefaultValue(){
        return getInt();
    }

     @Override
    public int getInt(){
        return 88;
    }
}


/**
 * 测试类
 */
public class Test {

    public static void main(String[] args) {
        BasicDAO basicDAO = new BasicDAO();
        //结果为 88 
        System.out.println(basicDAO.getIntDefaultValue());
    }
}
```

# 接口中定义静态方法
```java
private interface Defaulable {
    // the implementer may or 
    // may not implement (override) them.
    default String notRequired() { 
        return "Default implementation"; 
    }        
}

private static class DefaultableImpl implements Defaulable {
}

private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}

private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create( Supplier< Defaulable > supplier ) {
        return supplier.get();
    }
}

//下面的代码片段整合了默认方法和静态方法的使用场景：
public static void main( String[] args ) {
    Defaulable defaulable = DefaulableFactory.create( DefaultableImpl::new );
    System.out.println( defaulable.notRequired() );// Default implementation

    defaulable = DefaulableFactory.create( OverridableImpl::new );
    System.out.println( defaulable.notRequired() ); // Overridden implementation
}
```


# 原子 longAdder
jdk8引入LongAdder，比AtomicLong更高效了。
- AtomicLong的实现方式是内部有个value 变量，当多线程并发自增，自减时，均通过cas 指令从机器指令级别操作保证并发的原子性
- LongAdder中包含了一个Cell 数组，Cell是Striped64的一个内部类，顾名思义，Cell 代表了一个最小单元。Cell内部有一个非常重要的value变量，并且提供了一个cas更新其值的方法。

# try-with-resource
JDK7中新增了try-with-resource语法，try-with-resource并不是JVM虚拟机的新增功能，只是JDK实现了一个语法糖

```java
FileInputStream inputStream = null;
try {
    inputStream = new FileInputStream(new File("test"));
    System.out.println(inputStream.read());
} catch (IOException e) {
    throw new RuntimeException(e.getMessage(), e);
} finally {
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            throw new RuntimeException(e.getMessage(), e);
        }
    }
}

//当一个外部资源的句柄对象（比如FileInputStream对象）实现了AutoCloseable接口,可以简化
 try (FileInputStream inputStream = new FileInputStream(new File("test"))) {
    System.out.println(inputStream.read());
} catch (IOException e) {
    throw new RuntimeException(e.getMessage(), e);
}
```
try-with-resource时，如果对外部资源的处理和对外部资源的关闭均遭遇了异常，“关闭异常”将被抑制，“处理异常”将被抛出，但“关闭异常”并没有丢失，而是存放在“处理异常”的被抑制的异常列表中。

# Optional
是Java8提供的为了解决null安全问题的一个API
## 构造
JDK 提供三个静态方法来构造一个 Optional：
1.Optional.of(T value)
该方法通过一个非 null 的 value 来构造一个 Optional，返回的 Optional 包含了 value 这个值。对于该方法，传入的参数一定不能为 null，否则便会抛出 NullPointerException。
2.Optional.ofNullable(T value)
该方法和 of 方法的区别在于，传入的参数可以为 null
3.Optional.empty()
该方法用来构造一个空的 Optional，即该 Optional 中不包含值

## 使用
Optional 的 isPresent() 方法用来判断是否包含值，get() 用来获取 Optional 包含的值 —— 值得注意的是，如果值不存在，即在一个Optional.empty 上调用 get() 方法的话，将会抛出 NoSuchElementException 异常。

```java
SysUser user = getUserById(id);
if (user != null) {
    String username = user.getUsername();
    System.out.println("Username is: " + username); // 使用 username
}


//1、如果 Optional 中有值则将其返回，否则返回 orElse 方法传入的参数。
User user = Optional
        .ofNullable(getUserById(id))
        .orElse(new User(0, "Unknown"));
        
System.out.println("Username is: " + user.getUsername());

//2、如果 Optional 中有值，则对该值调用 consumer.accept，否则什么也不做。 所以对于上面的例子，我们可以修改为：
Optional<User> user = Optional.ofNullable(getUserById(id));
user.ifPresent(u -> System.out.println("Username is: " + u.getUsername()));

//3、orElseGet 与 orElse 方法的区别在于，orElseGet 方法传入的参数为一个 Supplier 接口的实现 —— 当 Optional 中有值的时候，返回值；当 Optional 中没有值的时候，返回从该 Supplier 获得的值。
User user = Optional
        .ofNullable(getUserById(id))
        .orElseGet(() -> new User(0, "Unknown"));
        
System.out.println("Username is: " + user.getUsername());


//4、orElseThrow 与 orElse 方法的区别在于，orElseThrow 方法当 Optional 中有值的时候，返回值；没有值的时候会抛出异常，抛出的异常由传入的 exceptionSupplier 提供。
User user = Optional
        .ofNullable(getUserById(id))
        .orElseThrow(() -> new EntityNotFoundException("id 为 " + id + " 的用户没有找到"));

//举一个 orElseThrow 的用途：在 SpringMVC 的控制器中，我们可以配置统一处理各种异常。查询某个实体时，如果数据库中有对应的记录便返回该记录，否则就可以抛出 EntityNotFoundException ，处理 EntityNotFoundException 的方法中我们就给客户端返回Http 状态码 404 和异常对应的信息 —— orElseThrow 完美的适用于这种场景。
@ExceptionHandler(EntityNotFoundException.class)
public ResponseEntity<String> handleException(EntityNotFoundException ex) {
    return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
}


//5、如果当前 Optional 为 Optional.empty，则依旧返回 Optional.empty；否则返回一个新的 Optional，该 Optional 包含的是：函数 mapper 在以 value 作为输入时的输出值。
Optional<String> username = Optional
        .ofNullable(getUserById(id))
        .map(user -> user.getUsername());
        
System.out.println("Username is: " + username.orElse("Unknown"));


//6、flatMap 方法与 map 方法的区别在于，map 方法参数中的函数 mapper 输出的是值，然后 map 方法会使用 Optional.ofNullable 将其包装为 Optional；而 flatMap 要求参数中的函数 mapper 输出的就是 Optional。
Optional<String> username = Optional
        .ofNullable(getUserById(id))
        .flatMap(user -> Optional.of(user.getUsername()))
        .flatMap(name -> Optional.of(name.toLowerCase()));
        
System.out.println("Username is: " + username.orElse("Unknown"));


//7、filter 方法接受一个 Predicate 来对 Optional 中包含的值进行过滤，如果包含的值满足条件，那么还是返回这个 Optional；否则返回 Optional.empty。
Optional<String> username = Optional
        .ofNullable(getUserById(id))
        .filter(user -> user.getId() < 10)
        .map(user -> user.getUsername());
        
System.out.println("Username is: " + username.orElse("Unknown"));

```

# java8 Supplier接口
在开发中，我们经常会遇到一些需要延迟计算的情形，比如某些运算非常消耗资源，如果提前算出来却没有用到，会得不偿失。在计算机科学中，有个专门的术语形容它：惰性求值。惰性求值是一种求值策略，也就是把求值延迟到真正需要的时候。在Java里，我们有一个专门的设计模式几乎就是为了处理这种情形而生的：Proxy。不过，现在我们有了新的选择：Supplier

## 定义
只有一个get的抽象类，没有默认的方法以及静态的方法，传入一个泛型T的，get方法，返回一个泛型T
```java
@FunctionalInterface
public interface Supplier<T> {
    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

这个接口，只是为我们提供了一个创建好的对象，这也符号接口的语义的定义，提供者，提供一个对象，
直接理解成一个创建对象的工厂，就可以了

## 例子

```java
Supplier<String> supplier = String::new;        
System.out.println(supplier.get());//""        
Supplier<Emp> supplierEmp = Emp::new;        
Emp emp = supplierEmp.get();        
emp.setName("dd");        
System.out.println(emp.getName());//dd

// supplier的实现demo
Supplier ultimateAnswerSupplier = new Supplier(){
    @Override
    public Integer get(){
        //Long time computation
        return 42;
    }
};

```

# 方法引用
方法引用使得开发者可以直接引用现存的方法、Java类的构造方法或者实例对象。方法引用和Lambda表达式配合使用，使得java类的构造方法看起来紧凑而简洁。

## 例子
```java
public static class Car {
    public static Car create( final Supplier< Car > supplier ) {
        return supplier.get();
    }              
    //注意：这个方法接受一个Car类型的参数。
    public static void collide( final Car car ) {
        System.out.println( "Collided " + car.toString() );
    }
    //注意：这个方法接受一个Car类型的参数：
    public void follow( final Car another ) {
        System.out.println( "Following the " + another.toString() );
    }
    //注意，这个方法没有定义入参
    public void repair() {   
        System.out.println( "Repaired " + this.toString() );
    }
}

//第一种方法引用的类型是构造器引用，语法是Class::new，或者更一般的形式：Class<T>::new。这个构造器没有参数。
final Car car = Car.create( Car::new );
final List< Car > cars = Arrays.asList( car );

//第二种方法引用的类型是静态方法引用，语法是Class::static_method。注意：这个方法接受一个Car类型的参数。
cars.forEach( Car::collide );

//第三种方法引用的类型是某个类的成员方法的引用，语法是Class::method，注意，这个方法没有定义入参：
cars.forEach( Car::repair );

//第四种方法引用的类型是某个实例对象的成员方法的引用，语法是instance::method。注意：这个方法接受一个Car类型的参数：
final Car police = Car.create( Car::new );
cars.forEach( police::follow );
```

# 注解
## 重复注解
自从Java 5中引入注解以来，这个特性开始变得非常流行，注解有一个很大的限制是：在同一个地方不能多次使用同一个注解。Java 8打破了这个限制，引入了重复注解的概念，允许在同一个地方多次使用同一个注解。
在Java 8中使用`@Repeatable`注解定义重复注解，实际上，这并不是语言层面的改进，而是编译器做的一个trick，底层的技术仍然相同。

## 拓宽注解的应用场景
Java 8拓宽了注解的应用场景。现在，注解几乎可以使用在任何元素上：局部变量、接口类型、超类和接口实现类，甚至可以用在函数的异常定义上

ElementType.TYPE_USER和ElementType.TYPE_PARAMETER是Java 8新增的两个注解，用于描述注解的使用场景

# Stream

Stream API是把真正的函数式编程风格引入到Java中。其实简单来说可以把Stream理解为MapReduce，当然Google的MapReduce的灵感也是来自函数式编程。她其实是一连串支持连续、并行聚集操作的元素。从语法上看，也很像linux的管道、或者链式编程，代码写起来简洁明了


# Date/Time API (JSR 310)

Java 8新的Date-Time API (JSR 310)受Joda-Time的影响，提供了新的java.time包，可以用来替代 java.util.Date和java.util.Calendar。一般会用到Clock、LocaleDate、LocalTime、LocaleDateTime、ZonedDateTime、Duration这些类

## JavaScript引擎Nashorn

Nashorn允许在JVM上开发运行JavaScript应用，允许Java与JavaScript相互调用



