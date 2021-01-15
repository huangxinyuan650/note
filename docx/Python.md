### Python基础
#### Python2和Python3的区别
- print -> print()
- 除法运算
- 异常
- 八进制字面量表示
等

#### Python数据类型
- 数字（Int、Float、Long、Complex）
- 字符串（str）
- 元组（tuple）
- 列表（list）
- 字典（dict）
- 集合（set）：集合中所有元素必须可散列，底层数据结构为哈希表（字典中只存放键没有值）

#### 可变类型与不可变类型
- 不可变类型：数字、字符串、元组（元素为不可变类型）
- 可变类型：所有映射

#### 哈希表、可散列与不可散列
- 哈希表：哈希函数+哈希冲突（1、再哈希 2、拉链法）
- 可散列数据类型：在对象生命周期中散列值不变（需实现__hash__、__eq__方法）。原子不可变类型、frozenset是可散列
- 不可散列数据类型：

#### 深拷贝、浅拷贝
- 浅拷贝：只复制对象本身
- 深拷贝：复制对象及所有对象关联的对象。（deepcopy中的memo来保存已经拷贝过的对象从而避免自引用递归）

#### 内存管理
- 引用计数：每个对象都有一个引用计数ob_refcnt，当引用数为0时内存就会释放掉。
- 标记清除：在标记阶段遍历所有对象若对象可达则标记可达，在清除阶段再次遍历所有对象，对未标记为可达的对象进行回收操作。
- 分代回收：对象存在的时间越长是垃圾的可能性就越小，应该尽量不回收。（层级分为0、1、2，每次垃圾回收扫描后存活的对象将被移到下一代中）

#### 实例方法、类方法、静态方法
- 实例方法：绑定到类的实例上，隐式传入self。（a.fun(*xargs,**kwargs)或者A.fun(a,*xargs,**kwargs)）
- 类方法（classmethod）：绑定到类对象上，隐式传入cls。(A.fun(*xargs,**kwargs)或者a.fun(*xargs,**kwargs))
- 静态方法（staticmethod）：没有参数绑定，和普通函数一样。(A.fun(*xargs,**kwargs)或者a.fun(*xargs,**kwargs))
- staticmethod为啥不直接用外部函数代替：函数属于类但不用访问类，通过子类的覆盖继承可更好的组织代码

#### 迭代器、生成器、协程
- 可迭代对象：遵循了可迭代协议（实现了iter()方法）的对象
- 迭代器：实现了迭代器协议（实现了iter和next方法）
- 生成器：数据的生成者，一种特殊的迭代器。创建方式有生成器表达式和生成器函数（yield）
```
    # fb generator
    def fb_ge(max):
        n, a, b = 0, 0, 1
        while n < max:
            yield b
            a, b, n = b, a+b, n+1
```
- 协程：协作式线程，可被挂起
```
def customer():
    _n = 1
    yield
    while _n < 100:
        _p = yield _n
        print(f'收到{_p}个产品\n开始消费...\n')
        time.sleep(0.5)
        print('消费完成\n')
        _n += 1


def productor():
    _c = customer()
    next(_c)
    for _ in range(100):
        print(f'产品传入{_c.send(_)}给消费者\n重新开始生产新产品')
        time.sleep(1)
        print('新批次产品生产完成')


productor()
```

#### 多线程、多进程

##### 多进程（操作系统分配资源的最小单元）

###### 进程状态
- 就绪态：具备执行条件，等待CPU（获取CPU后变成运行态）
- 运行态：正在使用CPU（使用完CPU还没计算完变成就绪态、等待其他资源时变成等待态）
- 等待态：暂不满足执行条件，待条件满足，才变成就绪态（等待条件满足后变成就绪态）

###### 孤儿进程、僵尸进程
- 孤儿进程：父进程早于子进程退出（孤儿进程将有init进程收养）。(无危害不用管)
- 僵尸进程：子进程早于父进程退出，但父进程未处理子进程退出状态，子进程变成僵尸进程（保留少量PCB信息在内存中）。（top的zombie代表僵尸进程数，ps -aux｜grep Z找到STAT为Z的进程，ps -ef｜grep 进程号找到父进程PPID，kill -9 PPID）

###### 进程间通信
- 管道：在内存中生成管道对象，多个进程操作同一个管道即可实现通信。（Pipe类，返回一个含有两个connection的元组，进程通过这两个connection进行通信）
- 消息队列：在内存中建立队列数据模型，多个进程操作队列来存取内容进行通信。（Queue类，put、get、full、empty、size、close）
- 共享内存：在内存中开辟一块对多个进程可见的空间，多个进程可读可写，但没次写入的都会覆盖上次内容。（Value、Array类，）
- 信号量：一个进程向另一个进程发送一个信号来传递某种信息，接收者根据信号做出相应操作。（Semaphore类，控制并发数，acquire+1，release-1）

```
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2021/1/10 12:11
# File: multi_process_demo.py
from multiprocessing import Process, Pipe, Value, Queue, Array, Semaphore, Manager
from os import getpid
import requests
import time


def down_fun(file_name):
    _s_start = time.time()
    _re = requests.get(f'http://127.0.0.1:2650/{file_name}')
    with open(f'/Users/huangxinyuan/develop_my/leetcode/source/{file_name}', 'wb') as _f:
        _f.write(_re.content)
        print(f'{file_name} has been saved!!!\n')
    print(f'{file_name}下载耗时：{time.time() - _s_start}\n下载进程号为：{getpid()}\n')


def down_one(file_name, msg_tool):
    print('P1 send message:Ping\n')
    # # 管道用法
    # msg_tool.send('Ping')

    # # 消息队列用法
    # msg_tool.put('Ping', block=True)

    # 共享内存用法 Value、Array
    # msg_tool.value = 'P'
    # msg_tool[0] = 'P'
    # msg_tool[1] = 'i'
    # msg_tool[2] = 'n'
    # msg_tool[3] = 'g'

    # # 信号量用法Semaphore
    # msg_tool.acquire()
    # print('P1 acquire lock!!!')

    # Manager 用法
    msg_tool['Ping'] = 'Pong'
    down_fun(file_name)


def down_two(file_name, msg_tool):
    # # 管道用法
    # print(f'P2 receive message {msg_tool.recv()}\n')

    # # 消息队列用法
    # print(f'P2 get message {msg_tool.get(block=True)}')

    # 共享内存用法 Value、Array
    # print(f'P2 get value {msg_tool.value}')
    # print(f'P2 get value {msg_tool[:]}')

    # # 信号量用法
    # msg_tool.release()
    # print('P2 release lock!!!')

    # Manager用法
    print(f'P2 read manager info {msg_tool}')
    down_fun(file_name)


def main():
    _start_time = time.time()
    # # 管道用法
    # _msg = Pipe()
    # _p1 = Process(target=down_one, args=('sea1.png', _msg[0],))
    # _p2 = Process(target=down_two, args=('sea2.png', _msg[1],))

    # # 消息队列用法
    # _msg = Queue(maxsize=5)

    # # 共享内存 Value（单个数字或者字符）、Array（数组要求每个元素类型相同）
    # _msg = Value(typecode_or_type='u')
    # _msg = Array(typecode_or_type='u', size_or_initializer=10)

    # # 信号量用法 Semaphore，参数控制并发数
    # _msg = Semaphore(1)

    # Manager 可以使用List、Dict、Value、Array、Semaphore等
    _msg = Manager().dict()

    _p1 = Process(target=down_one, args=('sea1.png', _msg,))
    _p2 = Process(target=down_two, args=('sea1.png', _msg,))

    _p1.start()
    _p2.start()
    _p1.join()
    _p2.join()
    print(f'总耗时：{time.time() - _start_time}\n')


if __name__ == '__main__':
    main()

```

##### 多线程（CPU调度的最小单元）

###### 线程间通信（直接global共享变量，加锁方式、或者使用queue）
- threading.Lock()：基本锁对象，每次只能锁一次，其余锁请求需等待锁释放后才能获取
- threading.Rlock()：可重入锁，可多次锁定多次释放，acquire和release成对出现即可
- threading.Condition()：acquire、release和Lock一样，但提供wait()（挂起）、notify(n)（对多唤醒n和线程）、notifyAll(唤醒所有wait线程)
- threading.Event()：线程中立个flag（默认为False），当一个或者多个线程遇到event.wait()阻塞时，直到flag变成True时(event.set())继续。提供了wait()(阻塞线程将flag设置为False)、set()(flag设置为True)、clear()(将flag设置为False)、isSet()(仅当flag为True时返回)

```
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2021/1/10 17:26
# File: future_demo.py
import requests
import time
from concurrent import futures

FILE_BASE = '~/develop_my/leetcode/source'


def download_fun(source_name):
    _re = requests.get(f'http://127.0.0.1:2650/{source_name}')
    save_source(content=_re.content, file_name=source_name)
    return source_name


def save_source(content, file_name):
    with open(f'{FILE_BASE}/{file_name}', 'wb') as f:
        f.write(content)
        print(f'{file_name} save successed!!!')


def down_main():
    _time1 = time.time()
    _list = [f'sea{_}.png' for _ in range(1, 101)]

    # # 多线程方案 concurrent.futures.ThreadPoolExecutor
    with futures.ThreadPoolExecutor(max_workers=30) as executor:
        # 方法一：直接使用executor执行所有任务
        # _re_list = executor.map(download_fun, _list)

        # 方法二：使用submit来一个一个任务添加，并通过futores.as_completed方法获取到已经执行完成的futures，使用futures.result获取结果
        _todo = []
        for _ in _list:
            _f = executor.submit(download_fun, _)
            _todo.append(_f)
            print(f'Schedule:{_}')
        for _ in futures.as_completed(_todo):
            print(f'Completed:{_.result()}')

    # 多进程方案 concurrent.futures.ProcessPoolExecutor
    # with futures.ProcessPoolExecutor() as executor:
    #     executor.map(download_fun, _list)

    # print(_re_list)
    # for _ in range(1, 101):
    #     _file_name = f'sea{_}.png'
    #     download_fun(_file_name)
    #     # save_source(content=_content, file_name=_file_name)
    print(f'总耗时：{(time.time() - _time1)}')


if __name__ == '__main__':
    down_main()


```

```
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/12/25 08:37
# File: multi_thread_demo.py

import requests
import uuid
import threading
import time
from queue import Queue

count = 0


# 全局变量加锁，传入锁
# def down_one(file_name: str, lock: threading.RLock):

def down_one(file_name: str, stat_queue):
    """
    下载单个文件
    """
    global count
    _start_time = time.time()
    _save_dir = '/Users/huangxinyuan/develop_my/leetcode/source'
    _response = requests.request(method='GET', url=f'http://127.0.0.1:2650/{file_name}')

    if _response.status_code == 200:
        with open(file=f'{_save_dir}/{uuid.uuid4()}{file_name}', mode='wb+') as f:
            f.write(_response.content)
        print('Success')
    else:
        print('Failed')
    # lock.acquire()
    # count += 1
    # lock.release()
    if stat_queue.empty():
        stat_queue.put(1)
        print('Queue +1')
    else:
        stat_queue.put(stat_queue.get() + 1)
        print('Queue +-1')
    print(f'下载{file_name}共耗时{time.time() - _start_time}\n已下载{count}个文件\n')


if __name__ == '__main__':
    _begin_time = time.time()
    # # 全局变量加锁
    # _lock = threading.RLock()
    # _t_list = [threading.Thread(target=down_one, args=(f'sea{_}.png', _lock,)) for _ in range(30)]

    # Queue
    _queue = Queue(maxsize=5)
    _t_list = [threading.Thread(target=down_one, args=(f'sea{_}.png', _queue,)) for _ in range(30)]

    [_.start() for _ in _t_list]
    [_.join() for _ in _t_list]
    print(f'总耗时：{time.time() - _begin_time}')

```
#### 装饰器
- 闭包：指延伸了作用域的函数，其中包含函数定义体中引用、但是不在定义体中定义的非全局变量。（函数访问定义体之外定义的非全局变量）
- 解释：在不改变函数调用方式的情况下 对函数进行额外功能的封装，装饰一个函数 转给他一个其他的功能

#### 单例：确保一个类只有一个实例（Python天然单例，因为模块在第一次导入时就生成pyc文件，再导入就直接加载pyc文件了）
```
# 装饰器
import threading
def single_w_demo(cls):
    _instance = {}

    def r_w(*xargs, **kwargs):
        if not _instance.get(cls):
            threading.Lock()
            if not _instance.get(cls):
                _instance[cls] = cls(*xargs, **kwargs)
            threading.RLock()
        return _instance[cls]

    return r_w


@single_w_demo
class B(object):
    pass

# 类
class C(object):
    _instance_lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if not hasattr(C, '_instance'):
            with C._instance_lock:
                if not hasattr(C, '_instance'):
                    setattr(C, '_instance', object.__new__(cls))
        return getattr(C, '_instance')
```

#### concurrent、asyncio
##### Futures
- done()，都不阻塞么，返回任务是否执行完成的布尔值
- add_done_callback()，传入可调用对象，当任务执行完成时调用
- result()，阻塞等待任务执行完成，concurrent中可传入timeout参数，asyncio.Futures不阻塞当未完成时抛出asyncio.InvalidStateError异常（建议用yield from futures）

##### asyncio
###### 可等待对象（协程、任务、Future）
- 协程函数：定义形式为 async def 的函数
- 协程对象：调用协程函数所返回的对象
###### 运行协程机制
- asyncio.run(coroutine)：接收一个协程并直接运行，管理asyncio事件循环、终结异步生成器、并关闭线程池（当有其他asyncio事件循环在同一线程中使用时此函数不可用）
- await：等待一个协程。创建一个协程对象后，await coroutine。
- asyncio.create_task(coroutine)：接受一个协程创建一个task准备执行并返回Task对象。
- asyncio.gather(*aws,loop=None,return_exceptions=False)：aws为可等待对象序列，并发运行多个协程，返回结果。（单个协程的执行异常或者取消不影响其他协程，若gather被取消则为执行完成的协程将被取消）
- asyncio.shield(aw,*,loop=None)：保护一个可等待对象防止被取消。
- asyncio.wait_for(aw,timeout,*,loop=None)：等待可等待对象aw执行完成，超过timeout后超时。（超时取消任务）
- asyncio.wait(aws,*,loop=None,time_out=None,return_when=ALL_COMPLETED)：并发运行可迭代对象aws中的可等待对象并进入阻塞状态直到满足return_when条件，返回结果为(done,pending)的task/future集合。（return_when参数为FIRST_COMPLETED、FIRST_EXCEPTION、ALL_COMPLETED，超时不取消任务）
- asyncio.as_completed(aws,*,loop=None,time_out=None)：并发运行可迭代对象aws中的可等待对象，返回一个协程的迭代器。
- asyncio.to_thread(func,/,*xargs,**kwargs)：在不同的线程中运行函数func，返回一个可获取func执行结果的协程。

###### 其他方法
- asyncio.current_task(loop=None)：获取当前运行的task实例。
- asyncio.all_tasks(loop=None)：返回事件循环所运行的所有未完成的任务实例对象集合。

###### Task使用说明
- 通过asyncio.create_task、loop.create_task、ensure_future等方法可创建task对象
- cancel(msg=None):是task对象抛出一个CancelledError异常给打包的协程。
- cancelled()：检测task对象是否已经被取消。
- done():判断一个task是否执行完成。
- result()：返回task返回结果。
- exception()：返回task执行异常。
- add_done_callback(callback,*,context=None)：添加一个任务完成式的回调。
- remove_done_callback(callback)：移除为任务添加的回调。
- get_coro()：返回task包装的协程对象。
- get_name()：返回task对象name。
- set_name(value)：设置task对象名称。

####### 基于生成器的协程
使用@asyncio.coroutine装饰的使用yield from语法的生成器，可等待future和其他协程
- asyncio.iscoroutine(obj)：判断obj是否为协程对象。
- asyncio.iscoroutinefunction(func)：判断func是否为协程函数。


#### 分布式锁
多进程、多线程均可通过锁来处理临界资源的竞争问题，但当使用集群时，多机针对临街资源的竞争需要使用分布式锁来完成
##### 实现方式
- 数据库：创建一张临界资源信息表，针对临界资源名称设置唯一性约束，使用资源前插入数据，若插入成功则可继续操作资源，操作完成后删除记录释放资源
- Redis：使用setnx命令（原子操作，成功返回1，失败返回0）申请锁，申请成功后操作资源，操作完成后删除key。（可使用expire命令设置获取锁使用时间，超时无效）
- zookeeper：

#### WSGI
Web Server Gateway Interface，指定了Web服务器到PythonWeb应用之间的标准接口，以提高Web应用在一系列Web服务器间的移植性。（web请求处理过程：浏览器发出请求-->Web服务器-->Web应用程序-->Web服务器-->浏览器）
##### WSGI规定格式
WSGI规定Web程序必须有一个可调用对象且该可调用对象接收两个参数，返回一个可迭代对象。
- environ：dict，包含请求的所有信息。
- start_response：在可调用对象中调用的函数，用来处理响应，参数包括状态码、响应头等。（主要处理状态码和header）
##### WSGI服务器代码启动说明
- WSGIServer-->HTTPServer-->socketserver.TCPServer-->BaseServer
- WSGIRequestHandler-->BaseHTTPRequestHandler-->socketserver.StreamRequestHandler-->BaseRequestHandler
- ServerHandler-->SimpleHandler-->BaseHandler-->
---
- 1、WSGIServer实例化，传入套接字所需IP、Port和handler处理类WSGIRequestHandler到init方法
- 2、TCPServer中的init：先调BaseServer的init方法注册了server_address、RequestHandlerClass(WSGIRequestHandler)等属性，然后创建socket套接字并绑定端口设置request_queue_size为5后开始socket监听。
- 3、WSGIServer实例调用set_app方法：传入app（处理请求方法）
- 4、WSGIServer实例启动服务：wsgi_server.serve_forever()，开始监听READ事件，当请求到达时通过_handle_request_noblock方法处理请求。
- 5、_handle_request_noblock方法：先get_request获取请求来的套接字，再verify_request方法验证请求，然后process_request方法处理请求，process_request方法会调finish_request方法
- 6、finish_request方法：调用WSGIRequestHandler处理请求，传入request、client_address、self(WSGIServer实例)
- 7、WSGIRequestHandler：调用init方法(BaseRequestHandler)实例化，接收request、client_address、WSGIServer实例并注册成实例属性，并调用setup方法(StreamRequestHandler)创建连接文件等，然后调用handle方法(WSGIRequestHandler)
- 8、handle方法：通过parse_request方法解析请求，然后传入self.rfile、self.wfile、sys.stderr、environ到ServerHandler创建ServerHandler实例(SimpleHandler的init，默认多线程multithread为true)，并将ServerHandler实例的request_handler设置为WSGIRequestHandler的实例，然后传入app名称调用run方法(BaseHandler)处理请求
- 9、run方法(BaseHandler)：调用setup_environ方法注入环境变量（请求信息等），传入environ和start_response调用app方法来实际处理请求，处理完成后并调用finish_response方法处理结果
```
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2021/1/15 09:37
# File: Application.py
from wsgiref.simple_server import make_server


def hello(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-Type', 'text/html')]
    start_response(status, response_headers)
    path = environ.get('PATH_INFO') and environ['PATH_INFO'][1:] or 'hello'
    return [f'<h1>{path}</h1>'.encode()]


def main():
    server = make_server(host='127.0.0.1', port=2650, app=hello)
    print('Start listening...')
    server.serve_forever()


if __name__ == '__main__':
    main()

```


#### 简易HttpServer
```
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/8/2 09:09
# File: http_service.py

import socket
import sys

STATUS_FLAG = True
BUFFER_SIZE = 1024
GET_PIC_RESPONSE_HEADER = 'HTTP/1.x 200 ok\r\nContent-Type: image/jpg\r\n\r\n'
BAD_REQUEST_HEADER = 'HTTP/1.x 400 BadRequest\r\nContent-Type: text/html\r\n\r\n'
GET_RESPONSE_HEADER = 'HTTP/1.x 200 ok\r\nContent-Type: text/html\r\n\r\n'
POST_RESPONSE_HEADER = 'HTTP/1.x 200 ok\r\nContent-Type: text/html\r\n\r\n'

_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
_socket.bind(('127.0.0.1', 2650))
_socket.listen(5)


def get(client, header):
    """
    GET请求处理
    :param client:
    :param header:
    :return:
    """
    print(header)
    _res = GET_RESPONSE_HEADER + 'Hello World\r\n' + '\r\n'.join(str(header).split('\\r\\n'))
    client.send(_res.encode('utf-8'))
    client.close()


def post(client, header):
    """
    POST请求处理
    :param client:
    :param header:
    :return:
    """
    print(header)
    _res = GET_RESPONSE_HEADER + 'Hello World\r\n' + header.split()
    client.send(_res.encode('utf-8'))
    client.close()


while STATUS_FLAG:
    _request_map = {
        'GET': 'get',
        'POST': 'post'
    }
    try:
        _conn, _addr = _socket.accept()
        _request_header = _conn.recv(1024)
        _list = str(_request_header)[2:].split('\\r\\n')
        _func = _request_map[_list[0].split(' ')[0]]
        _func = getattr(sys.modules[__name__], _func)
        _func(_conn, _request_header)
    except Exception as e:
        _res_str = BAD_REQUEST_HEADER
        _conn.send(_res_str.encode('utf-8'))
        print(e.args)

```

### Tornado
- 端口监听部分：
实际的socket套接字的创建是在bindSocket方法中，然后该方法被TCPServer的listen方法使用，而HttpServer继承了TCPServer并且initialize方法创建了TCPServer实例
```
HttpServer.initialize -> TCPServer.listen -> bindSocket -> socket.bind/listen
```

```
```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/7/2 08:44
# File: Standard.py

from functools import lru_cache, singledispatch

"""
setDefault(key,default).operate 相对于先get再赋值少了一次查询
ex: 将a注入到b中key为a的列表中
"""
a = [{'a':1}, {'a':2}, {'a':3}, {'a':4}, {'a':5}]
b = {'b': []}
[b.setdefault('a', []).append(_['a']) for _ in a]

"""
functools.lru_cache(maxsize=128,typed=False)：缓存被装饰函数结果，若typed为True则将结果分开保存
functools.singledispatch：实现dispatch功能（方法复写）
"""


@lru_cache()
def fibonacci(n):
    return n if n < 2 else fibonacci(n - 2) + fibonacci(n - 1)


@singledispatch
def obj_convert(obj):
    return obj


@obj_convert.register(str)
def _(obj):
    return str(obj)


@obj_convert.register(int)
def _(obj):
    return int(obj)


@obj_convert.register(tuple)
@obj_convert.register(list)
def _(obj):
    return list(obj)


```
