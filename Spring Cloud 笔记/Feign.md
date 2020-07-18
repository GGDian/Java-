## Feign

Feign 并非一定要在 Spring Cloud 下使用，单独使用也是没问题的。

**Feign的一个关键机制就是使用了动态代理**。

- 首先，如果你对某个接口定义了 `@FeignClient` 注解，**Feign 就会针对这个接口创建一个动态代理**。
- 接着你要是调用那个接口，本质就是会**调用 Feign 创建的动态代理**，这是核心中的核心。
- Feign的动态代理会根据你在接口上的 `@RequestMapping` 等注解，来**动态构造出你要请求的服务的地址**。
- 最后针对这个地址，发起请求、解析响应。



### Feign 和 Ribbon 的区别？

Ribbon 和 Feign 都是使用于调用用其余服务的，不过方式不同。

- 启动类用的注解不同。
  - Ribbon 使用的是 `@RibbonClient` 。
  - Feign 使用的是 `@EnableFeignClients` 。
- 服务的指定位置不同。
  - Ribbon 是在 `@RibbonClient` 注解上设置。
  - Feign 则是在定义声明方法的接口中用 `@FeignClient` 注解上设置。
- 调使用方式不同。
  - Ribbon 需要自己构建 Http 请求，模拟 Http 请求而后用 RestTemplate 发送给其余服务，步骤相当繁琐。
  - Feign 采使用接口的方式，将需要调使用的其余服务的方法定义成声明方法就可，不需要自己构建 Http 请求。不过要注意的是声明方法的注解、方法签名要和提供服务的方法完全一致。



```
OpenFeign` 也是运行在消费者端的，使用 `Ribbon` 进行负载均衡，所以 `OpenFeign` 直接内置了 `Ribbon
```



# 芋道

[Feign](https://github.com/OpenFeign/feign) 是由 Netflix 开源的**声明式的 HTTP 客户端**

[Spring Cloud OpenFeign](https://github.com/spring-cloud/spring-cloud-openfeign) 组件，将 Feign 集成到 Spring Cloud 体系中，实现服务的**声明式 HTTP 调用**。

相比使用 RestTemplate 实现服务的调用，**Feign 简化了代码的编写，提高了代码的可读性**，大大提升了开发的效率。

Spring Cloud OpenFeign 进一步将 Feign 和 [Ribbon 整合](https://github.com/spring-cloud/spring-cloud-openfeign/blob/2.2.x/spring-cloud-openfeign-core/src/main/java/org/springframework/cloud/openfeign/loadbalancer/FeignLoadBalancerAutoConfiguration.java)，提供了负载均衡的功能。另外，Feign 自身已经完成和 [Hystrix 整合](https://github.com/OpenFeign/feign/tree/master/hystrix)，**提供了服务容错的功能**。

```
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

这里我们没有主动引入 [`spring-cloud-netflix-ribbon`](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-netflix-ribbon) 依赖，因为 `spring-cloud-starter-alibaba-nacos-discovery` 和 `spring-cloud-starter-openfefign` 默认都引入了它

## 自定义 Feign 配置

在自定义 Feign 配置的时候，会有**全局**和**客户端**两种级别。相比来说，**客户端**级别是更细粒度的配置。针对每个服务，Spring Cloud OpenFeign 会创建一个 Feign 客户端，并且使用**服务名**作为 **Feign 客户端的名字**。

实现 Feign 自定义配置，可以通过**配置文件**和 **Spring JavaConfig** 两种方式。

在 Feign 中，定义了四种[日志级别](https://github.com/OpenFeign/feign/blob/master/core/src/main/java/feign/Logger.java#L128-L148)：

- `NONE`：不打印日志
- `BASIC`：只打印基本信息，包括请求方法、请求地址、响应状态码、请求时长
- `HEADERS`：在 `BASIC` 基础信息的基础之上，增加请求头、响应头
- `FULL`：打印完整信息，包括请求和响应的所有信息。

```yml
logging:
  level:
    cn.iocoder.springcloud.labx03.feigndemo.consumer.feign: DEBUG

feign:
  # Feign 客户端配置，对应 FeignClientProperties 配置属性类
  client:
    # config 配置项是 Map 类型。key 为 Feign 客户端的名字，value 为 FeignClientConfiguration 对象
    config:
      # 全局级别配置
      default:
        logger-level: BASIC
      # 客户端级别配置
      demo-provider:
        logger-level: FULL
```

① 在 `logging.level` 配置项下，添加自定义 Feign 接口所在包的日志级别为 `DEBUG`。Feign 定义的四种日志级别，针对的是日志内容的级别。最终打印日志时，**Feign 是调用日志组件的 `DEBUG` 级别打印日志**，所以这里需要设置为 `DEBUG` 级别。

② 在 `feign.client` 配置下，设置 Feign 客户端的配置，对应 [FeignClientProperties](https://github.com/spring-cloud/spring-cloud-openfeign/blob/master/spring-cloud-openfeign-core/src/main/java/org/springframework/cloud/openfeign/FeignClientProperties.java) 配置属性类。其中 `config` 配置项，**可以设置每个 Feign 客户端的配置**，并且 ***key* 为 Feign 客户端的名字**，*value* 为 [FeignClientConfiguration](https://github.com/spring-cloud/spring-cloud-openfeign/blob/master/spring-cloud-openfeign-core/src/main/java/org/springframework/cloud/openfeign/FeignClientProperties.java#L90-L238) 对象。

- `default` 为特殊的 *key*，用于**全局级别**的配置。
- `logger-level` 配置项，设置 Feign 的日志级别。

总结来说，**这里配置名字为 `demo-provider` 的 Feign 客户端的日志级别为 `FULL`**，全局级别的 Feign 客户端的日志级别为 `BASIC`。



## Spring JavaConfig 方式

![image-20200422145524241](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422145524241.png)



## 复杂参数

**额外**增加 `GET`、`POST` 类型请求的复杂参数的示例接口

### ProviderController

```java
@GetMapping("/get_demo")
public DemoDTO getDemo(DemoDTO demoDTO) {
    return demoDTO;
}

@PostMapping("/post_demo")
public DemoDTO postDemo(@RequestBody DemoDTO demoDTO) {
    return demoDTO;
}
```

```java
@GetMapping("/get_demo") // GET 方式一，最推荐
DemoDTO getDemo(@SpringQueryMap DemoDTO demoDTO);

@GetMapping("/get_demo") // GET 方式二，相对推荐
DemoDTO getDemo(@RequestParam("username") String username, @RequestParam("password") String password);

@GetMapping("/get_demo") // GET 方式三，不推荐
DemoDTO getDemo(@RequestParam Map<String, Object> params);

@PostMapping("/post_demo") // POST 方式
DemoDTO postDemo(@RequestBody DemoDTO demoDTO);
```

默认情况下，Feign **针对 POJO 类型**的参数，即使我们声明为 `GET` 类型的请求，也会自动转换成 `POST` 类型的请求。

`@SpringQueryMap` 注解的作用，相当于 Feign 的 [`@QueryMap`](https://github.com/OpenFeign/feign/blob/master/core/src/main/java/feign/QueryMap.java) 注解，将 POJO 对象转换成 [QueryString](https://en.wikipedia.org/wiki/Query_string)

## Feign 单独使用

我们**大多数**是通过 Feign 调用从 **Ribbon 负载均衡**选择的服务实例，而 Ribbon 是通过注册中心获取到的服务实例列表。

```java
@FeignClient(name = "iocoder", url = "http://www.iocoder.cn") // 注意，保持 name 属性和 url 属性的 host 是一致的。
public interface DemoProviderFeignClient {

//    @GetMapping("/echo")
//    String echo(@RequestParam("name") String name);

    @GetMapping("/") // 请求首页
    String echo(@RequestParam("name") String name);

}
```

## HTTP 客户端

默认情况下，Feign 通过 **JDK 自带的 HttpURLConnection** 封装了 [Client.Default](https://github.com/OpenFeign/feign/blob/819b2df8c54d9266abf4cde9b17ab7890ed95cc6/core/src/main/java/feign/Client.java#L58-L232)，**实现 HTTP 调用的客户端**。因为 HttpURLConnection **缺少对 HTTP 连接池的支持**，所以性能较低，在并发到达一定量级后基本会出现。

### 使用 Apache HttpClient

```
<!-- 引入 Feign Apache HttpClient 依赖 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

```yml
feign:
  # Feign Apache HttpClient 配置项，对应 FeignHttpClientProperties 配置属性类
  httpclient:
    enabled: true # 是否开启。默认为 true
    max-connections: 200 # 最大连接数。默认为 200
    max-connections-per-route: 50 # 每个路由的最大连接数。默认为 50。router = host + port
```

通过 `feign.httpclient` 配置项，我们可以开启 Feign Apache HttpClient，并进行自定义配置。在 [FeignHttpClientProperties](https://github.com/spring-cloud/spring-cloud-openfeign/blob/master/spring-cloud-openfeign-core/src/main/java/org/springframework/cloud/openfeign/support/FeignHttpClientProperties.java) 配置属性类中，还有其它配置项，胖友可以简单看看。

不过有一点要注意，虽然说 `feign.httpclient.enable` 默认为 `true` 开启，但是还是需要引入 `feign-httpclient` 依赖，才能创建 ApacheHttpClient 对象。

![image-20200422182503324](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422182503324.png)

### 使用 OkHttpClient

```
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

```yml
feign:
  httpclient:
    enabled: false # 是否开启。默认为 true
  okhttp:
    enabled: true # 是否开启。默认为 false
```

因为 `feign.httpclient.enabled` 配置项默认为 `true`，所以需要手动设置成 `false`，避免使用了 Feign Apache HttpClient

![image-20200422182437510](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200422182437510.png)

OkHttp 和 Apache HttpClient 在性能方面是基本**接近**的，有资料说 OkHttp 好一些，也有资料说 HttpClient 好一些。喜欢哪个用哪个

## 请求重试

Feign 和 Ribbon 都有请求重试的功能，两者都启用该功能的话，会产生冲突的问题。因此，有且只能启动一个的重试。**目前比较推荐的是使用 Ribbon 来提供重试**

在 Spring Cloud OpenFeign 中，默认创建的是 [NEVER_RETRY](https://github.com/OpenFeign/feign/blob/master/core/src/main/java/feign/Retryer.java#L99-L113) **不进行重试**。如此，我们**只需要配置 Ribbon 的重试**功能即可。

## Feign 与 RestTemplate 的对比

从开发效率、可维护性的角度来说，**Feign** 更加有优势。
从执行性能、灵活性的角度来说，**RestTemplate** 更加有优势。

个人推荐使用 **Feign 为主，RestTemplate 为辅**：

- 相比来说，开发效率、可维护性非常重要，要保证开发的体验。
- 执行性能的问题，因为 Feign 多一层 JDK 动态代理，所以会差一些。不过 HTTP 调用的整体性能的大头在网络传输和服务端的执行时间，所以 Feign 和 RestTemplate 的性能差距可以相对忽略。
- 灵活性的问题，99.99% 的情况下，Feign 都能够实现或者相对绕的实现；无法实现的情况下，在考虑采用 RestTemplate 进行实现。

# 原理

 是一个**http请求调用**的轻量级框架，可以以**Java接口注解**的方式**调用Http请求**，而不用像Java中通过封装HTTP请求报文的方式直接调用

Feign**通过处理注解，将请求模板化**，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的请求，这种请求相对而言比较直观。

在服务调用的场景中，我们经常调用基于Http协议的服务，而我们经常使用到的框架可能有**HttpURLConnection、Apache HttpComponnets、OkHttp3 、Netty**等等

Feign 默认底层通过JDK 的 `java.net.HttpURLConnection` 实现了`feign.Client`接口类,在每次发送请求的时候，**都会创建新的HttpURLConnection 链接**，这也就是为什么默认情况下Feign的性能很差的原因。可以通过拓展该接口，使用Apache HttpClient 或者OkHttp3等基于连接池的高性能Http客户端，我们项目内部使用的就是OkHttp3作为Http 客户端。