
# 函数式接口与lambda表达式的关系

在java中，lambda表达式与函数式接口是不可分割的，都是结合起来使用的。


对于函数式接口，我们可以理解为只有一个抽象方法的接口，除此之外它和别的接口相比并没有什么特殊的地方。为了确保函数式接口的正确性，我们可以给这个接口添加@FunctionalInterface注解（当然，也可以不加此注解），这样当其中有超过一个抽象方法时就会报错。

对于其他类型的接口，我们想要使用就需要定义一个类（或者使用匿名类）来实现那个接口和其中的方法，而函数式接口除了使用普通的方法来实现之外，还有一种更加简单的方法---就是使用lambda表达式。lambda表达式我们可以理解对于函数式接口和其中的抽象方法的具体实现，这样当有一个需要函数式接口参数的方法时，我们就可以给其传递一个对应的lambda表达式作为参数。执行的时候就会自动执行函数式接口中的唯一方法，也就是传递过去的lambda表达式了。

下面我们来举一个例子具体说明一下：

有如下代码

``````List<String> list = Arrays.asList("d", "h", "a", "z", "b");``````


我们要对其进行排序，有一个对应的list.sort(Comparator<? super E> c)方法，需要我们传递一个Comparator接口的实例，而Comparator之中唯一的抽象方法为int compare(T o1, T o2)，完全符合我们之前的函数式接口的定义，并且它还使用了@FunctionalInterface注解，所以除了普通的实现方法之外我们可以使用lambda表达式来实现这个方法，具体代码如下:

<pre><code>List<String> list = Arrays.asList("d", "h", "a", "z", "b");
list.sort((String a, String b) -> {
    return a.compareTo(b);
});</code></pre>

其相当于：

<pre><code>List<String> list = Arrays.asList("d", "h", "a", "z", "b");
Comparator<String> comparator = (String a, String b) -> {
    return a.compareTo(b);
};  // 使用lambda表达式实现函数式接口，并赋值
list.sort(comparator);</code></pre>



当然，对于上面的lambda表达式有很多简略写法，这是主要说明它和函数式接口的关系，关于lambda表达式的其他很多的使用方法大家可以去具体查询使用。



