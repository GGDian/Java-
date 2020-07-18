Hystrix 有两种隔离策略：

- **线程池**隔离
  - tomcat的线程会将请求的任务**交给服务的内部线程池**里面的线程来执行，tomcat的线程就可以去干别的事情去了
- **信号量**隔离

实际场景下，使用**线程池隔离居多**，因为支持**超时**功能。

|          | 线程池隔离               | 信号量隔离                  |
| -------- | ------------------------ | --------------------------- |
| 线程     | 与调用线程非相同线程     | 与调用线程相同（jetty线程） |
| 开销     | 排队、调度、上下文开销等 | 无线程切换，开销低          |
| 异步     | 支持                     | 不支持                      |
| 并发支持 | 支持（最大线程池大小）   | 支持（最大信号量上限）      |





### 聊聊 Hystrix 缓存机制？

Hystrix 提供缓存功能，作用是：

- 减少重复的请求数。
- 在同一个用户请求的上下文中，相同依赖服务的返回数据始终保持一致。

### 什么是 Hystrix 断路器？

Hystrix 断路器通过 HystrixCircuitBreaker 实现。

HystrixCircuitBreaker 有三种状态 ：

- `CLOSED` ：关闭
- `OPEN` ：打开
- `HALF_OPEN` ：半开



- **红线**：初始时，断路器处于CLOSED状态，链路处于健康状态。当满足如下条件，断路器从CLOSED变OPE状态：
  - **周期**( 可配，`HystrixCommandProperties.default_metricsRollingStatisticalWindow = 10000 ms` )内，总请求数超过一定**量**( 可配，`HystrixCommandProperties.circuitBreakerRequestVolumeThreshold = 20` ) 。
  - **错误**请求占总请求数超过一定**比例**( 可配，`HystrixCommandProperties.circuitBreakerErrorThresholdPercentage = 50%` ) 。
- **绿线** ：断路器处于 `OPEN` 状态，命令执行时，若当前时间超过断路器**开启**时间一定时间( `HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds = 5000 ms` )，断路器变成 `HALF_OPEN` 状态，**尝试**调用**正常**逻辑，根据执行是否成功，**打开或关闭**熔断器【**蓝线**】。



所谓 **熔断** 就是服务雪崩的一种有效解决方案。当指定时间窗内的请求失败率达到设定阈值时，系统将通过 **断路器** 直接将此请求链路断开。

这里所讲的 **熔断** 就是指的 `Hystrix` 中的 **断路器模式** ，你可以使用简单的 `@HystrixCommand` 注解来标注某个方法，这样 `Hystrix` 就会使用 **断路器** 来“包装”这个方法，每当调用时间超过指定时间时(**默认为1000ms**)，断路器将会中断对这个方法的调用。



**降级是为了更好的用户体验，当一个方法调用异常时，通过执行另一种代码逻辑来给用户友好的回复**。这也就对应着 `Hystrix` 的 **后备处理** 模式。