### change buffer

```java
// 表示 change buffer 的大小最多只能占用 buffer pool 的百分之几
innodb_change_buffer_max_size
```

### 重新统计索引信息

```
analyze table 表名;
```

### 扫描主键的行数

```
主键是直接按照表的行数来估计的，而表的行数，优化器直接用的是 show table status 的值
```

### flush效率

```java
// 这个参数会告诉 InnoDB 系统的磁盘能力	
innodb_io_capacity
// 脏页比例上限，默认是 75%
innodb_max_dirty_pages_pct
// 算出脏页比例和redolog日志中已经用比例，取其中较大的值记为 R，之后引擎就可以按照 innodb_io_capacity 定义的能力乘以 R% 来控制刷脏页的速度
//脏页比例
Innodb_buffer_pool_pages_dirty/Innodb_buffer_pool_pages_total
// 这个参数为 1 的时候，脏页刷盘时，如果邻居也是脏页，会一起拖下水进行刷盘
innodb_flush_neighbors
```



### binlog日志

系统会给 binlog cache 分配一片内存，每个线程一个。但是共用同一份 binlog 文件

```java
// 控制单个线程内 binlog cache 所占内存的大小
binlog_cache_size

```

write（写到文件系统的 page cache） 和 fsync（持久化到磁盘） 的时机，是由参数 sync_binlog 控制的

```
0,每次都只是write
1,每次提交事务都会执行 fsync
N,每个事务都 write，积累到 N 个事务后才 fsync
```



组提交

```
// 表示延迟多少微秒后才调用 fsync
binlog_group_commit_sync_delay
// 表示积累多少次以后才调用 fsync
binlog_group_commit_sync_no_delay_count

两个参数是或的关系，只要满足一个条件就会调用 fsync
```



binlog 是不能“被打断的”。一个事务的 binlog 必须连续写，因此要整个事务完成后，再一起写到文件里

### redo log 日志

redo log 的写入策略，innodb_flush_log_at_trx_commit

```
0，每次事务提交把 redo log 留在 redo log buffer
1，每次事务提交持久化到磁盘
2，每次事务提交把 redo log 写到 page cache
为 1 时，redo log 在 prepare 阶段就要持久化一次
```

事务执行中间过程的 redo log 也是直接写在 redo log buffer 中的，这些 redo log 也会被后台线程一起持久化到磁盘。也就是说，一个没有提交的事务的redo log，也是可能已经持久化到磁盘的



三种场景会让一个没有提交的事务的 redo log 写入到磁盘中：

* redo log buffer 占用的空间即将达到 innode_log_buffer_size 一半的时候，后台会主动写盘。事务还没提交，所有写盘动作是 write。并没有 fsync。只是留在了文件系统的 page cache
* 并行的事务提交时，顺带将这个事务的 redo log buffer 持久化到磁盘。
* 后台有个线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘

### WAL

* redo log 和 binlog 都是顺序写，磁盘的顺序写比随机写速度更快
* 组提交机制，可以大幅度降低磁盘的 IOPS 消耗



### 发送数据

```
net_buffer：由 net_buffer_length 定义的，默认是 16k
重复获取行，直到 net_buffer 写满，调用网络接口发出去
```



仅当一个线程处于“等待客户端接收结果”的状态，才会显示“**Sending to Client**”；而如果显示成“**Sending data**”，意思是正在执行





### Buffer Pool

大小由参数 innodb_buffer_pool_size 确定的，一般建议设置成可用物理内存的 60%-80%

```
// 默认一秒，LRU淘汰策略的参数
innodb_old_blocks_time
```

old 和 young 区的比例为 3：5





### Group by

内存临时表的大小有限制，参数 tmp_table_size 就是控制这个内存大小的，默认是 16 M

优化：

* 加索引

```mysql
alter table t1 add column z int generated always as(id % 100), add index(z);
```

* 直接排序

  在 group by 语句中加入 SQL_BIG_RESULT 这个提示（hint），就可以告诉优化器：这个语句涉及的数据量很大，请直接用磁盘临时表。

  MySQL 的优化器一看，磁盘临时表是 B+ 树存储，存储效率不如数组来得高。所以，既然你告诉我数据量很大，那从磁盘空间考虑，还是直接用数组来存吧。

```mysql
select SQL_BIG_RESULT id%100 as m, count(*) as c from t1 group by m;
```

1. 如果对 group by 语句的结果没有排序要求，要在语句后面加 order by null；
2. 尽量让 group by 过程用上表的索引，确认方法是 explain 结果里没有 Using temporary 和 Using filesort；
3. 如果 group by 需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大 tmp_table_size 参数，来避免用到磁盘临时表；
4. 如果数据量实在太大，使用 SQL_BIG_RESULT 这个提示，来告诉优化器直接使用排序算法得到 group by 的结果。