---
title: "Elasticsearch 从入门到进大厂"
status: 已发布
author: XiaoYi
---

# Elasticsearch 从入门到进大厂

> 这一篇从倒排索引讲起，一路到 Spring Data Elasticsearch 实战，最后手把手带你用 Canal 把 MySQL 数据实时同步到 ES。学完，搜索功能 + 数据同步两大难题一起解决。
>
> Let's use AI to create a better world
>
> ## 一、Elasticsearch 是啥 + 为什么快
>
> 一句话：Elasticsearch（简称 ES）是一个开源的分布式搜索和分析引擎，擅长海量数据的搜索、分析、计算。
>
> **核心：倒排索引（大厂必考）** MySQL 是「正排」：拿着 id 去查这一行的所有字段。ES 是「倒排」：把文档拆成词条（term），记录每个词条出现在哪些文档里，搜词条时直接定位。
>
> 文档 1：我爱吃苹果 文档 2：苹果很好吃
>
> 倒排索引（分词后）：
>
> ```
> 我     → [文档1]
> 爱     → [文档1]
> 吃     → [文档1, 文档2]
> 苹果   → [文档1, 文档2]
> 很     → [文档2]
> 好吃   → [文档2]
> ```
>
> 搜「苹果」→ 直接命中 文档 1、文档 2，不用全表扫
>
> 一句话：倒排索引 = 预先给每个词条建好「词条 → 文档」的映射，搜索时按词条直接定位，所以快。这是 ES 和 MySQL 最本质的区别。
>
> 表格
>
> |  | MySQL | Elasticsearch |
> | --- | --- | --- |
> | 强项 | 精确查询、事务 | 海量搜索、分词、聚合分析 |
> | 检索方式 | 正排（逐行扫） | 倒排索引（按词条定位） |
> | 数据形式 | 表 + 行 | JSON 文档 + 索引 |
> | 定位 | 系统数据存储 | 搜索、日志、数据分析 |
>
> ## 二、核心概念
>
> 表格
>
> | 概念 | 含义 | 类比 MySQL |
> | --- | --- | --- |
> | index | 索引库，一组文档的集合 | 表 |
> | document | 文档，一条数据，JSON 格式 | 行 |
> | mapping | 映射，定义字段类型、是否索引、分词器 | 表结构（schema） |
> | field | 字段 | 列 |
> | analyzer | 分词器，把文本拆成词条 | 无对应 |
> | shard | 分片，索引数据切成多份分布式存储 | 无对应 |
>
> 字段类型（mapping 里用）：`keyword`、`byte`、`short`、`integer`、`long`、`float`、`double`、`boolean`、`date`、`object`、`text`。 `keyword` 精确匹配（不分词），`text` 分词检索。
>
> ## 三、环境部署（Docker）
>
> ### 1. 起一个单节点 ES
>
> ```
> docker run -d --name es \
>   -p 9200:9200 \
>   -e discovery.type=single-node \
>   -e xpack.security.enabled=false \
>   -e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
>   elasticsearch:8.11.0
> ```
>
> ### 2. 开启安全认证（可选）
>
> ```
> # 重置 elastic 用户密码
> docker exec -it es bin/elasticsearch-reset-password -u elastic
> # 生成 Kibana 注册令牌（登录控制台用）
> docker exec -it es bin/elasticsearch-create-enrollment-token -s kibana
> ```
>
> 端口记牢：`9200` 是 ES 服务端口（程序连的），`9300` 是集群内部通信端口。
>
> ## 四、IK 中文分词器
>
> ES 自带标准分词器按 Unicode 文本分割（把标点 / 特殊字符当分隔符），适合简单英文，中文会被切成单个字，所以中文要用 IK 分词器。
>
> 表格
>
> | 分词器 | 特点 | 适用 |
> | --- | --- | --- |
> | standard | 默认，按空格 / 标点 | 简单英文 |
> | ik\_smart | 粗粒度，最少切分 | 适合检索 |
> | ik\_max\_word | 细粒度，穷举可能词 | 适合关键词提取 |
>
> IK 版本必须和 ES 版本一致，安装：
>
> ```
> docker exec es bin/elasticsearch-plugin install <IK 插件安装链接>
> docker restart es
> docker exec es bin/elasticsearch-plugin list   # 查看已装插件
> ```
>
> 大厂考点：`ik_smart` 适合检索（切得少、精准），`ik_max_word` 适合关键词提取（切得全、覆盖广）。
>
> ## 五、索引库 mapping 操作
>
> ### 1. 创建索引库
>
> ```
> PUT /index_name
> {
>   "mappings": {
>     "properties": {
>       "id": { "type": "long" },
>       "create_date": { "type": "date" },
>       "target_obj": {
>         "type": "object",
>         "index": true,
>         "properties": {
>           "sub_attr": { "type": "text", "index": true, "analyzer": "ik_smart" },
>           "other_sub_attr": { "type": "text", "index": false, "analyzer": "ik_smart" }
>         }
>       }
>     }
>   }
> }
> ```
>
> 关键：`"index": false` 表示该字段不建立倒排索引，不能被搜索（但能返回），省空间。`"index": true` 才能被检索。
>
> ### 2. 查 / 改 / 删
>
> ```
> GET /index_name                 # 查索引库
> PUT /index_name/_mapping        # 增量更新 mapping（只能加字段/改 index，不能改已存在的字段类型）
> DELETE /index_name              # 删索引库
> ```
>
> 大厂考点：mapping 一旦建立，字段类型不可改（只能新增字段或改 index 属性）。要改类型只能重建索引 + 数据迁移。
>
> ## 六、文档 CRUD
>
> ES 数据本质是 JSON 文档，操作就是增删改查：
>
> ```
> // 增（指定 id）
> POST /index_name/_doc/doc_id
> {
>   "key": "value",
>   "key2": { "sub_key": "sub_value" }
> }
>
> // 删
> DELETE /index_name/_doc/doc_id
>
> // 改（全量：先删再增，id 不存在则新增）
> PUT /index_name/_doc/doc_id
> {
>   "key": "new_value"
> }
>
> // 改（局部：只改指定字段）
> POST /index_name/_update/doc_id
> {
>   "doc": {
>     "key": "new_value"
>   }
> }
>
> // 查
> GET /index_name/_doc/doc_id
> ```
>
> 区分：`PUT _doc` 是全量覆盖（没传的字段会丢），`POST _update` 是局部更新（只改传的字段）。ES 还支持批处理（bulk）。
>
> ## 七、搜索（match /term/bool /range/ 高亮）
>
> ### 1. match（分词匹配）vs term（精确匹配）
>
> ```
> GET /index_name/_search
> {
>   "query": {
>     "match": { "key": "value" }   // 分词后匹配（text 字段用这个）
>   }
> }
> ```
>
> ```
> GET /index_name/_search
> {
>   "query": {
>     "term": { "id": 1 }           // 精确匹配，不分词（keyword/数值用这个）
>   }
> }
> ```
>
> 大厂考点：`match` 会先分词再匹配（适合 text），`term` 精确匹配不分词（适合 keyword / 数值）。text 字段不能直接 term，否则搜不到。
>
> ### 2. 布尔查询（must /should/must\_not /filter）
>
> ```
> {
>   "query": {
>     "bool": {
>       "must":     [ { "match": { "title": "搜索词" } } ],   // 必须满足（算分）
>       "should":   [ { "match": { "title": "搜索词" } } ],   // 最好满足（加分）
>       "must_not": [ { "match": { "title": "搜索词" } } ],   // 必须不满足
>       "filter":   [ { "range": { "create_date": { "gte": "start", "lte": "end" } } } ]  // 过滤（不算分，快）
>     }
>   }
> }
> ```
>
> 记牢：must 必须满足、should 最好满足、must\_not 必须不符、filter 过滤。filter 不参与评分，性能比 must 好，能用 filter 就用 filter。
>
> ### 3. 范围查询 + 高亮
>
> ```
> {
>   "query": {
>     "range": {
>       "create_time": { "gte": "2020-01-01", "lte": "2020-01-31" }
>     }
>   },
>   "highlight": {
>     "fields": {
>       "key": {
>         "pre_tags": ["<span style='color:red;'>"],
>         "post_tags": ["</span>"]
>       }
>     }
>   }
> }
> ```
>
> ## 八、Spring Data Elasticsearch 实战
>
> Spring Data 提供三剑客：`ElasticsearchRepository`（声明式 CRUD）、`ElasticsearchOperations`（复杂查询）、`ElasticsearchClient`（原生高级操作）。
>
> ### 1. 依赖 + 配置
>
> ```
> spring:
>   elasticsearch:
>     uris: https://localhost:9200
>     username: elastic
>     password: <password>
>     connection-timeout: 5s
>     socket-timeout: 25s
>     ssl:
>       verification-mode: none
> ```
>
> ### 2. 实体类
>
> ```
> @Document(indexName = "index_name", createIndex = false)
> public class Article {
>     @Id
>     private Long id;
>     @Field(type = FieldType.Date, format = DateFormat.date_time)
>     private Instant create_date;
>     @Field(type = FieldType.Object)
>     private TargetObj target_obj;
>
>     public static class TargetObj {
>         @Field(type = FieldType.Text, analyzer = "ik_smart")
>         private String sub_attr;
>         @Field(type = FieldType.Text, analyzer = "ik_smart", index = false)
>         private String other_sub_attr;
>         // getter & setter 省略
>     }
>     // getter & setter 省略
> }
> ```
>
> 规范建议：Java 属性用驼峰命名，ES 字段名用 `@Field(name = "xxx")` 指定，别直接用下划线命名 Java 字段（这是 Java 编码规范，面试也看这个）。
>
> ### 3. Repository（声明式，零实现）
>
> ```
> @Repository
> public interface ArticleRepository extends ElasticsearchRepository<Article, Long> {
>     // 方法名自动解析：按 target_obj.sub_attr 分页查
>     Page<Article> findByTargetObjSubAttr(String keyword, Pageable pageable);
>
>     // 复杂查询用 @Query 注解，?0 ?1 ?2 代表第 1/2/3 个参数
>     @Query("""
>       { "bool": { "must": [ {"match": {"target_obj.sub_attr": "?0"}} ],
>                   "filter": [ {"range": {"create_date": {"gte": "?1", "lte": "?2"}}} ] } }
>     """)
>     Page<Article> searchByKeywordAndDate(String keyword, String start_date, String end_date, Pageable pageable);
> }
> ```
>
> ### 4. Service（CRUD + 高亮 + 批量 + 局部更新）
>
> ```
> @Service
> @RequiredArgsConstructor
> public class ArticleService {
>     private final ArticleRepository articleRepository;
>     private final ElasticsearchOperations elasticsearchOperations;
>     private final ElasticsearchClient esClient;
>
>     // 新增：传新对象 save；修改：查出来改再 save
>     public Article save(Article article) {
>         return articleRepository.save(article);
>     }
>
>     public Article getById(Long id) {
>         return articleRepository.findById(id).orElse(null);
>     }
>
>     public void delete(Long id) {
>         articleRepository.deleteById(id);
>     }
>
>     // 高亮 + 布尔 + 分页搜索
>     public Page<Article> searchHighlight(String keyword, String startDate, String endDate, int page, int size) {
>         NativeQuery nativeQuery = NativeQuery.builder()
>                 .withQuery(q -> q.bool(
>                         b -> b.must(m -> m.match(ma -> ma.field("target_obj.sub_attr").query(keyword)))
>                                 .filter(f -> f.range(r -> r.field("create_date")
>                                         .gte(JsonData.of(startDate)).lte(JsonData.of(endDate))))
>                 ))
>                 .withPageable(PageRequest.of(page, size, Sort.by(Sort.Direction.DESC, "create_date")))
>                 .withHighlight(Highlight.builder().fields("target_obj.sub_attr",
>                         HighlightField.builder().options(HighlightOptions.builder()
>                                 .preTags("<span style='color:red;'>").postTags("</span>").build()).build()).build())
>                 .build();
>         SearchHits<Article> hits = elasticsearchOperations.search(nativeQuery, Article.class);
>         List<Article> list = hits.stream().map(SearchHit::getContent).collect(Collectors.toList());
>         return new PageImpl<>(list, PageRequest.of(page, size), hits.getTotalHits());
>     }
>
>     // 局部更新
>     public void updatePart(Long id, Article updateDoc) throws IOException {
>         UpdateRequest<Article, Article> req = UpdateRequest.of(
>                 u -> u.index("index_name").id(id.toString()).doc(updateDoc)
>         );
>         esClient.update(req, Article.class);
>     }
>
>     // 高性能批量写入
>     public void bulkSave(List<Article> articleList) throws IOException {
>         if (CollectionUtils.isEmpty(articleList)) return;
>         List<BulkOperation> ops = new ArrayList<>();
>         for (Article article : articleList) {
>             ops.add(BulkOperation.of(o -> o.index(
>                     i -> i.index("index_name").id(article.getId().toString()).document(article)
>             )));
>         }
>         BulkResponse resp = esClient.bulk(BulkRequest.of(b -> b.operations(ops)));
>         if (resp.errors()) {
>             throw new RuntimeException("Bulk写入失败: " + resp.items());
>         }
>     }
>
>     // 精确匹配（term，不分词）
>     public List<Article> termQuery(Long idVal) {
>         Criteria criteria = new Criteria("id").is(idVal);
>         CriteriaQuery criteriaQuery = new CriteriaQuery(criteria);
>         return elasticsearchOperations.search(criteriaQuery, Article.class)
>                 .stream().map(SearchHit::getContent).collect(Collectors.toList());
>     }
>
>     // 多字段同时检索
>     public Page<Article> multiMatchSearch(String kw, int page, int size) {
>         NativeQuery q = NativeQuery.builder()
>                 .withQuery(qb -> qb.multiMatch(m -> m
>                         .fields("target_obj.sub_attr", "target_obj.other_sub_attr")
>                         .query(kw).analyzer("ik_smart")))
>                 .withPageable(PageRequest.of(page, size))
>                 .build();
>         SearchHits<Article> hits = elasticsearchOperations.search(q, Article.class);
>         List<Article> data = hits.stream().map(SearchHit::getContent).collect(Collectors.toList());
>         return new PageImpl<>(data, PageRequest.of(page, size), hits.getTotalHits());
>     }
> }
> ```
>
> 三剑客分工：Repository 干简单 CRUD、Operations 干复杂查询 / 高亮、Client 干原生批量 / 局部更新。 Controller 层用构造注入（`@RequiredArgsConstructor + final` 字段），比 `@Autowired` 更安全（启动即校验，避免空指针）。
>
> ## 九、ES 与 MySQL 数据同步（4 种方案）
>
> 表格
>
> | 方案 | 原理 | 优缺点 |
> | --- | --- | --- |
> | 定时任务 | 定时扫 MySQL 全量 / 增量写入 ES | 简单，但有延迟、全量扫压力大 |
> | Logstash | 数据收集工具，定时拉取 | 官方配套，但仍非实时、配置较重 |
> | 双写 | 业务代码同时写 MySQL 和 ES | 实时，但要注意写入失败、数据不一致 |
> | Canal | 监听 binlog 自动同步 | 实时、无侵入，上线最推荐 |
>
> 结论：Canal 监听数据库 binlog，自动关联 ES 同步，是生产最推荐的方案 —— 业务代码不用改，实时性最好。
>
> ## 十、Canal 关联 ES（小白级完整指导）
>
> ### 1. Canal 是啥（一句话秒懂）
>
> Canal 伪装成 MySQL 的「从库」，骗主库把 binlog 推给它；它解析 binlog，把二进制日志翻译成我们看得懂的：哪张表、哪一行、是新增 / 修改 / 删除、变更前后的数据。
>
> `MySQL(主库) → binlog → Canal-Server(伪装从库) → SpringBoot(消费) → Elasticsearch`
>
> 架构核心：MySQL → Canal-Server → SpringBoot → Elasticsearch。
>
> ### 2. 前置条件：MySQL 开启 binlog（ROW 模式）
>
> ```
> # docker-compose 里给 mysql 加
> command:
>   - --server-id=1           # 主库唯一 id
>   - --log-bin=mysql-bin     # 开启 binlog
>   - --binlog_format=ROW     # 必须 ROW 模式
>   - --binlog_row_image=FULL # 记录完整行数据
> ```
>
> 为什么必须 ROW 模式：STATEMENT 模式只记录 SQL 语句，没有行数据，Canal 拿不到字段值。只有 ROW 模式才记录「变更前 / 后」的完整行。
>
> ### 3. 创建 canal 账号并授权
>
> 进入 MySQL 容器执行：
>
> ```
> -- % 代表任意主机
> CREATE USER canal@'%' IDENTIFIED BY 'canal_password';
> -- 适配 canal 1.1.5（MySQL8 默认 caching_sha2_password 它不兼容，必须改回 mysql_native_password）
> ALTER USER canal@'%' IDENTIFIED WITH mysql_native_password BY 'canal_password';
> GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO canal@'%';
> FLUSH PRIVILEGES;
>
> -- 校验 binlog 是否开启
> SHOW VARIABLES LIKE 'log_bin';
> SHOW VARIABLES LIKE 'binlog_format';
> ```
>
> 大坑（必背）：MySQL8 默认用 `caching_sha2_password`，canal 1.1.5 不兼容，必须 `ALTER USER ... WITH mysql_native_password`，否则连不上。
>
> ### 4. 启动 Canal-Server
>
> ```
> canal-server:
>   image: canal/canal-server:v1.1.5
>   container_name: canal-server
>   environment:
>     canal.instance.master.address: mysql:3306     # 用容器名，别写 127.0.0.1
>     canal.instance.dbUsername: canal
>     canal.instance.dbPassword: canal_password
>     canal.instance.filter.regex: test_db\\.article  # 监听 test_db 库的 article 表
>     canal.instance.destination: example            # 实例名，Java 客户端连接要用
> ```
>
> 关键配置：`filter.regex` 监听 `库名.表名`；监听全库用 `库名\\..*`。`destination` 默认 example，Java 客户端必须填这个。
>
> ### 5. SpringBoot 消费端（核心代码）
>
> ```
> @Component
> public class CanalSyncListener {
>     @Value("${canal.host}")
>     private String host;
>     @Value("${canal.port}")
>     private int port;
>     @Value("${canal.destination}")
>     private String destination;
>
>     private CanalConnector connector;
>     private volatile boolean running = true;   // 多线程内存可见性
>
>     @PostConstruct   // 构造后立即启动监听线程
>     public void start() {
>         new Thread(this::run, "canal-sync-thread").start();
>     }
>
>     private void run() {
>         connector = CanalConnectors.newSingleConnector(
>                 new InetSocketAddress(host, port), destination, "", "");
>         connector.connect();
>         connector.subscribe();
>         while (running) {
>             Message message = connector.getWithoutAck(1000); // 一次拉 1000 条 binlog
>             long batchId = message.getId();                   // 这批消息唯一批次号
>             int size = message.getEntries().size();
>             if (batchId == -1 || size == 0) {
>                 try { Thread.sleep(200); } catch (InterruptedException e) {
>                     Thread.currentThread().interrupt();
>                 }
>                 continue;
>             }
>             try {
>                 for (CanalEntry.Entry entry : message.getEntries()) {
>                     if (entry.getEntryType() != CanalEntry.EntryType.ROWDATA) continue; // ROWDATA 才是增删改
>                     CanalEntry.RowChange rowChange = CanalEntry.RowChange.parseFrom(entry.getStoreValue());
>                     String table = entry.getHeader().getTableName();
>                     for (CanalEntry.RowData rowData : rowChange.getRowDatasList()) {
>                         CanalEntry.EventType eventType = rowChange.getEventType();
>                         switch (eventType) {
>                             case INSERT:
>                             case UPDATE:
>                                 saveEs(rowData.getAfterColumnsList(), table);  // after = 新数据
>                                 break;
>                             case DELETE:
>                                 deleteEs(rowData.getBeforeColumnsList(), table); // before = 删除的数据
>                                 break;
>                             default:
>                                 break;
>                         }
>                     }
>                 }
>                 connector.ack(batchId);   // 确认成功，提交位点
>             } catch (Exception e) {
>                 connector.rollback(batchId);  // 失败回滚，下次重新拉取
>                 e.printStackTrace();
>             }
>         }
>         connector.disconnect();
>     }
>
>     private void saveEs(List<CanalEntry.Column> cols, String table) {
>         // 把 Column 列表组装成对象，用 ES 主键 id 调用 save（天然覆盖，实现幂等）
>     }
>
>     private void deleteEs(List<CanalEntry.Column> cols, String table) {
>         // 取主键 id，调用 deleteById 删除 ES 文档
>     }
>
>     @PreDestroy   // 容器销毁时停线程
>     public void stop() {
>         running = false;
>     }
> }
> ```
>
> ### 6. before /after 三个关键点（面试必考）
>
> 表格
>
> | 事件 | before | after |
> | --- | --- | --- |
> | INSERT | 空 | 新数据 |
> | UPDATE | 旧值 | 新值 |
> | DELETE | 删除的数据 | 空 |
>
> 所以 INSERT/UPDATE 取 after，DELETE 取 before。
>
> ### 7. ack /rollback 与幂等
>
> - ack：确认消费成功，提交位点，这条不再重复推送。
> - rollback：消费失败，这批 binlog 下次重新拉取。
> - 幂等：因为 rollback 会重发，所以写入 ES 必须幂等 —— 用 MySQL 主键 id 作为 ES 文档 id，save 天然覆盖。
>
> ### 8. 全量 + 增量流程（重要）
>
> ① 先全量：一次性把 MySQL 历史数据导入 ES ② 再增量：启动 Canal 监听后续变更
>
> Canal 只做增量同步，不管全量。所以上线顺序：先全量导历史数据，再开 Canal 监听增量。
>
> ### 9. 容器化专属坑（必背）
>
> 1. 没加同一个 Docker 网络 → 容器之间 ping 不通（用 docker network create 建网，容器名当域名）。
> 2. MySQL8 密码插件 → 必须 `ALTER USER ... WITH mysql_native_password`。
> 3. 地址写 127.0.0.1 → 容器内 127.0.0.1 是自己，要用容器名（mysql、es、canal-server）。
> 4. filter 正则转义 → `\\.` 才代表点，`.` 直接写会匹配任意字符。
> 5. 启动顺序 → app 依赖 mysql/canal/es，加 depends\_on（注意它只管启动顺序，不保证服务就绪）。
> 6. 时区 → 所有容器统一 Asia/Shanghai，否则同步时间错乱。
>
> ### 10. docker-compose 一键部署（完整版）
>
> ```
> services:
>   mysql:
>     image: mysql:8.0
>     container_name: mysql
>     networks: [sync_net]
>     ports: ["3306:3306"]
>     environment:
>       MYSQL_ROOT_PASSWORD: Root@123456
>       MYSQL_DATABASE: test_db
>     command:
>       - --server-id=1
>       - --log-bin=mysql-bin
>       - --binlog_format=ROW
>       - --binlog_row_image=FULL
>     restart: always
>
>   canal-server:
>     image: canal/canal-server:v1.1.5
>     container_name: canal-server
>     networks: [sync_net]
>     ports: ["11111:11111"]
>     environment:
>       canal.instance.master.address: mysql:3306
>       canal.instance.dbUsername: canal
>       canal.instance.dbPassword: Canal@123456
>       canal.instance.filter.regex: test_db\\.article
>       canal.instance.destination: example
>     restart: always
>
>   es:
>     image: elasticsearch:8.11.0
>     container_name: es
>     networks: [sync_net]
>     ports: ["9200:9200"]
>     environment:
>       discovery.type: single-node
>       xpack.security.enabled: false
>       ES_JAVA_OPTS: "-Xms1g -Xmx1g"
>     restart: always
>
>   app:
>     image: your-springboot-image:latest
>     container_name: es-sync-app
>     networks: [sync_net]
>     ports: ["8080:8080"]
>     depends_on: [mysql, canal-server, es]
>     restart: always
>
> networks:
>   sync_net:
>     driver: bridge
> ```
>
> ```
> docker compose up -d   # 一键启动
> docker logs -f canal-server   # 看到 start canal successful 即成功
> ```
>
> ## 大厂高频考点速查表
>
> 表格
>
> | 考点 | 答案要点 |
> | --- | --- |
> | 倒排索引原理 | 词条 → 文档映射，按词条定位，所以快 |
> | match vs term | match 分词匹配（text），term 精确（keyword / 数值） |
> | ik\_smart vs max | smart 检索，max\_word 关键词提取，版本必须一致 |
> | mapping 能改吗 | 字段类型不可改，只能加字段 / 改 index |
> | must/should/filter | 必须 / 最好 / 过滤（filter 不算分更快） |
> | 同步 4 方案 | 定时任务 / Logstash / 双写 / Canal（最推荐） |
> | Canal 本质 | 伪装 MySQL 从库，解析 binlog，只做增量同步 |
> | binlog 为何要 ROW | statement 无行数据，ROW 才有 before/after |
> | before/after | insert/update 取 after，delete 取 before |
> | 幂等怎么做 | 用主键 id 作 ES 文档 id，save 天然覆盖 |
> | 全量 + 增量 | 先全量导历史，再开 Canal 监听增量 |
