# redis

* [redis 阿里建议使用规范](https://www.infoq.cn/article/K7dB5AFKI9mr5Ugbs_px)

## Redis的内存淘汰策略：六种

>Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。

>1，noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。
>2，allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。
>3，allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
>4，volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
>5，volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。
>6，volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。



## Redis的过期策略：三种

>1，定时过期：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
>2，惰性过期：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
>3，定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。
(expires字典会保存所有设置了过期时间的key的过期时间数据，其中，key是指向键空间中的某个键的指针，value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。)

## 持久化
>持久化存储，指的是将内存的缓存永久存在磁盘中。也就是说我们的AOF和RDB持久化存储方式

### RDB持久化：Redis DataBase，快照方式
>1，持久化key之前，会检查是否过期，过期的key不进入RDB文件。
>2，数据载入数据库之前，会对key先进行过期检查，如果过期，不导入数据库
>3，将某一个时刻的内存快照（Snapshot），以二进制的方式写入磁盘的过程
>4，一种是手动触发，一种是自动触发

### AOF持久化： Append Only File，文件追加方式

>以独立日志的方式记录每次写命令， 重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用 是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式
>AOF的工作流程操作：命令写入 （append）、文件同步（sync）、文件重写（rewrite）、重启加载 （load）

>1，当key过期后，还没有被删除，此时进行执行持久化操作（该key是不会进入aof文件的，因为没有发生修改命令）。

>2，当key过期后，在发生删除操作时，程序会向aof文件追加一条del命令（在将来的以aof文件恢复数据的时候该过期的键就会被删掉）。

>3，因为AOF方式，向存储文件追加的是Redis的操作命令，而不是具体的数据，然而RDB确是存储的安全的二进制内容。
重写时，会先判断key是否过期，已过期的key不会重写到aof文件。

>4，即使在重写时，不验证是否过期，然而追加了del命令，测试无效的key同样会被删除。判断的情况是为了防止没有加入del命令的key

持久化配置 | 优缺点 
---------|----------
 always | 每条 Redis 操作命令都会写入磁盘，最多丢失一条数据，但会使得Redis的性能降低，但数据几乎是全的，基本不会存在丢失数据问题。 
 everysec | 每秒钟写入一次磁盘，最多丢失一秒的数据，对存取数据和性能折中，可以满足大部分使用场景。 
 no | 不设置写入磁盘的规则，根据当前操作系统来决定何时写入磁盘，一般不采用这种设置。 



>手动触发
>AOF 重写流程：AOF 是存放每条写命令的，所以会不断变大，达到一定的时候，AOF做rewrite操作，会重新生成一个新的AOF文件


### 混合持久化方式

>Redis 4.0 之后新增的方式，混合持久化是结合了 RDB 和 AOF 的优点，在写入的时候，先把当前的数据以 RDB 的形式写入文件的开头，再将后续的操作命令以 AOF 的格式存入文件，这样既能保证 Redis 重启时的速度，又能减低数据丢失的风险。
>AOF 重写时会把 Redis 的持久化数据，以 RDB 的格式写入到 AOF 文件的开头，之后的数据再以 AOF 的格式化追加的文件的末尾，如下图所示。

![](./res/redis-hunhe.jpg "")

持久化方式 | 优点 | 缺点
---------|----------|---------
 RDB | RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据 快照。非常适用于备份，全量复制等场景。与 AOF 格式的文件相比，RDB 文件可以更快的重启。RDB 对灾难恢复非常有用，它是一个紧凑的文件，可以更快的传输到远程服务器进行 Redis 服务恢复 |RDB方式数据没办法做到实时持久化/秒级持久化，RDB只能保存某个时间间隔的数据，如果在这个期间Redis故障了，就会丢失一段时间的数据。RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运 行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高。
 AOF | 意外情况下，丢失数据不多,具有一定实时性 | 对于相同的数据集来说，AOF 文件要大于 RDB 文件，从理论上说，RDB 比 AOF 更健壮
 混合 | 混合持久化结合了 RDB 和 AOF 持久化的优点，开头为 RDB 的格式，使得 Redis 可以更快的启动，同时结合 AOF 的优点，有减低了大量数据丢失的风险 | AOF 文件中添加了 RDB 格式的内容，会使得 AOF 文件的可读性会很差，不容易阅读； 如果开启混合持久化，就必须使用Redis 4.0 以及之后版本




## Redis为什么采用跳表而不是红黑树
>1，在做范围查找的时候，平衡树比skiplist操作要复杂。

### 跳表结构  vs 红黑树结构

## 有的key在删除时，可能导致内存没有及时释放
>当键被删除时，Redis并不总是释放(返回)内存到操作系统

### Hash 删除: hscan + hdel
```
public void delBigHash(String host, int port, String password, String bigHashKey) {
    Jedis jedis = new Jedis(host, port);
    if (password != null && !"".equals(password)) {
        jedis.auth(password);
    }
    ScanParams scanParams = new ScanParams().count(100);
    String cursor = "0";
    do {
        ScanResult<Entry<String, String>> scanResult = jedis.hscan(bigHashKey, cursor, scanParams);
        List<Entry<String, String>> entryList = scanResult.getResult();
        if (entryList != null && !entryList.isEmpty()) {
            for (Entry<String, String> entry : entryList) {
                jedis.hdel(bigHashKey, entry.getKey());
            }
        }
        cursor = scanResult.getStringCursor();
    } while (!"0".equals(cursor));
    
    //删除bigkey
    jedis.del(bigHashKey);
}

```

### List 删除: ltrim
```
public void delBigList(String host, int port, String password, String bigListKey) {
    Jedis jedis = new Jedis(host, port);
    if (password != null && !"".equals(password)) {
        jedis.auth(password);
    }
    long llen = jedis.llen(bigListKey);
    int counter = 0;
    int left = 100;
    while (counter < llen) {
        //每次从左侧截掉100个
        jedis.ltrim(bigListKey, left, llen);
        counter += left;
    }
    //最终删除key
    jedis.del(bigListKey);
}

```


### Set 删除: sscan + srem
```
public void delBigSet(String host, int port, String password, String bigSetKey) {
    Jedis jedis = new Jedis(host, port);
    if (password != null && !"".equals(password)) {
        jedis.auth(password);
    }
    ScanParams scanParams = new ScanParams().count(100);
    String cursor = "0";
    do {
        ScanResult<String> scanResult = jedis.sscan(bigSetKey, cursor, scanParams);
        List<String> memberList = scanResult.getResult();
        if (memberList != null && !memberList.isEmpty()) {
            for (String member : memberList) {
                jedis.srem(bigSetKey, member);
            }
        }
        cursor = scanResult.getStringCursor();
    } while (!"0".equals(cursor));
    
    //删除bigkey
    jedis.del(bigSetKey);
}

```

### SortedSet 删除: zscan + zrem
```
public void delBigZset(String host, int port, String password, String bigZsetKey) {
    Jedis jedis = new Jedis(host, port);
    if (password != null && !"".equals(password)) {
        jedis.auth(password);
    }
    ScanParams scanParams = new ScanParams().count(100);
    String cursor = "0";
    do {
        ScanResult<Tuple> scanResult = jedis.zscan(bigZsetKey, cursor, scanParams);
        List<Tuple> tupleList = scanResult.getResult();
        if (tupleList != null && !tupleList.isEmpty()) {
            for (Tuple tuple : tupleList) {
                jedis.zrem(bigZsetKey, tuple.getElement());
            }
        }
        cursor = scanResult.getStringCursor();
    } while (!"0".equals(cursor));
    
    //删除bigkey
    jedis.del(bigZsetKey);
}

```

## 主从同步

>主从同步分为 2 个步骤：同步和命令传播
>1，同步：将从服务器的数据库状态更新成主服务器当前的数据库状态。同步RDB
>2，命令传播：当主服务器数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的过程。同步aof操作

![主从同步](./res/redis-master-slave-sync.jpg "")

## 集群机制

### 主从模式:读写分离

![](./res/redis-master-slave.png "")

>主从模式难以在线扩容的缺点，Redis的容量受限于单机配置，是因为master 一个单机负责读写，slave负责读

### sentinel哨兵机制

>哨兵模式解决了主从复制不能自动故障转移，达不到高可用的问题，但还是存在难以在线扩容，Redis容量受限于单机配置的问题

![](./res/redis-sentinel.png "")

### cluster模式

>Cluster模式实现了Redis的分布式存储，即每台节点存储不同的内容，来解决在线扩容的问题
>能够实现自动故障转移，节点之间通过gossip协议交换状态信息，用投票机制完成slave到master的角色转换
>无中心架构，数据按照slot分布在多个节点
>集群中的每个节点都是平等的关系，每个节点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃，这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据


#### Cluster模式的具体工作机制：

>1，在Redis的每个节点上，都有一个插槽（slot），取值范围为0-16383
当我们存取key的时候，Redis会根据CRC16的算法得出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16383之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作
>2，为了保证高可用，Cluster模式也引入主从复制模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点
>3，当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点都宕机了，那么该集群就无法再提供服务了
>4，Cluster模式集群节点最小配置6个节点(3主3从，因为需要半数以上)，其中主节点提供读写操作，**从节点作为备用节点，不提供请求，只作为故障转移使用**

![](./res/redis-cluster.png "")