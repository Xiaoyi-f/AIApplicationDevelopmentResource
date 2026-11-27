---
title: "RabbitMQ 从入门到进大厂（四种交换机、消息可靠性、幂等一次讲透）"
status: 已发布
author: XiaoYi
---

# RabbitMQ 从入门到进大厂（四种交换机、消息可靠性、幂等一次讲透）

> 今天讲透 RabbitMQ。这一篇不只是「发一条消息、收一条消息」，而是把四种交换机、消息可靠性、消费幂等、死信队列这些大厂必考点一次讲清楚。
> 学完这篇，你就能说出「削峰填谷」「消息不丢」「不重复消费」到底怎么落地。
> Let's use AI to create a better world。

## 一、RabbitMQ 是啥 + 为什么用

一句话：**RabbitMQ 是一个开源消息中间件（消息队列），基于 AMQP 协议，负责在「生产者」和「消费者」之间暂存、转发消息。**

### 三大角色（用微信类比秒懂）

```
消息发送者  →  发布消息  →  消息 Broker  →  转发消息  →  消息接收者
（原调用方）               （微信服务器）              （原服务提供方）
                       管理、暂存、转发
```

|  |  |  |
| --- | --- | --- |
| 角色 | 类比 | 职责 |
| 生产者 | 发微信的人 | 发布消息 |
| Broker | 微信服务器 | 管理、暂存、转发消息 |
| 消费者 | 收微信的人 | 订阅并处理消息 |

### 三大价值（大厂高频）

|  |  |  |
| --- | --- | --- |
| 价值 | 说明 | 典型场景 |
| 削峰填谷 | 把瞬时高峰请求缓存到队列，慢慢消费 | 秒杀、抢购 |
| 解耦 | 生产者不直接依赖消费者，中间隔着队列 | 下单后异步通知库存/短信 |
| 异步 | 耗时操作丢给队列，主流程先返回 | 发邮件、记账 |

> 面试问「为什么用消息队列」，答这六个字：**削峰填谷、解耦、异步**。

---

## 二、核心概念（7 个角色）

|  |  |
| --- | --- |
| 概念 | 含义 |
| publisher | 生产者，发送消息的一方 |
| consumer | 消费者，消费消息的一方 |
| queue | 队列，存储消息 |
| exchange | 交换机，负责路由，**生产者发的消息由交换机决定投递到哪个队列** |
| virtual host | 虚拟主机，**数据隔离**，每个 vhost 相互独立，有各自的 exchange、queue |
| connection | TCP 连接，一个连接里可以开多个 channel（信道，复用连接） |
| binding | 绑定，把队列和交换机通过 routing key 关联起来 |

```
生产者 → exchange（交换机） → 通过 binding（routing key）→ queue（队列）→ 消费者
```

> 两个最容易混的点：① **消息不是直接发到队列，是先发到交换机**，交换机按路由规则转发到队列；② **virtual host 是逻辑隔离**，一个 RabbitMQ 实例能开多个 vhost，互不干扰。

---

## 三、AMQP 协议 + 管理界面

- **AMQP**：RabbitMQ 用的协议（Advanced Message Queuing Protocol）。

- **管理界面**：默认 `http://localhost:15672`，默认账号密码都是 `guest`。

> 大厂考点：**`5672` 是 AMQP 协议端口（程序连的），`15672` 是管理界面端口（浏览器看的）**，别搞混。

---

## 四、四种交换机

交换机（exchange）决定「消息去哪」，分四种：

|  |  |  |
| --- | --- | --- |
| 类型 | 路由规则 | 特点 |
| Direct | routing key **精确匹配** | 定向发送 |
| Fanout | 忽略 routing key，广播给所有绑定队列 | 广播 |
| Topic | routing key **通配符匹配** | 灵活主题订阅 |
| Headers | 按消息头匹配（很少用） | 基本不碰 |

**Topic 通配符规则（大厂必考）**：

```
*  匹配一个单词（一层）
#  匹配零个或多个单词（多层）

exchange.routing.*   → 匹配 exchange.routing.a / exchange.routing.b
exchange.routing.#   → 匹配 exchange.routing / exchange.routing.a / exchange.routing.a.b
```

> **纠正一个常见误区**：`*` 和 `#` 是 **Topic 交换机**的通配符，**Direct 交换机是精确匹配，不支持通配符**。很多人把两者混在一起记。

---

## 五、环境与配置

### 1. Docker 起一个（最快）

```
docker run -d --name rabbitmq \
  -e RABBITMQ_DEFAULT_USER=guest \
  -e RABBITMQ_DEFAULT_PASS=guest \
  -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management
```

> 环境变量记牢：**`RABBITMQ_DEFAULT_USER` / `RABBITMQ_DEFAULT_PASS`** 设默认账号密码。

### 2. application.yml

```
spring:
  rabbitmq:
    connection-timeout: 3s
    host: localhost
    port: 5672            # AMQP 协议端口
    virtual-host: /
    username: guest
    password: guest
    template:
      retry:
        enabled: true
        initial-interval: 1000ms
        multiplier: 1
        max-attempts: 3
    listener:
      simple:
        prefetch: 1        # 每次取 1 条，处理完再取（能者多劳模式）
```

---

## 六、Spring AMQP 快速上手

### 1. 发送消息（RabbitTemplate）

```
@SpringBootTest
public class SpringAmqpTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSendMessage() {
        String queueName = "simple.queue";
        String message = "Hello Spring AMQP";
        rabbitTemplate.convertAndSend(queueName, message); // 发到队列
    }
}
```

### 2. 接收消息（@RabbitListener）

```
@Component
public class SpringRabbitListener {
    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueue(String msg) throws InterruptedException {
        System.out.println("Spring 消费者接收到消息: [" + msg + "]");
    }
}
```

> 一个生产队列 + 多个消费者 → **竞争消费**：同一条消息只会被一个消费者处理，这是默认行为（轮询分发）。

---

## 七、三种交换机实战

### 1. Fanout 广播模式

```
@Configuration
public class FanoutConfig {
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("amq.fanout");  // 广播交换机
    }

    @Bean
    public Queue fanoutQueue() {
        return new Queue("fanout.queue");          // 队列
    }

    @Bean
    public Binding fanoutBinding(Queue fanoutQueue, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue).to(fanoutExchange); // 绑定（无 routing key）
    }
}
```

> Fanout 会把消息广播给**所有绑定到它的队列**，一个消息多个消费者都能收到。

### 2. Direct 定向模式

```
@Configuration
public class DirectConfig {
    @Bean
    public DirectExchange directExchange() {
        return ExchangeBuilder.directExchange("direct.exchange").build();
    }

    @Bean
    public Queue directQueue() {
        return new Queue("direct.queue");
    }

    @Bean
    public Binding directBinding(Queue directQueue, DirectExchange directExchange) {
        return BindingBuilder.bind(directQueue).to(directExchange)
                .with("direct.routing.key");   // 绑定 routing key
    }
}
```

> **纠正笔误**：交换机名是 `direct.exchange`，发送时别写成 `directExchange`（少个点，消息就发丢了）。

### 3. Topic 主题模式

```
@Bean
public TopicExchange topicExchange() {
    return new TopicExchange("topic.exchange");
}

@Bean
public Binding topicBinding(Queue topicQueue, TopicExchange topicExchange) {
    return BindingBuilder.bind(topicQueue).to(topicExchange)
            .with("order.*");   // 匹配 order.create / order.cancel 等
}
```

> Topic 用 `*`（一层）和 `#`（多层）做模糊匹配，适合「按业务主题订阅」的场景。

---

## 八、消息可靠性（大厂必考）

面试高频灵魂三问：**消息会丢吗？怎么保证不丢？怎么保证不被重复消费？** 这一节解决前两个。

### 1. 消息持久化（防 Broker 宕机丢消息）

```
@Bean
public Queue durableQueue() {
    return QueueBuilder.durable("durable.queue").build(); // 队列持久化
}

@Bean
public DirectExchange durableExchange() {
    return new DirectExchange("durable.exchange", true, false); // 交换机持久化
}
```

> 持久化 = 队列、交换机、消息都标记 durable，Broker 重启后消息还在。

### 2. 生产者确认（Publisher Confirm，防发送丢消息）

```
spring:
  rabbitmq:
    publisher-confirm-type: correlated   # 开启生产者确认
    publisher-returns: true              # 开启路由失败退回

rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
    if (ack) {
        System.out.println("消息成功到达交换机");
    } else {
        System.out.println("发送失败: " + cause);
    }
});
```

> 生产者确认分两步：**confirm（是否到交换机）+ return（交换机是否成功路由到队列）**。没到交换机 → confirm 返回 false；到了交换机但 routing key 不对 → 触发 return 退回。

### 3. 消费者确认（Consumer Ack，防处理丢消息）

默认是**自动 ack**：消息一到消费者就自动确认，但此时业务可能还没处理完就宕机了，消息就丢了。改成**手动 ack**：

```
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: manual   # 手动确认

@RabbitListener(queues = "durable.queue")
public void listen(Message msg, Channel channel) throws IOException {
    long deliveryTag = msg.getMessageProperties().getDeliveryTag();
    try {
        // 业务处理
        System.out.println("处理消息: " + new String(msg.getBody()));
        channel.basicAck(deliveryTag, false);   // 处理成功，确认
    } catch (Exception e) {
        channel.basicNack(deliveryTag, false, true); // 处理失败，重回队列
    }
}
```

> **可靠性的完整链路**：生产者 confirm（不丢在发送端）+ 消息持久化（不丢在 Broker）+ 消费者手动 ack（不丢在处理端）。三把锁，缺一不可。

---

## 九、消息幂等（防重复消费）

> **大厂必考：怎么保证消息不被重复消费？** 答案是**幂等**——同一个消息处理多次，结果和一次一样。

常见方案：**用唯一消息 ID + Redis 去重**。

```
@Component
public class MyListener {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @RabbitListener(queues = "direct.queue")
    public void onDirect(Message msg) {
        // 1. 取出消息唯一 ID
        String msgId = msg.getMessageProperties().getMessageId();

        // 2. setIfAbsent：key 不存在才创建返回 true，存在返回 false（天然原子性）
        Boolean first = stringRedisTemplate.opsForValue()
                .setIfAbsent("msg:" + msgId, "1", Duration.ofMinutes(10));

        // 3. 已经处理过，直接丢弃
        if (Boolean.FALSE.equals(first)) {
            return;
        }

        // 4. 真正业务
        String body = new String(msg.getBody(), StandardCharsets.UTF_8);
        System.out.println("定向收到:" + body);
    }
}
```

> 关键点：① **`setIfAbsent`（SETNX）是原子操作**，多个消费者并发时只有一个能 set 成功；② `Boolean.FALSE.equals(first)` 而不是 `!first`，因为 **Boolean 能装 null**，直接 `!first` 遇到 null 会 NPE；③ 消息 ID 用 `setCreateMessageIds(true)` 让转换器自动生成。

消息 ID 自动生成配置（配合 Jackson 序列化）：

```
@Bean
public MessageConverter messageConverter() {
    Jackson2JsonMessageConverter converter = new Jackson2JsonMessageConverter();
    converter.setCreateMessageIds(true);   // 自动创建消息 ID
    return converter;
}
```

> **纠正笔误**：接收端要用 `Message` 类型（不是 `String`）才能拿到 `getMessageProperties()`。用 `String` 接收是拿不到 messageId 的。

---

## 十、prefetch 能者多劳

```
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1    # 每次只取 1 条，处理完再取
```

> **能者多劳模式**：`prefetch: 1` 表示每个消费者每次只预取 1 条消息，处理完才取下一条。这样处理快的消费者能多拿消息，处理慢的不会积压一堆在本地。默认是「轮询平均分」，改成 1 就是「谁快谁多干」。

---

## 十一、死信队列 + 延迟队列

### 1. 死信队列（DLX）

消息变成「死信」的三种情况：① 消息被拒绝（nack 且不重回队列）；② 消息过期（TTL）；③ 队列满了。

```
@Bean
public Queue normalQueue() {
    return QueueBuilder.durable("normal.queue")
            .deadLetterExchange("dlx.exchange")   // 死信交换机
            .deadLetterRoutingKey("dlx.key")      // 死信路由键
            .build();
}
```

> 死信队列的典型用途：**处理失败的消息统一收集**，人工排查或告警，不让问题消息反复重试打爆系统。

### 2. 延迟队列（用 TTL + 死信实现）

```
@Bean
public Queue delayQueue() {
    return QueueBuilder.durable("delay.queue")
            .ttl(10000)                          // 消息 10 秒后过期
            .deadLetterExchange("dlx.exchange")  // 过期后进死信交换机
            .deadLetterRoutingKey("dlx.key")
            .build();
}
```

> 原理：**消息发到没有消费者的队列，等 TTL 过期变成死信，被转发到真正的消费队列**，实现「延迟消费」。典型场景：订单超时自动取消。

---

## 大厂高频考点速查表

|  |  |
| --- | --- |
| 考点 | 答案要点 |
| 为什么用消息队列 | 削峰填谷、解耦、异步 |
| 三大角色 | 生产者（发布）、Broker（暂存转发）、消费者（订阅） |
| 四种交换机 | Direct 精确 / Fanout 广播 / Topic 通配 / Headers 头 |
| `*` vs `#` | `*` 一层、`#` 多层，是 Topic 的通配符，不是 Direct |
| 5672 vs 15672 | 协议端口 / 管理界面端口 |
| virtual host | 逻辑隔离，独立 exchange/queue |
| 消息不丢三把锁 | 生产者 confirm + 持久化 + 消费者手动 ack |
| 消息重复消费 | 幂等：唯一消息 ID + Redis setIfAbsent 去重 |
| prefetch | 每次预取 1 条，能者多劳 |
| 死信队列 | 拒绝 / TTL 过期 / 队列满 → 进 DLX |
| 延迟队列 | TTL + 死信交换机实现 |
