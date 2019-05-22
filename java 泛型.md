# 泛型中? super T和? extends T
## extends
```java
//表示包括T在内的任何T的子类
List<? extends Number> foo3 = new ArrayList<? extends Number>();
// Integer extends Number
List<? extends Number> foo3 = new ArrayList<? extends Integer>();
// Double extends Number
List<? extends Number> foo3 = new ArrayList<? extends Double>();
```
1. 读取操作通过以上给定的赋值语句，你可以读取到Number，因为以上的列表要么包含Number元素，要么包含Number的类元素。 
你不能保证读取到Integer，因为foo3可能指向的是List<Double>。
你不能保证读取到Double，因为foo3可能指向的是List<Integer>。
2. 写入操作过以上给定的赋值语句，你能把一个什么类型的元素合法地插入到foo3中呢？ 
你不能插入一个Integer元素，因为foo3可能指向List<Double>。
你不能插入一个Double元素，因为foo3可能指向List<Integer>。
你不能插入一个Number元素，因为foo3可能指向List<Integer>。
>不能往List<? extends T>中插入任何类型的对象，因为你不能保证列表实际指向的类型是什么，你并不能保证列表中实际存储什么类型的对象。唯一可以保证的是，你可以从中读取到T或者T的子类。

## super
```java
//<? super T>表示包括T在内的任何T的父类
List<? super Integer> foo3 = new ArrayList<Integer>();
// Number is a superclass of Integer
List<? super Integer> foo3 = new ArrayList<Number>();
// Object is a superclass of Integer
List<? super Integer> foo3 = new ArrayList<Object>();
```
1. 读取操作通过以上给定的赋值语句，你一定能从foo3列表中读取到的元素的类型是什么呢？你不能保证读取到Integer，因为foo3可能指向List<Number>或者List<Object>。 
你不能保证读取到Number，因为foo3可能指向List<Object>。
唯一可以保证的是，你可以读取到Object或者Object子类的对象（你并不知道具体的子类是什么）。
2. 写入操作通过以上给定的赋值语句，你可以插入Integer对象，因为上述声明的列表都支持Integer。 
你可以插入Integer的子类的对象，因为Integer的子类同时也是Integer。
你不能插入Double对象，因为foo3可能指向ArrayList<Integer>。
你不能插入Number对象，因为foo3可能指向ArrayList<Integer>。
你不能插入Object对象，因为foo3可能指向ArrayList<Integer>。


## PECS
生产者（Producer）使用extends，消费者（Consumer）使用super。
- 如果你需要一个列表提供T类型的元素（即你想从列表中读取T类型的元素），你需要把这个列表声明成<? extends T>，比如List<? extends Integer>，因此你不能往该列表中添加任何元素。
- 如果一个列表使用T类型的元素（即你想把T类型的元素加入到列表中），你需要把这个列表声明成<? super T>，比如List<? super Integer>，因此你不能保证从中读取到的元素的类型。
- 如果一个列表即要生产，又要消费，你不能使用泛型通配符声明列表，比如List<Integer>。