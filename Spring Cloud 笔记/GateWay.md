# **Spring Cloud Gateway**

Spring Cloud Gateway 旨在为微服务架构提供一种**简单且有效的 API 路由的管理方式**，并基于 **Filter 的方式**提供网关的基本功能，例如说安全认证、监控、限流等等

> - 基于 Java 8 编码
> - 基于 Spring Framework 5 + Project Reactor + Spring Boot 2.0 构建
> - 支持动态路由，能够匹配任何请求属性上的路由
> - 支持内置到 Spring Handler 映射中的路由匹配
> - 支持基于 HTTP 请求的路由匹配（Path、Method、Header、Host 等等）
> - 集成了 Hystrix 断路器
> - 过滤器作用于匹配的路由
> - 过滤器可以修改 HTTP 请求和 HTTP 响应（增加/修改 Header、增加/修改请求参数、改写请求 Path 等等）
> - 支持 Spring Cloud DiscoveryClient 配置路由，与服务发现与注册配合使用
> - 支持限流

实现一个API网关接管所有的入口流量，类似Nginx的作用，将所有用户的请求转发给后端的服务器，但网关做的不仅仅只是简单的转发，也会针对流量做一些扩展，比如**鉴权、限流、权限、熔断、协议转换、错误码统一、缓存、日志、监控、告警**等，这样将通用的逻辑抽出来，由网关统一去做，业务方也能够更专注于业务逻辑，提升迭代的效率

```
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```

在 `spring-cloud-starter-gateway` 依赖中，会引入 WebFlux、Reactor、Netty 等等依赖

```yml
server:
  port: 8888

spring:
  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/blog
          filters:
            - StripPrefix=1
        - id: oschina # 路由的编号
          uri: https://www.oschina.net # 路由的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/oschina
          filters: # 过滤器，对请求进行拦截，实现自定义的功能，对应 FilterDefinition 数组
            - StripPrefix=1
```

- ID：编号，路由的唯一标识。

- URI：路由指向的目标 URI，即请求最终被转发的目的地。

  > 例如说，这里配置的 `http://www.iocoder.cn` 或 `https://www.oschina.net`，就是被转发的地址。

- [Predicate](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/AsyncPredicate.java)：谓语，作为路由的匹配条件。Gateway 内置了多种 Predicate 的[实现](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/)，提供了多种请求的匹配条件，比如说基于请求的 Path、Method 等等。

  > 例如说，这里配置的 [`Path`](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/PathRoutePredicateFactory.java) 匹配请求的 Path 地址。

- [Filter](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/GatewayFilter.java)：过滤器，对请求进行拦截，实现自定义的功能。Gateway 内置了多种 Filter 的[实现](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/)，提供了多种请求的处理逻辑，比如说限流、熔断等等。

  > 例如说，这里配置的 [`StripPrefix`](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/StripPrefixGatewayFilterFactory.java) 会将请求的 Path 去除掉前缀。可能有点不好理解，我们以第一个 Route 举例子，假设我们请求 http://127.0.0.1:8888/blog 时：
  >
  > - 如果**有**配置 `StripPrefix` 过滤器，则转发到的最终 URI 为 [http://www.iocoder.cn](http://www.iocoder.cn/)，正确返回首页
  > - 如果**未**配置 `StripPrefix` 过滤器，转发到的最终 URI 为 http://www.iocoder.cn/blog，错误返回 404

![image-20200422204202226](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422204202226.png)

- ① Gateway 接收客户端请求。

- ② 请求与 Predicate 进行匹配，获得到对应的 Route。匹配成功后，才能继续往下执行。

- ③ 请求经过 Filter 过滤器链，**执行前置（prev）处理逻辑**。

  > 例如说，修改请求头信息等。

- ④ 请求被 **Proxy Filter** **转发至目标 URI**，并最终获得响应。

  > 一般来说，目标 URI 是被代理的微服务，如果是在 Spring Cloud 架构中。

- ⑤ **响应经过 Filter 过滤器链，执行后置（post）处理逻辑**。

- ⑥ **Gateway 返回响应给客户端**。



使用 Gateway 提供的与 Spring Cloud 注册中心的集成，从注册中心获取服务列表，并以服务名作为目标 URI 来**自动**创建**动态路由**。



# Route Predicate

Gateway **内置**了多种 Route Predicate 实现，将请求匹配到对应的 Route 上。并且，多个 Route Predicate 是可以**组合**实现，满足我们绝大多数的路由匹配规则。

![image-20200422222116760](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422222116760.png)

一般情况下，我们主要使用 `Path` 和 `Host` 两个 Predicate，甚至只使用 `Path`

# 灰度发布

Gateway 内置的 [**Weight** Route Predicate](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/WeightRoutePredicateFactory.java) 实现非常有趣，提供了基于**权重**的匹配条件，为网关实现[**灰度发布**](http://www.iocoder.cn/Fight/Micro-service-deployment-blue-green-deployment-rolling-deployment-grayscale-release-canary-release/?self)提供了基础。

> 灰度发布（又名金丝雀发布）是指在黑与白之间，能够平滑过渡的一种发布方式。
>
> 在其上可以进行 A/B testing，即让一部分用户继续用产品特性 A，一部分用户开始用产品特性 B，如果用户对 B 没有什么反对意见，那么逐步扩大范围，把所有用户都迁移到 B 上面来。灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。

![image-20200422222307679](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422222307679.png)

```yml
spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: user-service-prod
          uri: http://www.iocoder.cn
          predicates:
            - Path=/**
            - Weight=user-service, 90
        - id: user-service-canary
          uri: https://www.oschina.net
          predicates:
            - Path=/**
            - Weight=user-service, 10
```

使用 `Weight` 匹配条件，设置**不同**的权重条件。其中，第一个参数为权重**分组**，需要配置成**相同**，一般和服务名相同即可；第二个参数为权重**比例**。

这里我们配置的 `uri` 暂时不是胖友可能希望的 `lb://user-service`，这是为什么呢？Gateway 的权重路由仅仅提供了灰度发布的基础，实际还是需要做一定的改造，例如说：

- 第一，Spring Cloud 微服务在注册到注册中心时，需要在元数据中带上**版本号**。例如说，`version = 1.0.0`、`version = 1.1.0`。

- 第二，Gateway 在 `lb://serviceId` 负载均衡选择请求的服务实例时，需要增加基于**版本号**的选择规则。

  > 友情提示：目前常用的负载均衡组件 Ribbon 暂未提供基于版本的负载均衡规则，需要胖友自己去略微拓展下，并不复杂噢。

# Route Filter

![image-20200422222800551](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422222800551.png)