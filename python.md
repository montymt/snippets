<!--ts-->
* [python](#python)
   * [安装python3](#安装python3)
   * [初始化项目](#初始化项目)
   * [logging](#logging)
   * [dotenv](#dotenv)
   * [MySQL](#mysql)
   * [websocket](#websocket)
   * [redis](#redis)
   * [requests](#requests)
<!--te-->
# python

## 安装python3

```shell
yum -y install centos-release-scl
yum -y install rh-python38
source /opt/rh/rh-python38/enable
mkdir .pip
cat > .pip/pip.conf <<EOF
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
EOF
```

## 初始化项目

```shell
pip3 install virtualenv
#pip install virtualenvwrapper
cd /usr/local
mkdir my-project && cd my-project
virtualenv -p /opt/rh/rh-python38/root/usr/bin/python3 venv
source venv/bin/active
#deactivate
pip install -r requirements.txt
#pip freeze > requirements.txt
```
## logging

```python
import logging
from logging import handlers

def init_logger(log_file):
    logger_instance = logging.getLogger()
    formatter = logging.Formatter('%(asctime)s - %(pathname)s[line:%(lineno)d] - %(levelname)s: %(message)s')
    log_handler = handlers.RotatingFileHandler(log_file, maxBytes=1024 * 1024 * 50, backupCount=9,
                                               encoding='utf8')
    log_handler.setFormatter(formatter)
    logger_instance.addHandler(log_handler)
    return logger_instance

logger = init_logger('test.log')

# default level is info
logger.warning("warning message")
```

## dotenv

https://github.com/theskumar/python-dotenv

`pip install python-dotenv`

```python
from dotenv import dotenv_values

# load .env
config = dotenv_values()

print(config['DB_HOST'])
```

## MySQL

python 3.7+

MySQL 5.7+

https://github.com/PyMySQL/PyMySQL

`pip3 install PyMySQL`

```python
import pymysql

connection = pymysql.connect(host='127.0.0.1', port=3306, db='blog', user='root', password='root',
                             cursorclass=pymysql.cursors.DictCursor)
with connection.cursor() as cursor:
    sql = "INSERT INTO `article` (time, title, author, content) " \
                      "VALUES (%s, %s, %s, %s)"
    cursor.execute(sql, (int(time.time()), row['title'], row['author'], row['content']))
connection.commit()
```

https://github.com/aio-libs/aiomysql

`pip3 install aiomysql`

## websocket

https://github.com/aaugustin/websockets

`pip3 install websockets`

```python
#!/usr/bin/env python

import asyncio
import signal

import websockets

async def echo(websocket):
    async for message in websocket:
        await websocket.send(message)

async def server():
    # Set the stop condition when receiving SIGTERM.
    loop = asyncio.get_running_loop()
    stop = loop.create_future()
    loop.add_signal_handler(signal.SIGTERM, stop.set_result, None)

    async with websockets.serve(echo, "localhost", 8765):
        await stop

asyncio.run(server())
```

## redis

```python
def batch_lpop(client, key, count):
    """
    一次性返回多个值。redis 6.2才有此功能
    :param client:
    :param key:
    :param count:
    :return:
    """
    p = client.pipeline()
    p.lrange(key, 0, count - 1)  # 从0开始
    p.ltrim(key, count, -1)  # 保留count开始到结束的值
    data = p.execute()
    return data
```

## requests

