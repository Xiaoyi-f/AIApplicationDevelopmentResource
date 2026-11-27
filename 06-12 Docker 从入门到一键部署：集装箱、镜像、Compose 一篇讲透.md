---
title: "Docker 从入门到一键部署：集装箱、镜像、Compose 一篇讲透"
status: 已发布
author: XiaoYi
---

# Docker 从入门到一键部署：集装箱、镜像、Compose 一篇讲透

> 今天把 Docker 从「集装箱是啥」讲到「docker compose 一键部署整个项目」，全配能抄的代码。 Let's use AI to create a better world。

## 一、Docker 到底解决什么问题？

你一定听过这句话：**「我电脑上跑得好好的，怎么到服务器上就不行了？」**

原因：环境不一样——Java 版本、依赖、系统库，差一点就崩。以前部署要在服务器上手动装一堆环境，又慢又容易错。

Docker 的解法：**把「代码 + 环境」打包成一个标准箱子，到哪都能原样运行。**

> 比喻：Docker 就是「集装箱」。你把货物（代码）和环境（依赖、配置）一起装进一个标准箱子，不管是本地、测试机、还是云服务器，这个箱子都能直接卸下来跑，不用管目标地是什么环境。

---

## 二、三个核心概念（先搞懂）

|  |  |  |
| --- | --- | --- |
| 概念 | 一句话 | 比喻 |
| **镜像 Image** | 打包好的模板（代码+环境） | 菜谱 / 集装箱的「设计图」 |
| **容器 Container** | 镜像跑起来的实例 | 按菜谱做出来的「一盘菜」 |
| **仓库 Registry** | 存镜像的地方 | 菜谱「书库」（Docker Hub） |

关系：

> 从 **仓库** 拉一个 **镜像** → 运行成一个 **容器**。一个镜像可以跑出无数个容器。

---

## 三、安装（一行）

```
# ubuntu / debian
curl -fsSL https://get.docker.com | sh

# 顺便安装 docker compose
sudo apt install docker-compose-plugin

# 验证
docker --version
docker compose version
```

---

## 四、跑第一个容器

```
# 拉镜像并且后台跑起来
docker run -d -p 80:80 nginx # -d 后台 / -p 端口映射
```

命令拆解：

- `docker run`：运行一个容器

- `-d`：后台运行（detached）

- `-p 80:80`：**端口映射**，把「宿主机的 80 端口」映射到「容器的 80 端口」——外面访问 80，就进到容器里了

- `nginx`：镜像名（本地没有会自动去 Docker Hub 拉）

跑完访问 `http://localhost`，能看到 Nginx 欢迎页。

---

## 五、常用命令（背下来）

```
docker pull 镜像名
docker images
docker run -d -p 80:80 镜像
docker ps
docker ps -a
docker stop 容器ID
docker rm 容器ID
docker exec -it 容器ID bash
docker logs 容器ID
docker rmi 镜像ID
```

> 记牢四个最常用的：**run 跑、ps 看、stop 停、logs 看日志。**

---

## 六、Dockerfile：构建自己的镜像

别人的镜像（nginx、mysql）能直接用。但你自己写的 Spring Boot 项目，怎么打包成镜像？靠 **Dockerfile**。

在项目根目录建一个 `Dockerfile`：

```
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/my-app.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

然后构建：

```
docker build -t my-app:latest .
```

再运行：

```
docker run -d -p 8080:8080 my-app:latest
```

> Dockerfile 就四个词：**FROM 基础、COPY 复制、EXPOSE 端口、CMD 启动命令。**

---

## 七、Docker Compose：一次编排多个容器

真实项目不可能只有一个容器——你有 Spring Boot、MySQL、Redis、Nginx，四个服务要一起跑。

一个个 `docker run` 又麻烦、又要手动管启动顺序。**Docker Compose** 就是：**写一个配置文件，一次把全家桶拉起来。**

配置文件叫 `docker-compose.yml`，放在项目根目录。

### 一份贴合你技术栈的完整配置

```
services:
  background:
    build: . # 使用当前目录下 Dockerfile 现场构建
    container_name: project_src
    networks:
      - sync_net
    ports:
      - "8080:8080"
    depends_on: # 启动顺序
      - db
      - redis
    restart: always

  db:
    image: mysql:8.0
    container_name: mysql
    networks:
      - sync_net
    environment:
      MYSQL_ROOT_PASSWORD: passwd
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"
    restart: always

  redis:
    image: redis:7-alpine
    container_name: redis
    networks:
      - sync_net
    command: ["redis-server", "--requirepass", "passwd", "--appendonly", "yes"]
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    restart: always

volumes:
  mysql_data:
    driver: local
  redis_data:
    driver: local

networks:
  sync_net:
    driver: bridge
```

### 一键启动 / 停止

```
docker compose pull # 拉取所有需要的镜像
docker compose -f compose.yml up -d
docker compose ps
docker compose down
docker compose stop
docker compose restart
docker compose logs -f 服务名
```

> 你上一篇文章学的 Nginx 配置，就挂在上面 `nginx.conf` 里，跟 Compose 无缝衔接。

### 补充 - 环境变量

```
--> MySQL
MYSQL_ROOT_PASSWORD
MYSQL_DATABASE

--> Redis
REDIS_ARGS="--requirepass mainredis --appendonly yes"

--> Elasticsearch
ELASTIC_PASSWORD

--> Postgre
POSTGRES_USER
POSTGRES_PASSWORD
POSTGRES_DB

--> Mongo
MONGO_INITDB_ROOT_USERNAME
MONGO_INITDB_ROOT_PASSWORD

--> RabbitMQ
RABBITMQ_DEFAULT_USER
RABBITMQ_DEFAULT_PASS

--> Neo4j
NEO4J_AUTH=neo4j/password
```

---

## 八、为什么 Docker 值得学（真实价值）

1. **部署不再痛苦**：环境全打包，到哪都能跑

1. **一条命令起全套**：`docker compose up -d`，MySQL/Redis/后端/Nginx 全起来

1. **删了重来很简单**：`docker compose down` 再 up，环境秒复原

1. **面试 + 接单加分**：现在后端岗位几乎都要求会 Docker

---

## 总结：一页速查

|  |  |
| --- | --- |
| 需求 | 命令 / 配置 |
| 跑容器 | `docker run -d -p 80:80 镜像` |
| 看容器 | `docker ps` / `docker ps -a` |
| 停 / 删 | `docker stop ID` / `docker rm ID` |
| 看日志 | `docker logs ID` |
| 进容器 | `docker exec -it ID bash` |
| 构建镜像 | `docker build -t 名字 .` |
| 一键编排 | `docker compose up -d` |
| 一键停止 | `docker compose down` |

> 一句话记牢：**镜像=打包好的模板，容器=跑起来的实例，Compose=一次拉起所有服务。** 核心就这三件事。

---
