---
title: "Git + GitHub + CI/CD：从本地存档到自动部署，一篇讲透"
status: 已发布
author: XiaoYi
---

# Git + GitHub + CI/CD：从本地存档到自动部署，一篇讲透

> 今天把 Git、GitHub、CI/CD 三个东西串成一条线，从「记个存档」讲到「push 一下自动上线」。
> Let's use AI to create a better world。

## 一、先搞懂三个东西的关系（一张图）

|  |  |  |
| --- | --- | --- |
| 工具 | 一句话 | 比喻 |
| **Git** | 本地管理代码版本 | 游戏里的「存档点」 |
| **GitHub** | 云端放代码、多人协作 | 「网盘」+「团队协作大厅」 |
| **CI/CD** | 自动测试、自动部署 | 「自动打包机器人」 |

它们是一条流水线：

> 你在本地用 **Git** 记存档 → 推到 **GitHub** 云端 → **CI/CD** 机器人看到有新代码，自动测试 + 自动部署上线。

---

## 二、Git 是啥？——游戏的存档点

写代码最怕什么？改着改着崩了，回不去了。

Git 就是给你一个**随时存档、随时回档**的能力：

- 每完成一点，就存个档（commit）

- 哪天改崩了，一键回到上一个存档

- 想开个新玩法，就开个新分支（branch），不行就删掉，主线不受影响

### Git 本地五件套（必会）

```
git init
git add .
git commit -m '说明'
git status
git log -- q退出
```

> 记牢三步曲：**改代码 → `git add .` → `git commit -m "..."`**。

---

## 三、分支：平行世界

分支 = **在主干旁边开一条岔路**，你在岔路上随便折腾，不影响主干。

```
git branch branch_name
git checkout branch_name
git checkout -b branch_name
git branch
```

改完、测试没问题了，再合回主干：

```
git checkout master
git merge branch_name -- 将branch_name分支合并到master
```

> 一句话：**主干稳定版，分支开发版，开发完 merge 回来。**

---

## 四、GitHub 是啥？——云端的代码仓库

Git 管本地，GitHub 管云端。把本地代码推到 GitHub，好处有三：

1. **备份**：电脑坏了，云端还有

1. **协作**：别人能一起改

1. **开源**：全世界都能看你的项目、给你点 star

### 本地 Git ↔ GitHub 联动四件套

```
# 初次关联远程仓库
git remote add origin https://github.com/xiaoyi-f/project.git

# 推上去
git push -u origin master

# 拉下来
git pull origin branch_name

# 克隆 - 带顶层文件夹
git clone -b branch_name https://github.com/xiaoyi-f/project.git
```

> 记牢：**push 是上传，pull 是下载，clone 是第一次整包下载。**

---

## 五、团队协作的核心：Fork + Pull Request

这是开源世界最常用的玩法（也是你以后做开源项目涨粉的路）：

1.**Fork**：把别人的项目「复制一份」到自己账号下

2.在自己的副本里改代码、`commit`、`push`

3.发起 **Pull Request（PR）**：请求原作者「把我改的合并进你的项目」

4.原作者 review 通过，就合并了

> 一句话：**Fork 抄一份 → 改完 push → 发 PR 求合并。** 这就是「参与开源」的完整姿势。

---

## 六、CI/CD 是啥？——自动打包机器人

以前发版是人工：改代码 → 手动测试 → 手动打包 → 手动传服务器 → 手动重启。烦，还容易漏。

CI/CD 就是把这一整套**自动化**：

- **CI（持续集成）**：每次 push 代码，自动跑测试，保证不坏

- **CD（持续部署）**：测试过了，自动打包、自动部署到服务器上线

你只管 push，剩下的机器人全包了。

---

## 七、GitHub Actions 实战（重头戏）

GitHub 内置的 CI/CD 工具叫 **GitHub Actions**。你只要在项目里放一个配置文件，它就会自动跑。

### 文件放哪

在项目根目录建这个文件：`.github/workflows/deploy.yml`

### 一个完整配置（贴合你的 Spring Boot + Docker + Nginx）

```
name: 构建并部署

on:
  push:
    branches: [ master ]

jobs:
  deploy:
    runs-on: ubuntu-latest # ubuntu 虚拟机

    steps:
      - name: 拉取代码
        uses: actions/checkout@v4

      - name: 设置 JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: 用 Maven 打包
        run: mvn -B package -DskipTests

      - name: 构建 Docker 镜像
        run: docker build -t my-app:latest .

      - name: 部署到服务器
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_KEY }}
          script:
            docker pull my-app:latest
            docker compose up -d
            nginx -s reload
```

### 这个配置在干嘛？（翻译成人话）

你每次 `git push` 到 main 分支，GitHub 就自动：

1.开一台干净虚拟机

2.拉下你的最新代码

3.装好 Java、用 Maven 打包

4.打包成 Docker 镜像

5.用 ssh 连到你的服务器，拉镜像、重启容器、刷新 Nginx

**全程你只做了一件事：`git push`。** 这就是 CI/CD 的爽点。

---

## 八、密钥别写死，放 Secrets（⚠️ 重要）

上面配置里的 `服务器地址`、`私钥` 都用 `${{ secrets.XXX }}` 占位，而不是直接写进去——因为：

- 写进代码里，一旦开源就**全世界都看到你的服务器密码**

- 正确做法：存到 GitHub 仓库的 **Settings → Secrets and variables → Actions** 里

> 这也正是你项目里的铁律：**密钥走环境变量/Secrets，禁止硬编码、禁止提交 Git。**

---

## 总结：一页速查

|  |  |
| --- | --- |
| 操作 | 命令 |
| 初始化 | `git init` |
| 暂存 | `git add .` |
| 存档 | `git commit -m "..."` |
| 看状态 | `git status` |
| 看历史 | `git log` |
| 新建分支 | `git checkout -b xxx` |
| 合并分支 | `git merge xxx` |
| 关联远程 | `git remote add origin 地址` |
| 上传 | `git push` |
| 下载 | `git pull` |
| 克隆 | `git clone 地址` |
| 自动部署 | 写 `.github/workflows/*.yml` |

> 一句话记牢：**Git 本地存档，GitHub 云端协作，CI/CD 自动上线。** 你 push，剩下的交给机器人。
