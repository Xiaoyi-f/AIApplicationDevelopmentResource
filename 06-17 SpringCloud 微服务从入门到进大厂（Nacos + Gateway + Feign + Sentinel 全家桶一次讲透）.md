---
title: "SpringCloud 微服务从入门到进大厂：Nacos + Gateway + Feign + Sentinel 全家桶一次讲透"
status: 已发布
author: XiaoYi
---

# SpringCloud 微服务从入门到进大厂：Nacos + Gateway + Feign + Sentinel 全家桶一次讲透

> 上一篇把 SpringBoot 单体的骨架搭好了。但真到了企业、大厂，一个项目很少是「一个大单体」，而是拆成一堆小服务——这就是微服务。
> 这篇不跟你绕概念，直接把现在最火、面试必问、招人必会的微服务全家桶一次讲透：**注册中心 Nacos、服务调用 OpenFeign、网关 Gateway、熔断限流 Sentinel、分布式事务 Seata、链路追踪 SkyWalking**，还有配套的 JWT 鉴权、分布式锁、消息队列、部署。
> 每个模块都标「大厂考点」，学完这篇，微服务面试和项目都够用。
> Let's use AI to create a better world。

---

## 一、微服务到底是啥？

一句话：**把一个大的单体应用，按业务拆成一个个能独立部署、独立开发、独立扩展的小服务，服务之间通过网络互相调用。**

```
单体：一个项目里塞下订单、用户、库存、支付所有模块
        ↓ 拆
微服务：user-service / order-service / goods-service / pay-service ...
        每个服务独立数据库、独立部署、独立扩缩容
```

|  |  |
| --- | --- |
| 对比项 | 说明 |
| 优点 | 独立部署、独立扩容、技术栈灵活、故障隔离、团队解耦 |
| 缺点 | 运维复杂、分布式事务、链路长、排查难 |
| 什么时候拆 | 业务复杂、团队大、需要独立发布时；小项目别硬拆 |

> **大厂考点**：单体 vs 微服务的优缺点？什么场景适合微服务？——本质是「复杂度换灵活性」，拆得越细，分布式问题越多。

---

## 二、先踩版本这个坑（必看）

SpringBoot 和 Spring Cloud 版本**严格对应**，配错直接起不来，这是新手第一道坎，也是面试常问。

| Spring Boot | Spring Cloud | Spring Cloud Alibaba |
| --- | --- | --- |
| 3.2.x / 3.3.x | 2023.0.x（Leyton） | 2023.0.x |
| 3.0.x / 3.1.x | 2022.0.x（Kilburn） | 2022.0.x |
| 2.6.x / 2.7.x | 2021.0.x（Jubilee） | 2021.0.x |

**本文以 SpringBoot 3.3.x + Spring Cloud 2023.0.x 为主力版本**（资料最多、企业最常用）。

> **大厂考点**：SpringBoot 2 和 3 的区别？——JDK 17 起步、javax 改 jakarta、`spring-cloud-starter-bootstrap` 默认关闭等。

---

## 三、注册中心 Nacos

### 1. 注册中心是干嘛的？

一句话：**服务的「通讯录」。每个服务启动时把自己地址注册进去，调用方从里面查地址，服务挂了还能自动踢掉。**

主流选择：**Nacos**（阿里，注册 + 配置二合一，最火）> Consul > Eureka（已停止维护，别再用了）。

### 2. 服务提供者接入

依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

配置 `application.yml`：

```yaml
spring:
  application:
    name: user-service
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

启动类加 `@EnableDiscoveryClient`（新版可省略），服务就自动注册上去了。

### 3. 关键概念

| 概念 | 说明 |
| --- | --- |
| 临时实例 | 靠心跳保活，挂了自动剔除（默认） |
| 持久实例 | 靠 Nacos 主动探测，适合数据库这类 |
| 服务发现 | 调用方实时拿到健康实例列表 |
| 权重 | 控制流量分配，灰度发布用 |

> **大厂考点**：Nacos 和 Eureka 区别？注册中心挂了服务还能调吗？（能，本地有缓存）AP 还是 CP？（Nacos 临时实例走 AP，持久实例走 CP）

---

## 四、配置中心 Nacos Config

一句话：**配置统一放 Nacos，改配置不用重启服务、不用重新打包，热更新生效。**

依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

配置 `bootstrap.yml`（注意：SpringBoot 3 需要先引入 bootstrap 依赖）：

```yaml
spring:
  application:
    name: user-service
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
```

- **Data ID 命名规则**：`{服务名}-{profile}.{后缀}`，如 `user-service-dev.yaml`。
- 配合 `@RefreshScope` 或 `@Value` + 动态刷新，实现配置热更新。

> **大厂考点**：配置中心作用？怎么实现配置动态刷新？多环境（dev/test/prod）怎么隔离（namespace + group）？

---

## 五、服务调用 OpenFeign + 负载均衡

### 1. OpenFeign 是什么？

一句话：**声明式 HTTP 客户端，写接口就像写本地方法一样调用远程服务，不用手写 RestTemplate 那堆样板代码。**

依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

用法：

```java
@FeignClient(name = "user-service")   // 服务名，自动负载均衡
public interface UserClient {
    @GetMapping("/user/{id}")
    User getUserById(@PathVariable("id") Long id);
}
```

调用方 `@EnableFeignClients` 开启，然后直接注入 `UserClient` 就能调。

### 2. 负载均衡

一句话：**调用方从注册中心拿到多个实例，按策略挑一个去调。**

> **重要**：Ribbon 已停更，现在用 **Spring Cloud LoadBalancer**（默认内置，无需额外配置）。Feign 默认就走 LoadBalancer。

常用负载均衡策略：轮询（默认）、随机、权重、一致性哈希。

### 3. 调用优化

- 超时、重试要配好，否则一个慢接口拖垮整个链路。
- 日志、熔断、降级可以挂在 Feign 上。

> **大厂考点**：Feign 原理（动态代理）？怎么实现负载均衡？Ribbon 和 LoadBalancer 区别？Feign 调用超时怎么配？

---

## 六、网关 Spring Cloud Gateway

### 1. 网关是干嘛的？

一句话：**所有请求的唯一入口，统一做路由转发、鉴权、限流、跨域、日志。**

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("user-route", r -> r.path("/user/**")
            .uri("lb://user-service"))   // lb 表示走负载均衡
        .build();
}
```

### 2. 核心概念

| 概念 | 说明 |
| --- | --- |
| Route | 路由：一个 id + 断言 + 目标 URI |
| Predicate | 断言：请求满足条件才匹配（路径/方法/Header） |
| Filter | 过滤器：转发前后处理（加请求头、鉴权、限流） |

### 3. 统一鉴权（超高频）

网关里用全局过滤器校验 JWT，通过后再放行到下游，**下游服务不用重复鉴权**。

```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        if (token == null || !JwtUtil.verify(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
}
```

> **大厂考点**：Gateway 和 Zuul 区别？（Gateway 基于 WebFlux 响应式，性能更好）Gateway 三大核心？网关作用？JWT 放网关还是服务里？

---

## 七、熔断限流 Sentinel

### 1. 为什么要熔断限流？

一句话：**防止「一个服务挂了，把调用它的服务也拖垮」，这是雪崩效应。**

Sentinel（阿里）替代已停更的 Hystrix，是现在国内最主流的熔断限流组件。

### 2. 核心规则

| 规则 | 作用 |
| --- | --- |
| 流控规则 | 限制 QPS / 线程数，超了就限流 |
| 熔断规则 | 异常比例/慢调用比例超阈值就熔断，快速失败 |
| 热点规则 | 针对热点参数限流 |
| 系统规则 | 按整体负载限流 |

依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

配置：

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080   # 控制台
```

代码里用 `@SentinelResource` 标记资源 + 指定降级处理方法（blockHandler / fallback）。

> **大厂考点**：雪崩效应怎么解决？熔断、限流、降级区别？Sentinel 和 Hystrix 区别？限流算法（滑动窗口、令牌桶、漏桶）？

---

## 八、分布式事务 Seata

### 1. 为什么有分布式事务？

一句话：**单体里一个事务能管一个库，微服务里「下单 + 扣库存 + 扣钱」跨了三个服务三个库，本地事务管不住了。**

Seata 是阿里开源的分布式事务解决方案，最常用。

### 2. 四种模式

| 模式 | 说明 | 场景 |
| --- | --- | --- |
| AT | 自动回滚（对业务无侵入，最常用） | 默认首选 |
| TCC | Try-Confirm-Cancel，业务侵入强 | 高性能、强一致 |
| SAGA | 长事务，补偿式 | 长流程 |
| XA | 强一致，性能差 | 银行等 |

### 3. AT 模式原理（必背）

```
一阶段：业务 SQL + 回滚日志一起提交（本地事务）
二阶段：都成功 → 删除回滚日志；有失败 → 用回滚日志反向补偿
```

三大角色：TC（事务协调者）、TM（事务发起者）、RM（资源参与者）。

> **大厂考点**：Seata AT 模式原理？AT、TCC、SAGA 区别？全局事务怎么保证最终一致？

---

## 九、认证鉴权 JWT + Spring Security

一句话：**登录发一个 JWT 令牌，之后请求都带 token，网关统一校验，实现无状态认证。**

| 方案 | 说明 |
| --- | --- |
| JWT | 无状态，自包含 token，适合分布式微服务 |
| OAuth2 | 第三方授权（微信登录那种） |
| Spring Security | 安全框架，管认证和授权 |

JWT 结构：`Header.Payload.Signature`（三段 Base64）。核心是签名防篡改，注意不要把敏感信息明文放 payload。

> **大厂考点**：JWT 和 Session 区别？JWT 怎么防止被篡改？token 过期怎么刷新？单点登录 SSO 怎么实现？

---

## 十、链路追踪 SkyWalking

一句话：**微服务调用链很长，一个请求穿过 N 个服务，链路追踪帮你把整条链路串起来，快速定位慢在哪、错在哪。**

| 方案 | 说明 |
| --- | --- |
| SkyWalking | 主流，无侵入（Java Agent 探针），UI 强大 |
| Micrometer + Zipkin | Sleuth 已并入 Micrometer Tracing |

接入方式：启动参数挂 SkyWalking Agent 探针即可，几乎零代码改动。

> **大厂考点**：链路追踪解决什么问题？traceId 怎么贯穿整个链路（放进日志 + 请求头透传）？

---

## 十一、消息队列 RabbitMQ / Kafka

一句话：**服务间异步解耦、削峰填谷、最终一致性的利器。**

| 选型 | 说明 |
| --- | --- |
| RabbitMQ | 延迟队列、可靠消息、复杂路由强 |
| Kafka | 超高吞吐、日志流、大数据场景 |
| RocketMQ | 阿里系，事务消息、电商场景 |

微服务里典型场景：下单后发消息异步扣库存、发短信、更新统计，实现「订单服务」和「库存服务」解耦。

> **大厂考点**：为什么用 MQ？怎么保证消息不丢、不重复（幂等）？顺序消息？死信队列？——这些在前面 RabbitMQ 篇已详细讲透。

---

## 十二、分布式锁 Redisson + 分布式 ID

### 1. 分布式锁

一句话：**单机 synchronized 锁不住多实例，跨服务要用 Redis 分布式锁，Redisson 是首选。**

```java
RLock lock = redissonClient.getLock("order:" + orderId);
try {
    if (lock.tryLock(3, 30, TimeUnit.SECONDS)) {
        // 业务
    }
} finally {
    if (lock.isHeldByCurrentThread()) lock.unlock();
}
```

### 2. 分布式 ID

一句话：**单机自增主键在分库分表/多实例下会重复，需要全局唯一 ID。**

| 方案 | 说明 |
| --- | --- |
| 雪花算法 Snowflake | 时间戳 + 机器号 + 序列号，最常用 |
| 美团 Leaf | 号段模式 + 雪花模式 |
| 数据库号段 | 一次取一批，减少 DB 压力 |

> **大厂考点**：分布式锁怎么实现？Redisson 看门狗原理？雪花算法结构？时钟回拨问题怎么处理？

---

## 十三、部署 Docker + K8s

一句话：**微服务几十个服务，手动部署会疯，必须上容器化。**

| 层 | 作用 |
| --- | --- |
| Docker | 把每个服务打成镜像，环境一致 |
| Docker Compose | 本地编排几个容器 |
| K8s | 生产级编排：自动扩缩容、滚动发布、服务发现、自愈 |
| CI/CD | GitHub Actions / Jenkins 自动化构建发布 |

典型链路：`代码提交 → CI 构建镜像 → 推镜像仓库 → K8s 滚动更新`。

> **大厂考点**：Docker 和虚拟机区别？K8s 核心组件（Pod / Service / Deployment）？服务怎么做到无感滚动发布？

---

## 附：微服务面试高频题 + 速查表

**高频面试题（背这些就够应付大多数面试）**

1. 微服务优缺点？什么场景适合拆？
2. Nacos 原理？注册中心挂了还能调吗？
3. Feign 原理？负载均衡怎么实现？
4. 网关作用？Gateway 三大核心？
5. 雪崩效应怎么解决？熔断、限流、降级区别？
6. Seata AT 模式原理？分布式事务方案对比？
7. JWT 和 Session 区别？SSO 怎么实现？
8. 分布式锁怎么实现？分布式 ID 有哪些方案？
9. 消息怎么保证不丢不重复？
10. 链路追踪怎么实现？

**速查表**

| 组件 | 一句话作用 |
| --- | --- |
| Nacos | 注册中心 + 配置中心 |
| OpenFeign | 声明式服务调用 |
| LoadBalancer | 客户端负载均衡 |
| Gateway | 统一入口 + 路由 + 鉴权 |
| Sentinel | 限流 + 熔断 + 降级 |
| Seata | 分布式事务（AT 最常用） |
| SkyWalking | 链路追踪 |
| Redisson | 分布式锁 |
| 雪花算法 | 分布式 ID |
| RabbitMQ/Kafka | 异步解耦 + 削峰 |
| Docker/K8s | 容器化 + 编排部署 |

> 学习建议：先把「Nacos + Feign + Gateway + Sentinel」这四个核心跑通一个 demo，剩下的 Seata、SkyWalking、分布式锁在真实项目里用到再深入。微服务的本质不在背概念，而在**把服务拆开、调起来、扛住流量**。
