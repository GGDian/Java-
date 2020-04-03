## config



## 服务端

* yml

  ```yaml
  spring:
    application:
      name: cloud-config-center   #注册进Eureka服务器的微服务名
    cloud:
      config:
        server:
          git:
            uri: git@....        #GitHub上面的git仓库名字
            ###搜索目录
            search-paths: 
              - springcloud-config
        ###选取分支
        label: master
  
  #服务注册到eureka地址
  eureka:
    client:
      service-url: 
        defaultZone: http://localhost:7001/eureka
  ```

* pom

  ```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
  </dependency>
  ```

* 主配置类加注解 @EnableConfigServer
* 访问是 http://ip/ 分支 / 文件名.yml

## 客户端

* pom

  ```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-config</artifactId>
  </dependency>
  ```

* 新建一个 bootstrap.yml；这个加载优先级比 application.yml 高

  ```yml
  spring:
    application:
      name: cloud-eureka-service
    cloud:
      #Config 客户端配置
      config:
        label: master #分支名称
        name: config  #配置文件名称
        profile: dev #读取后缀名称  合并为 http://ip//master/config-dev
        uri: http://ip+端口
  ```

* 

* ![image-20200329182200979](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200329182200979.png)







### github 修改配置后；客服端如何获得通知

* pom

  ```
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

* yml暴露监控端点

  ```yml
  
  management:
    endpoints:
      web:
        exposure:
          include: "*"
  ```

* 加 **@RefreshScope** 在业务类 Controller 上

* 然后运维人员发送 post 请求刷新即可