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