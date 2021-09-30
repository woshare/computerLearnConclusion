# 数据库优化

* [MRR](https://zhuanlan.zhihu.com/p/110154066)
* [ICP-MRR-BKA](https://zhuanlan.zhihu.com/p/110154066)


## Index Condition Pushdown(ICP)
>1，Index Condition Pushdown (ICP)是mysql使用索引从表中检索行数据的一种优化方式，从mysql5.6开始支持，mysql5.6之前，存储引擎会通过遍历索引定位基表中的行，然后返回给Server层，再去为这些数据行进行WHERE后的条件的过滤。mysql 5.6之后支持ICP后，如果WHERE条件可以使用索引，MySQL 会把这部分过滤操作放到存储引擎层，存储引擎通过索引过滤，把满足的行从表中读取出。ICP能减少引擎层访问基表的次数和 Server层访问存储引擎的次数。

>1，ICP的目标是减少从基表中读取操作的数量，从而降低IO操作
>2，对于InnoDB表，ICP只适用于辅助索引
>3，当使用ICP优化时，执行计划的Extra列显示Using indexcondition提示
>4，数据库配置 optimizer_switch="index_condition_pushdown=on”;


## Multi-Range Read (MRR)
>1，MRR 的全称是 Multi-Range Read Optimization，是优化器将随机 IO 转化为顺序 IO 以降低查询过程中 IO 开销的一种手段，这对IO-bound类型的SQL语句性能带来极大的提升，适用于range ref eq_ref类型的查询


>1，使数据访问有随机变为顺序，查询辅助索引是，首先把查询结果按照主键进行排序，按照主键的顺序进行书签查找
>2，减少缓冲池中页被替换的次数
>3，批量处理对键值的操作

>4，相关参数
>当mrr=on,mrr_cost_based=on，则表示cost base的方式还选择启用MRR优化,当发现优化后的代价过高时就会不使用该项优化
>当mrr=on,mrr_cost_based=off，则表示总是开启MRR优化
```

SET  @@optimizer_switch='mrr=on,mrr_cost_based=on';

```
>参数read_rnd_buffer_size 用来控制键值缓冲区的大小。二级索引扫描到文件的末尾或者缓冲区已满，则使用快速排序对缓冲区中的内容按照主键进行排序


## JOIN算法

### Nested Loop Join算法
>将驱动表/外部表的结果集作为循环基础数据，然后循环该结果集，每次获取一条数据作为下一个表的过滤条件查询数据，然后合并结果，获取结果集返回给客户端。Nested-Loop一次只将一行传入内层循环, 所以外层循环(的结果集)有多少行, 内存循环便要执行多少次，效率非常差。


### Block Nested-Loop Join算法
>将外层循环的行/结果集存入join buffer, 内层循环的每一行与整个buffer中的记录做比较，从而减少内层循环的次数。主要用于当被join的表上无索引。


### Batched Key Access算法
>当被join的表能够使用索引时，就先好顺序，然后再去检索被join的表。对这些行按照索引字段进行排序，因此减少了随机IO。如果被Join的表上没有索引，则使用老版本的BNL策略(BLOCK Nested-loop)。