### Hystrix

![image-20200328175852944](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200328175852944.png)



### 步骤

* pom

  ```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  </dependency>
  ```

* 主配置类激活 @EnableCircuitBreaker 

* 编写 Service 方法   **注意：兜底的方法参数要跟原来方法一致！！！**

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

* yml 文件

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

* DefaultProperties(defaultFallback="")；加在类上，默认兜底的方法；**这个方法就不能有参数了的**
* feignFallback：借助 @FeignClient 来实现，编写一个类，实现 @FeignClient 标注的接口；重写方法；方法体即对应的兜底内容；



### 熔断

* 注解为 @HystrixCommand

* 配置注解

  ```java
  @HystrixCommand(fallbackMethod = "paymentTimeoutHandler",commandProperties = {
          @HystrixProperty(name = "circuitBreaker.enabled",value = "true"), //是否开启断路器
          @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),//请求次数
          @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),// 时间窗口期
          @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60")//失败达到多少后跳闸
  })
  ```