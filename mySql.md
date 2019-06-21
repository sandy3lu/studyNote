
# 索引
Mysql的索引是一个数据结构，旨在使数据库高效的查找数据。
常用的数据结构是B+Tree，每个叶子节点不但存放了索引键的相关信息还增加了指向相邻叶子节点的指针，这样就形成了带有顺序访问指针的B+Tree，做这个优化的目的是提高不同区间访问的性能。
什么时候使用索引：

经常出现在group by,order by和distinc关键字后面的字段
经常与其他表进行连接的表，在连接字段上应该建立索引
经常出现在Where子句中的字段
经常出现用作查询选择的字段

# 优化
一般来说，要保证数据库的效率，要做好以下四个方面的工作：数据库设计、sql语句优化、数据库参数配置、恰当的硬件资源和操作系统，这个顺序也表现了这四个工作对性能影响的大小

## 数据库设计
基于三范式建立的模型是最有效保存数 据的方式，三范式最大的问题在于查询时通常需要join很多表，导致查询效率很低。所以有时候基于性能考虑，我们需要有意的违反三范式，适度的做冗余，以达到提 高查询效率的目的。

索引的字段必须是经常作为查询条件的字段;如果索引多个字段，第一个字段要是经常作为查询条件的。如果只有第二个字段作为查询条件，这个索引不会起到作用;　索引的字段必须有足够的区分度;Mysql 对于长字段支持前缀索引;

对表进行水平划分：如果一个表的记录数太多了，比如上千万条，而且需要经常检索，那么我们就有必要化整为零了。一个好的划分依据，有利于程序的简单实现，也可以充分利用水平分表的优势。比如系统界面上只提供按月查询的功能，那么把表按月 拆分成12个，每个查询只查询一个表就够了

对表进行垂直划分：有些表记录数并不多，可能也就2、3万条，但是字段却很长，表占用空间很大，检索表时需要执行大量I/O，严重降低了性能。这个时候需要把大的字段拆分到另一个表，并且该表与原表是一对一的关系。 

文件、图片等大文件用文件系统存储，不用数据库

## Sql语句优化工具

1. 慢日志
　　如果发现系统慢了，又说不清楚是哪里慢，那么就该用这个工具了。只需要为mysql配置参数，mysql会自己记录下来慢的sql语句。配置很简单，参数文件里配置：

　　slow_query_log=d:/slow.txt

　　long_query_time = 2

　　就可以在d:/slow.txt里找到执行时间超过2秒的语句了，根据这个文件定位问题吧。

2. Explain
　　现在我们已经知道是哪个语句慢了，那么它为什么慢呢?看看mysql是怎么执行的吧，用explain可以看到mysql执行计划。 如果在SELECT语句前放上关键词EXPLAIN，MySQL将解释它如何处理SELECT，提供有关表如何联接和联接的次序。

## 数据库参数配置

最重要的参数就是内存，我们主要用的innodb引擎，所以下面两个参数调的很大

innodb_additional_mem_pool_size = 64M
innodb_buffer_pool_size = 5G

## 合理的硬件资源和操作系统
读写分离

　　如果数据库压力很大，一台机器支撑不了，那么可以用mysql复制实现多台机器同步，将数据库的压力分散。


# 数据库三范式
1. 第一范式（1NF）：强调的是列的原子性，即列不能够再分成其他几列。

考虑这样一个表：【联系人】（姓名，性别，电话）
如果在实际场景中，一个联系人有家庭电话和公司电话，那么这种表结构设计就没有达到 1NF。要符合 1NF 我们只需把列（电话）拆分，即：【联系人】（姓名，性别，家庭电话，公司电话）。

2. 第二范式（2NF）：首先是 1NF，另外包含两部分内容，一是表必须有一个主键；二是没有包含在主键中的列必须`完全依赖于主键，而不能只依赖于主键的一部分`。

考虑一个订单明细表：【OrderDetail】（OrderID，ProductID，UnitPrice，Discount，Quantity，ProductName）。
因为我们知道在一个订单中可以订购多种产品，所以单单一个 OrderID 是不足以成为主键的，主键应该是（OrderID，ProductID）。显而易见 Discount（折扣），Quantity（数量）完全依赖（取决）于主键（OderID，ProductID），而 UnitPrice，ProductName 只依赖于 ProductID。所以 OrderDetail 表不符合 2NF。不符合 2NF 的设计容易产生冗余数据。
可以把【OrderDetail】表拆分为【OrderDetail】（OrderID，ProductID，Discount，Quantity）和【Product】（ProductID，UnitPrice，ProductName）来消除原订单表中UnitPrice，ProductName多次重复的情况。


3. 第三范式（3NF）：首先是 2NF，另外非主键列必须`直接依赖于主键`，不能存在传递依赖。即不能存在：非主键列 A 依赖于非主键列 B，非主键列 B 依赖于主键的情况。

考虑一个订单表【Order】（OrderID，OrderDate，CustomerID，CustomerName，CustomerAddr，CustomerCity）主键是（OrderID）。
其中 OrderDate，CustomerID，CustomerName，CustomerAddr，CustomerCity 等非主键列都完全依赖于主键（OrderID），所以符合 2NF。不过问题是 CustomerName，CustomerAddr，CustomerCity 直接依赖的是 CustomerID（非主键列），而不是直接依赖于主键，它是通过传递才依赖于主键，所以不符合 3NF。
通过拆分【Order】为【Order】（OrderID，OrderDate，CustomerID）和【Customer】（CustomerID，CustomerName，CustomerAddr，CustomerCity）从而达到 3NF。
