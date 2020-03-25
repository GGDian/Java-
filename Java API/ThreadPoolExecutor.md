ThreadPoolExecutor

* submit(Runnable task)
  * newTaskFor(task,null)
    * return new FutureTask<T>(runnable,value)
  * execute(RunnableFuture)



* Worker extends AQS implements Runnable

  * Thread thread

  * Runnable firstTask

  * volatile long completedTasks

  * ```java
    Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            // 调用 ThreadFactory 来创建一个新的线程
            this.thread = getThreadFactory().newThread(this);
    }
    ```

  * ```java
    public void run(){
    	runWorker(this);
    }
    ```



* execute
  * 获取线程池状态和数量 c = ctl.get()
  * 当前线程数 < corePoolSize
    * addWorker(command,true)
      * 返回
  * 失败了；判断线程池是否在运行；command 进入 workQueue
  * 如果队列满了，以 maximumPoolSize 为界创建新的 worker；addWorker(command,false)





* addWorker(Runnable firstTask, boolean core)
  * 获取线程运行时状态，如果线程池处于 shutdown；并且 firstTask 为空；队列不为空，即允许创建线程；
  * 还要判断工作线程数量是否超过了 core 或者 maxim
  * CAS 将工作线程数加 1
  * 增加失败；再获取线程池状态；如果变了要重新判断状态
  * 获取 mainLock 可重入锁
  * 把 firstTask 传给 worker 的构造方法 new Worker(firstTask)
  * 获取 worker 线程对象 ； Thread t = w.thread;
  * mainLock 加锁，防止在开启线程时，线程池被关闭
  * 再次判断状态
  * 把新创建的 worker 添加到 workers 这个 HashSet 中
  * 将 workerAdded 设置为 true；mainLock 解锁
  * 如果 workerAdded 为 true； 即设置成功；开启线程
  * workerStarrted = true;
  * finally -- >       workerStarted失败；执行 addWorkerFailed()
    * 获取 mainLock 锁
    * 从 workers 中 remove
    * 再把线程数 减 1
  * 返回 workerStarted
* runWorker(Worker w)
  * w.unlock
  * firstTask 不为空；或者 getTask() 不为空
  * w.lock()
  * 判断线程池状态，如果线程池状态大于等于 STOP，那么意味着该线程也要中断
  * 执行任务；task.run()
  * 执行完；置空 task，累加完成的任务数，释放 worker 的锁
  * 重新 getTask() 去获取队列中的锁
  * 最后执行线程关闭
    * getTask 返回 null，队列中已经没有任务需要执行了
    * 任务执行过程中发生了异常







```java
// 此方法有三种可能：
// 1. 阻塞直到获取到任务返回。我们知道，默认 corePoolSize 之内的线程是不会被回收的，
//      它们会一直等待任务
// 2. 超时退出。keepAliveTime 起作用的时候，也就是如果这么多时间内都没有任务，那么应该执行关闭
// 3. 如果发生了以下条件，此方法必须返回 null:
//    - 池中有大于 maximumPoolSize 个 workers 存在(通过调用 setMaximumPoolSize 进行设置)
//    - 线程池处于 SHUTDOWN，而且 workQueue 是空的，前面说了，这种不再接受新的任务
//    - 线程池处于 STOP，不仅不接受新的线程，连 workQueue 中的线程也不再执行
```

* getTask()
  * 判断状态
    * rs == SHUTDOWN && workQueue.isEmpty()，返回null
    * rs >= STOP ; 返回null
  * 判断当前线程是否允许回收 timed = allowCoreThreadTimeOut || wc > corePoolSize;
  * 