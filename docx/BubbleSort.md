
```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/6/2 22:18
# File: Sort.py


class BooSort(object):

    @staticmethod
    def sort(_list: list) -> list:
        """
        冒泡排序：(大数沉底)在未排序区间从起始位开始比较相邻两个元素，若前一个数大于后一个数进行交换操作（根据由小到大还是由大到小自行调整）
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
        """
        选择排序：默认先设置起始位置为最大或最小值的索引，然后遍历未排序区间，
        比较当前值是否比最大或者最小索引的值大或者小，若满足则将最大或者最小值的索引改为该索引，
        遍历结束将起始位置和最大或者最小索引的位置进行交换
        """

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
        """
        快速排序：在指定区间内设置中间值为起始值，然后使用双指针从始末位置开始遍历，
        先从末尾向开始方向遍历，当发现比中间值小则与左指针位置的值进行交换，
        然后再从左指针的位置开始向末尾遍历当发现比中间值大则与右指针位置的值进行交换，如此往复到左右指针相遇
        然后从起始位置到相遇位置和相遇位置到末尾分别执行上面逻辑
        """

        def _sort(_start, _end):
            _s, _e = _start, _end
            _m = _list[_s]
            while _s < _e:
                while _s < _e and _list[_e] >= _m:
                    # 找到一个小于_m的值
                    _e -= 1
                _list[_s] = _list[_e]
                while _s < _e and _list[_s] <= _m:
                    # 找到一个大于_m的值
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
        快速排序：在指定区间内设置中间值为起始值，然后使用双指针从始末位置开始遍历，
        先从末尾向开始方向遍历，当发现比中间值小则与左指针位置的值进行交换，
        然后再从左指针的位置开始向末尾遍历当发现比中间值大则与右指针位置的值进行交换，如此往复到左右指针相遇
        然后从起始位置到相遇位置和相遇位置到末尾分别执行上面逻辑
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
        插入排序：初始状态第一个数为有序区，其余为无需区，然后从无序区一个个插入到有序区
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
    归并排序：先拆待排序数组为两部分，然后分别排序，排完后为两个顺序数组，然后合并两个顺序数组即得到一个有序数组
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


class ShellSort(object):

    def sort(self, nums):
        """
        希尔排序
        先n/2分组，然后在n/2/2直到步长为1，分组后采用直接插入排序
        """
        _l = len(nums)
        _step = _l
        while _step >= 2:
            _step = _step // 2
            for _ in range(_step):
                """
                采用直接插入排序将分组后的数据进行排序
                """
                _i = _ + _step
                while _i < _l:
                    _j = _i
                    _t = nums[_i]
                    while _j > _:
                        if nums[_j - _step] > _t:
                            nums[_j] = nums[_j - _step]
                            _j -= _step
                        else:
                            break
                    nums[_j] = _t
                    _i += _step
        return nums


HEAP_TYPE_BIG = 'big'
HEAP_TYPE_SMALL = 'small'


class HeapDemo(object):
    """
    手动实现堆：数组
    主要实现，堆调整、堆排序（但排完序后堆就清空了QAQ）
    """

    def __init__(self, heap_type=HEAP_TYPE_BIG, max_size=None):
        """
        初始化堆，当有设置堆最大值时直接初始化堆大小
        """
        self._data = []
        self._max_size = max_size if max_size else None
        self._heap_type = heap_type

    def adjust_heap_bottom(self):
        """
        调整堆，当push或者pop元素后对堆进行调整
        自底向上开始调整堆
        """
        _i = len(self._data)
        _p = _i // 2
        while _i > 1 \
                and ((self._heap_type == HEAP_TYPE_BIG and self._data[_i - 1] > self._data[_p - 1])
                     or (self._heap_type == HEAP_TYPE_SMALL and self._data[_i - 1] < self._data[_p - 1])):
            self._data[_i - 1], self._data[_p - 1] = self._data[_p - 1], self._data[_i - 1]
            _i = _i // 2
            _p = _i // 2

    def adjust_heap_top(self):
        """
        当堆顶元素弹出时，自顶向下开始调整堆
        """
        _i = 1
        _l = len(self._data)
        while _i * 2 <= _l:
            if _i * 2 + 1 <= _l:
                _change_i = _i * 2 if (
                        (self._heap_type == HEAP_TYPE_BIG and self._data[_i * 2 - 1] > self._data[_i * 2])
                        or (self._heap_type == HEAP_TYPE_SMALL and self._data[_i * 2 - 1] < self._data[_i * 2])) \
                    else _i * 2 + 1
            else:
                _change_i = _i * 2
            if (self._heap_type == HEAP_TYPE_BIG and self._data[_i - 1] < self._data[_change_i - 1]) \
                    or (self._heap_type == HEAP_TYPE_SMALL and self._data[_i - 1] > self._data[_change_i - 1]):
                self._data[_i - 1], self._data[_change_i - 1], _i = self._data[_change_i - 1], self._data[
                    _i - 1], _change_i
            else:
                break

    def heap_push(self, value):
        """
        数据入堆，然后调整堆
        对于堆满操作，大顶堆先弹出最小
        """
        if not self._max_size or len(self._data) <= self._max_size:
            self._data.append(value)
        else:
            pass
        self.adjust_heap_bottom()

    def heap_pop(self):
        """
        出堆：从堆顶弹出元素，然后调整堆（由堆顶开始调整）
        """
        if len(self._data) < 1:
            raise Exception
        _pop_item = self._data[0]
        self._data[0], self._data[-1] = self._data[-1], self._data[0]
        self._data.pop()
        self.adjust_heap_top()
        return _pop_item

    def heap_sort(self):
        """
        堆排序，逐个出堆然后调整堆
        """
        _tmp_data = self._data[::]
        _sort_list = [self.heap_pop() for _ in range(len(self._data))]
        self._data = _tmp_data
        return _sort_list

    def top_k(self, k):
        """
        输出top-k
        """
        return self.heap_sort()[:k]

    @property
    def data(self):
        return self._data


class BinarySearch(object):
    """
    二分查找
    """

    def search(self, nums, aim):
        """
        """
        _low, _high = 0, len(nums) - 1
        while _low <= _high:
            _mid = _low + ((_high - _low) >> 1)
            if nums[_mid] == aim:
                return _mid
            elif nums[_mid] < aim:
                _low = _mid + 1
            else:
                _high = _mid - 1
        return '404'


s = BinarySearch()
print(s.search([1, 2, 3, 4, 5, 6, 7], 6))

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
_s6 = ShellSort()
print(_s1.sort([7, 4, 8, 4, 9, 3, 1, 1, 0, 4, 6, 8]))
print(_s2.sort([7, 4, 8, 4, 9, 3, 1, 1, 0, 4, 6, 8]))
print(_s3.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))
print(_s4.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))
print(_s5.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))
print(_s6.sort([7, 4, 8, 4, 9, 3, 7, 1, 1, 0, 4, 6, 8]))

```