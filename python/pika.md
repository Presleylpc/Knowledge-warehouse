http://www.rabbitmq.com/getstarted.html
https://blog.csdn.net/zyz511919766/article/details/41946521
### Docker启动
```
docker run -d --restart=always --name rabbitmq --publish 5671:5671 --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 --publish 15671:15671 --publish 15672:15672 rabbitmq:management 
```
### 简单队列

```
graph LR
send-->queue
queue-->receive
```
- send.py
```
import pika
import json

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()


channel.queue_declare(queue='hello')
channel.basic_publish(exchange='',
routing_key='hello',
body=json.dumps({'a': 123, 'b': 456}))
print(" [x] Sent 'Hello World!'")

```

- receive.py
```
import pika
import json

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()


channel.queue_declare(queue='hello')


def callback(ch, method, properties, body):
print(type(json.loads(body.decode('utf-8'))))


channel.basic_consume(callback,
queue='hello',
no_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

### 多监听者

```
graph LR
send-->queue
queue-->receive1
queue-->receive2
```

- new_task.py
```
import sys
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

message = ' '.join(sys.argv[1:]) or "Hello World!"
channel.basic_publish(exchange='',
routing_key='hello',
body=message)
print(" [x] Sent %r" % message)

```
- worker.py
```
import time
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()


channel.queue_declare(queue='hello')


def callback(ch, method, properties, body):
print(" [x] Received %r" % body)
time.sleep(body.count(b'.'))
print(" [x] Done")


channel.basic_consume(callback,
queue='hello',
no_ack=True) #去掉no_ack 则会进行消息确认

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
### 队列持久化
> 需要运用于生产者与消费者

```
channel.queue_declare（queue = 'task_queue'，durable = True）
```
> 发布消息

```
channel.basic_publish（exchange = ''，
routing_key = “task_queue”，
body = message，
properties = pika.BasicProperties（
delivery_mode = 2，＃make message persistent 
））
```
### 发布订阅模式
```
graph LR
P-->X
X-->Queue1
X-->Queue2
Queue1-->C1
Queue2-->C2
```
- receive_logs.py
```
#!/usr/bin/env python
#encoding:utf8
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
host='localhost'))
channel = connection.channel()

#作为好的习惯，在producer和consumer中分别声明一次以保证所要使用的exchange存在
channel.exchange_declare(exchange='logs', exchange_type='fanout')

#在不同的producer和consumer间共享queue时指明queue的name是重要的
#但某些时候，比如日志系统，需要接收所有的log message而非一个子集
#而且仅对当前的message 流感兴趣，对于过时的message不感兴趣，那么
#可以申请一个临时队列这样，每次连接到RabbitMQ时会以一个随机的名字生成
#一个新的空的queue，将exclusive置为True，这样在consumer从RabbitMQ断开后会删除该queue
result = channel.queue_declare(exclusive=True)

#用于获取临时queue的name
queue_name = result.method.queue

#exchange与queue之间的关系成为binding
#binding告诉exchange将message发送该哪些queue
channel.queue_bind(exchange='logs',
queue=queue_name)

print ' [*] Waiting for logs. To exit press CTRL+C'

def callback(ch, method, properties, body):
print " [x] %r" % (body,)

#从指定地queue中consume message且不确认
channel.basic_consume(callback,
queue=queue_name,
no_ack=True)

channel.start_consuming()
```
- emit_log.py
```
#!/usr/bin/env python
#encoding:utf8

import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(
host='localhost'))
channel = connection.channel()

#producer只能通过exchange将message发给queue
#exchange的类型决定将message路由至哪些queue
#可用的exchange类型：direct\topic\headers\fanout
#此处定义一个名称为'logs'的'fanout'类型的exchange，'fanout'类型的exchange简单的将message广播到它所知道的所有queue
channel.exchange_declare(exchange='logs',
exchange_type='fanout')

message = ' '.join(sys.argv[1:]) or "info: Hello World!"

#将message publish到名为log的exchange中
#因为是fanout类型的exchange，这里无需指定routing_key
channel.basic_publish(exchange='logs',
routing_key='',
body=message)

print " [x] Sent %r" % (message,)

connection.close()
```
#### 四种exchange类型
https://www.cnblogs.com/julyluo/p/6265775.html

#### RPC类型
- rpc_client
```
#!/usr/bin/env python
import pika
import uuid

class FibonacciRpcClient(object):
def __init__(self):
self.connection = pika.BlockingConnection(pika.ConnectionParameters(
host='localhost'))

self.channel = self.connection.channel()

result = self.channel.queue_declare(exclusive=True)
self.callback_queue = result.method.queue

self.channel.basic_consume(self.on_response, no_ack=True,
queue=self.callback_queue)

def on_response(self, ch, method, props, body):
if self.corr_id == props.correlation_id:
self.response = body

def call(self, n):
self.response = None
self.corr_id = str(uuid.uuid4())
self.channel.basic_publish(exchange='',
routing_key='rpc_queue',
properties=pika.BasicProperties(
reply_to = self.callback_queue,
correlation_id = self.corr_id,
),
body=str(n))
while self.response is None:
self.connection.process_data_events()
return int(self.response)

fibonacci_rpc = FibonacciRpcClient()

print(" [x] Requesting fib(30)")
response = fibonacci_rpc.call(30)
print(" [.] Got %r" % response)

```
- rpc_server

```
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
host='localhost'))

channel = connection.channel()

channel.queue_declare(queue='rpc_queue')

def fib(n):
if n == 0:
return 0
elif n == 1:
return 1
else:
return fib(n-1) + fib(n-2)

def on_request(ch, method, props, body):
n = int(body)

print(" [.] fib(%s)" % n)
response = fib(n)

ch.basic_publish(exchange='',
routing_key=props.reply_to,
properties=pika.BasicProperties(correlation_id = \
props.correlation_id),
body=str(response))
ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(on_request, queue='rpc_queue')

print(" [x] Awaiting RPC requests")
channel.start_consuming()

```