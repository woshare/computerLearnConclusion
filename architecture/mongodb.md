
## 类redolog的数据持久化方案
>对于write操作而言，首先写入journal日志，然后将数据在内存中修改（mmap），此后后台线程间歇性的将内存中变更的数据flush到底层的data files中，时间间隔为60秒（参见配置项“syncPeriodSecs”）；
write操作在journal文件中是有序的，为了提升性能，write将会首先写入journal日志的内存buffer中，当buffer数据达到100M或者每隔100毫秒，buffer中的数据将会flush到磁盘中的journal文件中；
如果mongodb异常退出，将可能导致最多100M数据或者最近100ms内的数据丢失，flush磁盘的时间间隔由配置项“commitIntervalMs”决定，默认为100毫秒。
mongodb之所以不能对每个write都将journal同步磁盘，这也是对性能的考虑，mysql的binlog也采用了类似的权衡方式。
开启journal日志功能，将会导致write性能有所降低，可能降低5~30%，因为它直接加剧了磁盘的写入负载，我们可以将journal日志单独放置在其他磁盘驱动器中来提高写入并发能力（与data files分别使用不同的磁盘驱动器


## 数据文件示例
>mongodb的数据将会保存在底层文件系统中，比如我们dbpath设定为“/data/db”目录，我们创建一个database为“test”，collection为“sample”，然后在此collection中插入数条documents。我们查看dbpath下生成的文件列表
-rw-------  1 mongo  mongo    16M 11  6 17:24 test.0
-rw-------  1 mongo  mongo    32M 11  6 17:24 test.1
-rw-------  1 mongo  mongo    64M 11  6 17:24 test.2
-rw-------  1 mongo  mongo   128M 11  6 17:24 test.3
-rw-------  1 mongo  mongo   256M 11  6 17:24 test.4
-rw-------  1 mongo  mongo   512M 11  6 17:24 test.5
-rw-------  1 mongo  mongo   512M 11  6 17:24 test.6
-rw-------  1 mongo  mongo    16M 11  6 17:24 test.ns

>序列号从0开始，逐个递增，数据文件从16M开始，每次扩张一倍（16M、32M、64M、128M...），在默认情况下单个data file的最大尺寸为2G，如果设置了smallFiles属性（配置文件中）则最大限定为512M；mongodb中每个database最多支持16000个数据文件，即约32T，如果设置了smallFiles则单个database的最大数据量为8T

## 默认系统级的admin和local库
>mongod中还有2个默认的database，系统级的，“admin”和“local”；它们的存储原理同上，
>其中“admin”用于存储“用户授权信息”，比如每个database中用户的role、权限等；
>“local”即为本地数据库，我们常说的oplog（replication架构中使用，类似与binlog）即保存在此数据库中。

* [持久化讲解的不错](http://blog.itpub.net/31561269/viewspace-2663773/)

## 索引
>索引：提高查询性能，默认情况下_id字段会被创建唯一索引；因为索引不仅需要占用大量内存而且也会占用磁盘，所以我们需要建立有限个索引，而且最好不要建立重复索引；每个索引需要8KB的空间，同时update、insert操作会导致索引的调整，会稍微影响write的性能，索引只能使read操作收益，所以读写比高的应用可以考虑建立索引

![](./res/mongo-index.jpeg "")