# Generators

**生成器函数是一种特殊的函数，它不会一次性执行完毕，而是可以“暂停”执行并“交出”（`yield`）一个值，然后在下次被调用时从“暂停”的地方继续执行。** 

### . 核心区别：`yield` vs `return`

要理解生成器，首先要明白 `yield` 关键字和 `return` 关键字的区别。

- **`return`**：
    
    - 当你使用 `return` 时，函数会立即终止。
        
    - 它会返回一个值（或者 `None`）。
        
    - 函数内部的所有状态（比如局部变量）都会被丢弃。
        
    - 你再次调用该函数时，它会从头开始执行。
        
- **`yield`**：
    
    - 当你使用 `yield` 时，函数会暂停执行。
        
    - 它会“交出”一个值（`yield` 后面的值）。
        
    - 函数内部的状态（局部变量等）会被完整保留。
        
    - 下次函数被“唤醒”时（通常是通过 `next()` 或 `for` 循环），它会从 `yield` 语句的下一行继续执行。

# Generatoers & Iterators

## 连接迭代器 (a_then_b)

`yield from` 语句可以让你在一个生成器函数中，非常方便地“委托”另一个迭代器（iterator）或可迭代对象（iterable）来产生值。

- 创建一个生成器，它首先 `yield` 迭代器 `a` 中的所有元素，然后再 `yield` 迭代器 `b` 中的所有元素。
- **传统方式 `a_then_b`：**
 ```python
    def a_then_b(a, b):
        for x in a:
            yield x
        for x in b:
            yield x
    ```

**使用 `yield from` 的 `a_then_b_`：**
```python
def a_then_b_(a, b):
    yield from a
    yield from b
```

代码变得极其简洁。`yield from a` 会自动遍历 `a` 并 `yield` 它的所有值。当 `a` 耗尽后，代码继续执行到 `yield from b`，再自动遍历 `b` 并 `yield` 它的所有值。

## 递归生成器 (countdown)

这个例子展示了 `yield from` 在递归（recursion）中的强大应用。

- **目标：** 创建一个生成器，从数字 `k` 开始倒数到 1。
    
- **代码 (countdown)：**
```python
    def countdown(k):
        if k > 0:
            yield k
            yield from countdown(k-1)
    ```
    
- **执行过程 (以 `countdown(5)` 为例)：**
    
    1. `countdown(5)` 被调用。`k=5` > 0。
        
    2. `yield 5`：生成器产出第一个值 `5`。
        
    3. `yield from countdown(4)`：执行被“暂停”，**控制权被委托**给一个新的 `countdown(4)` 生成器实例。
        
    4. `countdown(4)` 执行：
        
        - `yield 4`：这个 `4` 被 `yield from` 语句“捕获”并传递给_最外层_的调用者。
            
        - `yield from countdown(3)`：控制权又被委托给 `countdown(3)`。
            
    5. 这个过程不断重复（`yield 3`、`yield 2`、`yield 1`）。
        
    6. 当 `countdown(0)` 被调用时，`if k > 0` 为 `False`，该生成器立即结束（触发 `StopIteration`）。
        
    7. `yield from countdown(0)` 接收到 `StopIteration`，于是 `countdown(1)` 的 `yield from` 语句执行完毕，`countdown(1)` 也随之结束。
        
    8. 这个结束信号像多米诺骨牌一样传回上一层，直到 `countdown(5)` 中的 `yield from countdown(4)` 执行完毕，`countdown(5)` 也结束。
        
**这个例子说明了：`yield from` 可以非常优雅地处理递归生成器，将“子生成器”产生的值无缝地传递给“父生成器”的调用者，极大地简化了递归逻辑。**

# Filter函数

`filter()` 函数接收两个参数：

```python
filter(function, iterable)
```

1. **`function`**：
    
    - 这是一个“谓词函数”（Predicate Function），意思是它必须是一个返回布尔值（`True` 或 `False`）的函数。
        
    - `filter()` 会将 `iterable` 中的_每一个元素_都传递给这个函数。
        
    - 如果函数返回 `True`，该元素就被“保留”下来。
        
    - 如果函数返回 `False`，该元素就被“过滤”掉。
        
2. **`iterable`**：
    
    - 你想要进行筛选的序列，例如一个 `list`、`tuple` 或 `range` 对象等。
        

## 返回值（重要！）

在 Python 3 中，`filter()` **不会**立即返回一个新的列表。

它返回的是一个 **`filter` 对象**，这是一个**迭代器 (iterator)**。

- **优点：** 这样做非常**节省内存**（惰性求值）。它不会一次性创建包含所有结果的新列表，而是在你_需要_下一个值的时候才去计算它。
    
- **如何使用：** 要获取所有筛选后的结果，你通常需要将这个迭代器转换成一个列表（或元组等），最常见的方法是使用 `list()`。