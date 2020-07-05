
```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/6/2 22:18
# File: Sort.py


class BooSort(object):

    @staticmethod
    def sort(_list: list) -> list:
        def swap(n):
            _list[n - 1], _list[n] = _list[n], _list[n - 1]

        [
            swap(_j)
            for _i in range(len(_list) - 1)
            for _j in range(1, len(_list) - _i)
            if _list[_j - 1] > _list[_j]
        ]
        return _list


class ChooseSort(object):

    @staticmethod
    def sort(_list: list) -> list:

        for _i in range(len(_list) - 1):
            _min_i = _i
            for _j in range(_i + 1, len(_list)):
                if _list[_j] < _list[_min_i]:
                    _min_i = _j
            _list[_i], _list[_min_i] = _list[_min_i], _list[_i]
        return _list


class QuickSort(object):

    @staticmethod
    def sort(_list: list) -> list:

        def _sort(_start, _end):
            _s, _e = _start, _end
            _m = _list[_s]
            while _s < _e:
                while _s < _e and _list[_e] >= _m:
                    # 找到一个大于_m的值
                    _e -= 1
                _list[_s] = _list[_e]
                while _s < _e and _list[_s] <= _m:
                    # 找到一个小于_m的值
                    _s += 1
                _list[_e] = _list[_s]
            _list[_s] = _m
            print(_list)
            if _start < _s - 1:
                _sort(_start, _s - 1)
            if _end > _s + 1:
                _sort(_s + 1, _end)

        _sort(0, len(_list) - 1)
        return _list

    @staticmethod
    def sort_again(_list: list) -> list:
        """
        起始值为基准
        :param _list:
        :return:
        """

        def quick_sort(_start: int, _end: int):
            """

            :param _start:
            :param _end:
            :return:
            """
            _mid = _list[_start]
            _low, _high = _start, _end
            while _low < _high:
                while _low < _high and _list[_high] >= _mid:
                    _high -= 1
                _list[_low] = _list[_high]
                while _low < _high and _list[_low] <= _mid:
                    _low += 1
                _list[_high] = _list[_low]
            _list[_low] = _mid
            if _start < _low - 1:
                quick_sort(_start, _low - 1)
            if _end > _high + 1:
                quick_sort(_high + 1, _end)

        quick_sort(0, len(_list) - 1)
        return _list


_s1 = BooSort()
_s2 = ChooseSort()
_s3 = QuickSort()
print(_s1.sort([7, 4, 8, 4, 9, 3, 1, 1, 0, 4, 6, 8]))
print(_s2.sort([7, 4, 8, 4, 9, 3, 1, 1, 0, 4, 6, 8]))
print(_s3.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))

```