---
title: "SpringBoot 从入门到生产级后端"
status: 已发布
author: XiaoYi
---

# SpringBoot 从入门到生产级后端

> 上一篇把 Java 讲透了，这篇直接上你后端主力 SpringBoot。这一篇不是「Hello World」玩具教程，而是把一套能上生产的后端骨架完整拆给你看：分层架构、MyBatis-Plus、统一返回、全局异常、JWT 鉴权、Redis 高级玩法、定时任务、邮件推送、完整配置。
> 关键原理全部讲透 + 标注「大厂考点」，学完这篇，SpringBoot 面试和写项目都够用。
> Let's use AI to create a better world。

## 一、SpringBoot 是啥？

一句话：**SpringBoot 是对 Spring 全家桶的「一键封装」——约定大于配置，让你几行代码就搭起一个带内嵌服务器、自动配置好的 Web 后端。**

```
传统 Spring：手写一堆 XML 配置 + 自己装 Tomcat + 自己管理依赖版本
        ↓ SpringBoot 帮你干了
开箱即用：起步依赖 + 内嵌 Tomcat + 自动配置，专注写业务
```

|  |  |
| --- | --- |
| 概念 | 说明 |
| 定位 | 快速构建生产级 Spring 应用的脚手架 |
| 核心思想 | 约定大于配置（能自动的绝不让你手写） |
| 内嵌服务器 | Tomcat（默认）/ Jetty / Undertow |
| 配置方式 | application.yml / properties（零 XML） |
| 你用它干嘛 | 后端接口、微服务、中间件、你技术栈的整个后端 |

---

## 二、环境准备：JDK + Maven

### 1. 两个环境变量

```
%JAVA_HOME%   → JDK 安装目录
%MAVEN_HOME%  → Maven 安装目录
```

装完验证：

```
java -version
mvn -version
```

### 2. Maven 的配置与仓库

Maven 核心就两个东西：**`conf/settings.xml`（全局配置）+ `repository`（本地仓库）**。

|  |  |
| --- | --- |
| 配置项 | 作用 |
| 镜像 | 加速依赖下载（国内一般用阿里云镜像） |
| 自定义仓库 | 指定私有仓库地址 |
| 本地仓库 | 依赖下载后存放的地方（默认 `~/.m2`） |

> **重要**：SpringBoot 项目需要**对应符合版本的 Maven**（SpringBoot 3 要配 Maven 3.6+，太老的 Maven 会报版本不支持）。

### 3. POM 与 mvnw

- **POM（Project Object Model）**：项目对象模型，`pom.xml` 里描述项目依赖、构建配置，是 Maven 的核心文件。

- **`mvnw` / `mvnw.cmd`**：Maven 构建脚本（Wrapper 包装器）。**不用提前装 Maven**，启动脚本会自动下载符合项目版本的 Maven，保证团队构建环境一致。

```
mvn spring-boot:run        # 直接运行项目
```

> 大厂考点：`mvnw`（Maven Wrapper）解决「团队 Maven 版本不一致」问题，CI/CD 里统一用 `./mvnw` 而不是全局 `mvn`。

---

## 三、架构基础：C/S vs B/S、静态与动态资源

|  |  |
| --- | --- |
| 概念 | 说明 |
| C/S 架构 | Client/Server，客户端-服务器（桌面软件、App） |
| B/S 架构 | Browser/Server，浏览器-服务器（Web 应用，主流） |
| 静态资源 | 固定不变：HTML、CSS、JS、图片 |
| 动态资源 | 每次请求动态生成：接口返回的 JSON 数据 |

> 现在 99% 的后端做的都是 **B/S 架构 + 动态资源**：前端浏览器/App 请求你的接口，你返回 JSON。

---

## 四、起步依赖与启动流程

### 1. 起步依赖（Starter）

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

> **大厂考点：Maven 依赖传递性**——`spring-boot-starter-web` 内部依赖了 `spring-boot-starter-tomcat`，所以**你引入 web 起步依赖时，Tomcat 依赖已经悄悄进来了**。你运行 `main` 方法，其实启动的就是 SpringBoot **内嵌的 Tomcat**，你的项目自动部署到这个服务器上。

**常用起步依赖清单**：

|  |  |
| --- | --- |
| 依赖 | 作用 |
| spring-boot-starter-web | Web 开发（含内嵌 Tomcat） |
| mybatis-plus-boot-starter | MyBatis-Plus 集成 |
| mysql-connector-j | MySQL 驱动 |
| lombok | 简化实体类（@Data） |
| spring-boot-devtools | 热部署（开发用） |
| spring-boot-configuration-processor | 配置提示补全 |
| spring-boot-starter-data-redis | Redis 集成 |
| spring-boot-starter-mail | 邮件发送 |

### 2. 启动流程（大厂必考）

```
main 方法
   ↓
SpringApplication.run()
   ↓
读取 application.yml / application.properties
   ↓
创建 Spring 容器（IOC）
   ↓
自动配置（Auto Configuration）
   ↓
启动内嵌 Tomcat，部署项目
```

> 五个词背下来：**`main` → `run()` → 读配置 → 建容器 → 自动配置 → 启动 Tomcat`**。

### 3. banner.txt 水印

在 `resources` 目录新建 `banner.txt`，把水印文案写进去，**启动项目就会显示你自定义的水印**（可以放 ASCII 艺术字）。

---

## 五、分层架构与核心概念

### 1. 三层架构

```
Controller（控制层）→ 接收请求、参数校验、返回结果
        ↓
Service（业务层）→ 业务逻辑、事务
        ↓
Mapper（持久层）→ 操作数据库
```

> 原则：**高内聚、低耦合**——每层只干自己的事，通过接口解耦，方便替换和维护。

### 2. 三个对象（大厂高频）

|  |  |  |
| --- | --- | --- |
| 缩写 | 全称 | 用途 |
| PO | Persisted Object | 持久层对象，对应数据库表 |
| DTO | Data Transfer Object | 传输对象，接收前端入参 |
| VO | View Object | 视图对象，返回给前端的展示数据 |

### 3. IOC / DI / AOP（大厂必考三件套）

|  |  |  |
| --- | --- | --- |
| 缩写 | 全称 | 含义 |
| IOC | Inversion of Control | 控制反转：对象的创建交给 Spring 容器管理 |
| DI | Dependency Injection | 依赖注入：容器把依赖自动注入进来 |
| AOP | Aspect Oriented Programming | 面向切面编程：无侵入地加日志/事务/权限 |

> **Bean = 在 IOC 容器中创建、管理的对象。** 你写的 `@Service`、`@Controller` 都是 Bean，由 Spring 负责 new、负责装配、负责销毁。

### 4. 四个注解 + 注入方式（大厂高频）

|  |  |
| --- | --- |
| 注解 | 作用 |
| @Component | 通用组件 |
| @Controller | 控制层 |
| @Service | 业务层 |
| @Repository | 持久层（Mapper） |

```
// @Autowired：按类型注入（byType）
// @Resource ：按名称注入（byName）

@Autowired
private UserService userService;

@Resource
private UserMapper userMapper;

// 生产使用 参数构造注入
@RequiredArgsConstructor
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    private final UserLoginLogMapper userLoginLogMapper;
    private final VerificationCodeService verificationCodeService;
    private final BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder(10);
}
```

> 大厂考点：**`@Autowired` 默认 byType，`@Resource` 默认 byName**。接口有多个实现类时，`@Autowired` 会因类型冲突报错，得配合 `@Qualifier(beanName)`或改用 `@Resource(name=...)`。

---

## 六、配置文件 application.yml

### 1. 一个大坑（大厂考点）

```
# 0 开头的数据在 yml 里表示「八进制」！
phone: 0577          # 会解析成 8 进制，出问题
phone: "0577"        # ✅ 务必用引号包起来
```

> **注意**：`0` 开头的数据在 yml 中表示八进制，遇到电话号码、验证码这类「0 开头」的数据，**务必加引号**。

### 2. 多环境配置

```
spring:
  profiles:
    active: dev   # 激活 dev 环境，可切 prod
```

> 用 `application-dev.yml`、`application-prod.yml` 区分环境，通过 `profiles.active` 一键切换。

### 3. 完整配置见文末 [第十八章](#十八完整-applicationyml--logback)。

---

## 七、MyBatis-Plus 全攻略

MyBatis-Plus 是你技术栈的 ORM 主力。记住三件套：**`PO 实体类` — `Mapper 接口` — `mapper.xml`**。

### 1. MySQL 与 Java 类型对应

|  |  |
| --- | --- |
| MySQL | Java |
| TINYINT(1) | Boolean |
| INT | Integer |
| BIGINT | Long |
| DECIMAL(N, M) | BigDecimal |
| CHAR/VARCHAR/TEXT | String |
| DATETIME | LocalDateTime |

### 2. PO 实体类（核心注解）

```
@Data
@TableName("user")                        // 表名
public class User implements Serializable {

    private static final long serialVersionUID = 1L; // 不手写，JDK 自动分配

    @TableId(value = "id", type = IdType.AUTO) // 主键自增；分库分表用 IdType.ASSIGN_ID（雪花算法）
    private Long id;

    @TableField("user_name")              // 字段映射（驼峰转下划线）
    private String userName;

    @TableLogic(value = "0", delval = "1") // 逻辑删除：删除时改成 1，其他操作自动过滤
    private Integer deleted;              // 0 未删除，1 已删除

    @TableField(exist = false)            // 数据库里不存在的字段
    private String displayName;
}
```

> **大厂考点**：`serialVersionUID` 是序列化版本号。**改了类结构 JVM 会重新生成 UID**，反序列化时对不上就抛 `InvalidClassException`。跨系统传对象（Redis/RPC/MQ）务必显式固定它。实现 `Serializable` 接口是为了支持序列化/反序列化（Redis、RPC、Session 钝化、消息队列都要用）。

### 3. Mapper 接口（两种写法）

```
// 方式一：继承 BaseMapper，自带 CRUD，零 SQL
public interface UserMapper extends BaseMapper<User> {
    // 复杂查询才写方法
}

// 方式二：注解式 SQL（简单 SQL 直接写）
@Select("SELECT * FROM user WHERE id = #{id}")
User selectById(@Param("id") Long id);

@Insert("INSERT INTO user (user_name) VALUES (#{userName})")
int insert(User user);

@Update("UPDATE user SET user_name = #{userName} WHERE id = #{id}")
int updateById(@Param("id") Long id, @Param("userName") String userName);

@Delete("DELETE FROM user WHERE id = #{id}")
int deleteById(@Param("id") Long id);
```

> Mapper 接口需要 `@Mapper` 才能注入 Bean，但开发通常**在启动类上加 `@MapperScan("cn.xxx.mapper")`** 一次扫完。

### 4. mapper.xml（复杂 SQL）

```
<resultMap id="BaseResultMap" type="cn.xiaoyi.project.model.po.User">
    <id property="id" column="id" />
    <result property="userName" column="user_name" />
</resultMap>

<sql id="Base_Column_List">
    id, user_name, deleted
</sql>

<select id="selectById" resultType="cn.xiaoyi.project.model.po.User">
    SELECT <include refid="Base_Column_List"/> FROM user WHERE id = #{id}
</select>

<insert id="insertUser">
    INSERT INTO user (user_name) VALUES (#{userName})
</insert>

<update id="updateUser">
    UPDATE user SET user_name = #{userName} WHERE id = #{id}
</update>

<delete id="deleteById">
    DELETE FROM user WHERE id = #{id}
</delete>
```

> `BaseMapper` 与 `IService` 二选一：`BaseMapper` 直接操作单表，`IService` 封装了更多批量/事务方法，业务层更常用 `IService`。

### 5. 条件构造器 Wrapper（大厂高频）

```
LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
queryWrapper.eq(User::getUserName, "小翊")        // 等于
        .ne(User::getId, 1)                      // 不等于
        .gt(User::getAge, 18)                    // 大于
        .ge(User::getAge, 18)                    // 大于等于
        .lt(User::getAge, 60)                    // 小于
        .le(User::getAge, 60)                    // 小于等于
        .like(User::getUserName, "小")            // 模糊
        .between(User::getAge, 18, 30)           // 区间
        .in(User::getId, List.of(1, 2, 3))       // 包含
        .isNotNull(User::getEmail)               // 非空
        .orderByAsc(User::getId)                 // 升序
        .orderByDesc(User::getId)                // 降序
        .and(child -> child.eq(User::getSex, 1).eq(User::getStatus, 0))  // 且
        .or(child -> child.eq(User::getSex, 2)); // 或
```

> **用 `LambdaQueryWrapper` 而不是字符串列名**，因为 `User::getUserName` 这种 Lambda 写法是**编译期校验**，字段名改错会编译报错，杜绝了 `"user_name"` 手写错的问题。

### 6. 事务（大厂高频）

```
@Transactional(rollbackFor = Exception.class) // 触发异常就回滚
public void transfer() {
    // 通常回滚 INSERT / UPDATE / DELETE
    // 切勿吞掉异常，可主动抛出异常触发回滚
}
```

> **大厂考点：事务失效的常见场景**——① 方法不是 `public`；② 同类内部方法调用（this 调用不走代理）；③ 异常被 try-catch 吞掉没抛出；④ `rollbackFor` 没配，默认只回滚 `RuntimeException`，受检异常不回滚。所以**务必配 `rollbackFor = Exception.class`**。

### 7. 乐观锁 vs 悲观锁（大厂必考）

锁的作用：**解决多个请求同时修改同一条数据时的数据错乱，配合事务使用。**

**乐观锁（版本号）**——推荐，性能好：

```
// PO 里加版本号字段
@Version
private Long version;

// 配置拦截器
@Configuration
public class MyBatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

> 原理：更新时**自动拼接 `WHERE version = 旧值` 并自动 `version + 1`**。多个请求同时改，只有第一个成功，后面的发现版本号对不上就更新失败。

**悲观锁**——分两种：**共享锁（S 锁 / 读锁）** 和 **排他锁（X 锁 / 写锁）**：

|  |  |  |  |
| --- | --- | --- | --- |
| 类型 | 关键字 | 别称 | 含义 |
| 共享锁 | `FOR SHARE`（MySQL 8.0）/ `LOCK IN SHARE MODE`（5.7） | S 锁 / 读锁 | 别人也能加 S 锁一起读，但不能加 X 锁改 |
| 排他锁 | `FOR UPDATE` | X 锁 / 写锁 | 别人既不能加锁读、也不能改，只有我能改 |

> 一句话区分：**共享锁 = 大家都能看、不能改；排他锁 = 只有我一个人能看能改。**

悲观锁不能在通用 `BaseMapper` 里直接开，要**自定义 Mapper 方法**写对应的 SQL：

```
public interface UserMapper extends BaseMapper<User> {
    // 共享锁：读出来，允许别人也读，但不允许别人改
    @Select("SELECT * FROM user WHERE id = #{id} FOR SHARE")
    User selectByIdForShare(@Param("id") Long id);

    // 排他锁：读出来，别人既不能读也不能改，只有我能改
    @Select("SELECT * FROM user WHERE id = #{id} FOR UPDATE")
    User selectByIdForUpdate(@Param("id") Long id);
}
```

**关键：必须在事务里调用，行锁才会保持到事务结束。**典型场景「下单扣库存」：

```
@Service
public class OrderService {

    @Resource
    private UserMapper userMapper;

    // 事务保证：查 + 改 在同一个事务里，行锁从头锁到尾
    @Transactional(rollbackFor = Exception.class)
    public void deductStock(Long id) {
        // 1. 查出来，同时用 FOR UPDATE 锁住这一行
        User user = userMapper.selectByIdForUpdate(id);

        // 2. 判断 + 修改（此时其他事务改不了这一行，只能等）
        if (user.getStock() <= 0) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "库存不足");
        }
        user.setStock(user.getStock() - 1);
        userMapper.updateById(user);

        // 3. 事务提交后，行锁自动释放
    }
}
```

> 为什么必须加 @Transactional：FOR UPDATE 的行锁随事务提交/回滚才释放。如果不包事务，selectByIdForUpdate 执行完锁就没了，等于白加。
> 另一个坑：别用 queryWrapper.last("FOR UPDATE") 这种拼接方式，有 SQL 注入风险，直接自定义 Mapper 方法最安全。

|  |  |  |
| --- | --- | --- |
| 对比 | 乐观锁 | 悲观锁 |
| 实现 | 版本号（@Version） | 数据库行锁（FOR UPDATE） |
| 加锁时机 | 提交时才判断 | 读取时就加锁 |
| 性能 | 高（无锁等待） | 低（会阻塞） |
| 适用场景 | 读多写少、冲突少 | 写多、强一致性要求 |

### 8. 分页查询

```
@Configuration
public class MyBatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}

// 使用
Page<User> page = new Page<>(current, size);  // 页码从 1 开始
LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
queryWrapper.like(username != null, User::getUserName, username)
        .eq("deleted", 0);                    // 逻辑删除字段自动过滤

IPage<User> result = userService.page(page, queryWrapper);
List<User> userList = result.getRecords();   // 当前页数据
long total = result.getTotal();              // 总记录数
long pages = result.getPages();              // 总页数
long cur = result.getCurrent();              // 当前页
if (result.hasNext()) {                      // 是否还有下一页
    System.out.println("还有下一页");
}
```

> 小技巧：`like(condition, ...)` 第一个参数是条件，`username != null` 才拼接这个条件，**动态 SQL 一行搞定**。

---

## 八、统一返回 + 全局异常

这是你技术栈的规范核心：**`BaseResponse` 统一返回 + `ErrorCode` 错误码 + `BusinessException` 业务异常 + 全局异常拦截**。

### 1. 错误码枚举

```
@Getter
public enum ErrorCode {
    SUCCESS(0, "OK", ""),
    PARAMS_ERROR(40000, "请求参数错误", ""),
    NULL_ERROR(40001, "请求数据为空", ""),
    NOT_LOGIN(40100, "未登录", ""),
    NO_AUTH(40101, "无权限", ""),
    FORBIDDEN(40301, "禁止操作", ""),
    SYSTEM_ERROR(50000, "系统内部异常", "");

    private final int code;
    private final String message;
    private final String description;

    ErrorCode(int code, String message, String description) {
        this.code = code;
        this.message = message;
        this.description = description;
    }
}
```

### 2. 统一返回体

```
@Data
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor  // 以 final 或 @NonNull 字段作为构造参数
public class BaseResponse<T> implements Serializable {

    private int code;
    private T data;
    private String message;
    private String description;

    public BaseResponse(int code, T data, String message) {
        this(code, data, message, "");
    }

    public BaseResponse(int code, T data) {
        this(code, data, "", "");
    }

    public BaseResponse(ErrorCode errorCode) {
        this(errorCode.getCode(), null, errorCode.getMessage(), errorCode.getDescription());
    }
}
```

### 3. 返回工具类

```
public class ResultUtils {
    public static <T> BaseResponse<T> success(T data) {
        return new BaseResponse<>(0, data, "ok");
    }
    public static BaseResponse error(ErrorCode errorCode) {
        return new BaseResponse(errorCode);
    }
    public static BaseResponse error(int code, String message, String description) {
        return new BaseResponse(code, null, message, description);
    }
}
```

### 4. 业务异常

```
@Getter
public class BusinessException extends RuntimeException {
    private final int code;
    private final String description;

    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.code = errorCode.getCode();
        this.description = errorCode.getDescription();
    }

    public BusinessException(ErrorCode errorCode, String description) {
        super(errorCode.getMessage());
        this.code = errorCode.getCode();
        this.description = description;
    }
}
```

### 5. 全局异常拦截

```
@Slf4j
@RestControllerAdvice   // 全局统一异常拦截器
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public BaseResponse businessExceptionHandler(BusinessException e) {
        log.error("businessException: " + e.getMessage(), e);
        return ResultUtils.error(e.getCode(), e.getMessage(), e.getDescription());
    }

    @ExceptionHandler(RuntimeException.class)
    public BaseResponse runtimeExceptionHandler(RuntimeException e) {
        log.error("runtimeException", e);
        return ResultUtils.error(ErrorCode.SYSTEM_ERROR, e.getMessage(), "");
    }
}
```

> 价值：**Controller 里零 try-catch**，业务出错直接 `throw new BusinessException(...)`，由全局拦截统一转成 `BaseResponse` 返回，前端永远拿到统一格式。

---

## 九、参数校验

```
@Data
public class XxxDTO {
    @NotBlank(message = "name不能为空")
    @Size(min = 2, max = 20, message = "name长度需要在2到20之间")
    private String name;

    @Pattern(regexp = "^[a-zA-Z0-9]{0,30}$", message = "密码格式要正确")
    private String confirmPassword;

    @Email(message = "邮箱格式不正确")
    private String email;

    @Min(value = 0, message = "不能小于0")
    @Max(value = 100, message = "不能大于100")
    private Integer age;
}
```

> **使用时加 `@Valid` 才生效**，配合 `@RequestBody` 接收 JSON：

```
@RestController
@RequestMapping("/main")
public class XxxController {
    @Resource
    private XxxService xxxService;

    @PostMapping("/{id}")
    public BaseResponse<XxxVO> xxx(
            @PathVariable Long id,                    // 路径参数 /main/1
            @RequestParam(required = false) String name, // 查询参数 ?name=xx
            @RequestBody @Valid XxxDTO xxxDTO,        // 请求体 JSON
            HttpServletRequest request
    ) {
        Object uid = request.getAttribute("uid");     // 从过滤器里取用户 id
        XxxVO xxxVO = new XxxVO();
        return ResultUtils.success(xxxVO);
    }
}
```

> 三个参数注解分清：`@PathVariable`（路径）、`@RequestParam`（问号传参）、`@RequestBody`（JSON 请求体）。`@Valid` 紧跟在 `@RequestBody` 后，触发 DTO 校验。

---

## 十、跨域 CORS

```
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://access.xxx", "http://other.access.xxx")
                .allowCredentials(true)   // 允许携带 Cookie
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);            // 预检请求缓存时间
    }
}
```

> `allowCredentials(true)` 时，`allowedOrigins` 不能写 `*`（通配符），必须写具体域名，这是 CORS 规范限制，也是常见坑。

---

## 十一、认证鉴权：JWT + 过滤器

JWT（JSON Web Token）是无状态登录方案：**登录后发一个 token，之后每次请求带 token，服务端验签即可识别用户，不存 session。**

### 1. JWT 工具类

```
@Component
public class JwtUtils {
    @Value("${jwt.secret}")
    private String secretKey;

    @Value("#{ ${jwt.expire-hours} * 3600L * 1000 }")  // SpEL：小时转毫秒
    private long expireMs;

    // claims：存放用户信息的 Map
    public String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
                .addClaims(claims)
                .setExpiration(new Date(System.currentTimeMillis() + expireMs))
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }

    public Claims parseToken(String token) {
        if (token == null || token.isBlank()) {
            throw new IllegalArgumentException("token不能为空");
        }
        try {
            return Jwts.parser()
                    .setSigningKey(secretKey)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (ExpiredJwtException e) {
            throw new RuntimeException("token已过期");
        } catch (SignatureException e) {
            throw new RuntimeException("token签名非法，数据被篡改");
        } catch (MalformedJwtException e) {
            throw new RuntimeException("token格式错误");
        } catch (Exception e) {
            throw new RuntimeException("token解析失败");
        }
    }
}
```

> 大厂考点：**JWT 由 header.payload.signature 三段组成**，`signWith` 用密钥签名。改 payload 任何一位，signature 就验证不过（`SignatureException`）。所以 JWT 不能防偷窥（payload 只是 base64 编码，能解码），**只能防篡改**——敏感信息别放 payload。

### 2. 过滤器（统一鉴权）

```
@Slf4j
@WebFilter(urlPatterns = "/*", filterName = "jwtFilter")
public class JwtFilter implements Filter {

    private static final List<String> WHITE_LIST = Arrays.asList(
            "/api/login", "/api/register"
    );

    @Autowired
    private JwtUtils jwtUtils;

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
                         FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        String uri = request.getRequestURI();

        // 白名单放行
        if (WHITE_LIST.contains(uri)) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = request.getHeader("token");
        if (token == null || token.isBlank()) {
            send401(response, "token不能为空");
            return;
        }

        try {
            Claims claims = jwtUtils.parseToken(token);
            request.setAttribute("uid", claims.get("uid"));  // 把用户 id 传给后续 Controller
            filterChain.doFilter(request, response);
        } catch (RuntimeException e) {
            log.error("token校验失败: {}", e.getMessage());
            send401(response, e.getMessage());
        }
    }

    private void send401(HttpServletResponse response, String message) throws IOException {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json;charset=UTF-8");
        BaseResponse<?> failRes = ResultUtils.error(ErrorCode.NOT_LOGIN);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.writeValue(response.getWriter(), failRes);
    }
}
```

> **过滤器 vs 拦截器**：过滤器（Filter）是 Servlet 规范的，最早拦截；拦截器（Interceptor）是 Spring 的，在 DispatcherServlet 之后。**全局鉴权一般用 Filter（更底层），业务权限用 Interceptor**。

---

## 十二、Session + Redis

Session 存 Redis，实现分布式会话共享（多台服务器都能读到同一个登录态）。

### 1. Session 工具类

```
public class SessionUtil {
    public static HttpSession getSession() {
        RequestAttributes ra = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) ra;
        HttpServletRequest request = sra.getRequest();
        return request.getSession(true); // true：没有则自动创建；false：没有则返回 null
    }

    public static void set(String key, Object value) {
        getSession().setAttribute(key, value);
    }

    public static Object get(String key) {
        return getSession().getAttribute(key);
    }

    public static <T> T get(String key, Class<T> clazz) { // Long userId = SessionUtil.get("userId", Long.class);
        return clazz.cast(get(key));
    }

    public static void remove(String key) {
        getSession().removeAttribute(key);
    }

    public static void logout() {
        HttpSession session = getSession(false);
        if (session != null) session.invalidate();
    }
}
```

### 2. 登录拦截器

```
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        Object userId = SessionUtil.get("userId");
        if (userId == null) {
            response.setContentType("application/json;charset=utf-8");
            response.getWriter().write("{\"code\":401,\"msg\":\"请先登录\"}");
            return false;
        }
        return true;
    }
}

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/auth/login"); // 放行登录接口
    }
}
```

## 十二·补：sa-token 权限认证框架

> 前面手写了 JWT Filter 和 Session + LoginInterceptor，其实有个更省事的框架——**sa-token**，登录、鉴权、踢人、权限一行代码搞定。你技术栈的认证，直接用它，省掉一堆手写代码。

### 1. sa-token 是啥

一句话：**轻量级 Java 权限认证框架，登录认证 / 权限认证 / 分布式 Session / 单点登录 SSO / OAuth2 全包，比 Spring Security 简单太多。**

|  |  |  |
| --- | --- | --- |
| 框架 | 上手难度 | 特点 |
| Shiro | 中 | 老牌，功能全但配置多 |
| Spring Security | 高 | 重量级，学习曲线陡 |
| **sa-token** | **低** | **一行代码登录，注解鉴权，开箱即用** |

### 2. 引入依赖（SpringBoot 3）

```
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot3-starter</artifactId>
    <version>1.44.0</version>
</dependency>
```

> **大坑**：SpringBoot 3 必须用 `sa-token-spring-boot3-starter`。别用 `sa-token-reactor-spring-boot3-starter`（那是响应式 WebFlux 用的，Servlet 项目会报「未能获取有效的上下文」）。

### 3. application.yml 配置

```
sa-token:
  token-name: satoken      # token 名称（也是 cookie 名，默认 satoken）
  timeout: 2592000         # token 有效期（秒），默认 30 天，-1 永久有效
  active-timeout: -1       # 最低活跃频率，-1 不限制
  is-concurrent: true      # 是否允许同一账号多地同时登录（true 允许）
  is-share: false          # 多人登录是否共用一个 token（false 每次登录新建）
  token-style: random-32   # token 风格（uuid / simple-uuid / random-32 / random-64 / tik）
  is-log: true             # 是否输出操作日志
```

### 4. 核心 API（一行代码搞定）

```
import cn.dev33.satoken.stp.StpUtil;

// 登录：参数是账号 id，登录后自动生成 token
StpUtil.login(userId);

// 退出登录
StpUtil.logout();

// 是否已登录
boolean isLogin = StpUtil.isLogin();

// 校验登录，未登录抛 NotLoginException
StpUtil.checkLogin();

// 获取当前登录账号 id
Long userId = StpUtil.getLoginIdAsLong();

// 获取当前 token 值
String token = StpUtil.getTokenValue();

// 把指定账号踢下线
StpUtil.kickout(userId);

// 权限 / 角色判断
StpUtil.hasRole("admin");
StpUtil.hasPermission("user:add");
```

> 没有接口、没有 XML、没有复杂配置，`StpUtil.login(userId)` 一句就完成会话登录，这是它最爽的地方。

### 5. 注解鉴权（最常用）

```
@SaCheckLogin                          // 登录才能访问
@SaCheckRole("admin")                  // 必须有 admin 角色
@SaCheckPermission("user:add")         // 必须有 user:add 权限

// 多个权限满足其一即可（mode = OR）
@SaCheckPermission(value = {"user:add", "user:all"}, mode = SaMode.OR)
@GetMapping("/add")
public BaseResponse<Void> add() { ... }
```

> 注解是 sa-token 的杀手锏，比手写 `if (!StpUtil.hasPermission(...)) throw ...` 简洁得多，Spring Security 那种「配置一堆权限规则」的复杂度直接没了。

### 6. 路由拦截器（替代手写 LoginInterceptor）

```
@Configuration
public class SaTokenConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 全局拦截：每个请求先校验是否登录，白名单放行
        registry.addInterceptor(new SaInterceptor(handle -> StpUtil.checkLogin()))
                .addPathPatterns("/**")
                .excludePathPatterns("/auth/login", "/auth/register");
    }
}
```

> 对比上一章手写的 `LoginInterceptor`（要自己判断 `SessionUtil.get("userId")` 是否为空），这里 `new SaInterceptor(handle -> StpUtil.checkLogin())` 一行替代。

### 7. 集成 Redis（分布式会话）

多实例部署时，token 不能存内存（每台机器各存各的），要存 Redis：

```
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-redis-jackson</artifactId>
    <version>1.44.0</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

> 引入 `sa-token-redis-jackson` 后，sa-token 自动把 token 和 session 存到 Redis，**代码零改动**，天然支持分布式会话。

### 8. 前端怎么传 token

sa-token 默认从请求头里取 `satoken`（`token-name` 配置的名字）：

```
Authorization 或 satoken 请求头：satoken: <token 值>
```

> **注意与 JWT 的区别**：上一章 JWT 自定义 header 名是 `token`，sa-token 默认是 `satoken`。用了 sa-token 就统一从 `satoken` 读，别混用。

### 9. 三套方案怎么选

|  |  |  |
| --- | --- | --- |
| 方案 | 适用场景 | 特点 |
| JWT + Filter | 无状态、跨服务、不存服务端 | 轻量但自己管过期/续期/踢人 |
| Session + Redis | 传统会话、服务端可控 | 需手写工具类和拦截器 |
| **sa-token** | **绝大多数项目（推荐）** | **开箱即用，登录/权限/踢人/单点全包** |

> 结论：**新项目首选 sa-token**（省事、功能全、还支持单点登录/OAuth2）；只有需要「无状态、不依赖 Redis」的纯 API 网关场景才考虑 JWT。

---

---

## 十三、Redis 高级玩法

### 1. RedisTemplate（配置序列化）

```
@Configuration
public class RedisTemplateConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(RedisSerializer.string()); // key 用 string，值默认 JDK 序列化（对象转 JSON 字符串再存）
        return redisTemplate;
    }
}
```

五种数据类型操作：

```
@Resource
private RedisTemplate<String, Object> redisTemplate;

// String
redisTemplate.opsForValue().set("key", "data_json", 30, TimeUnit.MINUTES);
Object val = redisTemplate.opsForValue().get("key");
redisTemplate.delete("key");

// Hash
HashOperations<String, String, Object> hashOps = redisTemplate.opsForHash();
hashOps.put("key", "field", "data_json");
hashOps.get("key", "field");
Map<String, Object> map = hashOps.entries("key");

// List
redisTemplate.opsForList().rightPush("key", "data_json");
redisTemplate.opsForList().leftPop("key");

// Set
redisTemplate.opsForSet().add("key", "data_json", "other_data_json");
redisTemplate.opsForSet().remove("key", "data_json", "other_data_json");

// 通用
Boolean hasKey = redisTemplate.hasKey("key");
Boolean res = redisTemplate.expire("key", 10, TimeUnit.MINUTES); // 重置已存在键的过期时间，不存在返回 false
```

> 大厂考点：**RedisTemplate 默认用 JDK 序列化**，存进去是乱码二进制，可读性差、跨语言不通。生产要么配置 `GenericJackson2JsonRedisSerializer`，要么像上面**对象先转 JSON 字符串再存**。

### 2. Redisson（分布式锁 / 布隆 / 限流 / 延迟队列）

Redisson 是 Redis 的 Java 高级客户端，封装了分布式场景的一堆能力。

**配置**：

```
@Data
@Configuration
@ConfigurationProperties(prefix = "spring.data.redis")
public class RedisConfig {
    private String host;
    private int port;

    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        String redisAddress = String.format("redis://%s:%s", host, port);
        config.useSingleServer().setAddress(redisAddress).setDatabase(0);
        return Redisson.create(config);
    }
}
```

**可重入分布式锁**：

```
RLock lock = redissonClient.getLock("lockname");
try {
    boolean tryLock = lock.tryLock(10, TimeUnit.SECONDS); // 10s 内尝试拿锁
    if (tryLock) {
        System.out.println("拿到锁, 执行业务");
    }
} finally {
    if (lock.isHeldByCurrentThread()) {  // 只释放自己持有的锁
        lock.unlock();
    }
}
```

> 大厂考点：**Redisson 分布式锁 + 看门狗（watch dog）**——默认锁过期 30s，看门狗每 10s 自动续期，防止业务没执行完锁就过期。`tryLock(waitTime, leaseTime, unit)` 里 `leaseTime=-1` 才启用看门狗。

**布隆过滤器**：

```
RBloomFilter bloomFilter = redissonClient.getBloomFilter("bloom:key");
bloomFilter.tryInit(1000000, 0.01);  // 预期元素数量, 误判率
bloomFilter.add("data");             // 底层存的是序列化后的字节
boolean mightContain = bloomFilter.contains("data");
```

> 布隆过滤器：**判断「一定不存在」很准，判断「可能存在」有误判**。典型用途是**防缓存穿透**——先问布隆「这个 key 存不存在」，不存在直接返回，不去打数据库。

**限流器（令牌桶）**：

```
RRateLimiter rateLimiter = redissonClient.getRateLimiter("limit:api:submit");
rateLimiter.trySetRate(RateType.OVERALL, 2, 1, RateIntervalUnit.SECONDS); // 每秒产生 2 个令牌

boolean acquire = rateLimiter.tryAcquire(1);  // 拿一个令牌，拿不到就限流
if (!acquire) {
    throw new RuntimeException("请求过于频繁, 限流!");
}
```

**延迟队列（订单超时自动取消）**：

```
RBlockingQueue<OrderDTO> targetQueue = redissonClient.getBlockingQueue("delay:order");
RDelayedQueue<OrderDTO> delayedQueue = redissonClient.getDelayedQueue(targetQueue);

delayedQueue.offer(orderDTO, 10, TimeUnit.SECONDS); // 10 秒后才被消费
delayedQueue.remove(orderDTO);  // 用户支付成功后，必须删除延时任务

OrderDTO order = targetQueue.take(); // 阻塞获取到期任务
```

---

## 十四、缓存三大问题（大厂必考）

|  |  |  |
| --- | --- | --- |
| 问题 | 现象 | 解决方案 |
| 缓存穿透 | 查询「不存在」的数据，redis 没缓存，每次穿透到数据库 | ① 空结果也缓存（设短过期时间）② 布隆过滤器拦截 |
| 缓存击穿 | 热门数据突然过期，大量请求同时打数据库 | 互斥锁（分布式锁）让第一个请求重建缓存，其余等待 |
| 缓存雪崩 | 大量缓存同时过期，数据库瞬间压力巨大 | 给过期时间加随机值，避免同时失效 |

> 一句话记忆：**穿透 = 查没有的东西（布隆/空值缓存）；击穿 = 热点单点失效（加锁）；雪崩 = 大面积同时失效（过期加随机）。**

### Redis 三件套（防丢数据 / 防挂 / 防存不下）

|  |  |
| --- | --- |
| 问题 | 方案 |
| 防止数据丢失 | RDB 快照 + AOF 日志（两种持久化方案） |
| 避免服务器挂掉 | 主从复制 + 哨兵机制（主挂自动选新主顶上） |
| 数据量巨大存不下 | 淘汰策略删不常用数据 + 分片集群分散存储 |

---

## 十五、定时任务 @Scheduled

```
@Scheduled(cron = "${precache.recommend.cron}")
public void doPreCache() {
    RLock lock = redissonClient.getLock(LOCK_KEY);
    try {
        // wait=0 抢不到直接放弃，lease=-1 启用看门狗续锁
        boolean ok = lock.tryLock(0, -1, TimeUnit.MILLISECONDS);
        if (!ok) {
            log.info("推荐预热任务: 抢锁失败, 其他实例正在执行");
            return;
        }
        log.info("推荐预热任务: 获取锁成功, 开始执行");

        List<User> userList = userService.list();
        for (User user : userList) {
            Long userId = user.getId();
            try {
                List<User> recList = recommendService.getRecommendList(userId, 20);
                if (recList == null || recList.isEmpty()) continue;
                String key = "yupao:user:recommend:" + userId;
                valueOperations.set(key, recList, cacheTtl, TimeUnit.SECONDS);
            } catch (Exception e) {
                log.error("userId {} 缓存写入失败", userId, e);
            }
        }
        log.info("推荐预热任务: 执行完成");
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        log.error("推荐预热任务被中断", e);
    } catch (Exception e) {
        log.error("推荐预热任务全局异常", e);
    } finally {
        if (lock.isHeldByCurrentThread()) {  // 安全解锁：只释放自己持有的锁
            lock.unlock();
        }
    }
}
```

> 两个要点：① **启动类务必加 `@EnableScheduling`** 定时任务才生效；② **多实例部署时，定时任务要加分布式锁**，否则每台机器都跑一遍，重复执行。上面用 `tryLock(0, -1, ...)` 抢不到直接放弃，保证只有一个实例真正执行。

---

## 十六、邮件发送

> 生产上线用**专业的邮件推送服务**（阿里云邮件推送 / 腾讯云邮件推送），SMTP 只是基础用法。

```
@Service
public class MailService {
    @Autowired
    private JavaMailSender mailSender;

    private static final String AUTHOR_NAME = "author";
    private static final String SENDER = "15779544219@163.com";

    public void sendHtmlMail(String to, String subject, String htmlContent) {
        try {
            MimeMessage mimeMessage = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true, "UTF-8");
            helper.setFrom(AUTHOR_NAME + "<" + SENDER + ">");
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(htmlContent, true); // true = 发送 HTML 格式
            mailSender.send(mimeMessage);
        } catch (MessagingException e) {
            throw new RuntimeException("邮件发送失败", e);
        }
    }
}
```

> 注意：163 邮箱的 `password` 不是登录密码，是**授权码**（去邮箱设置里开启 SMTP 后获取）。

---

## 十七、接口文档 Knife4j

Knife4j（Swagger 增强版）自动生成接口文档。

```
@Configuration
@EnableSwagger2WebMvc
@Profile({"dev", "test"})   // 只在 dev/test 环境启用，生产不暴露
public class SwaggerConfig {
    @Bean
    public Docket defaultApi2() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.xxx.xxx.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("接口文档")
                .description("接口文档描述")
                .termsOfServiceUrl("https://github.com/xiaoyi-f")
                .contact(new Contact("xiaoyi", "https://github.com/xiaoyi-f", "xxx@xxx.com"))
                .version("1.0")
                .build();
    }
}
```

注解用法：

```
@Api(tags = "接口分组")                       // 类级别
public class ApiController {
    @ApiOperation(value = "接口摘要", notes = "接口描述") // 方法级别
    public Po api(@ApiParam(value = "参数名称", required = true) Long id) { ... }
}

@ApiModel(description = "xxx请求")              // DTO 类
public class XxxDTO {
    @ApiModelProperty(value = "参数描述", example = "xxx")
    private Long xxx;
}
```

> 大厂考点：**`@Profile({"dev","test"})` 限制接口文档只在开发/测试环境暴露**，生产环境关闭，防止接口文档泄露攻击面。

### 访问地址

启动后浏览器打开（端口和路径前缀以你 `application.yml` 为准）：

|  |  |
| --- | --- |
| 文档类型 | 访问地址 |
| Knife4j 增强版 | `http://localhost:8080/doc.html` |
| Swagger 原生 UI | `http://localhost:8080/swagger-ui.html` |

> 你前面配置了 `server.servlet.context-path: /api`，所以实际访问是 **`http://localhost:8080/api/doc.html`**（记得带上 context-path 前缀）。
>
> **版本坑**：上面 `@EnableSwagger2WebMvc` 是 **Knife4j 2.x + springfox** 的写法，适配 **SpringBoot 2.x**。你技术栈是 **SpringBoot 3**（`javax` 已改成 `jakarta`，springfox 不兼容），要用 **springdoc-openapi** + Knife4j 4.x，访问地址仍是 `/doc.html`。

### 版本配对（SpringBoot 3 必看）

|  |  |  |
| --- | --- | --- |
| SpringBoot 版本 | Knife4j 版本 | 依赖坐标 |
| 2.x（老项目） | 4.0 之前 | `knife4j-spring-boot-starter` |
| 2.x | 4.0+（Swagger2 规范） | `knife4j-openapi2-spring-boot-starter` |
| 2.x | 4.0+（OpenAPI3 规范） | `knife4j-openapi3-spring-boot-starter` |
| **3.x（你的技术栈）** | **4.0+** | **`knife4j-openapi3-jakarta-spring-boot-starter`** |

> **核心结论**：SpringBoot 3 只能用 **OpenAPI3 规范 + 带 `jakarta` 的依赖**。springfox 早已停更、不支持 jakarta，所以只能走 springdoc + OpenAPI3，坐标里**必须带 `jakarta`** 三个字，这是最容易搞错的点。

**注解对照（Swagger2 → OpenAPI3）**：

|  |  |
| --- | --- |
| Swagger2（你笔记里的写法） | OpenAPI3（SpringBoot 3 要用） |
| `@Api` | `@Tag` |
| `@ApiOperation` | `@Operation` |
| `@ApiParam` | `@Parameter` |
| `@ApiModel` | `@Schema` |
| `@ApiModelProperty` | `@Schema` |

> **已知坑**：Knife4j `4.5.0` 在 SpringBoot 3.x 下偶尔报 `NoSuchMethodError`，是底层 `springdoc-openapi-starter-webmvc-ui` 版本不匹配导致，解决方法是**在父 POM 显式指定 springdoc 版本（如 `2.8.9`）覆盖默认值**。
>
> **补充 -- OSS** 
>
> ```
> 依赖示例:
> <dependency>
>   <groupId>com.aliyun.oss</groupId>
>   <artifactId>aliyun-sdk-oss</artifactId>
>   <version>3.17.4</version>
> </dependency>
>
> // 准备阿里云信息
> endpoint --> oss 服务地址 
> access-key-id --> 访问密钥 ID 
> access-key-secret --> 访问密钥 Secret 
> bucket-name --> bucket 名称 
> --> 对于前端 建议设置关联自己的域名 从而确保前端直接显示对应的内容(图片等) 而避免弹出下载 
> --> 访问控制: 授权主体(创建用户) 授权策略(OSS)
>
> // 域名管理 
> 绑定自定义域名
> 先填写一个准备要绑定的域名, 自动校验主域名
> 创建TXT记录域名_dnsauth.要绑定的子域名 
> 创建CNAME记录域名要绑定的子域名 
> Tip: 创建解析不要多写主域名    
>
> // application.yml 对应的配置数据 
> aliyun:
>   oss:
>     endpoint: ${OSS_ENDPOINT}
>     access-key-id: ${OSS_ACCESS_KEY_ID}
>     access-key-secret: ${OSS_ACCESS_KEY_SECRET}
>     bucket: ${OSS_BUCKET}
>
> // 配置类 从 application.yml 读取配置数据 
> @Configuration
> public class OssConfig {
>
>     @Value("${aliyun.oss.endpoint}")
>     private String endpoint;
>
>     @Value("${aliyun.oss.access-key-id}")
>     private String accessKeyId;
>
>     @Value("${aliyun.oss.access-key-secret}")
>     private String accessKeySecret;
>
>     @Bean(destroyMethod = "shutdown")
>     public OSS ossClient() {
>         return new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
>     }
> }
>
>
> @Service
> public class OssService {
>
>     private final OSS ossClient;
>     private final String bucket;
>
>     public OssService(OSS ossClient, @Value("${aliyun.oss.bucket}") String bucket) {
>         this.ossClient = ossClient;
>         this.bucket = bucket;
>     }
>
>     // 普通上传
>     public String upload(MultipartFile file) throws IOException {
>         if (file.isEmpty()) {
>             throw new IllegalArgumentException("文件不能为空");
>         }
>         String objectKey = UUID.randomUUID() + "-" + file.getOriginalFilename();
>         ossClient.putObject(bucket, objectKey, file.getInputStream());
>         return objectKey;
>     }
>
>     // 指定文件夹上传（把文件上传到指定目录，例如：avatar/xxx.jpg）
>     public String uploadToFolder(MultipartFile file, String folderPath) throws IOException {
>         if (file.isEmpty()) {
>             throw new IllegalArgumentException("文件不能为空");
>         }
>         // folderPath 要求以 / 结尾，比如 "avatar/"
>         if (!folderPath.endsWith("/")) {
>             folderPath = folderPath + "/";
>         }
>         String objectKey = folderPath + UUID.randomUUID() + "-" + file.getOriginalFilename();
>         ossClient.putObject(bucket, objectKey, file.getInputStream());
>         return objectKey;
>     }
>
>     // 创建模拟文件夹
>     public void createFolder(String folderPath) {
>         // 必须以 / 结尾
>         if (!folderPath.endsWith("/")) {
>             folderPath = folderPath + "/";
>         }
>         ossClient.putObject(bucket, folderPath, "".getBytes());
>     }
>
>     // 递归删除文件夹（目录下所有文件+子目录全部删掉）
>     public void deleteFolder(String folderPath) {
>         if (!folderPath.endsWith("/")) {
>             folderPath = folderPath + "/";
>         }
>         ListObjectsV2Request req = new ListObjectsV2Request()
>                 .withBucketName(bucket)
>                 .withPrefix(folderPath)
>                 .withMaxKeys(1000);
>         ListObjectsV2Result result;
>         do {
>             result = ossClient.listObjectsV2(req);
>             for (OSSObjectSummary summary : result.getObjectSummaries()) {
>                 ossClient.deleteObject(bucket, summary.getKey());
>             }
>             req.setContinuationToken(result.getNextContinuationToken());
>         } while (result.isTruncated());
>     }
>
>     // 列出文件夹下所有文件（只当前层级，不递归子目录）
>     public List<String> listFileInFolder(String folderPath) {
>         if (!folderPath.endsWith("/")) {
>             folderPath = folderPath + "/";
>         }
>         List<String> keyList = new ArrayList<>();
>         ListObjectsV2Request req = new ListObjectsV2Request()
>                 .withBucketName(bucket)
>                 .withPrefix(folderPath)
>                 .withDelimiter("/") // 加delimiter，实现只看当前层，子文件夹作为commonPrefix
>                 .withMaxKeys(1000);
>         ListObjectsV2Result result;
>         do {
>             result = ossClient.listObjectsV2(req);
>             for (OSSObjectSummary summary : result.getObjectSummaries()) {
>                 keyList.add(summary.getKey());
>             }
>             req.setContinuationToken(result.getNextContinuationToken());
>         } while (result.isTruncated());
>         return keyList;
>     }
>
>     // 列出子文件夹（当前目录下的子文件夹）
>     public List<String> listSubFolder(String folderPath) {
>         if (!folderPath.endsWith("/")) {
>             folderPath = folderPath + "/";
>         }
>         List<String> subFolderList = new ArrayList<>();
>         ListObjectsV2Request req = new ListObjectsV2Request()
>                 .withBucketName(bucket)
>                 .withPrefix(folderPath)
>                 .withDelimiter("/")
>                 .withMaxKeys(1000);
>         ListObjectsV2Result result;
>         do {
>             result = ossClient.listObjectsV2(req);
>             subFolderList.addAll(result.getCommonPrefixes());
>             req.setContinuationToken(result.getNextContinuationToken());
>         } while (result.isTruncated());
>         return subFolderList;
>     }
>
>     // 预览 -- 临时访问链接 -- 有效期
>     public String generatePresignedUrl(String objectKey, long expireSeconds) {
>         Date expireDate = new Date(System.currentTimeMillis() + expireSeconds * 1000);
>         GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(bucket, objectKey);
>         request.setExpiration(expireDate);
>         URL url = ossClient.generatePresignedUrl(request);
>         return url.toString();
>     }
>
>     // 下载
>     public InputStream download(String objectKey) {
>         OSSObject object = ossClient.getObject(bucket, objectKey);
>         return object.getObjectContent();
>     }
>
>     // 删除单个文件
>     public void delete(String objectKey) {
>         ossClient.deleteObject(bucket, objectKey);
>     }
> }
>
>
> // Controller 
> @RestController
> @RequestMapping("/files")
> public class FileController {
>
>     private final OssService ossService;
>
>     public FileController(OssService ossService) {
>         this.ossService = ossService;
>     }
>
>     // 普通上传
>     @PostMapping
>     public String upload(@RequestParam("file") MultipartFile file) throws IOException {
>         return ossService.upload(file);
>     }
>
>     // 上传到指定文件夹
>     @PostMapping("/folder-upload")
>     public String uploadToFolder(@RequestParam("file") MultipartFile file,
>                                  @RequestParam("folderPath") String folderPath) throws IOException {
>         return ossService.uploadToFolder(file, folderPath);
>     }
>
>     // 创建文件夹
>     @PostMapping("/folder")
>     public void createFolder(@RequestParam("folderPath") String folderPath) {
>         ossService.createFolder(folderPath);
>     }
>
>     // 删除文件夹（递归清空）
>     @DeleteMapping("/folder")
>     public void deleteFolder(@RequestParam("folderPath") String folderPath) {
>         ossService.deleteFolder(folderPath);
>     }
>
>     // 查询文件夹内文件列表
>     @GetMapping("/folder/files")
>     public List<String> listFile(@RequestParam("folderPath") String folderPath) {
>         return ossService.listFileInFolder(folderPath);
>     }
>
>     // 查询文件夹下子文件夹
>     @GetMapping("/folder/sub")
>     public List<String> listSubFolder(@RequestParam("folderPath") String folderPath) {
>         return ossService.listSubFolder(folderPath);
>     }
>
>     // 预览
>     @GetMapping("/presigned-url")
>     public String getPresignedUrl(@RequestParam String objectKey,
>                                   @RequestParam(defaultValue = "1800") long expireSeconds) {
>         return ossService.generatePresignedUrl(objectKey, expireSeconds);
>     }
>
>     // 下载
>     @GetMapping("/download")
>     public void download(@RequestParam String objectKey, HttpServletResponse response) throws IOException {
>         try (InputStream input = ossService.download(objectKey)) {
>             response.setContentType("application/octet-stream");
>             response.setHeader("Content-Disposition", "attachment; filename=\"download\"");
>             input.transferTo(response.getOutputStream());
>         }
>     }
>
>     // 删除单个文件
>     @DeleteMapping
>     public void delete(@RequestParam String objectKey) {
>         ossService.delete(objectKey);
>     }
> }
>
>   /*
>   上传注意文件大小限制
>   spring:
>     servlet:
>       multipart:
>         max-file-size: 10MB
>         max-request-size: 10MB 
>   */
> ```
>
> 

### SpringBoot 3 完整配置（直接抄）

**第 1 步：pom.xml 加依赖**（坐标带 `jakarta`）：

```
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.5.0</version>
</dependency>
```

**第 2 步：application.yml 开启**：

```
springdoc:
  api-docs:
    enabled: true
  swagger-ui:
    enabled: true
knife4j:
  enable: true
```

**第 3 步：配置类**（springdoc 的 `OpenAPI` Bean，替代旧版 `Docket`）：

```
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;

@Configuration
@Profile({"dev", "test"})   // 只在 dev/test 环境启用，生产不暴露
public class Knife4jConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("接口文档")
                        .description("接口文档描述")
                        .contact(new Contact()
                                .name("xiaoyi")
                                .url("https://github.com/xiaoyi-f"))
                        .version("1.0"));
    }
}
```

**第 4 步：注解用法（OpenAPI3）**：

```
@Tag(name = "接口分组")                          // 类级别
public class ApiController {
    @Operation(summary = "接口摘要", description = "接口描述")  // 方法级别
    public Po api(@Parameter(description = "参数名称", required = true) Long id) { ... }
}

@Schema(description = "xxx请求")                  // DTO 类
public class XxxDTO {
    @Schema(description = "参数描述", example = "xxx")
    private Long xxx;
}
```

> 启动后访问 `http://localhost:8080/api/doc.html`（带你的 context-path）。`@Profile({"dev","test"})` 保证生产环境不暴露文档。

---

## 十八、完整 application.yml + logback

```
spring:
  profiles:
    active: dev               # 激活环境：dev / prod
  application:
    name: application-name
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/dbname?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8
    username: root
    password: passwd
  session:
    store-type: redis         # session 存 redis，分布式会话共享
  data:
    redis:
      url: redis://:password@127.0.0.1:6379
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 2
          max-wait: 2000ms
  mail:
    host: smtp.163.com
    port: 465
    username: xxx@163.com
    password: 授权码
    properties:
      mail:
        smtp:
          ssl:
            enable: true
          auth: true

server:
  address: 0.0.0.0
  port: 8080
  servlet:
    context-path: /api        # 全局路径前缀
    session:
      timeout: 604800
      cookie:
        name: JSESSIONID
        secure: false
        http-only: true
        domain: domain.cn     # 多子域名共享 cookie；不配置则自动分配当前访问域名
        path: /               # 默认值，整个站点都带 cookie

mybatis-plus:
  configuration:
    use-deprecated-executor: false
    map-underscore-to-camel-case: true   # 驼峰转下划线
    # log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 打印 SQL 日志
```

> `src/main/resources/banner.txt` 放水印；
>
> `src/main/resources/logback.xml` 填日志配置。

**cookie 参数解释**（大厂高频）：

|  |  |
| --- | --- |
| 参数 | 含义 |
| secure | true 时只在 HTTPS 下传 cookie |
| http-only | true 时 JS 读不到 cookie，防 XSS 窃取 |
| domain | 多子域名共享 cookie；不配则用当前域名 |
| path | 默认 `/`，整个站点都携带 |

---

## 大厂高频考点速查表

|  |  |
| --- | --- |
| 考点 | 答案要点 |
| 依赖传递 | starter 引入时内部依赖自动带进来（如 web 带 tomcat） |
| 自动配置原理 | run → 读配置 → 建容器 → @EnableAutoConfiguration → 启动 Tomcat |
| IOC / DI / AOP | 控制反转 / 依赖注入 / 面向切面（无侵入增强） |
| @Autowired vs @Resource | byType / byName |
| PO / DTO / VO | 持久层 / 传输 / 视图对象 |
| 乐观锁 vs 悲观锁 | 版本号 vs FOR UPDATE，性能与一致性取舍 |
| 事务失效场景 | 非 public、this 调用、异常被吞、rollbackFor 没配 |
| 缓存穿透/击穿/雪崩 | 布隆/空值缓存；互斥锁；过期加随机 |
| Redis 持久化 | RDB 快照 + AOF 日志 |
| Redis 高可用 | 主从复制 + 哨兵 |
| 分布式锁看门狗 | lease=-1 时自动续期，防锁提前过期 |
| 布隆过滤器 | 判「一定不存在」准，判「存在」有误判，防穿透 |
| JWT 防篡改 | signature 签名，payload 仅 base64 能解码 |
| Filter vs Interceptor | Filter 是 Servlet 规范更底层，Interceptor 是 Spring |
| serialVersionUID 坑 | 结构变了 UID 变，反序列化对不上报错 |
