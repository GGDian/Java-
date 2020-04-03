### GateWay

![image-20200329111153579](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200329111153579.png)

![image-20200329111825409](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200329111825409.png)

![image-20200329111942176](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200329111942176.png)











## 步骤

* pom

  ```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-gateway</artifactId>
  </dependency>
  ```

  pom 不能有 web 和 actuator 依赖

* yml；实现以用户名来查找，lb 开头

  ```yml
  spring:
    application:
      name: cloud-gateway
    cloud:
      gateway:
        discovery:
            locator:
              enabled: true           #开启从注册中心动态创建路由的功能，利用服务名进行路由
        routes:
        - id: payment_routh  #路由的ID，没有固定规则但要求唯一，建议配合服务名
  #          uri: http://localhost:8001   #匹配后提供服务的路由地址
            uri: lb://cloud-payment-service   #匹配后提供服务的路由地址
            predicates:
              - Path=/payment/get/**     #断言，路径相匹配的进行路由
              - After=2020-03-29T12:24:56.536+08:00[Asia/Shanghai]     #在这个时间后才可以访问
              - Cookie=username,zzyy     #第一个为名字，第二个是正则表达式
              - Header=X-Request-Id,\d+    #第一个为名字，第二个是正则表达式
  
        - id: payment_routh2  #路由的ID，没有固定规则但要求唯一，建议配合服务名
  #          uri: http://localhost:8001   #匹配后提供服务的路由地址
            uri: lb://cloud-payment-service   #匹配后提供服务的路由地址
            predicates:
              - Path=/payment/lb/**     #断言，路径相匹配的进行路由
  ```

### 另一种配置方法

![image-20200329115634274](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200329115634274.png)

### 全局过滤器

getQueryParams()；获取请求参数

![image-20200329172900483](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200329172900483.png)