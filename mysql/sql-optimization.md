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