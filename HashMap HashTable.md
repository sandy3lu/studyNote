
# HashMap与HashTable的区别
- HashMap是非线程安全的，HashTable是线程安全的（后者每个方法前都有synchronized关键字）。
- HashMap的键和值都允许有null值存在，而HashTable则不行。
- 因为线程安全的问题，HashMap效率比HashTable的要高。

# HashMap的实现机制

维护一个每个元素是一个链表的数组，而且链表中的每个节点是一个Entry[]键值对的数据结构。
实现了数组+链表的特性，查找快，插入删除也快。
对于每个key,他对应的数组索引下标是 int i = hash(key.hashcode)&(len-1);
每个新加入的节点放在链表首，然后该新加入的节点指向原链表首

使用HashMap要求添加的键类明确定义了hashCode()和equals()

## Hashcode的作用，与 equal 有什么区别？
Java对于eqauls方法和hashCode方法是这样规定的：
  (1) 如果两个对象相同（equals方法返回true），那么它们的hashCode值一定要相同；
  (2) 如果两个对象的hashCode相同，它们并不一定相同。



# HashMap，ConcurrentHashMap与LinkedHashMap的区别

ConcurrentHashMap是使用了锁分段技术技术来保证线程安全的，

锁分段技术：首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问


ConcurrentHashMap 是在每个段（segment）中线程安全的


LinkedHashMap维护一个双链表，可以将里面的数据按写入的顺序读出


## ConcurrentHashMap应用场景
ConcurrentHashMap把HashMap分成若干个Segmenet
1. get时，不加锁，先定位到segment然后在找到头结点进行读取操作。而value是volatile变量，所以可以保证在竞争条件时保证读取最新的值，如果读到的value是null，则可能正在修改，那么就调用ReadValueUnderLock函数，加锁保证读到的数据是正确的。
2. Put时会加锁，一律添加到hash链的头部。
3. Remove时也会加锁，由于next是final类型不可改变，所以必须把删除的节点之前的节点都复制一遍。
4. ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术。它使用了多个锁来控制对Hash表的不同Segment进行的修改。

ConcurrentHashMap的应用场景是高并发，但是并不能保证线程安全，而同步的HashMap和HashTable的是锁住整个容器，而加锁之后ConcurrentHashMap不需要锁住整个容器，只需要锁住对应的segment就好了，所以可以保证高并发同步访问，提升了效率。

ConcurrentHashMap能够保证每一次调用都是原子操作，但是并不保证多次调用之间也是原子操作

# HashMap 和 TreeMap
HashMap通过hashcode对其内容进行快速查找，而 TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。 
HashMap 非线程安全 TreeMap 非线程安全 

# List接口
一共有三个实现类，分别是ArrayList、Vector和LinkedList。List用于存放多个元素，能够维护元素的次序，并且允许元素的重复。

- ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。
- Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即`某一时刻只有一个线程能够写Vector`，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢。
- LinkedList使用双向链表实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。



