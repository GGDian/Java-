### Sentinel

* pom

  ```
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
  </dependency>
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>sentinel-datasource-nacos</artifactId>
  </dependency>
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```

* yml

  ```yml
  spring:
    application:
      name: cloudalibaba-sentinel-service
    cloud:
      nacos:
        discovery:
          # Nacos 服务注册中心地址
          server-addr: localhost:8848
      sentinel:
        transport:
          #配置Sentinel dashboard地址
          dashboard: localhost:8080
          #默认8719端口，假如被占用会自动从8719开始依次 +1扫描，直至找到未被占用的端口
          port: 8719
  ```







### 热点 key

用到了 @SentinelResource 注解；放在 handle 方法上

下面的配置是超过 1 秒 1 次即会被限流

![image-20200401200406111](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401200406111.png)

![image-20200401201251001](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401201251001.png)

![image-20200401201329009](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401201329009.png)

#### 参数例外项

![image-20200401201757580](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401201757580.png)

![image-20200401201905743](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401201905743.png)

### 系统规则：针对整个服务，在最外围进行限流



#### 服务一下线，在 Sentinel 控制台的流控规则中的设置也会消失，说明是临时节点

![image-20200401202554822](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401202554822.png)



### 自定义限流处理逻辑

理由：在注解中添加兜底属性方法，会导致耦合度过高，应该抽取出来

![image-20200401203344971](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401203344971.png)

![image-20200401203420489](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401203420489.png)

![image-20200401203515392](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401203515392.png)



#### 只配置 fallback

![image-20200401205821575](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401205821575.png)

![image-20200401210117780](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401210117780.png)



![image-20200401210222960](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401210222960.png)

#### fallback 和 blockHandler 两个都配置的话则 QPS 过高时，是 blockHandler 先生效

![image-20200401210533524](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401210533524.png)







### Sentinel 和 Feign 整合

* pom

  ```
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

* yml

  ```yaml
  # 激活Sentinel 对 Feign 的支持
  feign:
  	sentinel:
  		enabled: true
  ```

* 主启动类 @EnableFeignClients
* 接口 + 注解

![image-20200401212143209](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401212143209.png)

![image-20200401212057391](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401212057391.png)





#### Sentinel 的持久化配置

方式：把配置持久化到 nacos

* pom![image-20200401213009183](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401213009183.png)

* yml![image-20200401213130708](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401213130708.png)

* nacos 上的配置![image-20200401213746814](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401213746814.png)

![image-20200401213808577](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200401213808577.png)