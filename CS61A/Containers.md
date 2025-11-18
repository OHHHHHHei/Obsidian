# Aggregation

序列处理中的第三种常见模式是将序列中的所有值聚合为一个值。内置函数 `sum`、`min` 和 `max` 都是聚合函数的示例。

通过组合对每个元素进行计算、选择元素子集和聚合元素的模式，我们就可以使用序列处理的方法解决问题。

1. **`sum(iterable[, start])`**
    
    - **功能**: 计算一个可迭代对象中所有**数字**的和。
        
    - **参数**:
        
        - `iterable`: 必需的，包含数字的可迭代对象。
            
        - `start`: 可选的，一个初始值，会加到总和上。如果省略，默认为 0。
            
    - **注意**: 这个函数不能用于字符串。如果 `iterable` 为空，则返回 `start` 的值。
        
2. **`max(iterable[, key=func])`** 或 **`max(a, b, c, ...[, key=func])`**
    
    - **功能**: 找出最大项。
        
    - **用法**:
        
        - 当提供一个可迭代对象作为参数时，返回其中最大的元素。
            
        - 当提供两个或更多个独立的参数时，返回这些参数中最大的一个。
            
    - `key=func` 是一个可选参数，用于指定一个函数来决定如何比较元素的大小。
        
3. **`all(iterable)`**
    
    - **功能**: 判断一个可迭代对象中的所有元素是否都为 `True`（或者说，布尔值为 `True`）。
        
    - **返回值**:
        
        - 如果所有元素都为 `True`，则返回 `True`。
            
        - 如果可迭代对象为空，也返回 `True`。
            
        - 只要有一个元素为 `False`，就返回 `False`。
            

此外还有 min 和 any, 它们是 max 和 all 的补集：

-  `min()` 函数，与 `max()` 相对，用于查找最小值。
    
- `any()` 函数，与 `all()` 相对，用于判断可迭代对象中是否存在**任何一个**为 `True` 的元素。
# Strings

字符串字面量（string literals）可以表示任意文本，使用时将内容用单引号或双引号括起来。

```python
>>> 'I am string!'
'I am string!'
>>> "I've got an apostrophe"
"I've got an apostrophe"
>>> '您好'
'您好'
```

我们已经在代码中看到过字符串，比如文档字符串（docstring）、`print` 的调用中，以及 `assert` 语句中的错误消息。

字符串同样满足我们在本节开头介绍的序列的两个基本条件：它们具有长度且支持元素选择。字符串中的元素是只有一个字符的字符串。字符可以是字母表中的任何单个字母、标点符号或其他符号。

与其他编程语言不同，Python 没有单独的字符类型，任何文本都是字符串。表示单个字符的字符串的长度为 1。

```python
>>> city = 'Berkeley'
>>> len(city)
8
>>> city[3]
'k'
```

与列表一样，字符串也可以通过加法和乘法进行组合。

```python
>>> 'Berkeley' + ', CA'
'Berkeley, CA'
>>> 'Shabu ' * 2
'Shabu Shabu '
```

**成员资格（Membership）**：字符串的行为与 Python 中的其他序列类型有所不同。字符串抽象不符合我们对列表和范围描述的完整序列抽象。具体来说，成员运算符 `in` 应用于字符串时的行为与应用于序列时完全不同，它匹配的是子字符串而不是元素。（译者注：如果字符串的行为和列表的一样，则应该匹配字符串的元素，即单个字符，但实际上匹配的是任意子字符串）

```python
>>> 'here' in "Where's Waldo?"
True
```

**多行字面量（Multiline Literals）**：字符串可以不限于一行。跨越多行的字符串字面量可以用三重引号括起，我们已经在文档字符串中广泛使用了这种三重引号。

```python
>>> """The Zen of Python
claims, Readability counts.
Read more: import this."""
'The Zen of Python\nclaims, "Readability counts."\nRead more: import this.'
```

在上面的打印结果中，`\n`（读作“反斜杠 n”）是一个表示换行的单个元素。尽管它显示为两个字符（反斜杠和 "n" ），但为了便于计算长度和元素选择，它被视为单个字符。

**字符串强制转换（String Coercion）**：通过以对象值作为参数调用 `str` 的构造函数，可以从 Python 中的任何对象创建字符串。字符串的这一特性在用构造各种类型对象的描述性字符串时非常有用。

```python
>>> str(2) + ' is an element of ' + str(digits)
'2 is an element of [1, 8, 2, 8]'
```

# 字典

字典是 Python 中极为重要和常用的数据结构，掌握它对于编写高效、易读的代码至关重要。

### 1. 什么是字典？

你可以把 Python 的字典想象成一本真正的英文字典。在英文字典里，你通过一个**单词（key）** 去查找它的**释义（value）**。

Python 的字典也是如此，它是一个**键值对（key-value pair）**的集合。每个元素都由一个唯一的**键（key）**和与之对应的**值（value）**组成。

```python
# 一个表示个人信息的字典
person = {
    "name": "张三",
    "age": 30,
    "city": "北京",
    "is_student": False
}
print(person)
# 输出: {'name': '张三', 'age': 30, 'city': '北京', 'is_student': False}
```

在这个例子里，`"name"`, `"age"`, `"city"` 就是**键**，而 `"张三"`, `30`, `"北京"` 就是与它们分别对应的**值**。

### 2. 字典的核心特性

- **键值对结构**：数据以 `key: value` 的形式成对存储。
    
- **有序性 (Ordered)**：在 Python 3.7 及之后的版本中，字典是**有序的**。这意味着字典会记住你插入键值对的顺序。在早期的 Python 版本（3.6及以前）中，字典是无序的。
    
- **可变性 (Mutable)**：字典是可变的，你可以在创建之后随时添加、修改或删除其中的元素。
    
- **键的唯一性和不可变性**：
    
    - 字典中的**键必须是唯一的**，不能重复。如果你尝试用一个已存在的键添加新值，原来的值会被覆盖。
        
    - 字典的**键必须是不可变类型**（immutable），比如字符串（`str`）、数字（`int`, `float`）、布尔值（`bool`）或者元组（`tuple`）。列表（`list`）或者另一个字典（`dict`）不能作为键，因为它们是可变的。
        
    - 值（value）则可以是任意数据类型，甚至可以是另一个字典。

## 字典推导式

字典推导式的基本语法结构如下：
#### 基本形式

```python
{key_expression: value_expression for item in iterable}
```

- `{...}`：表示我们正在创建一个字典。

- `key_expression: value_expression`：这是新字典中每个键值对的表达式。

- `for item in iterable`：一个标准的 `for` 循环，用于遍历源可迭代对象（如列表、元组、集合等）中的每一项。
#### 带条件判断的形式

你还可以在后面加上一个 `if` 条件，用于筛选元素。

```python
{key_expression: value_expression for item in iterable if condition}
```

- `if condition`：只有当 `condition` 为 `True` 时，当前 `item` 才会被用来生成新的键值对