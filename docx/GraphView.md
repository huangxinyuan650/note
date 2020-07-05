```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/6/10 23:17
# File: GraphView.py

from queue import Queue


class GraphNode(object):

    def __init__(self, value: str):
        self.value = value
        self.children = []

    def add_child(self, value: str):
        _ = GraphNode(value)
        self.children.append(_)
        return _


class GraphView(object):

    def __init__(self):
        self.stack = []
        self.queue = Queue()
        self.read = []

    def DFS(self, graph: GraphNode):
        """
        深度优先
        一直入栈，没有就出栈
        :param graph:
        :return:
        """
        if graph not in self.read:
            print(graph.value)
            self.read.append(graph)
            # self.stack.append(graph)
            [self.DFS(_) for _ in graph.children]

    def BFS(self, graph: GraphNode):
        """
        广度优先
        一直读，读完入队列
        :param graph:
        :return:
        """
        if graph not in self.read:
            print(graph.value)
            self.read.append(graph)
            [self.queue.put(_) for _ in graph.children]
            while not self.queue.empty():
                self.BFS(self.queue.get())


if __name__ == '__main__':
    _root = GraphNode('A')
    _root.add_child('B').add_child('C').add_child('D')
    _root.add_child('E').add_child('F')
    _t = _root.add_child('G')
    _root.add_child('H').add_child('I').add_child('J')
    s = GraphView()
    s.BFS(_root)
    s.DFS(_root)

```