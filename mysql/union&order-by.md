# union 中使用order by失效


## 描述
>MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

## 语法
MySQL UNION 操作符语法格式：
```
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions]
UNION [ALL | DISTINCT] // 默认DISTINCT，表示去重了，All 表示运行重复
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions];
```

## union 和 order by 的通常使用方式

```
SELECT 列名称 FROM 表名称 UNION SELECT 列名称 FROM 表名称 ORDER BY 列名称；
SELECT 列名称 FROM 表名称 UNION ALL SELECT 列名称 FROM 表名称 ORDER BY 列名称；
```

## union链接的两个以上select 语句都需要 order by，会失效

### 语法不正确写法：报错。union在没有括号的情况下只能使用一个order by

```
SELECT * FROM t1 WHERE username LIKE 'l%' ORDER BY score ASC
UNION
SELECT * FROM t1 WHERE username LIKE '%m%' ORDER BY score ASC
```
#### 方案一：使用一个order by

#### 方案二：两个查询分别加括号，据说order by不能直接出现在union的子句中，但是可以出现在子句的子句中

```
(SELECT * FROM t1 WHERE username LIKE 'l%' ORDER BY sroce ASC)
UNION
(SELECT * FROM t1 WHERE username LIKE '%m%' ORDER BY score ASC)
```

#### 方案三：先各自排序，然后通过临时表嵌套再合并结果，注意排序后面必须加入 limit，否则order by不起作用

```
SELECT * FROM (SELECT * FROM t1 WHERE id IN (1,3,6) ORDER BY utime DESC limit 5) AS a 
UNION ALL
SELECT * FROM (SELECT * FROM t1 WHERE id IN (2,4,5) ORDER BY utime DESC limit 5) AS b
```

>注：或许对于这种简单的可能有效，但是一旦两个查询的排序复杂，可能就不能用了，例如我经历的例子，查询S1 排序=ORDER BY edit_update_time DESC,thumbs_up DESC，查询S2排序=ORDER BY level DESC,thumbs_up DESC ,edit_update_time DESC，并且最后的结果还要按S1的结果排在S2的结果之前，并且最后还要分页

>**在我的例子里，如上方案都失效**
>**最后的结果按照主键ID递增序**

### 我的成功的解决方案： 使用行号达到排序效果
```
 select * from (select @rownum := @rownum+1 as t_o,id,uid,show_status,thumbs_up,audit_status,edit_update_time,template_type from tb where uid='xx' and word='yy' and template_type in (1,2,3,4) ORDER BY edit_update_time DESC,thumbs_up DESC ) my_inspiration,(select @rownum:=0)t1 
UNION  
select * from (select @rownum := @rownum+1 as t_o,id,uid,show_status,thumbs_up,audit_status,edit_update_time,template_type from tb where uid!='xx' and word='yy' and show_status=2 and audit_status=1 and template_type in (1,2,3,4) ORDER BY level DESC,thumbs_up DESC ,edit_update_time DESC ) other_inspiration,(select @rownum:=0)t2 ORDER BY t_o asc;

```
