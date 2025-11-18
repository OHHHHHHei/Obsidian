# Lists
类似数组
```python
digits = [1, 8, 2, 8]
```
这样就创建了一个列表

# For 循环

Python 的 `for` 循环可以通过直接遍历元素值来简化函数，相比 `while` 循环无需引入变量名 `index`。

python

```python
>>> def count(s, value):
        """统计在序列 s 中出现了多少次值为 value 的元素"""
        total = 0
        for elem in s:
            if elem == value:
                total = total + 1
        return total
>>> count(digits, 8)
2
```

`for` 循环语句按以下过程执行：

1. 执行头部（header）中的 `<expression>`，它必须产生一个可迭代（iterable）的值

2. 对该可迭代值中的每个元素，按顺序：
    1. 将当前帧的 `<name>` 绑定到该元素值
    2. 执行 `<suite>`

## 序列解包（Sequence unpacking)

程序中的一个常见情况是序列的元素也是序列，但所有内部序列的长度是固定相同的。`for` 循环可以在头部的 `<name>` 中包含多个名称，来将每个元素序列“解包”到各自的元素中。

例如，我们可能有一个包含以列表为元素的 `pairs`，其中所有内部列表都只包含 2 个元素。

```python
>>> pairs = [[1, 2], [2, 2], [2, 3], [4, 4]]
```

此时我们希望找到有多少第一元素和第二元素相同的内部元素对，下面的 `for` 循环在头部中包括两个名称，将 `x` 和 `y` 分别绑定到每对中的第一个元素和第二个元素。

```python
>>> same_count = 0
>>> for x, y in pairs:
        if x == y:
            same_count = same_count + 1
>>> same_count
2
```

这种将多个名称绑定到固定长度序列中的多个值的模式称为序列解包（sequence unpacking），这与赋值语句中将多个名称绑定到多个值的模式类似。

# Range

`range` 是 Python 中的另一种内置序列类型，用于表示整数范围。范围是用 `range` 创建的，它有两个整数参数：起始值和结束值加一。

```python
>>> range(1, 10)  # 包括 1，但不包括 10
range(1, 10)
```

将 `range` 的返回结果传入 `list` 构造函数，可以构造出一个包含该 `range` 对象中所有值的列表，从而简单的查看范围中包含的内容。

```python
>>> list(range(5, 8))
[5, 6, 7]
```

如果只给出一个参数，参数将作为双参数中的“结束值加一”，获得从 0 到结束值的范围。（译者注：其实单参数就相当于默认了起始值从 0 开始）

```python
>>> list(range(4))
[0, 1, 2, 3]
```

范围通常出现在 `for` 循环头部中的表达式，以指定 `<suite>` 应执行的次数。一个惯用的使用方式是：如果 `<name>` 没有在 `<suite>` 中被使用到，则用下划线字符 "_" 作为 `<name>`。

```python
>>> for _ in range(3):
        print('Go Bears!')

Go Bears!
Go Bears!
Go Bears!
```

对解释器而言，这个下划线只是环境中的另一个名称，但对程序员具有约定俗成的含义，表示该名称不会出现在任何未来的表达式中。
# 列表推导式 (List Comprehension)

许多序列操作可以通过对序列中的每个元素使用一个固定表达式进行计算，并将结果值保存在结果序列中。在 Python 中，列表推导式是执行此类计算的表达式。

```python
>>> odds = [1, 3, 5, 7, 9]
>>> [x+1 for x in odds]
[2, 4, 6, 8, 10]
```

上面的 `for` 关键字并不是 `for` 循环的一部分，而是列表推导式的一部分，因为它被包含在方括号里。子表达式 `x+1` 通过绑定到 `odds` 中每个元素的变量 `x` 进行求值，并将结果值收集到列表中。

另一个常见的序列操作是选取原序列中满足某些条件的值。列表推导式也可以表达这种模式，例如选择 `odds` 中所有可以整除 25 的元素。

```python
>>> [x for x in odds if 25 % x == 0]
[1, 5]
```

列表推导式的一般形式是：

```python
[<map expression> for <name> in <sequence expression> if <filter expression>]
```

为了计算列表推导式，Python 首先执行 `<sequence expression>`，它必须返回一个可迭代值。然后将每个元素值按顺序绑定到 `<name>`，再执行 `<filter expression>`，如果结果为真值，则计算 `<map expression>`，`<map expression>` 的结果将被收集到结果列表中。

# zip函数

zip可以接受一个或者多个可迭代对象，并且把这些对象中相应位置的元素打包成一个个元组，并且返回一个包含这些元组的迭代器

```python
names = ['Alice', 'Bob', 'Charlie']
scores = [95, 88, 76]

# 使用 zip 将两个列表打包
zipped_data = zip(names, scores)

# zip() 返回的是一个 zip 对象（迭代器），需要转换为列表才能看到全部内容
print(zipped_data)  
# 输出: <zip object at 0x...>

# 转换为列表查看
print(list(zipped_data))
# 输出: [('Alice', 95), ('Bob', 88), ('Charlie', 76)]
```

如果 `zip` 接收的列表长度不同 **`zip` 会在最短的那个列表耗尽后停止。** 它不会报错，也不会用任何值去填充

```Python
names = ['Alice', 'Bob', 'Charlie', 'David'] # 4个元素
scores = [95, 88, 76] # 3个元素

zipped_data = zip(names, scores)

print(list(zipped_data))
# 输出: [('Alice', 95), ('Bob', 88), ('Charlie', 76)]
# 'David' 被忽略了，因为 scores 列表在第3个元素之后就结束了。
```
# 切片 (slicing)

在 Python 中，切片的通用语法是 `[start:stop:step]`

在 Python 中，序列切片的表达方式类似于元素选择，都使用方括号，方括号中的冒号用于分隔起始索引和结束索引。

如果起始索引或结束索引被省略则默认为极值：当起始索引被省略，则起始索引为 0；当结束索引被省略，则结束索引为序列长度，即取到序列最后一位。

```python
>>> digits [1, 8, 2, 8]
>>> digits[0:2]
[1, 8]
>>> digits[1:]
[8, 2, 8]
```

省略 `start` -> `[:stop]`

省略 `stop` -> `[start:]`

同时省略 `start` 和 `stop` -> `[:]` 或 `[::step]`

`[:]` (步长为默认的 1)
`[::step]` (指定步长)

# 递归

列表同样可以进行递归操作
根据上面的切片操作
我们通常将列表分为第一个和除了第一个的部分
根据这个来进行递归操作