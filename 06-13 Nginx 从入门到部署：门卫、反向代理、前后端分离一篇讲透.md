---
title: "Nginx 从入门到部署：门卫、反向代理、前后端分离一篇讲透"
status: 已发布
author: XiaoYi
---

# Nginx 从入门到部署：门卫、反向代理、前后端分离一篇讲透

> 今天把 Nginx 从「它是干嘛的」讲到「反向代理 + 前后端分离部署」，全配能直接抄的配置。 Let's use AI to create a better world。

## 一、Nginx 到底是个啥？

一句话：**Nginx 是一个「门卫」兼「交通指挥」。**

你部署一个网站，浏览器（访客）要访问你的服务器。谁来接待这些访客、把请求导到正确的地方？就是 Nginx。

它站在最前面，干三件大事：

|  |  |  |
| --- | --- | --- |
| 能力 | 一句话 | 比喻 |
| 静态网站托管 | 直接把 HTML/CSS/JS 发给访客 | 门卫递宣传单 |
| 反向代理 | 把请求转发给后面的程序（如 Spring Boot） | 前台接待，转给后台部门 |
| 负载均衡 | 请求多了，分给多台机器 | 交通指挥，分流 |

> 记牢一个词：**反向代理**——这是 Nginx 最核心、最值钱的技能。

---

## 二、安装（一行）

```
# ubuntu / debian
sudo apt install nginx

# 用 docker 更省事
docker run -d --name nginx -p 80:80 nginx
```

装完访问 `http://你的服务器IP`，能看到 Nginx 欢迎页，就装好了。

---

## 三、配置文件长啥样？

主配置文件在 `/etc/nginx/nginx.conf`，但真正干活的是**一个个 `server` 块**。

结构先看懂，后面全在这个骨架上改：

```
# 一个 server 块 = 一个 [站点] / [服务]
server {
    listen 80;
    server_name xxx.cn;

    # 访问路径 / 时候 处理方式
    location / {
        root /var/www/html; # 去文件夹找
        index index.html; # 默认打开文件
    }
}
```

三个关键词：

- **`server`**：一个网站（一个域名 + 端口）

- **`listen`**：监听哪个端口

- **`location`**：不同的 URL 路径，走不同的处理

---

## 四、能力一：托管静态网站

把你的网页文件（`index.html` 等）放到服务器某个文件夹，让 Nginx 直接发出去：

```
server {
    listen 80;
    server_name my-site.com;

    root /var/www/my-site;
    index index.html;
}
```

把文件丢进 `/var/www/my-site`，访问 `my-site.com` 就能看到网页了。

---

## 五、能力二：反向代理（重点！）

### 为什么要反代？

你的后端程序（比如 Spring Boot）跑在 `8080` 端口。但你不想让用户直接访问 `http://ip:8080`（丑、不安全、暴露端口），而是想让他们访问 `http://api.my-site.com`。

Nginx 就站在前面：**用户访问 80 端口，Nginx 偷偷把请求转给 8080 的后端**，再把结果带回来。用户从头到尾只看到 80。

```
server {
    listen 80;
    server_name api.my-sit.com;

    location / {
        # 代理转发给本机 8080 端口
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host; # 访客访问的域名
        proxy_set_header X-Real-IP $remote_addr; # 真实客户IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # 转发链路
        proxy_set_header Authorization $http_authorization; # 认证头 / cookies自动转发
    }
}
```

> 记牢：**`proxy_pass` = 把请求转给谁。** 这一行就是反向代理的全部精髓。

---

## 六、前后端分离部署（你最常用的场景）

你的项目是 Vue3 前端 + Spring Boot 后端，怎么用 Nginx 一次搞定？

**思路：同一个域名，`/` 走前端静态文件，`/api/` 走后端。**

```
server {
    listen 80;
    server_name my-app.com;

    # 前端
    location / {
        root /var/www/dist;
        try_files $uri $uri/ /index.html # $uri $uri/ 均没有匹配资源就自动返回 /index.html
    }

    # 后端接口
    location /api/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

这样：浏览器访问 `my-app.com` 看到前端页面，页面里调 `my-app.com/api/xxx` 就自动转到后端，**前后端同域，还避开了跨域问题**。

---

## 七、能力三：负载均衡

用户多了，一台后端扛不住，就起多台，让 Nginx 轮流分发：

```
# 定义一组后端
upstream backend {
    # 其他服务器 本机 均可填写
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend; # 转发到上面后端组, 自动轮询
    }
}
```

> `upstream` = 一个「后端小分队」，Nginx 默认一人一个轮流来。

---

## 八、HTTPS（上线必配）

http 是明文传输，不安全。用 `certbot` 免费申请证书 + 自动配 HTTPS：

```
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d my-app.com
```

跑完它会自动改你的配置，把 http 升级成 https，还帮你自动续期。

---

## 九、常用命令（背下来）

```
nginx -t # 检查配置错误
nginx -s reload
nginx -s stop
nginx -s restart
```

> 改完任何配置，先 `nginx -t` 检查，没问题再 `nginx -s reload`。这是标准动作。

---

## 十、和你的 Docker + Nginx 串起来

你项目里 Nginx 是跑在 Docker 里的，配置方式和上面完全一样，只是把配置文件挂载进容器：

```
docker run -d --name nginx \
    -p 80:80 -p 443:443 \
    -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v /var/www/dist:/var/www/dist:ro \
    nginx
```

- `-v` 把本地配置文件、前端文件挂进容器

- 容器里的 Nginx 配置，和你在宿主机上写的**一模一样**

所以：**学好 Nginx 配置，Docker 里、裸机里、云服务器里，都是同一套。**

---

## 总结：一页速查

|  |  |
| --- | --- |
| 需求 | 关键配置 |
| 托管静态网站 | `root /路径;` |
| 反向代理 | `proxy_pass http://127.0.0.1:8080;` |
| 前端 + 后端分离 | `location /` 静态 + `location /api/` 反代 |
| 负载均衡 | `upstream` + `proxy_pass http://名字;` |
| 单页应用路由 | `try_files $uri $uri/ /index.html;` |
| 检查配置 | `nginx -t` |
| 热重载 | `nginx -s reload` |

> 一句话记牢：**Nginx = 门卫。静态文件直接递，动态请求转给后端，请求多就分流。** 核心就一个 `proxy_pass`。
