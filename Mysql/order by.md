### order by

* sort_buffer_size，MySQL 为排序开辟的内存（sort_buffer）的大小
* max_length_for_sort_data，MySQL 中专门控制用于排序的行数据的长度的一个参数；要排序的字段超过这个值时，会改用 rowId 排序
* 如果内存足够，可以多利用内存，尽量减少磁盘访问

### 优化

* 原因是原来的数据是无序的
* 建索引，保证要排序的字段有序即可