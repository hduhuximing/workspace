## [《分布式可配置化服务框架m-RPC》](http://www.xuxueli.com/xxl-rpc/#/?id=《分布式服务框架xxl-rpc》)

## [一、简介](http://www.xuxueli.com/xxl-rpc/#/?id=一、简介)

### [1.1 概述](http://www.xuxueli.com/xxl-rpc/#/?id=_11-概述)

m-RPC 是一个分布式服务框架，提供稳定高性能的RPC远程服务调用功能。拥有"高性能、分布式、注册中心、负载均衡、服务治理"等特性。现已开放源代码，开箱即用。

### [1.2 特性](http://www.xuxueli.com/xxl-rpc/#/?id=_12-特性)

- 1、快速接入：接入步骤非常简洁，两分钟即可上手；
- 2、服务透明：系统完整的封装了底层通信细节，开发时调用远程服务就像调用本地服务，在提供远程调用能力时不损失本地调用的语义简洁性；
- 3、多调用方案：支持 SYNC、ONEWAY、FUTURE、CALLBACK 等方案；
- 4、多通讯方案：支持 TCP 和 HTTP 两种通讯方式进行服务调用；其中 TCP 提供可选方案 NETTY 或 MINA ，HTTP 提供可选方案 NETTY_HTTP 或 Jetty；
- 5、多序列化方案：支持 HESSIAN、HESSIAN1、PROTOSTUFF、KRYO、JACKSON 等试试 以及自实现的   [jfire-codejson](https://gitee.com/eric_ds/jfire-codejson) 
- 6、负载均衡/软负载：提供丰富的负载均衡策略，包括：轮询、随机、LRU、LFU、一致性HASH等；
- 7、注册中心：可选组件，支持服务注册并动态发现；可选择不启用，直接指定服务提供方机器地址通讯；选择启用时，内置可选方案：“XXL-REGISTRY 轻量级注册中心”（推荐）、“ZK注册中心”、“Local注册中心”等；
- 8、服务治理：提供服务治理中心，可在线管理注册的服务信息，如服务锁定、禁用等；
- 9、服务监控：可在线监控服务调用统计信息以及服务健康状况等（计划中）；
- 10、容错：服务提供方集群注册时，某个服务节点不可用时将会自动摘除，同时消费方将会移除失效节点将流量分发到其余节点，提高系统容错能力。
- 11、解决1+1问题：传统分布式通讯一般通过nginx或f5做集群服务的流量负载均衡，每次请求在到达目标服务机器之前都需要经过负载均衡机器，即1+1，这将会把流量放大一倍。而XXL-RPC将会从消费方直达服务提供方，每次请求直达目标机器，从而可以避免上述问题；
- 12、高兼容性：得益于优良的兼容性与模块化设计，不限制外部框架；除 spring/springboot 环境之外，理论上支持运行在任何Java代码中，甚至main方法直接启动运行；
- 13、泛化调用：服务调用方不依赖服务方提供的API；

### [1.3 背景](http://www.xuxueli.com/xxl-rpc/#/?id=_13-背景)

RPC（Remote Procedure Call Protocol，远程过程调用），调用远程服务就像调用本地服务，在提供远程调用能力时不损失本地调用的语义简洁性；

一般公司，尤其是大型互联网公司内部系统由上千上万个服务组成，不同的服务部署在不同机器，跑在不同的JVM上，此时需要解决两个问题：

- 1、如果我需要依赖别人的服务，但是别人的服务在远程机器上，我该如何调用？
- 2、如果其他团队需要使用我的服务，我该怎样发布自己的服务供他人调用？

“XXL-RPC”可以高效的解决这个问题：

- 1、如何调用：只需要知晓远程服务的stub和地址，即可方便的调用远程服务，同时调用透明化，就像调用本地服务一样简单；
- 2、如何发布：只需要提供自己服务的stub和地址，别人即可方便的调用我的服务，在开启注册中心的情况下服务动态发现，只需要提供服务的stub即可；

## [二、快速入门（springboot版本）](http://www.xuxueli.com/xxl-rpc/#/?id=二、快速入门（springboot版本）)

### [2.1 准备工作](http://www.xuxueli.com/xxl-rpc/#/?id=_21-准备工作)

- 1、编译项目

  源码目录介绍：

  - /doc
  - /xxl-rpc-core ：核心依赖；
  - /xxl-rpc-samples ：示例项目；
    - /xxl-rpc-sample-frameless ：无框架版本示例；
    - /xxl-rpc-sample-springboot ：springboot版本示例；
      - /xxl-rpc-sample-springboot-api ：公共API接口
      - /xxl-rpc-sample-springboot-client ：服务消费方 invoker 调用示例；
      - /xxl-rpc-sample-springboot-server ：服务提供方 provider 示例;
    - / xxl-rpc-sample-jfinal ：jfinal版本示例；

### [2.2 配置部署 “分布式服务注册中心XXL-REGISTRY”](http://www.xuxueli.com/xxl-rpc/#/?id=_22-配置部署-分布式服务注册中心xxl-registry)

推荐使用 "XXL-REGISTRY" 作为注册中心。可前往 XXL-REGISTRY (https://github.com/xuxueli/xxl-registry ) 查看部署文档。非常轻量级，一分钟可完成部署工作。

### [2.3 项目中使用XXL-RPC](http://www.xuxueli.com/xxl-rpc/#/?id=_23-项目中使用xxl-rpc)

以示例项目 “xxl-rpc-sample-springboot” 为例讲解；

#### [2.3.1 开发“服务API”](http://www.xuxueli.com/xxl-rpc/#/?id=_231-开发服务api)

开发RPC服务的 “接口 / interface” 和 “数据模型 / DTO”；

```undefined
可参考如下代码：
com.xxl.rpc.sample.api.DemoService
com.xxl.rpc.sample.api.dto.UserDTO
```

#### [2.3.2 配置开发“服务提供方”](http://www.xuxueli.com/xxl-rpc/#/?id=_232-配置开发服务提供方)

- 1、配置 “maven依赖”：

需引入：XXL-RPC核心依赖 + 公共API接口依赖

```undefined
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-rpc-core</artifactId>
    <version>${parent.version}</version>
</dependency>
```

- 2、配置“服务提供方 ProviderFactory”

```undefined
// 参考代码位置：com.xxl.rpc.sample.server.conf.XxlRpcProviderConfig

@Bean
public XxlRpcSpringProviderFactory xxlRpcSpringProviderFactory() {

    XxlRpcSpringProviderFactory providerFactory = new XxlRpcSpringProviderFactory();
    providerFactory.setPort(port);
    providerFactory.setServiceRegistryClass(XxlRegistryServiceRegistry.class);
    providerFactory.setServiceRegistryParam(new HashMap<String, String>(){{
        put(XxlRegistryServiceRegistry.XXL_REGISTRY_ADDRESS, address);
        put(XxlRegistryServiceRegistry.ENV, env);
    }});

    logger.info(">>>>>>>>>>> xxl-rpc provider config init finish.");
    return providerFactory;
}
```

| ProviderFactory 参数 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| netType              | 服务通讯方案，可选范围：NETTY（默认）、MINA、NETTY_HTTP、JETTY； |
| serialize            | 序列化方案，可选范围: HESSIAN（默认）、HESSIAN1、PROTOSTUFF、KRYO、JACKSON； |
| ip                   | 服务方IP，为空自动获取机器IP，支持手动指定；                 |
| port                 | 服务方端口，默认 7080                                        |
| accessToken          | 服务鉴权Token，非空时生效；                                  |
| serviceRegistryClass | 服务注册中心，可选范围：LocalServiceRegistry.class、ZkServiceRegistry.class；支持灵活自由扩展； |
| serviceRegistryParam | 服务注册中心启动参数，参数说明可参考各注册中心实现的 start() 的方法注释； |

- 3、开发“服务实现类”

实现 “服务API” 的接口，开发业务逻辑代码；

```undefined
可参考如下代码：
com.xxl.rpc.sample.api.DemoService

注意：
1、添加 “@Service” 注解：被Spring容器扫描识别为SpringBean；
2、添加 “@XxlRpcService” 注解：被 “XXL-RPC” 的 ProviderFactory 扫描识别，进行Provider服务注册，如果开启注册中心同时也会进行注册中心服务注册； 
```

| “@XxlRpcService” 注解参数 | 说明                                                     |
| ------------------------- | -------------------------------------------------------- |
| version                   | 服务版本，默认空；可据此区分同一个“服务API” 的不同版本； |

#### [2.3.3 配置开发“服务消费方”](http://www.xuxueli.com/xxl-rpc/#/?id=_233-配置开发服务消费方)

- 1、配置 “maven依赖”：

需引入：XXL-RPC核心依赖 + 公共API接口依赖

```undefined
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-rpc-core</artifactId>
    <version>${parent.version}</version>
</dependency>
```

- 2、配置“服务消费方 InvokerFactory”

```undefined
// 参考代码位置：com.xxl.rpc.sample.client.conf.XxlRpcInvokerConfig

@Bean
public XxlRpcSpringInvokerFactory xxlJobExecutor() {

    XxlRpcSpringInvokerFactory invokerFactory = new XxlRpcSpringInvokerFactory();
    invokerFactory.setServiceRegistryClass(XxlRegistryServiceRegistry.class);
    invokerFactory.setServiceRegistryParam(new HashMap<String, String>(){{
        put(XxlRegistryServiceRegistry.XXL_REGISTRY_ADDRESS, address);
        put(XxlRegistryServiceRegistry.ENV, env);
    }});

    logger.info(">>>>>>>>>>> xxl-rpc invoker config init finish.");
    return invokerFactory;
}
```

| InvokerFactory 参数  | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| serviceRegistryClass | 服务注册中心，可选范围：LocalServiceRegistry.class、ZkServiceRegistry.class；支持灵活自由扩展； |
| serviceRegistryParam | 服务注册中心启动参数，参数说明可参考各注册中心实现的 start() 的方法注释； |

- 3、注入并实用远程服务

```undefined
// 参考代码位置：com.xxl.rpc.sample.client.controller.IndexController

@XxlRpcReference
private DemoService demoService;

……
UserDTO user = demoService.sayHi(name);
……
```

| “@XxlRpcReference” 注解参数 | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| netType                     | 服务通讯方案，可选范围：NETTY（默认）、MINA、NETTY_HTTP、JETTY； |
| serializer                  | 序列化方案，可选范围: HESSIAN（默认）、HESSIAN1、PROTOSTUFF、KRYO、JACKSON； |
| address                     | 服务远程地址，ip:port 格式；选填；非空时将会优先实用该服务地址，为空时会从注册中心服务地址发现； |
| accessToken                 | 服务鉴权Token，非空时生效；                                  |
| version                     | 服务版本，默认空；可据此区分同一个“服务API” 的不同版本；     |
| timeout                     | 服务超时时间，单位毫秒；                                     |
| callType                    | 请求类型，可选范围：SYNC（默认）、ONEWAY、FUTURE、CALLBACK； |

#### [2.3.4 测试](http://www.xuxueli.com/xxl-rpc/#/?id=_234-测试)

```undefined
// 参考代码位置：com.xxl.rpc.sample.client.controller.IndexController
```

代码中将上面配置的消费方 invoker 远程服务注入到测试 Controller 中使用，调用该服务，查看看是否正常。 如果正常，说明该接口项目通过XXL-RPC从 client 项目调用了 server 项目中的服务，夸JVM进行了一次RPC通讯。

访问该Controller地址即可进行测试：http://127.0.0.1:8081/?name=jack

## [三、快速入门（frameless 无框架版本）](http://www.xuxueli.com/xxl-rpc/#/?id=三、快速入门（frameless-无框架版本）)

得益于优良的兼容性与模块化设计，不限制外部框架；除 spring/springboot 环境之外，理论上支持运行在任何Java代码中，甚至main方法直接启动运行；

以示例项目 “xxl-rpc-sample-frameless” 为例讲解；该示例项目以直连IP方式进行演示，也可以选择接入注册中心方式使用，如接入 XXL-REGISTRY。

### [3.1 API方式创建“服务提供者”：](http://www.xuxueli.com/xxl-rpc/#/?id=_31-api方式创建服务提供者：)

```undefined
// 参考代码位置：com.xxl.rpc.sample.server.XxlRpcServerApplication

// init
XxlRpcProviderFactory providerFactory = new XxlRpcProviderFactory();
providerFactory.initConfig(NetEnum.NETTY, Serializer.SerializeEnum.HESSIAN.getSerializer(), -1, -1, null, 7080, null, null, null);

// add services
providerFactory.addService(DemoService.class.getName(), null, new DemoServiceImpl());

// start
providerFactory.start();

while (!Thread.currentThread().isInterrupted()) {
    TimeUnit.HOURS.sleep(1);
}

// stop
providerFactory.stop();
```

### [3.2 API方式创建“服务消费者”：](http://www.xuxueli.com/xxl-rpc/#/?id=_32-api方式创建服务消费者：)

```undefined
// 参考代码位置：com.xxl.rpc.sample.client.XxlRpcClientAplication

// init client
DemoService demoService = (DemoService) new XxlRpcReferenceBean(NetEnum.JETTY, Serializer.SerializeEnum.HESSIAN.getSerializer(), CallType.SYNC,
        DemoService.class, null, 500, "127.0.0.1:7080", null, null).getObject();

// test
UserDTO userDTO = demoService.sayHi("[SYNC]jack");
System.out.println(userDTO);
```

## [四、系统设计](http://www.xuxueli.com/xxl-rpc/#/?id=四、系统设计)

### [4.1 系统架构图](http://www.xuxueli.com/xxl-rpc/#/?id=_41-系统架构图)

![输入图片说明](https://raw.githubusercontent.com/xuxueli/xxl-rpc/master/doc/images/img_DNq6.png)

### [4.2 核心思想](http://www.xuxueli.com/xxl-rpc/#/?id=_42-核心思想)

提供稳定高性能的RPC远程服务调用功能，简化分布式服务通讯开发。

### [4.3 角色构成](http://www.xuxueli.com/xxl-rpc/#/?id=_43-角色构成)

- 1、provider：服务提供方；
- 2、invoker：服务消费方；
- 3、serializer: 序列化模块；
- 4、remoting：服务通讯模块；
- 5、registry：服务注册中心；
- 6、admin：服务治理、监控中心：管理服务节点信息，统计服务调用次数、QPS和健康情况；（非必选，暂未整理发布...）

### [4.4 RPC工作原理剖析](http://www.xuxueli.com/xxl-rpc/#/?id=_44-rpc工作原理剖析)

![输入图片说明](https://raw.githubusercontent.com/xuxueli/xxl-rpc/master/doc/images/img_XEVY.png)

概念：

- 1、serialization：序列化，通讯数据需要经过序列化，从而支持在网络中传输；
- 2、deserialization：反序列化，服务接受到序列化的请求数据，需要序列化为底层原始数据；
- 3、stub：体现在XXL-RPC为服务的api接口；
- 4、skeleton：体现在XXL-RPC为服务的实现api接口的具体服务；
- 5、proxy：根据远程服务的stub生成的代理服务，对开发人员透明；
- 6、provider：远程服务的提供方；
- 7、consumer：远程服务的消费方；

RPC通讯，可大致划分为四个步骤，可参考上图进行理解：（XXL-RPC提供了多种调用方案，此处以 “SYNC” 方案为例讲解；）

- 1、consumer发起请求：consumer会根据远程服务的stub实例化远程服务的代理服务，在发起请求时，代理服务会封装本次请求相关底层数据，如服务iface、methos、params等等，然后将数据经过serialization之后发送给provider；
- 2、provider接收请求：provider接收到请求数据，首先会deserialization获取原始请求数据，然后根据stub匹配目标服务并调用；
- 3、provider响应请求：provider在调用目标服务后，封装服务返回数据并进行serialization，然后把数据传输给consumer；
- 4、consumer接收响应：consumer接受到相应数据后，首先会deserialization获取原始数据，然后根据stub生成调用返回结果，返回给请求调用处。结束。

### [4.5 TCP通讯模型](http://www.xuxueli.com/xxl-rpc/#/?id=_45-tcp通讯模型)

![输入图片说明](https://raw.githubusercontent.com/xuxueli/xxl-rpc/master/doc/images/img_b1IX.png)

consumer和provider采用NIO方式通讯，其中TC通讯方案可选NETTY或MINA具体选型，高吞吐高并发；但是仅仅依靠单个TCP连接进行数据传输存在瓶颈和风险，因此XXL-RPC在consumer端自身实现了内部连接池，consumer和provider之间为了一个连接池，当尽情底层通讯是会取出一条TCP连接进行通讯（可参考上图）。

### [4.6 sync-over-async](http://www.xuxueli.com/xxl-rpc/#/?id=_46-sync-over-async)

![输入图片说明](https://raw.githubusercontent.com/xuxueli/xxl-rpc/master/doc/images/img_pMtS.png)

XXL-RPC采用NIO进行底层通讯，但是NIO是异步通讯模型，调用线程并不会阻塞获取调用结果，因此，XXL-RPC实现了在异步通讯模型上的同步调用，即“sync-over-async”，实现原理如下，可参考上图进行理解：

- 1、每次请求会生成一个唯一的RequestId和一个RpcResponse，托管到请求池中。
- 2、调度线程，执行RpcResponse的get方法阻塞获取本次请求结果；
- 3、然后，底层通过NIO方式发起调用，provider异步响应请求结果，然后根据RequestId寻找到本次调用的RpcResponse，设置响应结果后唤醒调度线程。
- 4、调度线程被唤醒，返回异步响应的请求数据。

### [4.7 注册中心](http://www.xuxueli.com/xxl-rpc/#/?id=_47-注册中心)

XXL-RPC的注册中心，是可选组件，支持服务注册并动态发现；

可选择不启用，直接指定服务提供方机器地址通讯；

选择启用时，内置可选方案：“XXL-REGISTRY 轻量级注册中心”（推荐）、“ZK注册中心”、“Local注册中心”等；

##### [a、XXL-REGISTRY 轻量级注册中心（推荐）](http://www.xuxueli.com/xxl-rpc/#/?id=a、xxl-registry-轻量级注册中心（推荐）)

推荐使用内置的 "XXL-REGISTRY" 作为注册中心。可前往 XXL-REGISTRY (https://github.com/xuxueli/xxl-registry ) 查看部署文档。非常轻量级，一分钟可完成部署工作。

更易于集群部署、横向扩展，搭建与学习成本更低，推荐采用该方式；

##### [b、ZK注册中心](http://www.xuxueli.com/xxl-rpc/#/?id=b、zk注册中心)

内置“ZK注册中心”，可选组件，结构图如下：

![输入图片说明](https://raw.githubusercontent.com/xuxueli/xxl-rpc/master/doc/images/img_m3Ma.png)

原理：
XXL-RPC中每个服务在zookeeper中对应一个节点，如图"iface name"节点，该服务的每一个provider机器对应"iface name"节点下的一个子节点，如图中"192.168.0.1:9999"、"192.168.0.2:9999"和"192.168.0.3:9999"，子节点类型为zookeeper的EPHMERAL类型，该类型节点有个特点，当机器和zookeeper集群断掉连接后节点将会被移除。consumer底层可以从zookeeper获取到可提供服务的provider集群地址列表，从而可以向其中一个机器发起RPC调用。

### [4.8 在线服务目录](http://www.xuxueli.com/xxl-rpc/#/?id=_48-在线服务目录)

服务提供方新增 "/services" 服务目录功能，可查看在线服务列表；暂时仅针对JETTY通讯方案，浏览器访问地址 "{端口地址}/services" 即可。

### [4.9 如何切换“通讯方案”选型](http://www.xuxueli.com/xxl-rpc/#/?id=_49-如何切换通讯方案选型)

XXL-RPC提供多中通讯方案：支持 TCP 和 HTTP 两种通讯方式进行服务调用；其中 TCP 提供可选方案 NETTY 或 MINA ，HTTP 提供可选方案 NETTY_HTTP 或 Jetty；

如果需要切换XXL-RPC“通讯方案”，只需要执行以下两个步骤即可：

- a、引入通讯依赖包，排除掉其他方案依赖，各方案依赖如下：
  - NETTY：依赖 netty-all ；
  - MINA：依赖 mina-core ；
  - NETTY_HTTP：依赖 netty-all ；
  - JETTY：依赖 jetty-server + jetty-client；
- b、修改通讯枚举，需要同时在“服务方”与“消费方”两端一同修改，通讯枚举属性代码位置如下：
  - 服务工厂 "XxlRpcSpringProviderFactory.netType" ：可参考springboot示例组件初始化代码；
  - 服务引用注解 "XxlRpcReference.netType" | 服务Bean对象 "XxlRpcReferenceBean.netType" ：可参考springboot示例组件初始化代码；

### [4.9 如何切换“注册中心”选型](http://www.xuxueli.com/xxl-rpc/#/?id=_49-如何切换注册中心选型)

XXL-RPC的注册中心，是一个可选组件，不强制依赖；支持服务注册并动态发现；
可选择不启用，直接指定服务提供方机器地址通讯；
选择启用时，原生提供多种开箱即用的注册中心可选方案，包括：“XXL-RPC原生轻量级注册中心”、“ZK注册中心”、“Local注册中心”等；

如果需要切换XXL-RPC“注册中心”，只需要执行以下两个步骤即可：

- a、引入注册注册中心依赖包，排除掉其他方案依赖，各方案依赖如下：
  - XXL-RPC原生轻量级注册中心：轻量级、无第三方依赖；
  - ZK注册中心：依赖 zookeeper
  - Local注册中心：轻量级、零依赖；
- b、修改注册中心配置，需要同时在“服务方”与“消费方”两端一同修改，代码位置如下：
  - XxlRpcSpringProviderFactory.serviceRegistryClass：注册中心实现类，可选：XxlRegistryServiceRegistry.class、LocalServiceRegistry.class、ZkServiceRegistry.class
  - XxlRpcSpringProviderFactory.serviceRegistryParam：注册中心启动参数，各种注册中心启动参数不同，可参考其 start 方案了解；

### [4.10 泛化调用](http://www.xuxueli.com/xxl-rpc/#/?id=_410-泛化调用)

XXL-RPC 提供 "泛华调用" 支持，服务调用方不依赖服务方提供的API；泛化调用通常用于框架集成，比如 "网关平台、跨语言调用、测试平台" 等； 开启 "泛华调用" 时服务方不需要做任何调整，仅需要调用方初始化一个泛华调用服务Reference （"XxlRpcGenericService"） 即可。

| “XxlRpcGenericService#invoke” 请求参数 | 说明                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| String iface                           | 服务接口类名                                                 |
| String version                         | 服务版本                                                     |
| String method                          | 服务方法                                                     |
| String[] parameterTypes                | 服务方法形参-类型，如 "int、java.lang.Integer、java.util.List、java.util.Map ..." |
| Object[] args                          | 服务方法形参-数据                                            |

```undefined
// 服务Reference初始化-注解方式示例
@XxlRpcReference
private XxlRpcGenericService genericService;

// 服务Reference初始化-API方式示例
XxlRpcGenericService genericService = (XxlRpcGenericService) new XxlRpcReferenceBean(……).getObject();

// 调用方示例
Object result = genericService.invoke(
            "com.xxl.rpc.sample.server.service.Demo2Service",
            null,
            "sum",
            new String[]{"int", "int"},
            new Object[]{1, 2}
    );


// 服务方示例
public class Demo2ServiceImpl implements Demo2Service {

    @Override
    public int sum(int a, int b) {
        return a + b;
    }

}
```