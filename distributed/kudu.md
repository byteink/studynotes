
# 2 Kudu at a high level

## 2.1 Tables and schemas
每个表包含有限个列，每列包含name、type和nullability，这些列的有序子集可以作为主键
用户可以在任意时间发起alter table命令来增加或删除某些列，但是限制不能删除主键列

显式的列类型：
为列指定具体类型而不是是个bytes可以允许使用类型特定的编码方式，
也允许我们暴露类SQL的元数据给其他系统

kudu不提供二级索引，也不提供除了主键外的唯一性约束（unique constraints）

## 2.2 Write operations
当前不提供多行事务API，单行修改跨列原子执行

## 2.3 Read operations
只提供一个Scan操作来实现从table中获取数据。
scan时用户可以添加任意个断言来过滤结果，当前提供两种断言：
一个列和一个常数的比较；主键范围的组合；
用户可以在scan时指定一个投影（projection），投影包含需要获取的列的子集。
指定子集可以提供性能。


## 2.5 Consistency Model
Kudu给客户端提供了两种一致性的选择。默认是快照一致性（snapshot consistency)。
一次scan产生一个snapshot，保证不会出现违反因果关系的异常。
同时保证从单个client来看read-your-writes一致性

Kudu提供选项在clients之间传播timestamps：
当执行完一个write后，用户可以先client library获取一个timestmap token
这个token可以通过外部渠道传播给另外一个客户端，然后传递给Kudu API，
保持跨越两个客户端的write操作的因果关系

如果觉得传播token太繁琐，Kudu也提供了像Spanner一样的commit-wait。
如果一个write启用了commit-wait，当执行完后client可能会延迟一段时间，来保证任何后续的write操作是因果正确的
由于缺少特定的时间同步硬件，这会给write操作引入显著的延迟（100-1000ms默认的NTP配置）。

timestamp的分配基于HybridTime算法。


## 2.6
Kudu不允许用户write操作时指定timestamp
但read操作时可以指定timestamp，可以让多个分布式任务读到一致性的快照。


# 3 Architechure

## 3.1 Cluster roles
Master server负责元信息，Tablet server负责数据

## 3.2 Partitioning
tables按水平分区。

行根据主键的值被映射到某个tablet，因此像insert或者updates随机访问操作只会影响一个tablet。
对于大的表，推荐每个机器上有个10-100个tablets，每个tablet可以是数十GB。

Kudu支持灵活的partition schema。partition schema是一个计算主键partition key的函数。
每个tablet包括一个连续的partition key范围。

partition schema由一个可选的range-partitioning rule 加上零到多个hashpartitioning rules

## 3.3 Replication
使用raft来复制tablet。

write操作发送给leader，leader使用一个本地的lock manager来支持并发操作，
产生一个MVCC timestamp，然后通过raft来完成请求。如果多数副本接受这个write，记录到WAL，
然后这个write就可以被提交。

raft参数：500ms心跳，1500ms选举超时
对Raft的改进：
1. 

# 4 Tablet storage

## 4.6 INSERT path
每个tablet有一个MemRowSet
为了保证主键的唯一性，必须在插入新行前查询所有已存在的DiskRowSets
为了高效地查询DiskRowSets，每个DiskRowSets存储了一个已存在key集合的Bloom Filter
Bloom filer按4KB page分块，每个对应一小段key range，并使用B树所有这些pages。
这些pages也会缓存在一个LRU结构。

另外对于每个DiskRowSet，存储它的最小和最大的主键，使用区间树和key边界来索引DiskRowSet。


## 4.9 Delta Compaction
只compaction更新比较频繁的列，避免IO操作


## 4.10 RowSet Compaction
基于Key merge两个或多个DiskRowSets，输出新的DiskRowsets，也按32MB分卷，笔迷阿妈太大的DRS。
Compaction的目的：
- 可以有机会移除被删除的行
- 减少DRS之间key range的重叠







