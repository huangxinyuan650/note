## Celery
### 说明 
- Celery是一个强大的分布式任务队列的异步处理框架,它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab）

### 组件说明
[ ] 任务模块 Task: 包含异步任务(业务调用)和定时任务（beat中定时任务）
[ ] 消息中间件 Broker: 即为任务调度队列，接收任务生产者发来的消息（即任务），将任务存入队列
[ ] 任务执行单元 Worker: 执行任务的处理单元，实时监控消息队列，获取队列中调度的任务，并执行它
[ ] 任务结果存储 Backend: 用于存储任务的执行结果，以供查询

### 异步任务
- 1. 创建一个celery实例
创建一个celery.Celery实例，并指定broker和backend，添加celery任务（用@app.task装饰函数即可）

- 2. 启动celery worker
Celery worker -A Celery实例所在模块名 --loglevel=info
参数： -A 指定了Celery 实例的位置，celery 会自动在该文件中寻找Celery实例 
参数：--loglevel 指定了日志级别，默认为warning,也可以 -l info来表示

- 3. 调用任务
可以通过调用celery的delay或者apply_async方法来调用任务
delay可以理解为apply_async的快捷方式，appy_async支持更多的参数，一般形式如下：
apply_async(args=(), kwargs={}, route_name=None, **options)
常用参数：
countdown:指定多少秒后执行任务
eta: 指定任务的具体被调度时间，参数类型为datetime
expires: 任务过期时间，参数类型可以是int，也可以是datetime

### 配置、定时任务
Celery 除了可以执行异步任务，也支持执行周期性任务（定时任务），Celery Beat 进程通过读取配置的内容，周期性地将定时任务发往任务队列
[ ] BROKER_URL: broker
[ ] CELERY_RESULT_BACKEND: backend
[ ] CELERY_TIMEZONE: 制定时区
[ ] CELERY_IMPORTS = (,): 指定导入的任务的模块
[ ] CELERYBEAT_SCHEDULE: 定时任务， 可以指定间隔时间发送，也可以指定固定时间发送
通过开启celery beat 进程，定时任务将在对应时刻发送到broker:
Celery beat -A celery_app

### [更详细信息直接参考官方文档](http://docs.jinkan.org/docs/celery/getting-started/introduction.html)


