## Celery
### 一、概念
Celery 是一个强大的 分布式任务队列 的 异步处理框架，它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab）

可以看到，Celery 主要包含以下几个模块：
1. 任务模块 Task:包含异步任务和定时任务。其中，异步任务通常在业务逻辑中被触发并发往任务队列，而定时任务由 Celery Beat 进程周期性地将任务发往任务队列。
2. 消息中间件 Broker: Broker，即为任务调度队列，接收任务生产者发来的消息（即任务），将任务存入队列。Celery 本身不提供队列服务，官方推荐使用 RabbitMQ 和 Redis 等。
3. 任务执行单元 Worker: Worker 是执行任务的处理单元，它实时监控消息队列，获取队列中调度的任务，并执行它。
4. 任务结果存储 Backend: Backend 用于存储任务的执行结果，以供查询。同消息中间件一样，存储也可使用 RabbitMQ, redis 和 MongoDB 等。
### 二、创建异步任务
1. 创建一个celery实例
创建一个celery实例，并指定broker和backend，添加celery任务（用@app.task装饰函数即可）
2. 启动celery worker
Celery worker -A tasks --loglevel=info
参数： -A 指定了celery 实例的位置，celery 会自动在该文件中寻找celery实例 
参数--loglevel 指定了日志级别，默认为warning,也可以 -l info来表示。
3. 调用任务
可以通过调用delay()方法和apply_async()方法来调用任务。
Delay可以理解为apply_async()的快捷方式，appy_async支持更多的参数，一般形式如下：
apply_async(args=(), kwargs={}, route_name=None, **options)
常用参数：
Coundown:指定多少秒后执行任务
Ela: 指定任务的具体被调度时间，参数类型为datetime
Expries: 任务过期时间，参数类型可以是int，也可以是datetime
### 三、配置文件和定时任务
Celery 除了可以执行异步任务，也支持执行周期性任务（Periodic Tasks），或者说定时任务。Celery Beat 进程通过读取配置文件的内容，周期性地将定时任务发往任务队列。
1.  BROKER_URL: 指定broker
2.  CELERY_RESULT_BACKEND: 指定 backend
3.  CELERY_TIMEZONE: 制定时区
4.  CELERY_IMPORTS = (,): 指定导入的任务模块
5.  CELERYBEAT_SCHEDULE: 定时任务， 可以指定间隔时间发送，也可以指定固定时间发送。
通过开启celery beat 进程，定时任务将在对应时刻发送到broker:
Celery beat -A celery_app