





## Eureka

![image-20200327171219159](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200327171219159.png)

#### 构建过程

1. 配置 pom 文件
2. 配置yml文件
3. 服务端加注解 @EnableEurekaServer；连接注册端用 @EnableEurekaClient

```
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```

```
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```



```yml
eureka:
  instance:
    # eureka服务端的实例名称
    # 单机 hostname: localhost
    hostname: localhost
#    hostname: eureka7001.com
    # Eureka客户端向服务端发送心跳的时间间隔,单位为秒(默认是30秒)
#    lease-renewal-interval-in-seconds: 1
    # Eureka服务端在收到最后一次心跳后等待时间上限 ,单位为秒(默认是90秒),超时剔除服务
#    lease-expiration-duration-in-seconds: 2
#  server:
    # 禁用自我保护,保证不可用服务被及时删除
#    enable-self-preservation: true
#    eviction-interval-timer-in-ms: 2000
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      # 单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 相互注册
#      defaultZone: http://eureka7002.com:7002/eureka/
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```





![image-20200327183043173](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200327183043173.png)



![image-20200327184234240](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200327184234240.png)





## 需要注意的点

* restTemplate 上面要加上 @loadBalance 注解实现负载均衡
* 调用服务名即可，切记服务名前面要加上访问协议；如 http://

```
 public static final String PAYMENT_URL = "http://CLOUD-PROVIDER-SERVICE";
```

* 加注解 @EnableDiscoveryClient；可以用 @Resource 得到 DiscoveryClient；得到在注册中心所有服务的信息，包括端口号，ip，服务名
* 同个服务开集群；name 名字不需要变化，因为是同一个服务；交给负载均衡即可



* Eureka 实现的是 CAP 中的 AP 分支
* 某一时刻某个微服务不可用了，Eureka 不会立刻清理，依旧会对该服务的信息进行保存





## 关闭自我保护

```yml
eureka:
  server:
    # 禁用自我保护,保证不可用服务被及时删除
    enable-self-preservation: true
    # 时间两秒
    eviction-interval-timer-in-ms: 2000
```

### 客户端发送心跳包设置

```yml
eureka:
  instance:
    # Eureka客户端向服务端发送心跳的时间间隔,单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    # Eureka服务端在收到最后一次心跳后等待时间上限 ,单位为秒(默认是90秒),超时剔除服务
    lease-expiration-duration-in-seconds: 2
```