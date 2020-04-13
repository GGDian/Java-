yml![image-20200328161557082](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200328161557082.png)

![image-20200328161927355](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200328161927355.png)



#### HTTP 客户端

#### 步骤

* pom文件

```
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

* 主启动类开启 @EnableFeignClients（basePackages = "包路径"）

  * 比如 basePackages = "com.atguigu.gulimall.merber.feign"

* service层写接口；方法的签名要与服务端的一致

  ```java
  @Component
  //服务名
  @FeignClient(value = "cloud-provider-service")
  public interface PaymentFeignService {
  
      @GetMapping(value = "/payment/get/{id}")
      public CommonResult getPaymentById(@PathVariable("id") Long id);
  }
  ```

* controller层掉用 service 层

  ```java
  @GetMapping(value = "/payment/get/{id}")
  public CommonResult getPaymentById(@PathVariable("id") Long id){
      return paymentFeignService.getPaymentById(id);
  }
  ```



Feign 客户端默认的超时时间 1 秒

### 超时设置

feign 内置了 ribbon

```yml
ribbon:
  # 请求的超时时间，默认是1秒
  ReadTimeout: 5000
  # 建立连接超时的时间
  ConnectionTimeout: 5000
```



### 日志增强

* 配置类

  ```java
  @Configuration
  public class FeignConfig {
      @Bean
      Logger.Level feignLoggerLevel(){
          return Logger.Level.FULL;
      }
  }
  ```

* yml 配置

  ```yml
  logging:
    level: 
      com.atguigu.springcloud.service.PaymentFeignService: debug
  ```