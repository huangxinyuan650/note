```python
# _*_ coding:utf-8_*_
# Author:   Ace Huang
# Time: 2020/6/10 22:36
# File: TreeView.py


class TreeNode(object):

    def __init__(self, value: str):
        self.value = value
        self.left = None
        self.right = None

    def set_child(self, value: str, child_type: int = 0):
        """
        设置孩子节点
        :param value:
        :param child_type:
        :return:
        """
        setattr(self, 'left' if not child_type else 'right', TreeNode(value))
        return getattr(self, 'left' if not child_type else 'right')


class ReadAllSolution(object):

    def __init__(self):
        pass

    def read_first(self, tree: TreeNode):
        """
        先序遍历 root->left->right
        :param tree:
        :return:
        """
        if tree:
            print(tree.value)
            self.read_first(tree.left)
            self.read_first(tree.right)

    def read_middle(self, tree: TreeNode):
        """
        中序遍历 left->root->right
        :param tree:
        :return:
        """
        if tree:
            self.read_middle(tree.left)
            print(tree.value)
            self.read_middle(tree.right)

    def read_last(self, tree: TreeNode):
        """
        后序遍历 left->right->root
        :param tree:
        :return:
        """
        if tree:
            self.read_last(tree.left)
            self.read_last(tree.right)
            print(tree.value)


if __name__ == '__main__':
    _root = TreeNode('A')
    _root.set_child('B').set_child('D', 1)
    _root.set_child('C', 1).set_child('E')
    s = ReadAllSolution()
    s.read_first(_root)
    print('=======')
    s.read_middle(_root)
    print('=======')
    s.read_last(_root)

```