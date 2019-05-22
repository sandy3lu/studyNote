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


# 原子long
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

