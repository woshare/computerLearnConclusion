


## 嵌套事务

>当sql解释器遇到 start transaction 时候会触发commit… !!!    

begin_1  sql_1  begin_2  sql_2  sql_3 commit_1  rollback_1  . 

begin_2 被执行的时候， sql_1 已经就被提交了， 当你再去执行commit_1的时候，那么sql_2 和 sql_3 就被提交了.    这时候你再去rollback，一定用都没有….    因为先前都提交完了，你能回滚啥

>因为START TRANSACTION会隐式的提交session中所有当前的更改，结束已有的事务，并打开一个新的事务

```
mysql> select * from ceshi;  
+------+  
|  n   |  
+------+  
|    1 |  
+------+  
1 row in set (0.00 sec)  
  
mysql> start transaction ;  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> insert into ceshi values(2);  
Query OK, 1 row affected (0.00 sec)  
  
mysql> start transaction ;  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> insert into ceshi values(3);  
Query OK, 1 row affected (0.00 sec)  
  
mysql> commit;  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> rollback;  
Query OK, 0 rows affected (0.00 sec)  
```