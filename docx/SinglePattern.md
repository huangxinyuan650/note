```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/7/5 11:31
# File: SinglePattern.py


def single_instance(cls):
    """

    :param cls:
    :return:
    """
    _instance = {}

    def get_instance():
        """:return:
        """
        return _instance.setdefault(cls, cls())

    return get_instance


@single_instance
class A(object):

    def __init__(self):
        self.count = 0

    def counter(self):
        self.count += 1
        print(self.count)


if __name__ == '__main__':
    _ins1 = A()
    _ins1.counter()
    _ins2 = A()
    _ins2.counter()
    print(_ins1 == _ins2)

```