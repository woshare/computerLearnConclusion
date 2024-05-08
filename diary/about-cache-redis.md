# 关于缓存

关于缓存，你会想到什么

一般就是分布式缓存，常用的有redis，memcache等
再就是本地缓存

如何去闭环的去思考总结一个领域

![java全栈知识体系](https://pdai.tech/md/db/nosql-redis/db-redis-x-cache.html)
## curd

### 数据创建
>1，一个key下数据量不要太多，string 类型控制在 10KB 以内，hash、list、set、zset 元素个数不要超过 5000
>2，一般还是需要设置超时淘汰，要注意，同时淘汰造成雪崩
>3，key的命名
