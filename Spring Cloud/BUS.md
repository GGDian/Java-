## BUS

### 服务端

* pom

  ```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  ```

* mq配置；服务暴露需要 pom 中添加 actuator 

  ```yml
  rabbitmq:
  	host: localhost
  	port: 5672
  	username: guest
  	password: guest
  #暴露bus 刷新配置的端点
  management:
  	endpoints:
  		web:
  			exposure:
  				include: 'bus-refresh'
  ```

* 客户端也需要加入mq配置
* 手动向服务端发送一个 post 请求手动刷新配置
* ConfigClient实例都监听MQ中同一个 topic (默认是springCloudBus)；当一个服务刷新数据的时候，它会把这个信息放入到 Topic 中，这样其他监听同一个 Topic 的服务就能得到通知，然后去更新自身的配置；



### 精准通知，不全部进行广播

* http://localhost:端口/actuator/bus-refresh/config-client:3355
* 即只通知端口3355的服务