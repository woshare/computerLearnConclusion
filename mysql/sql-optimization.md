# SQL优化

## 优化基本原则

>原则一：尽量避免全表扫描。
>原则二：通过索引优化

>1，**最左前缀匹配原则**：mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配。所以要尽量把“=”条件放在前面，把这些条件放在最后。复合索引也是同理。
>2，尽量选择 **区分度高且是业务常用的列作为索引**。在构建复合索引时，也应注意顺序。
>3，当取出的数据超过 **全表数据的20%** 时，不会使用索引。【待求证】
>4，**避免在 like 查询中将 %放在开头**：1）不使用索引：like ‘%L%’；2）使用索引：like ‘L%
>5，尽量将or 转换为 union all：这是何原理？可能和mysql实现有关，union all会并发？可能不是。1）不使用索引：select * from user where name=’a’ or age=’20’；2）使用索引：
select * from user where name=’a’ union all select * from user where age=’20’。or可能导致不会使用索引，导致全表扫描。对于or+没有索引的age这种情况，假设它走了userId的索引，但是走到age查询条件时，它还得全表扫描，也就是需要三步过程：全表扫描+索引扫描+合并 如果它一开始就走全表扫描，直接一遍扫描就完事。mysql是有优化器的，处于效率与成本考虑，遇到or条件，索引可能失效，看起来也合情合理。
>6，**字段加函数不会使用索引，字段加运算符不会使用索引，索引列不能参与计算**
>7，使用组合索引时，必须要包括第一个列,**索引(A, B)相当于创建了索引(A)和索引(A, B)**
>8，尽量避免使用is null或is not null
>9，不等于（!=，<>）不会使用索引
>10，尽量使用表连接（join）代替子查询select * from t1 where a in (select b from t2)
>11，性能方面，表连接 > (not) exists > (not) in
>12，避免使用HAVING子句, **HAVING 只会在检索出所有记录之后才对结果集进行过滤**. 这个处理需要排序,总计等操作. 如果能通过WHERE子句限制记录的数目,那就能减少这方面的开销
>13，**类型要一致**
>14，尽量减少select *，避免全表扫描
>15，在适当的时候，使用覆盖索引：通常在使用索引检索数据之后，需要访问磁盘上数据表文件读取所需要的列，这种操作成为 **回表**。若索引中包含查询的所有列，则不需要回表操作，直接从索引文件中读取数据即可，这种索引成为 **覆盖索引**
>16，=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式
>17，尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可
>18，明知只有一条查询结果，那请使用 “LIMIT 1”，避免全表扫描
>19，索引数量，一般不要超过5个
>20，尽可能使用varchar/nvarchar 代替 char/nchar。首先变长字段存储空间小，可以节省存储空间
>21，除非确实需要服务器去重，否则就一定要使用UNION ALL，如果没有ALL关键字，MySQL会给临时表加上DISTINCT选项，这会导致整个临时表的数据做唯一性检查，这样做的代价非常高   

## 优化工具与方法
>1，explain
>2，开启慢查询日志
>3，show  processlist；
>4，日志分析工具 MySQLdumpslow
>5，第三方工具：美团技术团队的 SQLAdvisor，给出索引优化建议的工具

## 优化案例

### 避免多个范围条件

 

实际开发中，我们会经常使用多个范围条件，比如想查询某个时间段内登录过的用户：

 
select user.* from user where login_time > '2017-04-01' and age between 18 and 30;
 

这个查询有一个问题：它有两个范围条件，login_time列和age列，MySQL可以使用login_time列的索引或者age列的索引，但无法同时使用它们。


## 索引失效原理

* [索引失效原理-good](https://cloud.tencent.com/developer/article/1704743)


###  先举一个遵循最佳左前缀法则的例子
select * from testTable where a=1 and b=2
分析如下：

首先a字段在B+树上是有序的，所以我们可以通过二分查找法来定位到a=1的位置。

其次在a确定的情况下，b是相对有序的，因为有序，所以同样可以通过二分查找法找到b=2的位置。

再来看看不遵循最佳左前缀的例子
select * from testTable where b=2
分析如下：

我们来回想一下b有顺序的前提：在a确定的情况下。

现在你的a都飞了，那b肯定是不能确定顺序的，在一个无序的B+树上是无法用二分查找来定位到b字段的。

所以这个时候，是用不上索引的。大家懂了吗？

### 范围查询右边失效原理
举例
select * from testTable where a>1 and b=2
分析如下：

首先a字段在B+树上是有序的，所以可以用二分查找法定位到1，然后将所有大于1的数据取出来，a可以用到索引。

b有序的前提是a是确定的值，那么现在a的值是取大于1的，可能有10个大于1的a，也可能有一百个a。

大于1的a那部分的B+树里，b字段是无序的（开局一张图），所以b不能在无序的B+树里用二分查找来查询，b用不到索引。

### like索引失效原理


一、%号放右边（前缀）:where name like "a%"

由于B+树的索引顺序，是按照首字母的大小进行排序，前缀匹配又是匹配首字母。所以可以在B+树上进行有序的查找，查找首字母符合要求的数据。所以有些时候可以用到索引。

二、%号放左边：where name like "%a"

是匹配字符串尾部的数据，我们上面说了排序规则，尾部的字母是没有顺序的，所以不能按照索引顺序查询，就用不到索引。

三、两个%%号：where name like "%a%"

这个是查询任意位置的字母满足条件即可，只有首字母是进行索引排序的，其他位置的字母都是相对无序的，所以查找任意位置的字母是用不上索引的

## 联合索引

```
SELECT * FROM table WHERE a > 1 and b = 2; 
```
如何建立索引?
如果此题回答为对(a,b)建立索引，那都可以回去等通知了。
此题正确答法是，对(b,a)建立索引。如果你建立的是(a,b)索引，那么只有a字段能用得上索引，毕竟 **最左匹配原则遇到范围查询就停止匹配**。
如果对(b,a)建立索引那么两个字段都能用上，优化器会帮我们调整where后a,b的顺序，让我们用上索引

>如果我们创建了(area, age,salary)的复合索引，那么其实相当于创建了(area,age,salary)、(area,age)、(area)三个索引，这被称为最佳左前缀特性

* [联合索引](https://www.cnblogs.com/rjzheng/p/12557314.html)

* [美团mysql-索引-不错](https://tech.meituan.com/2014/06/30/mysql-index.html)
* [mysql-基本查询原理-不错](https://dbaplus.cn/news-155-1531-1.html)


## 锁
>1，加锁的方式：自动加锁。对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁；对于普通SELECT语句，InnoDB不会加任何锁；
>2，显示加锁：
>1），共享锁：select * from tableName where ... + lock in share more
>2），排他锁：select * from tableName where ... + for update


### 行锁分析
>1，如何分析行锁定：show status like 'innodb_row_lock%';
>2，比较重要的是：innodb_row_lock_time_avg 平均等待时，innodb_row_lock_waits等待总次数，innodb_row_lock_time等待总时长


## 5步优化

SQL优化一般步骤
>1、通过慢查日志等定位那些执行效率较低的SQL语句
>2、explain 分析SQL的执行计划
需要重点关注type、rows、filtered、extra。

type由上至下，效率越来越高

ALL 全表扫描
index 索引全扫描
range 索引范围扫描，常用语<,<=,>=,between,in等操作
ref 使用非唯一索引扫描或唯一索引前缀扫描，返回单条记录，常出现在关联查询中
eq_ref 类似ref，区别在于使用的是唯一索引，使用主键的关联查询
const/system 单条记录，系统会把匹配行中的其他列作为常数处理，如主键或唯一索引查询
null MySQL不访问任何表或索引，直接返回结果 虽然上至下，效率越来越高，但是根据cost模型，假设有两个索引idx1(a, b, c),idx2(a, c)，SQL为"select * from t where a = 1 and b in (1, 2) order by c";如果走idx1，那么是type为range，如果走idx2，那么type是ref；当需要扫描的行数，使用idx2大约是idx1的5倍以上时，不会用idx1，否则会用idx2
Extra

Using filesort：MySQL需要额外的一次传递，以找出如何按排序顺序检索行。通过根据联接类型浏览所有行并为所有匹配WHERE子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行。
Using temporary：使用了临时表保存中间结果，性能特别差，需要重点优化
Using index：表示相应的 select 操作中使用了覆盖索引（Coveing Index）,避免访问了表的数据行，效率不错！如果同时出现 using where，意味着无法直接通过索引查找来查询到符合条件的数据。
Using index condition：MySQL5.6之后新增的ICP，using index condtion就是使用了ICP（索引下推），在存储引擎层进行数据过滤，而不是在服务层过滤，利用索引现有的数据减少回表的数据。
>3、show profile 分析
了解SQL执行的线程的状态及消耗的时间。默认是关闭的
```
set profiling = 1;
SHOW PROFILES ;
SHOW PROFILE FOR QUERY  #{id};
```
>4、trace
trace分析优化器如何选择执行计划，通过trace文件能够进一步了解为什么优惠券选择A执行计划而不选择B执行计划。
```
set optimizer_trace="enabled=on";
set optimizer_trace_max_mem_size=1000000;
select * from information_schema.optimizer_trace;


```
>5、确定问题并采用相应的措施
>优化索引
>优化SQL语句：修改SQL、IN 查询分段、时间查询分段、基于上一次数据过滤
>改用其他实现方式：ES、数仓等
>数据碎片处理

* [如上优化方法和场景](https://www.cnblogs.com/powercto/p/14410128.html)

### 大分页优化
>1，一种是把上一次的最后一条数据，也即上面的c传过来，然后做“c < xxx”处理
>2，**采用延迟关联的方式进行处理，减少SQL回表，但是要记得索引需要完全覆盖才有效果**
```
select t1.* from _t t1, (select id from _t where a = 1 and b = 2 order by c desc limit 10000, 10) t2 where t1.id = t2.id;
```