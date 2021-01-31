
```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/6/2 22:18
# File: Sort.py


class BooSort(object):

    @staticmethod
    def sort(_list: list) -> list:
        """
        可优化，当某次冒泡不再出现交换操作时说明数据已经有序即可跳出排序
        """
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

class InsertSort(object):

    def sort(self, nums: list) -> list:
        """
        插入排序，初始状态第一个数为有序区，其余为无需区，然后从无序区一个个插入到有序区
        """
        if len(nums) <= 1:
            return nums
        for _ in range(1, len(nums)):
            _tmp = nums[_]
            while _ > 0:
                if nums[_ - 1] > _tmp:
                    nums[_] = nums[_ - 1]
                    _ -= 1
                else:
                    break
            nums[_] = _tmp
        return nums

class MergeSort(object):
    """
    归并排序
    """

    def sort(self, nums: list) -> list:
        """
        空间复杂度有点高，数组合并？
        """

        def merge(sub1, sub2):
            """
            合并两个顺序数组
            """
            _re = []
            _i_1 = _i_2 = 0
            _l1, _l2 = len(sub1), len(sub2)
            while _i_1 < _l1 and _i_2 < _l2:
                if sub1[_i_1] < sub2[_i_2]:
                    _re.append(sub1[_i_1])
                    _i_1 += 1
                else:
                    _re.append(sub2[_i_2])
                    _i_2 += 1

            _re += sub1[_i_1:] if _i_1 < _l1 else sub2[_i_2:]
            return _re

        def merge_sort(sub_nums):
            """
            先拆了排序，然后再合并
            """
            _l = len(sub_nums)
            if _l == 1:
                return sub_nums
            else:
                return merge(merge_sort(sub_nums[:_l // 2]), merge_sort(sub_nums[_l // 2:]))

        return merge_sort(nums)


class BucketSort(object):

    def sort(self, nums):
        """
        桶排序
        将数据丢到有序的范围桶中，然后桶内数据再使用快排进行排序，即完成对整个数据的排序
        适用场景：数据无限，内存有限，分布均匀（逐步读取大文件，根据范围追加到范围的桶文件中，然后针对每个文件进行加载然后快速排序）
        """


class CountingSort(object):

    def sort(self):
        """
        计数排序
        桶排序的特殊情况：桶内数据大小相同
        """


class RadixSort(object):

    def sort(self, nums):
        """
        基数排序：采用稳定排序，根据前_i-1个元素划分为一个桶
        待排序数据可按位拆分，不足用0补齐
        """
        _k = 11
        for _i in range(_k):
            """
            第_i位数据排序
            先找到0-(_i-1)相同的作为一个桶，然后排序
            """
            pass


def get_middle_node(header):
    """
    判断一个链表的中间节点
    快慢指针，快指针一次访问两个节点，慢指针一次访问一个节点，当快指针到达表尾时慢指针为中间节点
    """
    _fast_cursor = _slow_cursor = header
    while _fast_cursor and _fast_cursor.next:
        _fast_cursor = _fast_cursor.next.next
        _slow_cursor = _slow_cursor.next
    if _fast_cursor:
        # 节点奇数
        return _slow_cursor.next
    else:
        # 节点偶数
        return _slow_cursor


_s1 = BooSort()
_s2 = ChooseSort()
_s3 = QuickSort()
_s4 = InsertSort()
_s5 = MergeSort()
print(_s1.sort([7, 4, 8, 4, 9, 3, 1, 1, 0, 4, 6, 8]))
print(_s2.sort([7, 4, 8, 4, 9, 3, 1, 1, 0, 4, 6, 8]))
print(_s3.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))
print(_s4.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))
print(_s5.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))

```