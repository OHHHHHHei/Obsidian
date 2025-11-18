Python 和许多其他编程语言都提供了一种统一的方法来按照顺序地处理容器内的元素，称为迭代器（iterators）迭代器是一种对象，提供对值逐一顺序访问的功能。 迭代器抽象有两个组件：

- 检索下一个元素的机制
- 到达序列末尾并且没有剩余元素，发出信号的机制

在迭代器上调用 `iter` 将返回该迭代器，而不是其副本。 Python 中包含此行为，以便程序员可以对某个值调用 `iter` 来获取迭代器，而不必担心它是迭代器还是容器。**“在迭代器上调用 `iter` 将返回该迭代器”** 这个设计，是一种被称为 **“幂等（Idempotent）”** 操作的体现。无论你对一个迭代器调用 `iter()` 多少次，你得到的结果都是一样的——就是那个迭代器本身。

# enumerate函数

`enumerate()` 函数返回一个 `enumerate` 对象，它是一个**迭代器**。每次迭代，它会生成一个包含两个元素的元组 `(index, value)`。

```python
fruits = ['apple', 'banana', 'cherry']

for index, fruit in enumerate(fruits):
    print(f"索引 {index} 对应水果: {fruit}")
```

一个非常常见的场景是，我们希望序号从 `1` 开始，而不是 `0`。这时 `start` 参数就派上用场了。

```python
tasks = ['写报告', '开会', '回复邮件']

# 我们希望任务列表从 1 开始编号
for number, task in enumerate(tasks, start=1):
    print(f"任务 {number}: {task}")
```

`enumerate()` 返回的是一个迭代器。如果你想一次性看到它生成的所有内容，可以用 `list()` 或 `tuple()` 将其转换。