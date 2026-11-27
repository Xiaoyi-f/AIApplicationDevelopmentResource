---
title: "FastAPI 教学：从第一个接口到异步 ORM"
status: 已发布
author: XiaoYi
---

# FastAPI 教学：从第一个接口到异步 ORM

> 后端你可能会 Spring Boot，但 Python 圈里，FastAPI 是现在做 AI 服务、做接口最快的那条路——**自带文档、自动校验、天然支持异步**。 这篇把 FastAPI 从「hello 接口」一路讲到异步 ORM、乐观锁、悲观锁、分页、测试，全是能直接抄进项目的真代码。
>
> Let's use AI to create a better world。

## 一、FastAPI 是啥？

一句话：**用 Python 写后端接口最快、最爽的框架**。

你写一个函数，它自动帮你变成 HTTP 接口，还**白送一套交互式文档**（`/docs`）。

三个特点记住就够：

|  |  |
| --- | --- |
| 特点 | 好处 |
| **自动文档** | 启动后访问 `/docs`，接口全在网页上，能直接点着测试 |
| **自动校验** | 用 Pydantic 声明字段类型，不合法直接 422 拒绝，不用你手写判断 |
| **原生异步** | `async def` 就是异步接口，高并发不吃亏 |

> 它在你技术栈里的位置：Spring Boot 是「企业级重装」，FastAPI 是「轻快利器」。做 AI Agent、RAG、爬虫服务，我基本都用 FastAPI 起接口。

---

## 二、环境 + 第一个应用

```
# standard 佩戴标准配套包
pip install "fastapi[standard]" uvicorn

from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def hello():
    return {"message": "hello"}

# 启动 uvicorn main:app --reload
```

浏览器打开 `http://127.0.0.1:8000` 看到返回，打开 `/docs` 看到文档。

> `uvicorn main:app` —— `main` 是文件名，`app` 是变量名。改完代码 `--reload` 自动重启。

---

## 三、应用元信息 + 文档自定义

`/docs` 里那些标题、描述、版本，都能配：

```
app = FastAPI(
    title="FastAPI",
    description="描述",
    version="1.0.0",
    terms_of_service="https://xxx.xxx",
    contact={
        "name": "XiaoYi",
        "url": "https://xxx.xx",
        "email": "dev@example.com"
    }
)
```

> `/docs` 是 Swagger 界面，`/redoc` 是另一种风格的文档，同一个接口两套皮肤。

---

## 四、参数怎么传进来（5 种来源）

FastAPI 里「参数从哪来」由你声明的位置决定，一次看全：

```
from fastapi import FastAPI, Header, Cookie, Query

@app.get("/item/{item_id}")
async def api(
    item_id: int,
    q: str = Query(default="1", ge=1), # gt lt le
    user_agent: str = Header(None),
    token: str = Cookie(None)
):
    return {"item_id": item_id, "q": q, "ua": "user_agent", "token": token}
```

|  |  |  |
| --- | --- | --- |
| 来源 | 写法 | 例子 |
| 路径参数 | 函数参数名和路径里同名 | `/item/{item_id}` → `item_id` |
| 查询参数 | `Query(...)` | `?page=1` |
| 请求头 | `Header(...)` | `User-Agent` |
| Cookie | `Cookie(...)` | 浏览器自动带 |
| 请求体 | `DTO`（Pydantic 模型） | POST 的 JSON |

> **请求体**是重头戏，用 Pydantic 模型声明，见下一节。

---

## 五、DTO + 字段校验（Pydantic）

```
from pydantic import BaseModel, Field, EmailStr

class DTO(BaseModel):
    name: str = Field(default=None, min_length=0, max_length=10, description="姓名")
    price: int | float = Field(ge=0, description="价格")
    email: EmailStr

@app.post("/item")
# 不符合自动要求自动返回422校验错误
async def create(dto: DTO):
    return {"name": dto.name, "price": dto.price, "email": dto.email}
```

- `Field` 能做**长度、范围、描述**等校验，不合法 FastAPI 自动返回 422

- `EmailStr` 自动验邮箱，需要先装 `pip install email-validator` | | | | --- | --- |

- `int | float` 是 Python 3.10+ 的联合类型写法，等价于「整数或小数都行」

---

## 六、返回响应（自定义内容 + 设 Cookie）

```
from fastapi.responses import RedirectResponse, JSONResponse

@app.get("/redirect")
async def redirect():
    return RedirectResponse(url="/item")

@app.get("/login")
async def login():
    resp = JSONResponse(content={"code": 0, "data": "ok"})
    resp.set_cookie(
        key="session_token",
        value="abc123",
        max_age=3600,
        httponly=True,
        secure=False, # 生产上 https 建议 True
        path="/"
    )

    return resp
```

> 顺带回顾：上一章讲 Nginx 反代时说的 Cookie「域名对不上」的坑，这里的 `set_cookie` 就是那个「发卡」的地方。

---

## 七、跨域 CORS（前后端分离必配）

Vue 前端跑在 5173，后端跑在 8000，浏览器默认不让跨域请求。加这个中间件解决：

```
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    max_age=3600
)
```

---

## 八、文件上传

```
from fastapi import UploadFile, File
import shutil

@app.post("/files")
# File() 表示以multipart/form-data形式上传文件 ... 表示必填
async def upload(files: list[UploadFile] = File(...)):
    with open(f"uploads/{files[0].filename}", "wb") as buffer:
        shutil.copyfileobj(files[0].file, buffer)
    return {"content-type": files[0].content_type, "size": files[0].size}
```

两个关键点：

- **`= File(...)` 必须写**：笔记里漏了这步。不写的话 FastAPI 会把 `files` 当成查询参数而不是文件，直接报错。

- **`shutil.copyfileobj`**：边读边写，比一次性 `read()` 省内存，大文件更稳。

> 单个文件就写 `file: UploadFile = File(...)`，多个才用 `list[UploadFile]`。

---

## 九、SSE 流式输出（AI 对话必用）

AI 一个字一个字往外蹦，靠的就是 SSE：

```
import json
from fastapi.responses import StreamingResponse

@app.get("/sse")
async def sse():
    async def gen():
        yield f"data: {json.dumps({"status": "connected"})}\n\n"
        yield f"event: event_name\ndata: {json.dumps({"data": "data"})}\n\n"

    return StreamingResponse(gen(), media_type="text/event-stream", headers={
        "Cache-Control": "no-cache",
        "X-Accel-Buffering": "no"
    })
```

---

## 十、SQLModel 异步 ORM（增删改查）

### 1. 准备

```
# pip install sqlmodel aiomysql

from sqlalchemy.ext.asyncio import create_async_engine # 异步引擎
from sqlmodel.ext.asyncio.session import AsyncSession # 异步会话
from sqlmodel import SQLModel

engine = create_async_engine("mysql+aiomysql://root:password@127.0.0.1:3306/dbname")

async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(SQLModel.metadata.create_all) # 初次自动建表

"""
基本类型:
bool
int
Decimal
str
"""

from sqlmodel import Field, Column, BigInteger

class Base(SQLModel):
    __abstract__ = True

    def to_dict(self):
        """遍历所有列, 取属性值拼字典"""
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}

class Table(Base, table=True):
    __tablename__ = "my_table"

    id: int = Field(default=None, primary_key=True, sa_type=BigInteger, sa_column_kwargs={"autoincrement": True}, nullable=False, unique=True)
```

### 4. 增删改查（四件套）

```
from sqlmodel import select

# 增
async def create():
    async with AsyncSession(engine) as session:
        session.add(Table())
        await session.commit()

# 删
async def delete(row_id: int):
    async with AsyncSession(engine) as session:
        # Table 自定义模型类名
        row = await session.get(Table, row_id)
        if row:
            await session.delete(row)
            await session.commit()

# 改
async def update(row_id: int):
    async with AsyncSession(engine) as session:
        row = await session.get(Table, row_id)
        if row:
            row.name = "new_name"
            await session.commit()

# 查
async def query():
    async with AsyncSession(engine) as session:
        stat = select(Table).where(Table.id > 0)
        row = await session.exec(stat).first() # 查不到返回None
        rows = (await session.exec(stat)).all() # 拿全部
```

---

## 十一、并发控制：乐观锁 vs 悲观锁

这是后端面试和真实项目都会碰的「并发扣钱」问题。

### 乐观锁（版本号，先读再改，改时比对）

```
from sqlmodel import update

old_version = 0

stat = (
    update(Table)
    .where(Table.id == row_id, Table.version == old_version)
    .values(name="new_value", version=old_version + 1)
)

res = await session.exec(stat)
# res.rowcount 受影响行数 0 表示他人改动过 需要重试
```

> 思路：不锁表，改的时候带上「我看到的版本号」，如果版本变了说明有人抢先，就更新失败重试。

### 悲观锁（直接上锁，阻塞别人）

```
# 排他锁: 其他事务既不可读也不可写, 等我提交
stat = select(Table).where(Table.id == row_id).with_for_update()
row = (await session.exec(stat)).first() # 自动上锁
row.balance -= 100
await session.commit() # 提交自动释放锁

# 共享锁: 别人可读, 不能写
stat = select(Table).where(Table.id == row_id).with_for_update(read=True)
```

|  |  |  |
| --- | --- | --- |
|  | 乐观锁 | 悲观锁 |
| 思路 | 不锁，改时比对版本 | 直接锁住记录 |
| 并发冲突少 | 合适，性能好 | 略重 |
| 并发冲突多 | 频繁重试 | 更稳，但可能阻塞 |

---

## 十二、分页查询

```
from math import ceil
from sqlmodel import func, select

async def page_query(cur_page: int, page_size: int):
    offset_val = (cur_page - 1) * page_size

    async with AsyncSession(engine) as session:
        stat = select(Table).order_by(Table.id.asc()).offset(offset_val).limit(page_size)
        rows = (await session.exec(stat)).all()

        count_stmt = select(func.count()).select_from(Table)
        total = (await session.exec(count_stmt)).one() # 取出该数字 -- 必须有且只有一条
        total_pages = ceil(total / page_size)

    return {"rows": rows, "total": total, "total_pages": total_pages}
```

---

## 十三、测试（不用真起服务也能测）

```
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_hello():
    resp = client.get("/")
    assert resp.status_code == 200
    assert resp.json()["message"] == "hello"
```

> 改完接口跑一遍 `pytest`，比手动点 `/docs` 靠谱得多。

---

## 十四、一页速查

|  |  |
| --- | --- |
| 你想干嘛 | 写法 |
| 路径参数 | `/item/{item_id}` → 函数参数 `item_id` |
| 查询参数 | `q: str = Query(default="1")` |
| 请求头 | `ua: str = Header(None)` |
| Cookie | `token: str = Cookie(None)` |
| 请求体 | `dto: DTO`（Pydantic 模型） |
| 文件上传 | `files: list[UploadFile] = File(...)` |
| 重定向 | `return RedirectResponse(url="/x")` |
| 设 Cookie | `resp.set_cookie(key=..., value=...)` |
| 跨域 | `app.add_middleware(CORSMiddleware, ...)` |
| 流式 | `StreamingResponse(gen(), media_type="text/event-stream")` |
| 异步会话 | `async with AsyncSession(engine) as session:` |
| 上锁 | `.with_for_update()` / `.with_for_update(read=True)` |
| 分页 | `.offset(n).limit(m)` |

---

> 这篇把 FastAPI 的主干 + SQLModel 异步 ORM + 并发锁都串完了，你手里的后端线（Spring Boot ↔ FastAPI）算是补齐了。
