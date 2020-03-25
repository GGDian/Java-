group by

* 新建临时表，扫描索引，把数据取出后放进临时表中
  * 临时表大小有限制，由参数 tmp_table_size 控制大小，默认16M，太小的时候，会先把数据放进内存临时表，插入一部分数据后，发现不够用，才转成磁盘临时表，磁盘临时表默认使用的引擎是 InnoDB
* 遍历结束后会根据索引字段进行排序，返回结果集给客户端
  * 排序又会用到内存临时表，放到 sort_buffer 中进行排序；如果是 rowid排序，还要再回内存临时表取值
* 如果不想在最后阶段排序，可以在 group by 后加上 order by null



### 索引优化：

前提：

* 不管是内存临时表还是磁盘临时表，group by 逻辑都需要构造一个带唯一索引的表，执行代价比较高；

原因：

* group by 的语义逻辑，是统计不同的值出现的个数，但是，每一行的值的结果都是无序的，所以需要一个临时表，来记录并统计结果

解决：

* 扫描过程中保证出现的数据有序

具体：

* generated column

```mysql
alter table t1 add column z int generated always as(id%100),add index(z);
```

如此，group by 时就可以用上索引z

```mysql
select z,count(*) as c from t1 group by z;
```

### 直接排序优化：

* 在语句中加入 SQL_BIG_RESULT 这个提示，告知优化器：这个语句涉及的数据量很大，请直接使用磁盘临时表
* 优化器会直接用数据来存，因为效率比 B+ 树高
  * 初始化 sort_buffer，确定放入一个整型字段，记为 m
  * 扫描索引，依次放入要排序的值
  * 扫描完成，对 sort_buffer 的字段 m 做排序；如果sort_buffer不够用，会用磁盘临时表
  * 排序完成，会得到一个有序数组