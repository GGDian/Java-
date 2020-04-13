### Nacos

![image-20200405152516942](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200405152516942.png)

![image-20200404140253933](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200404140253933.png)



nacos 自带负载均衡，支持 AP 和 CP 切换

* pom

  ```
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```

* yml

  ```yml
  spring:
    application:
      name: nacos-provider
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848
          
  management:
    endpoints:
      web:
        exposure:
          include: '*'
  ```

* 主配置类加载@EnableDiscoveryClient





### 配置文件管理

* ```
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  </dependency>
  ```

* bootstrap.yml 这个优先级比 application.yml 高；优先去远端仓库获得配置

  ```yml
  spring:
    application:
      name: nacos-provider
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848
        config:
        	server-addr: localhost:8848 #Nacos 作为配置中心地址
        	file-extension: yaml  #指定yaml格式的配置
        	group: 分组名
        	namespace: 命名空间id
  ```

  

* 控制器 controller 加上注解 @RefreshScope；支持 Nacos 的动态刷新功能

* Nacos 上配置文件的名称为 ${prefix}-${spring.profile.active}.${file-extension}；即服务名-激活分支.yaml；就是  **dataId**







### Nginx

![image-20200330142550862](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200330142550862.png)

#### 启动

![image-20200330142854306](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200330142854306.png)

![image-20200405195107115](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200405195107115.png)