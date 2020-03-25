PriorityBlockingQueue

* take
  * 获取锁
  * while ---> dequeue== null ? noEmpty.await()
    * size 为0 返回 null
    * 否则，取头节点返回
    * 取尾节点进行树化 siftDownComparable
      * 从顶向下开始树化，找到尾节点合适的位置
      * n >>> 1   ----->    定位 half
      * 从头节点开始，左右节点比较，找出左右节点比较小的，再跟尾节点比较，如果尾节点比较小；则 break；k 为尾节点的新位置
      * 尾节点比较大，把比较小的子节点赋值给头节点；必须向下判断遍历

