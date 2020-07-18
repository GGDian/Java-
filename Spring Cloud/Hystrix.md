### Hystrix

保证服务调用方的线程**不会被长时间、不必要地占用**，从而避免了故障在分布式系统中的蔓延，乃至雪崩

![image-20200328175852944](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200328175852944.png)



### 哪些情况会触发降级

* 程序运行异常

* 超时
* 服务熔断触发服务降级
* 线程池/信号量打满也会导致服务降级

### 服务熔断

达到最大服务访问后，直接拒绝访问；调用降级的方法返回

服务的降级 -> 进而熔断->恢复调用链路

### 服务限流

秒杀高并发等操作，一秒钟 N 个，有序进行



服务端与客户端都可以进行降级

### 步骤

* pom

  ```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  </dependency>
  ```

* 主配置类激活 @EnableCircuitBreaker 

* 编写 **Service** 方法   **注意：兜底的方法参数要跟原来方法一致！！！**

  ```java
  @HystrixCommand(fallbackMethod = "paymentTimeoutHandler",commandProperties = {
          @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
  })
  //超时三秒即服务降级；
  public String paymentTimeout(){
      try {
          TimeUnit.SECONDS.sleep(5);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
      return "成功！";
  }
  public String paymentTimeoutHandler(){
      return "超时了喂！";
  }
  ```

### 客户端开启降级

* pom

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

* yml 文件；与 feign 整合，开启降级功能

  ```yml
  feign:
    hystrix:
      enabled: true
  ```

* 主类加注解 @EnableHystrix

* controller 层；**注意：兜底的方法参数要跟原来方法一致！！！**

  ```java
  @GetMapping(value = "/payment/get/{id}")
  @HystrixCommand(fallbackMethod = "paymentTimeoutHandler",commandProperties = {
          @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "2000")
  })
  public String getPaymentById(@PathVariable("id") Long id){
      return paymentFeignService.getPaymentById(id);
  }
  
  public String paymentTimeoutHandler(Long id){
      return "服务端没给我返回哦！";
  }
  ```





### 优化代码

每个业务方法对应一个兜底的方法，代码膨胀

统一和自定义的分开

* DefaultProperties(defaultFallback="")；加在类上，默认全局兜底的方法；**这个方法就不能有参数了的**

![image-20200421155003228](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200421155003228.png)

* feignFallback：借助 @FeignClient 来实现，编写一个类，实现 @FeignClient 标注的接口；重写方法；方法体即对应的兜底内容；

![image-20200421155458821](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200421155458821.png)



### 熔断

* 注解为 @HystrixCommand

* 配置注解

  ```java
  @HystrixCommand(fallbackMethod = "paymentTimeoutHandler",commandProperties = {
          @HystrixProperty(name = "circuitBreaker.enabled",value = "true"), //是否开启断路器
          @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),//请求次数
          @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),// 时间窗口期
          @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60")//失败率达到多少后跳闸
  })
  ```

![image-20200421155925289](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200421155925289.png)

熔断打开，



![image-20200421160726158](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200421160726158.png)

![image-20200421161442333](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200421161442333.png)



![image-20200421161541145](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200421161541145.png)