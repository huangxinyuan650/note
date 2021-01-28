## Celery
### 说明 
- Celery是一个强大的分布式任务队列的异步处理框架,它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab）

### 组件说明
* [ ] 任务模块 Task: 包含异步任务(业务调用)和定时任务（beat中定时任务）
* [ ] 消息中间件 Broker: 即为任务调度队列，接收任务生产者发来的消息（即任务），将任务存入队列
* [ ] 任务执行单元 Worker: 执行任务的处理单元，实时监控消息队列，获取队列中调度的任务，并执行它
* [ ] 任务结果存储 Backend: 用于存储任务的执行结果，以供查询

### 异步任务
- 1. 创建一个celery实例
创建一个celery.Celery实例，并指定broker和backend，添加celery任务（用@app.task装饰函数即可）

- 2. 启动celery worker
```
Celery worker -A Celery实例所在模块名 --loglevel=info
参数： -A 指定了Celery 实例的位置，celery 会自动在该文件中寻找Celery实例 
参数：--loglevel 指定了日志级别，默认为warning,也可以 -l info来表示
```

- 3. 调用任务
可以通过调用celery的delay或者apply_async方法来调用任务
delay可以理解为apply_async的快捷方式，appy_async支持更多的参数，一般形式如下：

```
apply_async(args=(), kwargs={}, route_name=None, **options)
常用参数：
countdown:指定多少秒后执行任务
eta: 指定任务的具体被调度时间，参数类型为datetime
expires: 任务过期时间，参数类型可以是int，也可以是datetime
```


### 配置、定时任务
Celery 除了可以执行异步任务，也支持执行周期性任务（定时任务），Celery Beat 进程通过读取配置的内容，周期性地将定时任务发往任务队列
* [ ] BROKER_URL: broker
* [ ] CELERY_RESULT_BACKEND: backend
* [ ] CELERY_TIMEZONE: 制定时区
* [ ] CELERY_IMPORTS = (,): 指定导入的任务的模块
* [ ] CELERYBEAT_SCHEDULE: 定时任务， 可以指定间隔时间发送，也可以指定固定时间发送

```
通过开启celery beat 进程，定时任务将在对应时刻发送到broker:
Celery beat -A celery_app
```

### [更详细信息直接参考官方文档](http://docs.jinkan.org/docs/celery/getting-started/introduction.html)

## 消息队列
- 可靠性保证
- 持久化
- 广播模式
- 延时队列：使用zset实现，到期时间戳为score排序，轮询zset确认是否有到期消息需要处理
### Redis
#### list
使用LPUSH向队列发消息，BRPOPLPUSH（可以支持等待超时时间和存储取出后的消息来做持久化）来消费并备份消息
#### pub/sub
使用publish向队列发送消息，使用psubscribe订阅特定模式消息（消息无法持久化、无法收到历史消息）
#### [stream](./Redis.md)
- 生产者：使用xadd来向队列发送消息
- 消费者：使用xread（xreadgroup）从消息队列中读取消息（可block、可读历史消息，可读最新消息、使用group还可以多个group订阅且消费会需要确认）

### RabbitMQ
#### 组成
- Broker：标示消息队列服务器实体
- Virtual Host（虚拟主机）：用于标示一批交换机、消息队列和相关对象
- Exchange（交换机）：接收消息生产者发送的消息并将消息根据路由规则转给服务器中的消息队列
- Queue（消息队列）：存储消息并将消息发送给消费者
- Banding（绑定）：用于消息队列和交换机之间的关联
- Channel（信道）：多路复用连接中一条双向连接的数据流通道（建立在TCP连接之上的虚链接开销较小，消息发布、订阅消息、接收消息都通过信道完成）。
- Connection：网络连接
- Publisher（消息的生产者）：向交换机发送消息的应用程序
- Consumer（消息的消费者）：从消息队列中获取消息的应用程序
- Message（消息）：消息是不具名的，有消息头和消息体组成
- Bing key：消息队列接受的key
- Routing key：消息携带的需要路由的key

#### Exchange类型
- fanout：将所有发给Exchange的消息都路由到所有绑定的Queue
- direct：将消息的Routing Key与Queue的Bing Key完全匹配的消息路由到对应的Queue中
- topic：将消息的Routing Key与Queue的Bing Key进行匹配（模糊匹配，Bing Key和Routing Key都是点号分隔的字符串，*匹配一个单词，#匹配零个或多个单词），匹配上的消息路由到对应的Queue中
- header：根据消息中的headers属性与创建Exchange是设置的属性进行匹配，将匹配上的消息路由到对应的Queue中



