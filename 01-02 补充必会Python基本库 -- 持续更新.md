---
title: "补充必会Python基本库 -- 持续更新"
status: 已发布
author: XiaoYi
---

# 补充必会Python基本库 -- 持续更新

```
import uuid

session_id = uuid.uuid4().hex

from datetime import datetime

def now_str() -> str:
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

import random

n = random.randint(1, 10) # [1, 10] 整数
f = random.uniform(1, 10) # [1, 10] 浮点数

lst = [1, 2, 3, 4, 5, 6]
items = random.choices(lst, k=3) # 随机选取可重复
sample = random.sample(lst, 3) # 随机选取不重复

# 打乱旧列表顺序
random.shuffle(lst)

import argparse

parser = argparse.ArgumentParser(description="演示脚本")
parser.add_argument("input", help="输入文件路径")
parser.add_argument("--num", type=int, default=0, help="设置数字")
parser.add_argument("--debug", action="store_true", help="开启调试模式")

args = parser.parse_args()

print(args.input, args.num, args.debug)

import subprocess

ret = subprocess.run(["python", "--version"], capture_output=True, text=True)
print(ret.returncode, ret.stdout, ret.stderr)

from pydantic import BaseModel, Field

class User(BaseModel):

    id: int
    name: str
    age: int | None = Field(default=None, description="年龄")

data = {"id": "0", "name": "XiaoYi"}
user = User(**data)

print(user.id, user.name, user.age) # 自动将str转int
print(user.model_dump()) # 转字典
print(user.model_dump_json()) # 转JSON

# bad = User(id="0", name="test") 跑错ValidationError

import logging
import os

from app.config import LOG_DIR # 我们将日志存放目录配置到config模块

_FORMAT = "%(asctime)s - %(levelname)s - [%(name)s] - %(message)s"

def get_logger(name: str = "crawler") -> logging.Logger:
    logger = logging.getLogger(name)
    if logger.handlers:
        return logger

    logger.setLevel(logging.INFO)
    formatter = logging.Formatter(_FORMAT)

    stream_handler = logging.StreamHandler()
    stream_handler.setLevel(logging.INFO)
    stream_handler.setFormatter(formatter)
    logger.addHandler(stream_handler)

    file_handler = logging.FileHandler(os.path.join(LOG_DIR, "crawler.log"), encoding="utf-8")
    file_handler.setLevel(logging.INFO)
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)

    return logger

import pytest

def test_assert():
    assert 2 > 1
    assert 0.1 + 0.2 == pytest.approx(0.3)

# scope -> function, class, module, session
@pytest.fixture(scope="function", autouse=False)
def db_conn():
    conn = "模拟数据库连接"
    yield conn
    print("模拟关闭数据库连接")

# monkeypatch -> pytest内置夹具
def test_inner(monkeypatch):
    """将目标函数替换为lambda函数"""
    monkeypatch.setattr("Python.Base.static_method", lambda: "mock")

# 万能假对象
from unittest.mock import MagicMock

virtual_obj = MagicMock()

# pytest.ini -- pytest 命令自动执行 pytest 文件夹里面所有测试文件
"""
[pytest]
testpaths = test
pythonpath = .
addopts = -v -s
"""
```
