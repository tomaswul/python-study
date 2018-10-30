---
title: Django日志处理
date: 2018-10-30 16:31:34
tags:
---

#### 1.日志的配置

日志处理是每个网站以及程序必须要做的功能，完整的日志可以追踪网站或者程序可能会出现的各种问题。

logging的使用需要对项目的setting.py进行配置。

```python
# 配置日志
LOGGING = {
    # 必须是1
    'version': 1,
    # True表示禁用日志
    'disable_existing_loggers': False,
    # 指定写入到日志文件中的日志格式
    'formatters': {
      'default': {
          'format': '%(name)s %(asctime)s %(message)s'
      },

    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'filename': '%s/log.txt' % os.path.join(BASE_DIR, 'log'),
            'formatter': 'default',
            # 当日志文件大于5M,则做自动备份
            'class': 'logging.handlers.RotatingFileHandler',
            'maxBytes': 5 * 1024 * 1024,
        }
    },
    'loggers': {
        '': {
            'handlers': ['console'],
            'level': 'INFO',
        }
    }
```

进行配置之后需要在中间件里面对要进行日志记录的信息进行捕获并把捕获到的信息存储在指定的log文件里面

```python
import time

import logging
class LoggingMiddleware(MiddlewareMixin):
	def process_request(self,request):
		# 记录当前请求服务器的时候，请求参数，请求内容
		request.init_time = time.time()
		request.init_body = request.body
		return None

	def process_response(self,request,response):
		# 记录返回响应的时间和访问服务器的时间差(可以进行代码优化)，记录返回状态码
		# 响应时间差
		times = time.time() - request.init_time

		# 响应状态码
		code = response.status_code
		# 响应内容
		try:
			res_body = response.content
		except Exception as e:
			res_body = None
		# 请求内容
		req_body = request.init_body
		msg = '%s %s %s %s' % (times, code, res_body, req_body)
		# 写入日志
		try:
			logging.info(msg)
		except Exception as e:
			logging.critical('log error,Exception: %s'% e)
		return response
		
```

日志的详细信息

#### 2. 日志logging模块

logging模块可以收集记录错误，警告等调试信息，在程序中可以捕获这些信息，并且甚至可以将错误的重要信息等都可以通过邮件发送给开发者

##### 

##### 1.1 logging的组成

```
Loggers

Handlers

Filters

Formatters

```

##### 

##### 1.1 Loggers

Logger 为日志系统的入口。每个logger 是一个具名的容器，可以向它写入需要处理的消息。

每个logger 都有一个日志级别。日志级别表示该logger 将要处理的消息的严重性。

Python 定义以下几种日志级别：

```
DEBUG：用于调试目的的底层系统信息

INFO：普通的系统信息

WARNING：表示出现一个较小的问题。

ERROR：表示出现一个较大的问题。

CRITICAL：表示出现一个致命的问题。

```

日志级别等级CRITICAL > ERROR > WARNING > INFO > DEBUG > NOTSET

##### 

##### 1.2 Handlers

Handler 决定如何处理logger 中的每条消息。它表示一个特定的日志行为。

与logger 一样，handler 也有一个日志级别。如果消息的日志级别小于handler 的级别，handler 将忽略该消息。

Logger 可以有多个handler，而每个handler 可以有不同的日志级别。

##### 

##### 1.3 Filters

Filter 用于对从logger 传递给handler 的日志记录进行额外的控制。

##### 

##### 1.4 Formatters

日志记录需要转换成文本。

Formatter 表示文本的格式。Fomatter 通常由包含日志记录属性的Python 格式字符串组成；

你也可以编写自定义的fomatter 来实现自己的格式。

如下展示了formatters格式:[![图](https://github.com/coco369/knowledge/raw/master/django/images/django_logging_model.png)](https://github.com/coco369/knowledge/blob/master/django/images/django_logging_model.png)

##### 1.5 日志常见的写入方法

```python
 import logging

 # 获取logger，logger用于接收日志信息，并且丢给handlers进行处理
 logger = logging.getLogger(__name__)

 # logger接收日志信息的几个方法，如下:
 logger.debug()
 logger.info()
 logger.warning()
 logger.error()
 logger.critical()
```

