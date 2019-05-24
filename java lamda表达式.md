
# lambda表达式

最简单的Lambda表达式可由逗号分隔的参数列表、->符号和语句块组成
```java
Arrays.asList( "a", "b", "d" ).forEach( ( String e ) -> System.out.println( e ) );

//如果Lambda表达式需要更复杂的语句块，则可以使用花括号将该语句块括起来，类似于Java中的函数体：
Arrays.asList( "a", "b", "d" ).forEach( e -> {
    System.out.print( e );
    System.out.print( e );
} );


//Lambda表达式可以引用类成员和局部变量（会将这些变量隐式得转换成final的）：
String separator = ","; // 相当于 final String separator = ",";
Arrays.asList( "a", "b", "d" ).forEach( 
    ( String e ) -> System.out.print( e + separator ) );


//Lambda表达式有返回值，返回值的类型也由编译器推理得出。
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> {
    int result = e1.compareTo( e2 );
    return result;
} );

```


# 函数式接口与lambda表达式的关系

在java中，lambda表达式与函数式接口是不可分割的，都是结合起来使用的。`函数接口指的是只有一个函数的接口，这样的接口可以隐式转换为Lambda表达式`

为了确保函数式接口的正确性，我们可以给这个接口添加`@FunctionalInterface`注解，这样当其中有超过一个抽象方法时就会报错。默认方法和静态方法不会破坏函数式接口的定义.
```java
@FunctionalInterface
public interface FunctionalDefaultMethods {
    void method();

    default void defaultMethod() {            
    }        
}
```

对于其他类型的接口，我们想要使用就需要定义一个类（或者使用匿名类）来实现那个接口和其中的方法，而函数式接口除了使用普通的方法来实现之外，还有一种更加简单的方法---就是使用lambda表达式。

lambda表达式我们可以理解对于函数式接口的抽象方法的具体实现，这样当有一个需要函数式接口参数的方法时，我们就可以给其传递一个对应的lambda表达式作为参数。执行的时候就会自动执行函数式接口中的唯一方法，也就是传递过去的lambda表达式了。

## 例子
我们要对list其进行排序，有一个对应的list.sort(Comparator<? super E> c)方法，需要我们传递一个Comparator接口的实例，而Comparator之中唯一的抽象方法为int compare(T o1, T o2)，符合函数式接口的定义，并且它还使用了@FunctionalInterface注解，所以可以使用lambda表达式来实现这个方法:
```java
List<String> list = Arrays.asList("d", "h", "a", "z", "b");
list.sort((String a, String b) -> {
    return a.compareTo(b);
});

//其相当于：
Comparator<String> comparator = (String a, String b) -> {
    return a.compareTo(b);
};  // 使用lambda表达式实现函数式接口，并赋值
list.sort(comparator);
```





