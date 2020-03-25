ReentrantLock：

* lock
  * acquire(1) (AQS中)
    * tryAcquire(1)  子类中
      * 如果 state 为 0，CAS 尝试抢锁变为 1
      * 当前线程重入，state+1
    * acquireQueued(addWaiter(Node.EXCLUSIVE),1)
      * addWaiter
        * 取到 tail，判断是否为空
          * 不为空，CAS 将新节点设置为 tail
        * enq(node)
          * 取到 tail，判断是否为空
          * 为空，初始化 head 和 tail，new Node()
          * 不为空，CAS将新节点设置为 tail
      * acquireQueued(addWaiter(Node.EXCLUSIVE),1)
        * for()
        * 拿到新节点的前驱节点；如果为 head ，尝试抢锁，tryAcquire(1)
          * 抢锁成功，设置节点为 head
        * 抢锁失败或者不为 head；将前驱节点的 waitStatus 变为 -1；然后在下一个循环把新节点的线程挂起
        * 从挂起中醒来又会去拿当前节点的前驱节点，判断进行抢锁，如果抢锁失败，则又会被挂起
* unlock
  * release(1)
    * tryRelease(1)
      * 获取 state 并减 1，如果 state 为 0，即返回 true (此刻已经拥有锁，所以不需要用 CAS)
    * 获取 head ，如果 head != null && waitstatus != 0，则唤醒下一个节点
    * unparkSuccessor(h);
      * CAS 将 head.waitstatus 变为 0；
      * 获取下个节点，如果为 null，或者已经取消（waitStatus>0）；从后面往前面遍历，找到最前面未被取消的节点；进行唤醒





Condition: 实现类为 AQS 中的 ConditionObject，即代码都在 AQS中

* await

  * addConditionWaiter() 添加节点到条件队列

    * 获取 lastWaiter；
    * 如果 lastWaiter 不为空 且已经取消等待，unlinkCancelledWaiters();
      * 链表操作，分为两种情况；头节点取消或中间节点取消
    * 新建节点 node
    * 加到条件队列中（如果 lastWaiter 为空，赋值给 firstWaiter；否则，赋值给 lastWaiter.nextWaiter）

  * fullyRelease(node) //释放可重入锁

    * 获取 state，调用 release(state)

  * isOnSyncQueue(node) //节点是否在阻塞队列中

    * node.waitStatus == condition(-2) || node.prev==null 返回 false
    * node.next != null; 返回true
    * 从后往前遍历链表；判断是否有相等

  * 节点不在阻塞队列中，挂起等待唤醒

  * 节点唤醒（有可能是在进阻塞队列的时候被唤醒；也有可能在队头被唤醒），判断是否中断；checkInterruptWhileWaiting(node)

    * 中断？(transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT
      * CAS 将 node.waitStatus 从 -2 变成 0
      * 成功即线程在 signal 前已经中断，执行节点入阻塞队列操作，enq(node)；返回true；返回 THROW_IE(-1)
      * 否则节点是在 signal 后中断；
    * 没有中断，返回 0

  * acquireQueued(node,saveState) //进入阻塞队列去获取锁

    * 如果不是头节点，或者抢锁失败；会把前驱节点的 waitStatus 变成 -1，进入挂起状态
    * 等待获取锁

  * ```
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    ```

    如果被中断了自己进入队列，即node.nextWaiter 还不等于 null，还存在队列的引用中；要清除掉的

  * interruptMode != 0；即 node 在阻塞队列或者条件队列中被中断了

    * 如果是在条件队列里面被中断，则最后会抛出异常
    * 如果是在阻塞队列里面被中断，最后会设置线程为中断状态

* signal   //将 node 从条件队列中搬到阻塞队列中；

  * firstWaiter 不为空；执行 doSignal 操作
    * firstWaiter.nextWaiter = null
    * transferForSignal(first) //转移
      * CAS 将 node.waitStatus 从 -2 变为 0 
      * enq(node) 入阻塞队列；返回前驱 node
      * 如果前驱 node 的 ws>0 唤醒 node ; 否则CAS 变为 -1；失败唤醒node





CountDownLatch

* await()
  * acquireSharedInterruptibly(1)
    * tryAcquireShared(arg) < 0
      * (getState() == 0) ? 1 : -1; 即就是 state 不等于 0，await 可以一直加节点
    * doAcquireSharedInterruptibly(arg);
      *  Node node = addWaiter(Node.SHARED); 加入新节点
      * 获取前驱节点；为 head -> 去看看 state 等于 0 了没有；等于 0 我们就可以冲了！！！
      * 门还没有耶（state不为0；还要继续 countDown 才行呀）；先挂起一段时间，等前驱节点唤醒
      * 节点被唤醒；如果前驱节点为 head；判断当前 state 等于 0 了没有
      * 可以了，执行 setHeadAndPropagate(node,r)
        * 设置 node 为 head
        * node.next != null；执行 doReleaseShared()；继续唤醒下一个节点
* countDown()
  * releaseShared(1)
    * tryReleaseShared(arg)
      * 减1；然后 CAS 进行设置值；如果 state==0；执行唤醒 node 操作
    * doReleaseShared()
      * 获取 head；
        * head != null && head != tail
          * ws == -1 --》 CAS 操作 -1 变成 0  -----》唤醒下一个节点 unparkSuccessor(h)
          * ws == 0 ---》 CAS 操作 0 变成 -3    失败-------》 continue 下一个循环
        * 执行 一系列操作后，h 依然等于 head 即 break;





CyclicBarrier

* await()
  * dowait(false, 0L)
    * 获取 lock ；执行 lock 方法
    * 获得 generation ；判断是否破了，破了即抛出 BrokenBarrierExcption();
    * 线程被中断，打破栅栏（breakBarrier()），抛出 InterruptedException()；
    * --count；等于 0 执行 command 任务，执行 nextGeneration()
      * trip.signalAll()；
      * count = parties;
      * generation = new Generation;
    * 不等于 0 ；即要进行阻塞操作，借助 trip.await()
      * 被中断，捕获到 InterruptedException异常；
      * 如果 generation 还没有变化；栅栏没被破坏；破环栅栏 breakBarrier()
        * generation.broken = true;
        * count = parties;
        * trip.signalAll();
      * 否则，中断当前线程；
    * 被唤醒后，判断栅栏是否被破坏，true 即 抛出 BrokenBarrierException();
    * generation 变化了，返回；
    * nanos <= 0L ,即超时了；打破栅栏；抛出 TimeoutException();



Semaphore  有公平锁，非公平锁之分

acquire

* acquireSharedInterruptibly(1)
  * tryAcquireShared(arg) < 0
    * 公平锁：当前队列中是否有在排队，有即返回 -1
    * 获取 state 减去 1；不小于 0 ； CAS 更新 state，返回 state
    * 没有获取锁，即要进入队列

release

* releaseShared(1)
  * tryReleaseShared(arg)
    * CAS 更新 state，成功就返回 true；直接唤醒后续节点
    * 但是后续节点唤醒后又会去抢锁，调用 tryAcquireShared

