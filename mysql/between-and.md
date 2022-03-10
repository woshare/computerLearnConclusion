# between-and 索引失效
>1，加上limit会使用索引

ALTER TABLE testread add INDEX tm(`create_time`,`update_time`)

-- EXPLAIN SELECT * from testread where create_time BETWEEN '2022-03-09 16:17:28' AND '2022-03-11 16:17:28' # on index

-- EXPLAIN SELECT * from testread where create_time BETWEEN '2022-03-09 16:17:28' AND '2022-03-11 16:17:28' limit 0,10 # using index

-- EXPLAIN SELECT * from testread where update_time BETWEEN '2022-03-09 16:17:28' AND '2022-03-11 16:17:28' limit 0,10；# no index

EXPLAIN SELECT * from testread where '2022-03-09 16:17:28' BETWEEN create_time AND update_time  # using index

-- EXPLAIN SELECT * from testread where '2022-03-09 16:17:28' >= create_time  # using index


EXPLAIN SELECT * from testread where '2022-03-09 16:17:28' <= update_time  # no index