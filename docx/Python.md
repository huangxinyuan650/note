### Python基础
#### Python数据类型
- 数字（Int、Float、Long、Complex）
- 字符串（str）
- 元组（tuple）
- 列表（list）
- 字典（dict）
- 集合（set）：集合中所有元素必须可散列，底层数据结构为哈希表（字典中只存放键没有值）
#### 可变类型与不可变类型
- 可变类型：数字、字符串、元组（元素为不可变类型）
- 不可变类型：所有映射
#### 哈希表、可散列与不可散列
- 哈希表：哈希函数+哈希冲突（1、再哈希 2、拉链法）
- 可散列数据类型：在对象生命周期中散列值不变（需实现__hash__、__eq__方法）。原子不可变类型、frozenset是可散列
- 不可散列数据类型：
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
- 孤儿进程：父进程早于子进程退出（孤儿进程将有init进程收养）
- 僵尸进程：子进程早于父进程退出，但父进程未处理子进程退出状态，子进程变成僵尸进程（保留少量PCB信息在内存中）

###### 进程间通信
- 管道：在内存中生成管道对象，多个进程操作同一个管道即可实现通信
- 消息队列：在内存中建立队列数据模型，多个进程操作队列来存取内容进行通信
- 共享内存：在内存中开辟一块对多个进程可见的空间，多个进程可读可写，但没次写入的都会覆盖上次内容
- 信号量：一个进程向另一个进程发送一个信号来传递某种信息，接收者根据信号做出相应操作
- 信号量：
- 套接字：

##### 多线程（CPU调度的最小单元）

###### 线程间通信（直接共享内存了，加锁方式）
- threading.Lock()
- threading.Rlock()
- threading.Condition()
- threading.Event()

#### 装饰器
- 闭包：
- 解释：在不改变函数调用方式的情况下 对函数进行额外功能的封装，装饰一个函数 转给他一个其他的功能

#### 单例


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

**闭包:**
指延伸了作用域的函数，其中包含函数定义体中引用、但是不在定义体中定义的非全局变量。（函数访问定义体之外定义的非全局变量）
***
**装饰器：**
```python
def dec_test(args):
    print args
    def des_2(func):
        print func.__name__
        return func
    return des_2

@dec_test(args="huang”)
def fun_1(arg):
    print arg
fun_1("hxy")
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











当装饰器带有参数时，需要在装饰器内部重新再定义一个方法来接收被修饰方法对象经过处理再返回被修饰方法然后在修饰器中再将新建方法返回
```python
__new__,__init__,__call__:(先new再init)
class Test(object):
    def __new__(cls, *args, **kwargs):
        print 'new'
        return super(Test, cls).__new__(cls, *args, **kwargs)#不加不会调用用init方法
```
#__new__必须有返回值，返回实例化出来的实例，如果__new__创建的是当前类的实例，会自动调用__init__函数，通过return语句里面调用的__new__函数的第一个参数是cls来保证是当前类实例，如果是其他类的类名，；那么实际创建返回的就是其他类的实例，其实就不会调用当前类的__init__函数，也不会调用其他类的__init__函数。

```python
    def __init__(self, x, y):
        print 'Init'
        self.x = x
        self.y = y
        self.z = (self.x, self.y)
    def __call__(self, *args, **kwargs):
        print 'call'
t = Test(1, 2)
t()
```
对象通过提供__call__(slef, [,*args [,**kwargs]])方法可以模拟函数的行为，如果一个对象x提供了该方法，就可以像函数一样使用它，也就是说x(arg1, arg2...) 等同于调用x.__call__(self, arg1, arg2) 
***
**静态方法和类方法：**
@staticmethod 通过类名调用，
@classmethod    通过类对象实例调用并将对象本身作为参数传入
***
**字符串格式化**
"{index1},{index2}".format(*list)
"{key1},{key2}".format(**dict)
f"{var1},{var2}" var1、var2该语句可访问的变量
***
xrange(x,y)生成一个x到y的等比数列