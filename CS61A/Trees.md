# 树的结构
### 核心思想：树的表示方法

一个数据抽象的设计过程

这个实现的核心思想是：**一棵树就是一个列表**。==（定义表示）==

- 列表的**第一个元素** (`tree[0]`) 是这棵树（或子树）的根节点的**值或标签（label）**。
    
- 列表的**其余所有元素** (`tree[1:]`) 组成一个新的列表，其中包含了所有的**分支（branches）**，而每个分支本身也是一棵符合同样规则的树。

一个树有一个 **根标签（root label）** 和一系列 **分支（branch）。** 我们规定 **树的每个分支都是一棵树** ，没有分支的树称为**叶子（leaf）。** 树中包含的任何树都称为该树的子树（例如分支的分支）。树的每个子树的根称为该树中的一个**节点（node）。** 

树的数据抽象由构造函数 `tree`、找出标签 `label` 和找出分支 `branches` 组成。==（定义接口）==
我们从简化版本开始讲起。

### 注意：`list()` 函数不会多套一层`[]` 
当 `list()` 函数的参数**已经是一个列表（或任何可迭代对象，如元组）** 时，它的作用是：

**遍历你给它的那个列表，取出里面的每一个元素，然后用这些元素创建一个全新的列表。**

它本质上是在**制作一个“浅拷贝”（Shallow Copy）**。

```python
>>> def tree(root_label, branches=[]):
        for branch in branches:
            assert is_tree(branch), '分支必须是树'
        return [root_label] + list(branches)

>>> def label(tree):
        return tree[0]

>>> def branches(tree):
        return tree[1:]
```

只有当树有根标签并且所有分支也是树时，树才是结构良好的。在 `tree` 构造函数中使用了 `is_tree` 函数以验证所有分支是否结构良好。
`tree` 中的`branches` 一定要使用`tree` 函数来构造，而不是一个列表，因为那样会违反抽象屏障（abstraction barrieres）。

```python
>>> def is_tree(tree):
        if type(tree) != list or len(tree) < 1:
            return False
        for branch in branches(tree):
            if not is_tree(branch):
                return False
        return True
```

`is_leaf` 函数检查树是否有分支，若无分支则为叶子节点。

```python
>>> def is_leaf(tree):
        return not branches(tree)
```

这是一个样例

```python
>>> t = tree(3, [tree(1), tree(2, [tree(1), tree(1)])])
>>> t
[3, [1], [2, [1], [1]]]
>>> label(t)
3
>>> branches(t)
[[1], [2, [1], [1]]]
>>> label(branches(t)[1])
2
>>> is_leaf(t)
False
>>> is_leaf(branches(t)[0])
True
```

# Tree Processing

 
