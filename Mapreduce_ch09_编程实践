# MapReduce编程实践
### Reduce Join
1. Reduce Join：在reduce端进行连接，是MapReduce框架实现join操作最常见的方式，其具体原理如下：
- Map端的主要工作：为来自不同表（文件）的key/value对打标签以区别不同来源的记录。然后用连接字段作为key,其余部分和新加的标志作为value,最后进行输出。
- reduce端的主要工作：在reduce端以连接字段作为key的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录（在map阶段已经打标志）分开，最后进行合并就ok了。
2. Reduce Join的实现一：
- 适用场景：两个表连接
- 实现方式：二次排序
3. Reduce Join的实现二：
- 适用场景：两个表连接
- 实现方式：笛卡尔积
4. Reduce Join的实现三：
- 适用场景：一个大表和一个小表连接
- 实现方式：分布式缓存
5. Reduce join存在一些不足，之所以会存在reduce join这种方式，是因为整体数据被分割了，每个map task只处理一部分数据而不能够获取到所有需要的join字段，因此我们可以充分利用mapreduce框架的特性，让他按照join key进行分区，将所有join key相同的记录集中起来进行处理，所以reduce join这种方式就出现了。
这种方式的缺点很明显就是会造成map和reduce端也就是shuffle阶段出现大量的数据传输，效率很低。

### Map Join
1. Map Join的实现一：
- 使用场景：一张表十分小，一张表很大。
- 用法：在提交作业的时候先将小表文件放到该作业的DistributedCache中，然后从DistributedCache中取出该小表进行join(比如放到HashMap等等容器中)，然后扫描大表，看大表中的每条记录的join key/value值是否能够在内存中找到相同join key的记录，如果有则直接输出结果。
DistributedCache是分布式缓存的一种实现，它在整个MapReduce框架中起着相当重要的作用，他可以支撑我们写一些相当复杂高效的分布式程序。说回到这里，JobTracker在作业启动之前会获取到DistributedCache的资源uri列表，并将对应的文件分发到各个涉及到该作业的任务的TaskTracker上，另外，关于DistributedCache和作业的关系，比如权限、存储路径区分、public和private等属性。
2. Map Join的实现二：
- 使用场景：一张表在数据库，一张表很大
- 另外还有一种比较变态的Mapjoin方式，就是结合Hbase来做Mapjoin操作。这种方式完全可以突破内存的控制，是你毫无忌惮的使用Mapjoin，而且效率也非常不错。


### Semi Join
1. 使用场景：一个大表（内存放不下），一个超大表
> semijoin就是所谓的半连接，它是reduce join的一个变种，就是在map端过滤掉一些数据，在网络中只传输参与连接的数据不参与连接的数据不必再网络中进行传输，从而减少了shuffle的网络传输量，使得整体的效率得到提高，其他思想和reduce join是一摸一样的。更通俗易懂即是，将小表中参与join的key单独抽出来通过DistributedCache分发到相关节点，然后将其取出放到内存中（可以放在HashSet中），在map阶段扫描连接表，将join key不在内存HashSet中的记录过滤掉，让那些参与join的记录通过shuffle传输到reduce端进行join操作，其他的和reduce join都是一样的。
2. Semi Join的实现一：
- 使用场景：一个大表（内存放不下），一个超大表
- 实现方式：分布式缓存

### Reduce join + Bloom Filter
1. 使用场景：一个大表（表中的key内存仍然放不下），一个超大表
> 在某些情况下，Semijoin抽取出来的小表的key集合在内存中仍然存放不下，这个时候可以使用BloomFilter以节省空间。Bloom Filter最常见的作用是；判断某个元素是否在一个集合里面。它最重要的两个方法是add()和membershipTest()。
因而可将小表中的key保存到Bloom Filter中，在map阶段过滤大表，可能有一些不在小表中的记录没有过滤掉（但是在小表中的记录一定不会过滤掉），这没有关系，只不过增加了少量的网络IO而已。
> Bloom Filter参数计算方式：
n:小表中的记录数。
m:位数组大小，一般m是n的倍数，倍数越大误判率就越小，但是也有内存限制，不能太大，这个值需要反复测试得出。
k:hash个数，最有hash个数值为：k=In2*(m/n) 


### 总结1
1. 三种join方式适用于不同的场景，其处理效率上相差很大，其主要导致的因素是网络传输，Map join效率最高，其次是semijoin,最低的是reduce join。另外，写分布式大数据处理程序时，最好要对整体要处理的数据分布情况有一个了解，这可以提高我们代码的效率，使数据的倾斜度降到最低，使我们的代码倾向性更好。
2. MR有三种不同的运行模式：
- local模式：需要处理的数据在local，程序运行也在local
- client模式：需要处理的数据在hdfs，程序运行在local
- yarn模式：需要处理的数据在hdfs，程序提交到hdfs
