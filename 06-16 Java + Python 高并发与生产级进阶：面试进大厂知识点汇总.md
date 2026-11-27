---
title: "Java + Python 高并发与生产级进阶：面试进大厂知识点汇总"
status: 已发布
category: "进大厂 · 高并发 · 生产级"
---

# Java + Python 高并发与生产级进阶：面试进大厂知识点汇总

> 这篇是「进大厂」冲刺篇：把 Java / Python 两个技术栈里**高并发、生产级、面试高频**的知识点浓缩成一份速查清单。
> 每个知识点给「一句话结论 + 关键代码/参数 + 一道高频面试题」，适合面试前过一遍、写简历前查漏补缺。

---

## 一、Java 高并发（JUC 核心）

### 1. 线程与线程池

- **结论**：线程是稀缺资源，生产环境一律用线程池，禁止 `new Thread()` 裸创建。
- 线程池 7 参数：核心线程数、最大线程数、存活时间、时间单位、工作队列、线程工厂、拒绝策略。

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    8,                          // 核心线程
    16,                         // 最大线程
    60, TimeUnit.SECONDS,       // 空闲存活
    new LinkedBlockingQueue<>(1000),   // 工作队列
    new ThreadPoolExecutor.CallerRunsPolicy()  // 拒绝策略：让提交者自己跑
);
```

- **核心线程数怎么定**：CPU 密集型 ≈ CPU 核数 + 1；IO 密集型 ≈ 核数 × 2（或核数 / (1 - 阻塞系数)）。
- **拒绝策略**：AbortPolicy（抛异常，默认）、CallerRunsPolicy（提交者执行）、DiscardPolicy（丢弃）、DiscardOldestPolicy（丢最老）。
- **高频面试题**：线程池的 7 个参数？任务提交后是怎么被执行的（核心 → 队列 → 最大 → 拒绝）？

### 2. 锁：synchronized / ReentrantLock / AQS

- **结论**：`synchronized` 是 JVM 内置锁，1.6 后经过锁升级优化；`ReentrantLock` 是 API 锁，更灵活，两者底层都依赖 AQS。
- **区别**：ReentrantLock 支持公平锁、可中断、超时、Condition 分组唤醒、tryLock。
- **AQS**：AbstractQueuedSynchronizer，用 volatile state + CLH 队列实现，ReentrantLock / CountDownLatch / Semaphore 都基于它。

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();  // 必须 finally 释放
}
```

- **高频面试题**：synchronized 和 ReentrantLock 区别？AQS 原理？

### 3. volatile / CAS / ABA

- **volatile**：保证可见性 + 禁止指令重排，**不保证原子性**。适合状态标记、双重检查锁（DCL）。
- **CAS**：Compare-And-Swap，CPU 原子指令，`AtomicInteger` 等原子类的底层。
- **ABA 问题**：CAS 判断值没变就以为没被改过，实际可能被改过又改回。用版本号解决：`AtomicStampedReference`。

```java
// DCL 单例：volatile 防止指令重排导致的半初始化对象
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
```

- **高频面试题**：volatile 的可见性和禁止重排怎么实现的（内存屏障）？ABA 怎么解决？

### 4. ConcurrentHashMap

- **结论**：线程安全的 Map，读不加锁，写只锁桶（Node），并发度远高于 Hashtable。
- **JDK 1.7**：分段锁 Segment + 数组链表；**JDK 1.8**：CAS + synchronized 锁单个桶，链表超 8 转红黑树。
- **高频面试题**：ConcurrentHashMap 为什么比 Hashtable 快？1.7 和 1.8 实现差异？

### 5. ThreadLocal 与内存泄漏

- **结论**：线程本地变量，底层是 Thread 内部的 ThreadLocalMap，key 是弱引用。
- **泄漏原因**：ThreadLocal 被回收后，key 变 null，但 value 仍被线程引用，若线程是线程池里的长生命周期线程，value 无法回收。
- **解决**：用完必须 `remove()`；项目里用静态 ThreadLocal 存用户上下文（登录态、TraceId）。

```java
private static final ThreadLocal<String> USER = new ThreadLocal<>();
try {
    USER.set(uid);
    // 业务
} finally {
    USER.remove();  // 必须清理
}
```

- **高频面试题**：ThreadLocal 为什么内存泄漏？怎么避免？父子线程怎么传值（InheritableThreadLocal / TransmittableThreadLocal）？

### 6. 并发工具类

| 工具 | 作用 |
|------|------|
| CountDownLatch | 等 N 个任务都完成再继续（一次性） |
| CyclicBarrier | 一组线程互相等待到齐再一起跑（可复用） |
| Semaphore | 限流，控制同时访问的线程数 |
| BlockingQueue | 阻塞队列，生产者-消费者模型 |

- **高频面试题**：CountDownLatch 和 CyclicBarrier 区别？阻塞队列实现原理？

### 7. 异步 CompletableFuture

- **结论**：Java 8 的异步编程利器，链式编排多个异步任务，避免回调地狱。

```java
CompletableFuture.supplyAsync(() -> queryOrder())
    .thenApply(order -> calcPrice(order))
    .thenAccept(price -> System.out.println(price))
    .exceptionally(e -> { log.error("err", e); return null; });
```

- **高频面试题**：thenApply / thenCompose / thenCombine 区别？

### 8. 死锁与排查

- **四个必要条件**：互斥、占有且等待、不可剥夺、循环等待。破坏任一即可避免。
- **排查**：`jstack <pid>` 看 `Found one Java-level deadlock`；`jps` 找进程；Arthas `thread -b` 定位死锁。

---

## 二、JVM（进大厂必考）

### 1. 内存区域

- **堆**：对象实例，分新生代（Eden + S0 + S1）和老年代，GC 主战场。
- **栈**：线程私有，栈帧（局部变量表、操作数栈），方法调用。
- **方法区/元空间**：类元数据、常量池。
- **程序计数器、本地方法栈**。

### 2. GC 算法与收集器

- **算法**：标记-清除（碎片）、标记-复制（新生代）、标记-整理（老年代）。
- **收集器**：
  - 新生代：Serial、ParNew、Parallel Scavenge（吞吐量优先）。
  - 老年代：CMS（低延迟，已废弃）、G1（大堆、可预测停顿，主流）、ZGC（超低延迟，JDK 11+）。
- **高频面试题**：对象从新生代到老年代的路径？什么对象会进老年代（大对象、年龄超阈值、动态年龄判定）？

### 3. 常用调优参数

```bash
-Xms2048m -Xmx2048m            # 堆初始/最大，生产建议相等避免抖动
-Xmn512m                        # 新生代
-XX:+UseG1GC                    # 使用 G1
-XX:MaxGCPauseMillis=200        # 期望停顿
-XX:+HeapDumpOnOutOfMemoryError # OOM 自动 dump
-XX:HeapDumpPath=/tmp/heap.hprof
```

### 4. OOM 排查

- 拿到 dump 后：`jmap -dump:live,format=b,file=heap.hprof <pid>`，用 MAT / JProfiler / Arthas `heapdump` 分析。
- 常见 OOM：`heap space`（堆满）、`Metaspace`（类加载过多）、`unable to create native thread`（线程开太多）。

---

## 三、Java 生产级工程

### 1. 缓存三大问题

| 问题 | 场景 | 解决 |
|------|------|------|
| 穿透 | 查不存在的 key | 布隆过滤器、空值缓存 |
| 击穿 | 热点 key 失效 | 互斥锁重建、逻辑过期 |
| 雪崩 | 大量 key 同时失效 | 过期时间加随机、多级缓存、限流 |

- **高频面试题**：缓存穿透、击穿、雪崩分别是什么？怎么解决？

### 2. 分布式锁

- **Redis 方案**：`SET key value NX PX 30000`（原子加锁 + 过期时间），释放用 Lua 脚本校验 value 再删，避免误删。
- **Redisson**：封装了看门狗自动续期、可重入、公平锁，生产首选。
- **ZooKeeper 方案**：临时顺序节点 + watch，强一致但性能低。

### 3. 幂等

- **结论**：同一请求重复提交，结果不变。
- **方案**：数据库唯一索引、乐观锁（version）、分布式锁、token 机制、状态机约束。
- **高频面试题**：MQ 消息重复消费怎么保证幂等？支付接口怎么防重复下单？

### 4. 限流与熔断

- **限流算法**：固定窗口、滑动窗口、漏桶（匀速）、令牌桶（允许突发）。
- **熔断降级**：Sentinel / Hystrix，达到阈值后快速失败，避免雪崩。
- **高频面试题**：令牌桶和漏桶区别？Sentinel 的限流规则？

### 5. 消息可靠性（RabbitMQ）

- **生产者确认**：confirm 模式 + 消息持久化。
- **消费者确认**：手动 ack，处理成功才 ack。
- **幂等**：消费端去重（消息 id + Redis setnx）。
- **高频面试题**：消息怎么保证不丢？死信队列、延迟队列怎么实现？

### 6. 分布式事务

- **方案对比**：
  - 2PC/XA：强一致，性能差，阻塞。
  - TCC：Try-Confirm-Cancel，业务侵入，性能好。
  - 最终一致性（本地消息表 / MQ）：可用性最高，主流选择。
  - Seata：阿里开源，AT 模式（自动回滚）。
- **高频面试题**：2PC、TCC、最终一致性区别？你们项目用的哪种？

### 7. 监控与日志

- **监控**：Prometheus + Grafana（指标）、SkyWalking / Zipkin（链路追踪）、Arthas（线上诊断）。
- **日志**：Logback，结构化日志带 TraceId，全链路可追踪。
- **健康检查**：Spring Boot Actuator `/actuator/health`。

---

## 四、Python 高并发

### 1. GIL 与多线程 / 多进程

- **结论**：GIL（全局解释器锁）导致 CPython 多线程**无法并行跑 CPU 密集型**任务；IO 密集型多线程仍然有效（等待 IO 时释放 GIL）。
- **选型**：
  - CPU 密集 → `multiprocessing` 多进程。
  - IO 密集 → `threading` 多线程 或 `asyncio` 协程。

```python
# 多进程：绕过 GIL，真正并行
from multiprocessing import Pool

def work(x):
    return x * x

with Pool(4) as p:
    result = p.map(work, range(100))
```

### 2. asyncio 协程

- **结论**：单线程内通过事件循环调度，IO 等待时让出控制权，适合高并发 IO（爬虫、网关、API）。
- **核心**：`async def` / `await` / `asyncio.create_task` / `asyncio.gather`。

```python
import asyncio
import httpx

async def fetch(url):
    async with httpx.AsyncClient() as client:
        return await client.get(url)

async def main():
    urls = ["http://a.com"] * 100
    # 并发执行 100 个请求
    results = await asyncio.gather(*[fetch(u) for u in urls])

asyncio.run(main())
```

- **高频面试题**：协程、线程、进程区别？事件循环原理？`asyncio.gather` 和 `wait` 区别？

### 3. FastAPI 异步

- **结论**：`async def` 端点跑在事件循环里；同步 `def` 端点会被 FastAPI 丢进线程池执行，避免阻塞循环。
- **高频面试题**：FastAPI 为什么快（Starlette + Pydantic + uvicorn/uvloop）？async def 和 def 端点区别？

### 4. 异步生态库

- 数据库：`asyncpg`（PostgreSQL）、`aiomysql`、`motor`（MongoDB）。
- 网络：`httpx`、`aiohttp`。
- 事件循环加速：`uvloop`（uvicorn 默认）。

### 5. Celery 任务队列

- **结论**：分布式异步任务队列，配合 Redis / RabbitMQ 做 broker，处理耗时任务（发邮件、报表、爬虫调度）。
- **高频面试题**：Celery 怎么保证任务不丢？重试机制？beat 定时任务？

---

## 五、面试进大厂

### 1. 高频面试题清单（Java）

- JVM：内存模型、GC、调优、OOM 排查。
- 并发：线程池、AQS、锁升级、volatile/CAS、ConcurrentHashMap、ThreadLocal。
- Spring：IOC/AOP、Bean 生命周期、循环依赖、事务失效场景。
- MySQL：索引（B+ 树）、事务隔离级别、MVCC、锁、慢查询优化。
- Redis：数据结构、持久化、主从/哨兵/集群、缓存三问题、分布式锁。
- MQ：可靠性、幂等、顺序消费、延迟队列。
- 分布式：CAP、BASE、一致性算法、分布式事务、限流熔断、分布式 ID（雪花算法）。

### 2. 算法刷题方向

- 数组/链表/栈/队列 → 二叉树/图 → 动态规划 → 贪心/回溯。
- 目标：LeetCode 高频 200 题，重点掌握 Hot 100。
- 套路：双指针、滑动窗口、哈希、快慢指针、单调栈、前缀和。

### 3. 系统设计（常见场景）

- **秒杀系统**：缓存预热、库存扣减（Redis 预扣 + MQ 异步落库）、限流、防超卖、防重复下单。
- **短链服务**：发号器 + Base62 编码 + 重定向。
- **IM 系统**：Netty-SocketIO / WebSocket + 心跳 + 消息可靠性 + 离线消息。
- **排行榜**：Redis ZSet。
- **设计套路**：先谈需求与量级（QPS/数据量），再分层（接入层 → 应用层 → 数据层），最后谈扩展与容灾。

### 4. 项目亮点怎么包装

- 用 STAR 法则讲项目：背景（Situation）→ 任务（Task）→ 行动（Action）→ 结果（Result）。
- 突出**量级**（QPS、数据量、压测结果）和**踩坑**（怎么发现、怎么解决）。
- 别只说「用了 Redis」，要说「为什么用、解决了什么问题、优化前后对比」。

---

## 附：速查表

| 主题 | 一句话答案 |
|------|-----------|
| 线程池核心线程数 | CPU 密集 ≈ 核数+1；IO 密集 ≈ 核数×2 |
| volatile | 可见性 + 禁重排，不保证原子性 |
| CAS 问题 | ABA，用版本号（AtomicStampedReference） |
| ConcurrentHashMap | 1.8 CAS + synchronized 锁单桶，红黑树 |
| ThreadLocal 泄漏 | 用完 remove() |
| 缓存穿透/击穿/雪崩 | 布隆过滤 / 互斥重建 / 随机过期 |
| 分布式锁 | Redisson（看门狗续期） |
| 消息不丢 | 生产 confirm + 持久化 + 消费手动 ack |
| 分布式事务 | 2PC / TCC / 最终一致性（Seata） |
| Python GIL | 多线程不能并行 CPU 任务，IO 密集可用线程/协程 |
| asyncio | 单线程事件循环，适合 IO 高并发 |
| FastAPI 快 | Starlette + Pydantic + uvloop |
