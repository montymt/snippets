# python

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

