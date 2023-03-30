---
title: Python日志库logging
tags: [Python, logging]
categories: [Python]
date: 2023-03-30 15:33:05
description: Python日志库logging的使用
cover: http://cdn-hw-static.shanhutech.cn/bizhi/staticwp/202212/13746e6880e009d49378a81f0dec5474--569975149.jpg
---

# 1. logging日志库

参考：

[logging官方文档](https://docs.python.org/3/howto/logging.html)

[python logging模块使用教程 - 简书](https://www.jianshu.com/p/feb86c06c4f4)

[python日志：logging模块使用 - 知乎](https://zhuanlan.zhihu.com/p/360306588)



## 1.1 日志类型

| Level      | When it’s used                                                                                                                                                         |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DEBUG`    | Detailed information, typically of interest only when diagnosing problems.                                                                                             |
| `INFO`     | Confirmation that things are working as expected.                                                                                                                      |
| `WARNING`  | An indication that something unexpected happened, or indicative of some problem in the near future (e.g. ‘disk space low’). The software is still working as expected. |
| `ERROR`    | Due to a more serious problem, the software has not been able to perform some function.                                                                                |
| `CRITICAL` | A serious error, indicating that the program itself may be unable to continue running.                                                                                 |




## 1.2 修改日志格式

函数格式：`logging.basicConfig(filename='', filemode='', format='', datefmt='', level='')`

使用`logging.basicConfig(format='')`，通过修改`format`参数指定日志输出格式

```Python
import logging
logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)

logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%m/%d/%Y %I:%M:%S %p')  # asctime表示详细日期

```



`basicConfig`参数信息：

| 关键字   | 描述                                                                                        |
| -------- | ------------------------------------------------------------------------------------------- |
| filename | 创建一个FileHandler，使用指定的文件名，而不是使用StreamHandler。                            |
| filemode | 如果指明了文件名，指明打开文件的模式（如果没有指明filemode，默认为'a'）。                   |
| format   | handler使用指明的格式化字符串。                                                             |
| datefmt  | 使用指明的日期／时间格式。                                                                  |
| level    | 指明根logger的级别。                                                                        |
| stream   | 使用指明的流来初始化StreamHandler。该参数与'filename'不兼容，如果两个都有，'stream'被忽略。 |




format格式：

| 格式           | 描述                   |
| -------------- | ---------------------- |
| %(levelno)s    | 打印日志级别的数值     |
| %(levelname)s  | 打印日志级别名称       |
| %(pathname)s   | 打印当前执行程序的路径 |
| %(filename)s   | 打印当前执行程序名称   |
| %(funcName)s   | 打印日志的当前函数     |
| %(lineno)d     | 打印日志的当前行号     |
| %(asctime)s    | 打印日志的时间         |
| %(thread)d     | 打印线程id             |
| %(threadName)s | 打印线程名称           |
| %(process)d    | 打印进程ID             |
| %(message)s    | 打印日志信息           |




## 1.3 logging的高级应用

logging采用模块化设计，包含四种组件：**Loggers记录器**、**Handlers处理器**、**Formatters格式化器**、**Filters过滤器**。

### 1.3.1 Loggers记录器

给应用程序提供可记录日志的接口。

#### 1. 调用接口

```Python
# logger属于单例模式，即__name__相同，则获取的logger相同
logger = logging.getLogger(__name__)
```

#### 2. 设置日志级别

```Python
logger.setLevel()
```

#### 3. 与处理器的联用

```Python
logger.addHandler()
logger.removeHandler()
```



### 1.3.2 Handlers处理器

常见的处理器

- StreamHandler：屏幕输出
    - `sh = logging.StreamHandler(stream=None)`
- FileHandler：文件记录
    - `fh = logging.FileHandler(filename, mode='a', encoding=None, delay=False)`
- BaseRotatingHandler：标准的分割文件日志
- RotatingFileHandler：按文件大小记录日志
- TimeRotatingFileHandler：按时间记录日志



`fh.setFormatter(formatter1)`：给处理器`sh`设置日志格式，`formatter1`表示格式化器的实例



### 1.3.3 Formatters格式化器

调用接口：

```Python
formatter = logging.Formatter(fmt=None, datefmt=None)
```

fmt表示日志格式，datefmt表示日期格式



### 1.3.4 Filters过滤器

调用接口：

```Python
filter = logging.Filter(name='')
```



### 1.3.5 logging示例代码

```Python
import logging

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s %(filename)s %(levelname)s %(message)s',
                    datefmt='%a %d %b %Y %H:%M:%S',
                    filename='my.log',
                    filemode='w')

logging.info('This is a info.')
logging.debug('This is a debug message.')
logging.warning('This is a warning.')

#记录器
logger1 = logging.getLogger("logger1")
logger1.setLevel(logging.DEBUG)
print(logger1)
print(type(logger1))


#处理器
#1.标准输出
sh1 = logging.StreamHandler()
sh1.setLevel(logging.DEBUG)


# 2.文件输出
# 没有设置输出级别，将用logger1的输出级别(并且输出级别在设置的时候级别不能比Logger的低!!!)，设置了就使用自己的输出级别
fh1 = logging.FileHandler(filename="fh.log",mode='a')
fh1.setLevel(logging.DEBUG)

# 格式器
fmt1 = logging.Formatter(fmt="%(asctime)s - %(levelname)-9s - %(filename)-8s : %(lineno)s line - %(message)s")

fmt2 = logging.Formatter(fmt="%(asctime)s - %(name)s - %(levelname)-9s - %(filename)-8s : %(lineno)s line - %(message)s"
                        ,datefmt="%Y/%m/%d %H:%M:%S")

#给处理器设置格式
sh1.setFormatter(fmt1)
fh1.setFormatter(fmt2)

#记录器设置处理器
logger1.addHandler(sh1)
logger1.addHandler(fh1)

#打印日志代码
logger1.debug("This is  DEBUG of logger1 !!")
logger1.info("This is  INFO of logger1 !!")
logger1.warning("This is  WARNING of logger1 !!")
logger1.error("This is  ERROR of logger1 !!")
logger1.critical("This is  CRITICAL of logger1 !!")

```

由于处理器设置了两个，所以控制台和日志文件都会有输出。

