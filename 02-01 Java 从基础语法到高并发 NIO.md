---
title: "Java 从基础语法到高并发 NIO"
status: 已发布
author: XiaoYi
---

# Java 从基础语法到高并发 NIO

> 技术栈里「Java + SpringBoot」是后端主力。这篇不跟你整虚的，从 `import java` 讲起，一路干到**容器、IO、多进程、多线程、线程池、反射、代理、注解、加密、NIO**。 每个知识点都配能跑的代码，看完这篇，Java 面试题能答个七七八八。
>
> Let's use AI to create a better world。

## 一、Java 是啥？

一句话：**Java 是一门「一次编译、到处运行」的强类型面向对象语言，靠 JVM（Java 虚拟机）屏蔽了底层差异。**

```
你写的 .java 源码
    ↓ 编译（javac）
字节码 .class 文件
    ↓ 运行（JVM 解释执行）
任意装了 JVM 的平台（Windows / Linux / Mac）
```

|  |  |
| --- | --- |
| 概念 | 说明 |
| 定位 | 强类型、面向对象、跨平台 |
| 核心 | JVM —— Java 虚拟机，跨平台的秘密 |
| 打包 | `java -jar app.jar` 直接跑打好的 jar 包 |
| 生态 | SpringBoot、Netty、Hadoop、Kafka 都是它 |
| 你用它干嘛 | 后端接口、爬虫服务、分布式中间件 |

> 你技术栈里的 `Netty-SocketIO`、`RabbitMQ`、`SpringBoot`，底层全是 Java。学透这一篇，后面全是降维打击。

---

## 二、环境与第一个程序

装好 JDK，验证：

```
java -version
javac -version
```

第一个程序（记住：**Java 一个文件一个 public 类，类名 = 文件名**）：

```
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

编译 + 运行：

```
javac Hello.java
java Hello
```

> IDEA 三个高频快捷键：**`Alt + Enter`（万能修复 / 导入）**、**`Ctrl + F12`（查看类结构）**、**`Ctrl + Shift + F10`（运行当前文件）**。

---

## 三、基础语法速览

### 1. 八大基本类型 + 包装类

|  |  |  |
| --- | --- | --- |
| 基本类型 | 包装类 | 默认值 |
| byte | Byte | 0 |
| short | Short | 0 |
| int | Integer | 0 |
| long | Long | 0L |
| float | Float | 0.0f |
| double | Double | 0.0d |
| boolean | Boolean | false |
| char | Character | \u0000 |

> 记忆：**8 个基本类型，`long` 结尾加 `L`，`float` 结尾加 `F`。**

### 2. 装箱与拆箱

```
Integer intWrapper = 10; // 自动装箱
int base = intWrapper; // 自动拆箱
int intBase = intWrapper.intValue(); // 手动拆箱
Integer wrapper = Integer.valueOf(10); // 手动装箱 优先走缓存池 (-128!127)
int num = Integer.parseInt("10"); // 字符串转数字
```

> **装箱优先走缓存池**：`Integer.valueOf(10)` 在 `-128~127` 范围内直接复用缓存对象，这也是面试常问的「Integer 缓存」。

### 3. 访问修饰符 + 关键字

|  |  |
| --- | --- |
| 类别 | 内容 |
| 访问权限 | private < default(包访问) < protected < public |
| 类关键字 | class / interface / abstract / extends / implements / new / static / final |
| 流程控制 | if / else / switch / case / default / break / continue / return |
| 异常 | try / catch / throw / throws / finally |
| 其他 | import / package / super / this / void / true / false |

### 4. `==` vs `equals`（面试必考）

```
// == 比较: 基本类型比 [值], 引用类型 [地址]
// equals 比较: 逻辑值 (可重写)
String a = new String("hello");
String b = new String("hello");
a == b; // false
a.equals(b); // true
```

> **一句话：`==` 比「引用地址/基值」，`equals` 比「逻辑内容」（前提是重写了）。** 你自定义类想用 `equals` 比较内容，记得重写它。

---

## 四、常用类：String / StringBuilder / BigDecimal

### 1. String（不可变）

```
String str = "HelloJava";
char[] charArray = str.toCharArray();
str.length();
str.substring(0, 5);
str.contains("Java");
```

> `String` 底层是 `final` 字符数组，**不可变**。频繁拼接别用 `+`，用 `StringBuilder`。

### 2. StringBuilder（可变，线程不安全，快）

```
StringBuilder sb = new StringBuilder("");
sb.append("a").append("b");
int len = sb.length();
String s = sb.toString();
```

### 3. BigDecimal（金额计算，别用 double）

```
BigDecimal number = new BigDecimal("0");
BigDecimal otherNumber = new BigDecimal("0");
number.add(otherNumber);
number.subtract(otherNumber);
number.multiply(otherNumber);
number.divide(otherNumber);
```

> **金额必须用 `BigDecimal`，构造时传字符串**（`new BigDecimal("0")`），不能传 double，否则精度照样丢。

### 4. 时间 LocalDateTime

```
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String timeStr = now.format(dtf);
```

> 新项目统一用 `java.time` 包（`LocalDate` / `LocalDateTime`），别再碰老掉牙的 `Date` / `SimpleDateFormat`。

---

## 五、数组与容器

### 1. 数组

```
int[] arr = new int[10];
int[] arr2 = {1, 2, 3, 4, 5, 6};
```

### 2. 容器家族

```
Collection (单列集合接口)
    |—— List 有序可重复
    |    |__ ArrayList 数组实现 查快
    |    |__ LinkedList 链表实现 增删快
    |__ Set 无序不可重复
          |__ HashSet 纯无序
          |__ LinkedHashSet 维护插入顺序 (内部使用双链表)
          |__ TreeSet 自动排序 自然序

Map (双列键值对接口)
    |__ HashMap 纯无序
    |__ LinkedHashMap 维护插入顺序
    |__ TreeMap 自动按 key 排序 自然序
```

### 3. 遍历（三种）

```
for (Object item : collection) {}

Iterator iter = collection.iterator();
while (iter.hasNext()) { Object item = iter.next() }

ListIterator listIter = list.listIterator();
while (listIter.hasNext() && listIter.hasPrevious()) {
    Object item = listIter.next();
    Object item = listIter.previous();
}
```

> 遍历集合时要**删除元素**，用迭代器的 `remove()-->next()之后删除刚next()`，别用增强 for 直接删（会抛 `ConcurrentModificationException`）。

---

## 六、Stream 流

> **凡是实现了 `java.util.Collection` 接口的集合，都支持 `stream()`**，进而支持 `map()`、`filter()` 等链式操作，尾部用 `toList()` / `toSet()` 收集结果。

```
List<Integer> res = List.of(1, 2, 3, 4, 5).stream()
                        .filter(n -> n % 2 == 0)
                        .map(n -> n * 2)
                        .toList();

// 方法引用
list.forEach(System.out::println);
```

> Lambda `() -> {}` 和方法引用 `ClassName::function` 是 Stream 的灵魂，配合 `map/filter/reduce` 写起来像在写 SQL。

---

## 七、IO 流（字符流 / 字节流）

IO 流两个维度：**字节流 vs 字符流**、**输入流 vs 输出流**。

|  |  |
| --- | --- |
| 维度 | 分类 |
| 按数据单位 | 字节流（图片/视频）、字符流（文本） |
| 按方向 | 输入流（读）、输出流（写） |

### 1. 字节流（Buffered 封装提速）

```
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("path"));
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("path", true)); // true = 追加
```

> **Buffered 为什么快？** 它把源流包了一层缓冲区，减少系统从「用户态到内核态」的调用次数。缓冲区在 JVM 堆内存（用户态），默认 `1024 * 8` 字节即 **8KB**。

### 2. 字符流（指定编码）

```
BufferedReader br = new BufferedReader(
        new InputStreamReader(new FileInputStream("in.txt"), StandardCharsets.UTF_8));
BufferedWriter bw = new BufferedWriter(
        new OutputStreamWriter(new FileOutputStream("out.txt", true), StandardCharsets.UTF_8));
```

> 常用套路：`byte[] buf = new byte[1024]` / `read(buf)` / `"str".getBytes(StandardCharsets.UTF_8)` / `write(data)` / `flush()` / `close()`。

### 3. Hutool 一键读写（省事神器）

```
// in / out 使用以上对应 输入流 和 输出流 即可
List<String> lines = IoUtil.readLines(in, "UTF-8", new ArrayList<>());
String content = IoUtil.readUtf8(in); // 读完 文本类文件
byte[] bytes = IoUtil.readBytes(in); // 读完 二进制文件

// 写入
IoUtil.writeUtf8(content, false, out); // 内容, 是否追加, 输出流
IoUtil.copy(in, out); // 拷贝流
IoUtil.close(in); // 关闭流
```

> 生产里别手写一堆 `try-finally`，`cn.hutool.core.io.IoUtil` 一行搞定。这也是你技术栈常用库。

---

## 八、多进程：ProcessBuilder

Java 里能「拉起」别的程序（比如调用你的 Python 脚本）：

```
ProcessBuilder pb = new ProcessBuilder("python3", "test.py");
pb.directory(new File("/home/user")); // 设置子进程工作目录
pb.inheritIO(); // 子进程输入输出绑定到当前控制台
Process p = pb.start(); // 创建子进程 同JVM隔离
long pid = p.pid(); // 子进程 pid
boolean alive = p.isAlive();
int code = p.waitFor(); // 阻塞等待子进程结束
boolean done = p.waitFor(10, TimeUnit.SECONDS); // 最多等待10s, 超时返回false
p.destroy(); // 温和关闭
p.destroyForcibly(); // 强制杀死

long jvmPid = ProcessHandle.current().pid(); // 当前 JVM 自己pid
```

> 三个概念背下来：**进程 = 系统资源分配的基本单位**，具备**独立性、动态性、并发性**。子进程和 JVM 是完全隔离的两块内存。

---

## 九、多线程：三种开启方式

先分清两个词：

|  |  |
| --- | --- |
| 词 | 含义 |
| 并行 | 同一时刻，多个指令在**多个 CPU** 上同时执行 |
| 并发 | 同一时刻，多个指令在单个/多个 CPU 上**交替执行** |

> **线程 = CPU 调度的基本单位**，一个进程里能同时跑多个任务（线程）。并行/并发由操作系统调度，你只管写。

### 方式一：继承 Thread

```
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("线程任务执行" + i);
        }
    }
}
// 启动
MyThread mt = new MyThread();
mt.start();
```

### 方式二：实现 Runnable（无返回值，可再继承别的类）

```
class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("自己的线程任务" + i);
        }
    }
}
// 启动（共用同一个 Runnable 实例，容易发生线程安全竞争）
MyRunnable mr = new MyRunnable();
new Thread(mr, "threadName").start();
```

### 方式三：实现 Callable（有返回值）

```
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i <= 99; i++) sum += i;
        return sum;
    }
}
// 启动
FutureTask<Integer> task = new FutureTask<>(new MyCallable());
new Thread(task, "threadName").start();
Integer result = task.get();   // 阻塞方法，拿到返回值前会卡住
```

### 线程常用 API

```
thread.getName();                 // 获取线程名
thread.setName("threadName");     // 设置线程名
Thread.currentThread();           // 获取当前线程（静态方法）
Thread.sleep(1000);               // 休眠 1000ms（静态方法）
thread.setPriority(10);           // 设置优先级 1~10（默认 5，越大抢到 CPU 概率越大）
thread.setDaemon(true);           // 设为守护线程
```

> **大坑**：直接写 `thread.run()` 只是普通方法调用，**不会开启新线程**，会在主线程里串行执行！必须用 `start()`。

### 中断标签机制 + ThreadLocal

```
thread.interrupt();                        // 打中断标记（协作式，不是强杀）
boolean wasInterrupted = Thread.interrupted(); // 静态：返回并清除当前线程的中断标记
Thread.currentThread().isInterrupted();    // 实例：查询当前线程是否被中断
thread.isInterrupted();                    // 查询指定线程的中断标记

// ThreadLocal：线程局部变量，每个线程一份，互不干扰
ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("value");
String value = threadLocal.get();
threadLocal.remove();                      // 用完记得 remove，防内存泄漏
```

> Java 是**抢占式调度**（允许随机性执行），所以多线程结果可能每次不一样，这是特性不是 bug。

---

## 十、线程安全与锁

**问题根源**：线程间资源默认共用，并行/并发让代码「分线程分行块」执行，产生数据错乱（比如超卖）。

> **锁对象可以是任意对象，但多条线程必须用「同一把锁」。** 锁能解决数据安全，但也会降低效率。

### 1. synchronized（同步代码块 / 同步方法）

```
class TicketTask1 implements Runnable {
    int tickets = 100;

    @Override
    public void run() {
        while (true) {
            synchronized (TicketTask1.class) {   // 同步代码块，锁 = 类字节码对象
                if (tickets == 0) break;
                System.out.println(Thread.currentThread().getName() + "卖出第" + tickets + "张票");
                tickets--;
            }
        }
    }
}
// 同步方法：static 锁类对象，非 static 锁 this
// static synchronized void method() {}
// private synchronized void method() {}
```

### 2. ReentrantLock（更灵活，推荐）

```
class TicketTask2 implements Runnable {
    int tickets = 100;
    ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            try {
                lock.lock();                 // 上锁
                if (tickets == 0) break;
                System.out.println(Thread.currentThread().getName() + "卖出第" + tickets + "张票");
                tickets--;
            } finally {
                lock.unlock();               // 必须在 finally 里解锁
            }
        }
    }
}
```

> 对比：`synchronized` 语法简单，`ReentrantLock` 能手动控制、可中断、可公平锁。**核心记住 `lock.lock()` 和 `lock.unlock()` 成对出现，unlock 放 finally。**

---

## 十一、线程池

> **线程池用来减少线程创建/销毁的开销**——线程复用，别来一个任务 new 一个线程。

JDK 自带 `Executors` 能快速建池，但**开发中通常不咋用**（无法精细配置），推荐自定义 `ThreadPoolExecutor`。

```
ThreadPoolExecutor pool = new ThreadPoolExecutor(
        2,                                    // 核心线程数（正式员工）
        5,                                    // 最大线程数（正式 + 临时工）
        60, TimeUnit.SECONDS,                 // 空闲时间 + 单位
        new ArrayBlockingQueue<>(10),         // 任务队列（有界，指定排队人数）
        Executors.defaultThreadFactory(),     // 线程工厂
        new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);

for (int i = 1; i <= 16; i++) {
    pool.submit(() -> System.out.println(Thread.currentThread().getName() + " 提交了任务"));
}
```

**7 个核心参数**：核心线程数、最大线程数、空闲时间、时间单位、任务队列、线程工厂、拒绝策略。

**两条判断铁律**（面试爱考）：

```
临时线程什么时候创建？ → 任务数 > 核心线程数 + 队列容量
拒绝策略什么时候触发？ → 任务数 > 最大线程数 + 队列容量
```

**四种拒绝策略**：

|  |  |
| --- | --- |
| 策略 | 行为 |
| AbortPolicy（默认） | 丢弃任务并抛 `RejectedExecutionException` |
| DiscardPolicy | 丢弃任务，不抛异常（不推荐） |
| DiscardOldestPolicy | 抛弃队列里等最久的任务，再收当前任务 |
| CallerRunsPolicy | 由调用线程自己执行，绕过线程池，到主线程里面执行 |

```
// 自定义拒绝策略示例
new ThreadPoolExecutor.AbortPolicy() {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        log.error("线程池饱和, 任务被拒绝");
        super.rejectedExecution(r, e);
    }
}
```

> 队列分两种：**有界队列** `ArrayBlockingQueue<>(N)`、**无界队列** `LinkedBlockingQueue<>()`。生产建议用有界 + 明确拒绝策略，别让任务无限堆积撑爆内存。

---

## 十二、枚举

枚举 = **一行就是一条预设好的记录**，适合错误码、状态机这种固定常量集。

```
enum ErrorCode {
    // 一行就是一条预设好的记录
    SUCCESS(0, "ok", ""),
    PARAMS_ERROR(40000, "请求参数错误", ""),
    NULL_ERROR(40001, "请求数据为空", ""),
    NOT_LOGIN(40100, "未登录", ""),
    NO_AUTH(40101, "无权限", ""),
    SYSTEM_ERROR(50000, "系统内部异常", "");

    // 三个"格子"，用来存上面每一行的3个信息
    private final int code;
    private final String message;
    private final String description;

    // 构造：就是把括号里的三个值，放进上面三个格子里
    ErrorCode(int code, String message, String description) {
        this.code = code;
        this.message = message;
        this.description = description;
    }
}
```

> 这跟你技术栈里 `ErrorCode + BusinessException + 全局异常拦截` 的统一返回规范是同一套思想——错误码用枚举集中管理，别到处散落魔法数字。

---

## 十三、类加载器与双亲委派

### 1. 类加载三阶段

```
加载 → 链接 → 初始化
加载：按「包名 + 类名」找到类，用流读进内存，创建一个 Class 对象
链接：验证（是否符合 JVM 规范）→ 准备（给 static 变量分配空间、设默认值）→ 解析
初始化：按程序员代码的主观计划，初始化类变量和资源
```

> 加载时机：**用到的时候才加载**（懒加载）。

### 2. 类加载器层次（逻辑父子关系，非 extends）

```
Bootstrap ClassLoader   启动类加载器（C 写的，拿不到引用 → null），加载 JDK 自带类
        ↓ parent
Platform ClassLoader    平台类加载器，加载 lib/modules（JDK9 前叫扩展加载器，加载 jre/lib/ext）
        ↓ parent
Application ClassLoader 应用程序类加载器，加载你 classpath 下的类
        ↓ parent
自定义类加载器            上级是 Application

xxx.class.getClassLoader();   // 获取该类的类加载器
```

### 3. 双亲委派模式

> 收到加载请求，**自己不先加载，先委托父加载器**；父加载器还不行就再往上，一路递归到顶层 Bootstrap；父能加载就返回，父加载不了，子才自己加载。

好处：**保证核心类不被篡改**（比如你自己写个 `java.lang.String` 也加载不进去，因为顶层早就加载了官方的）。

---

## 14. 反射：框架的灵魂

> 反射：**运行时动态获取任意类的属性和方法，并动态调用**。你自己写的代码你当然知道，但框架（Spring 等）不知道，它要靠反射去操作你写的类。

### 获取字节码对象（三种）

```
Class.forName("全类名");   // 全类名 = 包名.类名
类名.class;
对象.getClass();
```

### 获取成员 / 方法并调用

```
Class clazz = obj.getClass();
Field[] fields = clazz.getDeclaredFields();  // 所有成员变量
Method[] methods = clazz.getDeclaredMethods(); // 所有方法

for (Field field : fields) {
    field.getName();   // 变量名
    field.getType();   // 变量类型
}
for (Method method : methods) {
    method.getName();        // 方法名
    method.getReturnType();  // 返回值类型
    for (Parameter p : method.getParameters()) {  // JDK8+ 支持
        p.getName();
        p.getType();
    }
}

// 调用某个方法
if ("func".equals(method.getName())) {
    method.setAccessible(true);        // 调用私有方法前先赋权
    method.invoke(clazz.newInstance(), args);  // 反射调用
}
```

> 读资源文件常用：`类字节码对象.getClassLoader().getResourceAsStream("file.txt")`，根路径是 `target/classes`（即 classpath）。

---

## 十五、代理：静态与动态

### 1. 静态代理（手动写代理类）

```
public interface Sale {
    void sale(double money);
}
public class Owner implements Sale {
    public void sale(double money) {
        System.out.println("出售商品, 获得" + money);
    }
}
public class SaleProxy implements Sale {
    private Owner owner;                    // 持有被代理对象
    public SaleProxy(Owner owner) { this.owner = owner; }
    public void sale(double money) {
        money += 20000;                     // 增强：抽成
        owner.sale(money);                  // 调原生逻辑
    }
}
```

> 缺点：每代理一个类就要手写一个代理类，代码膨胀。

### 2. 动态代理（JDK 反射自动生成）

```
UserService target = new UserServiceImpl();   // 原始目标对象
UserService proxy = (UserService) Proxy.newProxyInstance(
        ClassLoader.getSystemClassLoader(),   // 参数1：类加载器
        new Class[]{UserService.class},       // 参数2：要实现的接口
        new InvocationHandler() {             // 参数3：所有方法调用都进 invoke
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if ("addUser".equals(method.getName())) {
                    long start = System.currentTimeMillis();  // 前置增强
                    Object result = method.invoke(target, args); // 调原生方法
                    long end = System.currentTimeMillis();    // 后置增强
                    System.out.println("耗时：" + (end - start) + "ms");
                    return result;
                }
                return method.invoke(target, args);
            }
        }
);
proxy.addUser("张三");   // ★用代理对象调用，才会触发增强
```

> 关键：**JDK 动态代理基于接口**，`InvocationHandler` 的 `invoke` 是统一入口，前置/后置增强逻辑都写这里。你后面学 Spring AOP 会发现原理一模一样。

---

## 十六、注解与 AOP

### 1. 常见内置注解

```
@Override              // 标识重写
@Deprecated            // 标记已过时
@SuppressWarnings("all") // 压制警告
```

### 2. 自定义注解

```
@Target(ElementType.METHOD)          // 能加在哪：TYPE/FIELD/METHOD/PARAMETER/CONSTRUCTOR/LOCAL_VARIABLE
@Retention(RetentionPolicy.RUNTIME)  // 保留到哪：SOURCE/CLASS/RUNTIME
public @interface MyAnnotation {
    String attr() default "defaultValue";  // 属性
}
```

### 3. AOP 增强（切面）

```
@Aspect
@Component
public class MyAnnotationAspect {
    @Pointcut("@annotation(com.example.project.annotation.MyAnnotation)")
    public void pointcut() {}

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        MyAnnotation annotation = signature.getMethod().getAnnotation(MyAnnotation.class);
        String attr = annotation.attr();          // 读注解属性

        long start = System.currentTimeMillis();  // 前置增强
        Object result = joinPoint.proceed();      // 执行目标方法
        long cost = System.currentTimeMillis() - start;
        System.out.println("方法耗时：" + cost + "ms");
        return result;
    }
}
```

> AOP = **注解 + 动态代理 + 反射**三件套的合体。理解了这个，Spring 的「日志、事务、权限」切面就通透了。

---

## 十七、密码加密

**核心原则：密码绝不能明文存库，必须加盐（SALT）哈希。**

### 1. MD5（加盐）

```
private static final String SALT = "盐值";
public static String md5(String rawPwd) {
    String content = rawPwd + SALT;
    return DigestUtils.md5DigestAsHex(content.getBytes(StandardCharsets.UTF_8));
}
```

### 2. SHA-256（加盐）

```
public static String sha256(String rawPwd) throws NoSuchAlgorithmException {
    String content = SALT + rawPwd;
    MessageDigest md = MessageDigest.getInstance("SHA-256");
    byte[] hashBytes = md.digest(content.getBytes(StandardCharsets.UTF_8));
    return bytesToHex(hashBytes);
}
private static String bytesToHex(byte[] bytes) {
    StringBuilder sb = new StringBuilder();
    for (byte b : bytes) {
        String hex = Integer.toHexString(0xff & b);
        if (hex.length() == 1) sb.append('0');
        sb.append(hex);
    }
    return sb.toString();
}
```

### 3. BCrypt（自带随机盐，生产首选）

```
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(10); // 算力 4~31，线上一般 10
String hashPwd = encoder.encode(rawPwd);                       // 加密（自带随机盐）
boolean match = encoder.matches(rawPwd, hashPwd);              // 校验
```

> **MD5/SHA 要手动加盐，BCrypt 自带随机盐且不可逆**。生产密码用 `BCrypt`（来自 `org.springframework.security`），最省心。

---

## 十八、NIO：高并发的秘密

> **NIO（New/Non-blocking I/O）= Java 为了弥补传统 `java.io` 性能不足，为高并发、大吞吐提供的方案。**

NIO 四大特性：

|  |  |
| --- | --- |
| 特性 | 说明 |
| 面向缓冲区 | Buffer（缓冲区）读写 |
| 非阻塞 | 不等，没数据就返回 |
| 有 Selector | **单线程管多连接**（核心优势） |
| 双向 | Channel（通道）可读可写 |

```
// 服务端
Selector selector = Selector.open();
ServerSocketChannel server = ServerSocketChannel.open();
server.bind(new InetSocketAddress(8080));
server.configureBlocking(false);                     // 非阻塞
server.register(selector, SelectionKey.OP_ACCEPT);   // 监听「接受连接」事件

while (true) {
    selector.select();                               // 阻塞，直到有事件就绪
    Iterator<SelectionKey> it = selector.selectedKeys().iterator();
    while (it.hasNext()) {
        SelectionKey key = it.next();
        it.remove();                                 // ★必须移除，否则重复处理

        if (key.isAcceptable()) {                    // 新连接
            SocketChannel sc = server.accept();
            sc.configureBlocking(false);
            sc.register(selector, SelectionKey.OP_READ);
        } else if (key.isReadable()) {               // 可读
            SocketChannel sc = (SocketChannel) key.channel();
            ByteBuffer buf = ByteBuffer.allocate(1024);
            int n = sc.read(buf);
            if (n == -1) { sc.close(); continue; }   // ★对端关闭
            buf.flip();                              // 写模式 -> 读模式
            String msg = StandardCharsets.UTF_8.decode(buf).toString();
            sc.write(ByteBuffer.wrap(("收到: " + msg).getBytes()));
        }
    }
}

// 客户端
SocketChannel sc = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8080));
sc.write(ByteBuffer.wrap("hello".getBytes()));
ByteBuffer buf = ByteBuffer.allocate(1024);
sc.read(buf);
buf.flip();
System.out.println(StandardCharsets.UTF_8.decode(buf));
sc.close();
```

> 你技术栈里的 **Netty 底层就是 NIO**。看懂上面这段 `Selector + Channel + Buffer`，Netty 再往下走就是封装和优化，门槛已经过了。

## 十九、JUnit

```
// 单元测试 --> 集成测试

public class UserServiceTest {

  @BeforeAll
  public static void testBeforeAll() {
    System.out.println("所有测试开始");
  }

  @AfterAll
  public static void testAfterAll() {
    System.out.println("所有测试结束");
  }

  @BeforeEach
  public void testBefore() {
    System.out.println("测试开始");
  }

  @AfterEach
  public void testAfter() {
    System.out.println("测试结束");
  }

  @Test // @Test 标记的方法必须为 public void 修饰 底层原因与规范
  @DisplayName("测试获取用户年龄") // 指定测试名字
  public void testGetAge() {
    Integer age = new UserService().getAge("id");

    Assertions.assertEquals(18, age, "两个值相等");
    Assertions.assertNotEquals(16, age, "两个值不相等");
    Assertions.assertNull(age, "值为 null");
    Assertions.assertNotNull(age, "值不为 null");
    Assertions.assertTrue(age > 16, "大于 16"); // 检查条件是否为true
    Assertions.assertFalse(age < 16, "小于 16"); // 检查条件是否为false
    Assertions.assertSame(age, age, "两个对象引用相等"); // 检查两个对象引用地址是否相等
  }
}
```

---

## 速查表

|  |  |
| --- | --- |
| 要做什么 | 用什么 |
| 金额计算 | `BigDecimal("0")`，传字符串 |
| 频繁拼接字符串 | `StringBuilder` |
| 时间格式化 | `LocalDateTime` + `DateTimeFormatter` |
| 有序可重复集合 | `ArrayList` / `LinkedList` |
| 去重 | `HashSet` / `LinkedHashSet` |
| 链式处理集合 | `stream().map().filter().toList()` |
| 读大文件 | `BufferedInputStream` + 循环读 8KB |
| 调 Python 脚本 | `ProcessBuilder` |
| 开线程（无返回） | 继承 `Thread` / 实现 `Runnable` |
| 开线程（有返回） | 实现 `Callable` + `FutureTask.get()` |
| 线程安全计数 | `synchronized` / `ReentrantLock` |
| 复用线程 | `ThreadPoolExecutor`（7 参数） |
| 固定常量集 | `enum` |
| 动态操作类 | 反射 `Class` / `Method.invoke` |
| 无侵入增强方法 | 动态代理 `Proxy.newProxyInstance` |
| 密码存储 | `BCryptPasswordEncoder` |
| 高并发网络 | NIO `Selector` + `Channel` |
