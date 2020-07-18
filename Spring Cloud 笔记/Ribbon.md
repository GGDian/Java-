### Ribbon

主要提供客户侧的软件负载均衡算法

Spring Cloud Ribbon 是一个基于 **HTTP 和 TCP** 的**客户端负载均衡**工具，它基于 Netflix Ribbon 实现。通过 Spring Cloud 的封装，可以让我们轻松地将**面向服务的 REST 模版**请求自动转换成**客户端负载均衡的服务调用**。

比Nginx更注重的是**承担并发而不是请求分发**，可以直接感知后台动态变化**来指定分发策略**



默认的负载均衡算法是 Round Robin 算法，顺序向下轮询

- **`RoundRobinRule`**：轮询策略。`Ribbon` 默认采用的策略。若经过一轮轮询没有找到可用的 `provider`，其最多轮询 10 轮。若最终还没有找到，则返回 `null`。
- **`RandomRule`**: 随机策略，从所有可用的 `provider` 中随机选择一个。
- **`RetryRule`**: 重试策略。先按照 `RoundRobinRule` 策略获取 `provider`，若获取失败，则在指定的时限内重试。默认的时限为 500 毫秒。





- 首先，Ribbon 会从 **Eureka Client** 里**获取到对应的服务列表**。
- 然后，Ribbon 使用负载均衡算法获得使用的服务。
- 最后，Ribbon 调用对应的服务。



### 芋道

Ribbon 是一个进程间通信的客户端库

- 负载均衡
- 容错机制
- 支持 HTTP、TCP、UDP 等多种协议，并支持异步和响应式的调用方式
- 缓存与批处理

客户端级别的负载均衡可以有**更好的性能**，因为不需要多经过一层代理服务器。并且，服务端级别的负载均衡需要额外考虑代理服务的高可用，以及请求量较大时的负载压力。



> 在 Spring Cloud Common 项目中，定义了[LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java) 接口，作为通用的负载均衡客户端，提供从指定服务中选择一个实例、对指定服务发起请求等 API 方法。而想要集成到 Spring Cloud 体系的负载均衡的组件，需要提供对应的 **LoadBalancerClient** 实现类。

这种方法实现有缺点，当调用失败后需要自己**手动**写逻辑，使用 LoadBalancerClient 重新选择一个该服务的实例，后交给 RestTemplate 再发起调用

而加了 LoadBalance 注解的 restTemplate 可以**自动**通过 LoadBalancerClient 重新选择一个该服务的实例，再次发起调用

```java
  @Autowired
        private LoadBalancerClient loadBalancerClient;

        @GetMapping("/hello")
        public String hello(String name) {
            // 获得服务 `demo-provider` 的一个实例
            ServiceInstance instance = loadBalancerClient.choose("demo-provider");
            // 发起调用
            String targetUrl = instance.getUri() + "/echo?name=" + name;
            String response = restTemplate.getForObject(targetUrl, String.class);
            // 返回结果
            return "consumer:" + response;
        }
```

在默认配置下，**与 Nacos 整合，Ribbon 采用 [ZoneAvoidanceRule](https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/ZoneAvoidanceRule.java) 负载均衡策略**，在未配置所在区域的情况下，和**轮询**负载均衡策略是相对等价的。所以服务消费者 `demo-consumer` 调用服务提供者 `demo-provider` 时，顺序将请求分配给每个实例。

默认情况下，Ribbon 采用 ZoneAvoidanceRule 规则。因为大多数公司是单机房，所以一般只有一个 zone，而 ZoneAvoidanceRule 在仅有一个 zone 的情况下，会退化成轮询的选择方式，所以会和 RoundRobinRule 规则类似。

#### 自定义 Ribbon 配置

在自定义 Ribbon 配置的时候，会有**全局**和**客户端**两种级别。相比来说，**客户端**级别是更细粒度的配置。针对每个服务，Spring Cloud Netflix Ribbon 会创建一个 Ribbon 客户端，并且使用**服务名**作为 **Ribbon 客户端的名字**。

实现 Ribbon 自定义配置，可以通过**配置文件**和 **Spring JavaConfig** 两种方式。

####  配置文件

通过在配置文件中，添加 `{clientName}.ribbon.{key}={value}` 配置项，设置**指定名字**的 Ribbon 客户端的**指定属性**的**值**

```yml
demo-provider:  
	ribbon:    
		NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则全类名
```

#### Spring JavaConfig 方式

我们使用 **Spring JavaConfig** 的方式，实现 Ribbon **全局**和 **客户端**两种级别的自定义配置



对于 DefaultRibbonClientConfiguration 和 UserProviderRibbonClientConfiguration 两个配置类，我们并没有和 DemoConsumerApplication 启动类放在**一个包路径**下

`@RibbonClients` 注解，通过 `defaultConfiguration` 属性声明 Ribbon **全局**级别的自定义配置，通过 `value` 属性声明多个 Ribbon **客户端**级别的自定义配置。

`@RibbonClient` 注解，声明一个 Ribbon **客户端**级别的自定义配置，其中 `name` 属性用于设置 Ribbon 客户端的**名字**。

为了避免多个 Ribbon 客户端级别的配置类创建的 Bean 之间互相冲突，Spring Cloud Netflix Ribbon 通过 [SpringClientFactory](https://github.com/wojesen/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/ribbon/SpringClientFactory.java) 类，**为每一个 Ribbon 客户端创建一个 Spring 子上下文**

不过这里要注意，因为 DefaultRibbonClientConfiguration 和 UserProviderRibbonClientConfiguration 都创建了 IRule Bean，而 DefaultRibbonClientConfiguration 是在 Spring **父上下文**生效，会和 UserProviderRibbonClientConfiguration 所在的 Spring **子上下文**共享。这样就导致从 Spring 获取 IRule Bean 时，存在**两个**而不知道选择哪一个。因此，我们声明 UserProviderRibbonClientConfiguration 创建的 IRule Bean 为 `@Primary`，优先使用它。

```java
// RibbonConfiguration.java
@Configuration
@RibbonClients( value = {@RibbonClient(name = "demo-provider", configuration = 			 				UserProviderRibbonClientConfiguration.class)  // 客户端级别的配置
                },
              defaultConfiguration = DefaultRibbonClientConfiguration.class // 全局配置
              )
public class RibbonConfiguration {}
// DefaultRibbonClientConfiguration.java
@Configuration
public class DefaultRibbonClientConfiguration {    
    @Bean    
    public IRule ribbonDefaultRule() {        
        return new RoundRobinRule();   
    }
}
// UserProviderRibbonClientConfiguration.java
@Configuration
public class UserProviderRibbonClientConfiguration {    
    @Bean    
    @Primary    
    public IRule ribbonCustomRule() {        
        return new RandomRule();    
    }
}
```

#### 实践建议

- 对于 Ribbon **客户端**级别的自定义配置，推荐使用配置文件的方式，简单方便好管理。在配置文件的方式无法满足的情况下，使用 Spring JavaConfig 的方式作为补充。不过绝大多数场景下，都基本不需要哈~
- 对于 Ribbon **全局**级别的自定义配置，暂时只能使用 Spring JavaConfig 的方式
- 配置文件方式的**优先级**高于 Spring JavaConfig 方式，客户端级别的**优先级**高于全局级别



#### Nacos 自定义负载均衡规则

在 [Spring Cloud Alibaba Nacos Discovery](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-nacos-discovery) 组件，在和 Ribbon 集成时，提供了自定义负载均衡规则 [NacosRule](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-discovery/src/main/java/com/alibaba/cloud/nacos/ribbon/NacosRule.java)。规则如下：

- 第一步，获得**健康**的方服务实例列表
- 第二步，优先选择**相同 Nacos 集群**的服务实例列表，保证**高性能**。如果选择不到，则允许使用其它 Nacos 集群的服务实例列表，保证**高可用**
- 第三步，从服务实例列表按照**权重**进行**随机**，选择一个服务实例返回

```yml
demo-provider:  
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule

```

#### 饥饿加载

默认配置下，Ribbon 客户端是在**首次**请求服务时，才创建该服务的对应的 Ribbon 客户端。

**好处**是项目在启动的时候，能够更加快速，因为 Ribbon 客户端创建时，需要从注册中心获取服务的实例列表，需要有网络请求的消耗。

**坏处**是首次请求服务时，因为需要 Ribbon 客户端的创建，会导致请求比较慢，严重情况下会导致请求超时。

因此，Spring Cloud Netflix Ribbon 提供了 `ribbon.eager-load` 配置项，允许我们在项目启动时，**提前**创建 Ribbon 客户端。翻译成中文就是**“饥饿加载”**。



```yml
ribbon:  
# Ribbon 饥饿加载配置项，对应 RibbonEagerLoadProperties 配置类  
	eager-load: enabled: true # 是否开启饥饿加载。默认为 false 不开启    
	clients: user-provider # 开启饥饿加载的 Ribbon 客户端名字。如果有多个，使用 , 逗号分隔。
```

在生产环境下，一定要开启 Ribbon 饥饿加载。



#### HTTP 客户端

Ribbon **只负责**服务实例的选择，提供负载均衡的功能，而服务的 HTTP 调用则是交给 **RestTemplate** 来完成。实际上，Ribbon 也是提供 HTTP 调用功能的。

```yml
ribbon:
#  okhttp:
#    enabled: true # 设置使用 OkHttp，对应 OkHttpRibbonConfiguration 配置类
  restclient:
    enabled: true # 设置使用 Jersey Client，对应 RestClientRibbonConfiguration 配置类
#  httpclient:
#    enabled: true # 设置使用 Apache HttpClient，对应 HttpClientRibbonConfiguration 配置类
```



#### 请求重试

在 Spring Cloud 中，提供 `spring.cloud.loadbalancer.retry` 配置项，通过设置为 `true`，开启负载均衡的重试功能

```yml
ribbon:
  restclient:
    enabled: true # 设置使用 Jersey Client，对应 RestClientRibbonConfiguration 配置类

demo-provider:
  ribbon:
    ConnectTimeout: 1000 # 请求的连接超时时间，单位：毫秒。默认为 1000
    ReadTimeout: 1 # 请求的读取超时时间，单位：毫秒。默认为 1000
    OkToRetryOnAllOperations: true # 是否对所有操作都进行重试，默认为 false。
    MaxAutoRetries: 0 # 对当前服务的重试次数，默认为 0 次。
    MaxAutoRetriesNextServer: 1 # 重新选择服务实例的次数，默认为 1 次。注意，不包含第 1 次哈。
```

① 设置 `ribbon.restclient.enable` 配置项为 `true`，因为我们通过 Ribbon 实现请求重试，所以需要**使用 Ribbon 内置的 HTTP 客户端**进行请求服务。

② `ConnectTimeout` 和 `ReadTimeout` 两个配置项，**用于设置请求的连接和超时时间**。

这里，我们设置 `ReadTimeout` 配置项为 1，用于模拟请求服务超时，从而演示请求重试的效果。

③ `OkToRetryOnAllOperations`、`MaxAutoRetries`、`MaxAutoRetriesNextServer` 三个配置项，设置请求重试相关的参数。具体每个参数的解释，看下配置文件中的注释哈。

`MaxAutoRetries` 和 `MaxAutoRetriesNextServer` 两个配置项可能略微难以理解，艿艿再简单描述下。

- 第一步，在使用 Ribbon 选择一个服务实例后，如果请求失败，重试 `MaxAutoRetries` 请求直到成功。
- 第二步，如果经过 `1 + MaxAutoRetries` 次，请求一个服务实例还是失败，重新使用 Ribbon 选择一个新的服务实例，重复第一步的过程。最多可以重新选择 `MaxAutoRetriesNextServer` 次新的服务实例。

也就是说，在服务实例足够的情况下，最多会发起 `(1 + MaxAutoRetries) * (1 + MaxAutoRetriesNextServer)` 请求。不过一般情况下，推荐设置 `MaxAutoRetries` 为 0，不重试当前实例。

在 ② 和 ③ 中的参数，如果想要**全局**级别的配置，可以添加到 `ribbon` 配置项下。例如说：

```yml
ribbon:
    ConnectTimeout: 1000 # 请求的连接超时时间，单位：毫秒。默认为 1000
    ReadTimeout: 1 # 请求的读取超时时间，单位：毫秒。默认为 1000
    OkToRetryOnAllOperations: true # 是否对所有操作都进行重试，默认为 false。
    MaxAutoRetries: 0 # 对当前服务的重试次数，默认为 0 次。
    MaxAutoRetriesNextServer: 1 # 重新选择服务实例的次数，默认为 1 次。注意，不包含第 1 次哈。
```

