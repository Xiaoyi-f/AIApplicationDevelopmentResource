---
title: "上手Python看这篇足以"
status: 已发布
author: XiaoYi
---

# 上手Python看这篇足以

Python是一种动态解释性脚本语言，作者：吉多·范罗苏姆，大家常叫他"龟叔"

> 动态：运行时候才确定变量不用提前声明类型
>
> 解释性：不用一次性编译为二进制文件执行，而是由解释器逐行执行命令
>
> 脚本：用来快速写短小任务，侧重简单快速开发

安装Python [Welcome to Python.org](https://www.python.org/ "Welcome to Python.org") 前往官网下载对应操作系统版本即可

注意：不要安装过新版本，一般可以查看版本情况（bugfix / security / pre-release）下载最老的一个security版本最为稳定

下载好Python之后，自带一个IDLE（集成开发学习环境）用于给小白学习和练习（鼓励以此为实验台 / 练习场） --> pip install ipython 高级版本IDLE

pip工具也是下载好python之后自带的，等价于下载python包（依赖库）的应用商店

我们开发项目一般会使用 python -m venv .venv 创建一个虚拟环境然后再activate激活虚拟环境作为我们的开发环境，之后 pip list -> pip config list -> pip install uv -> uv pip install pip-conf-mirror

-> pip-conf-mirror --list -> pip-conf-mirror --uv <source\_name>

> 常用命令：
>
> uv pip install / uninstall package
>
> uv pip list
>
> uv pip show package
>
> uv pip freeze > requirements.txt
>
> uv pip install -r requirements.txt

## Python正确学习方式

> Python是一种简单的面向对象高级编程语言，不需要自我进行内存管理，包括类型都是自检

### 1.对Python基础进行认识

通用优先级和标识符规则与其他语言同理 -- pass

基本内置函数：min max sum len count round range map(lambda arg: pass, iterable) filter(lambda arg: pass, iterable)

Python支持 '''内容''' """内容""" 作为文档字符串，同时文档字符串也可作为文档注释

通过 import keyword -> print(keyword.kwlist) # 查看Python内置关键词 -> # 开头为行注释

我们首要任务就是认识所有关键词，这是学习任何语言的基本一步

补充：Python现在早已支持 match 块 （自行AI了解学习，锻炼AI运用学习能力）

### 2.认识Python数据类型

int float bool complex

list tuple dict set str iter

之后我们把这些数据类型搞清楚同时学会它们的常用方法便已经掌握Python基础60%

list常用方法 -> append insert pop remove clear

dict常用方法 -> keys values items get update pop("key") clear

set常用方法 -> add update remove discard clear

str常用方法 -> replace split join find title lower upper strip

iter函数可以将可迭代类型转换为迭代器，通过next函数逐个取出

推导式是Python中一种语法糖，eg：{key, value for key, value in enumerate(iterable, start=0)}

**Decimal -->**

```
from decimal import Decimal

n = Decimal("123.456")

# 配合 sqlmodel -->
from sqlmodel import Field, SQLModel

class Job(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    salary: Decimal = Field(max_digits=6, decimal_places=3)
```

### 3.学习Python装饰器

```
def decorator(*args, **kwargs):
    def decorate(func):
        def wrapper(*args, **kwargs):
            # setup
            res = func(*args, **kwargs)
            # teardown
            return res
        return wrapper
    return decorate
```

### 4.学习Python类 -- 面向对象

> 面向对象三大特性：封装、继承、多态

```
from abc import ABC, abstractmethod
from enum import Enum

class Base(ABC):

    def __init__(self, name):
        self.name = name

    @abstractmethod
    def abstract_method(self, *args):
        raise NotImplementedError

    @staticmethod
    def static_method():
        pass

class ImplBase(Base):

    def __init__(self):
        super().__init__()

    def abstract_method(self, *args):
        pass

    @property
    def upper_name(self):
        return self.name.upper()

class ErrorCode(Enum):

    SUCCESS = (0, "ok", "")
    PARAMS_ERROR = (40000, "参数错误", "")
    NULL_ERROR = (40001, "空参错误", "")
    NOT_LOGIN = (40100, "未登录", "")
    NOT_AUTH = (40101, "无权限", "")
    SYSTEM_ERROR = (50000, "系统内部异常", "")

    def __init__(self, code, message, description):
        self.code = code
        self.message = message
        self.description = description
```

 Tip：封装、继承、多态需要自我进一步学习，再进一步学习 Python协议 最后多看源码

### 5.文件读写

```
# 正则
import re

TEXT = "target_text"
re_expr = re.compile(r"^[^0-9a-zA-Z].*.{3,6}$", re.DOTALL | re.IGNORECASE)
matched_resp = re.findall(re_expr, TEXT) # iterable

# 临时文件夹
import sys, shutil, tempfile, os

sys.path.insert(0, os.path.join(os.path.dirname(os.path.abspath(__file__)), "target"))
temp_path = tempfile.mkdtemp(prefix="temp_dir", dir=".")
shutil.rmtree(temp_path, ignore_errors=True)

# 基本文件读写 - newline设置为空用于读取特定模块对应文件，让模块自己接管换行处理[csv]
with open("f", "a+", encoding="utf-8", newline="") as f:
    text = f.read()
    line_text = f.readline()
    f.write("text")
    f.writelines(["line\n", "other_line\n"])
    f.seek(0, 0)
    f.seek(0, 2)

# 环境变量文件读写 -- uv pip install python-dotenv
from dotenv import load_dotenv
import os

load_dotenv()

os.getenv("ENV_VAR")

# JSON文件读写
import json

source = {"key": "value"}

json_str = json.dumps(source, indent=2, ensure_ascii=False)
json_data = json.loads(json_str)

data = json.load("file_obj")
json.dump(source, "file_obj", indent=2, ensure_ascii=False)

# CSV文件读写
import csv

dict_reader = csv.DictReader("file_obj") # dict_reader.fieldnames

fieldnames = ["name", "skill", "score"]
dict_writer = csv.DictWriter("file_obj", fieldnames=fieldnames)
dict_writer.writeheader()
dict_writer.writerow({"name": "XiaoYi", "skill": "Java", "score": 98})
dict_writer.writerows("dict_list")
```

### 6.进程池与线程池

```
from multiprocessing import Manager
from threading import Lock as ThreadLock
from concurrent import futures

# (Target / target) 代表 (Thread / thread) / (Process / process)
"""
def targetWorker(arg, lock):
    with lock:
        return "result"

args = [1, 2, 3]

with futures.TargetPoolExecutor(max_workers=5) as executor:
    lock = TargetLock() # Process -> Manager().Lock()
    futures_list = [executor.submit(targetWorker, arg, lock) for arg in args]

    for future in futures.as_completed(futures_list):
        print(future.result())
"""

import threading

_thread_local = threading.local()

# Tip: 全局只写一个threading.local()
# 线程 1 设置 _thread_local.name = "t1"
# 线程 2 设置 _thread_local.name = "t2"
# 两个线程访问的是同名属性，但底层是两块完全独立的数据，不会互相覆盖！
def worker():
    # 每个线程自己的name，别的线程看不到
    _thread_local.name = threading.current_thread().name
    print(_thread_local.name)
```
