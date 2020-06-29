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
“{index1},{index2}”.format(*list)
“{key1},{key2}”.format(**dict)
***
python测试框架
***
xrange(x,y)生成一个x到y的等比数列