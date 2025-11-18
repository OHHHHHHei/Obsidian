# 什么是高阶函数

**定义**：一个函数如果满足以下**至少一个**条件，它就是高阶函数：
1. **接受其他函数作为参数**。
2. **将函数作为返回值**。
---
# 寻找通用模式

将目标进行抽象化处理，例如
- 计算 `1*1 + 2*2 + ... + n*n` (平方和)
- 计算 `1*1*1 + 2*2*2 + ... + n*n*n` (立方和)
- 计算一个比较复杂的关于 π 的级数求和
这几个计算任务的**基本模式**是完全一样的：都是对从 1 到 n 的一系列数字，先对每个数字进行某种特定操作（求平方、求立方、或某个复杂计算），然后将所有结果累加起来。
如果我们为每种计算都写一个独立的函数，代码会显得非常重复。因此，的核心思想就是：**将这种通用的“求和模式”抽象出来，创建一个更强大的、能够表达“求和”这个概念本身的函数。**
### 关键技术一：将函数作为参数

这是本章介绍的第一个，也是最核心的高阶函数技巧。网页中定义了一个非常通用的 `summation` 函数，其结构大致如下：
```python
def summation(n, term):
    total, k = 0, 1
    while k <= n:
        total, k = total + term(k), k + 1
    return total
```
这个 `summation` 函数接受两个参数：

- `n`: 需要累加的项数。
- `term`: **一个函数**。这个函数定义了“对每一项 k 具体做什么操作”。

**如何使用它？**

1. **计算平方和**:
```python
def square(x): 
    return x * x 
summation(10, square) # 将 square 函数作为参数传进去
```
1. **计算立方和**:
```python
def cube(x):
    return x * x * x
summation(10, cube) # 将 cube 函数作为参数传进去
```
通过这种方式，`summation` 函数封装了通用的“累加”逻辑，而 `square` 和 `cube` 函数则作为可插拔的“组件”，定义了每一项的具体计算细节。这就实现了逻辑的完美分离和复用。
### 关键技术二：将函数作为返回值

高阶函数的另一种形式是“函数工厂”，即一个函数它的返回值是**另一个新的函数**。

网页中举了一个 `make_adder` 的例子：
```python
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

- `make_adder` 函数接受一个参数 `n`。
- 它的返回值**不是一个数字**，而是内部定义的 `adder` **这个函数**。

**如何使用它？**
```python
# 创建一个“加 5”的函数
add_five = make_adder(5) 

# 现在 add_five 就是一个函数，可以拿来用了
result = add_five(10)  # result 会是 15
```
`make_adder` 就像一个可以制造各种“加法器”的工厂。你给它一个数字 `n`，它就为你量身定做一个“加 `n`”的函数。
___
**“传入的参数被返回的函数记住了”**，这就是这个模式最核心、最有用的心智模型。这个特性在计算机科学中被称为 **闭包 (Closure)**，它是实现很多高级功能和设计模式的基础。


# 柯里化

柯里化是一种将一个接受**多个参数**的函数，转变为一系列**只接受单个参数**的函数的技术。

柯里化的本质是：**用函数来接收参数，每接收一个参数，就返回一个等待接收下一个参数的新函数，直到接收完所有参数才最终返回值。**

我们可以定义函数来自动进行柯里化，以及逆柯里化变换（uncurrying transformation）：
```python
>>> def curry2(f):
        """返回给定的双参数函数的柯里化版本"""
        def g(x):
            def h(y):
                return f(x, y)
            return h
        return g
>>> def uncurry2(g):
        """返回给定的柯里化函数的双参数版本"""
        def f(x, y):
            return g(x)(y)
        return f
>>> pow_curried = curry2(pow)
>>> pow_curried(2)(5)
32
>>> map_to_range(0, 10, pow_curried(2))
1
2
4
8
16
32
64
128
256
512
```


# 函数装饰器

## 1. 包装函数

```python
# 这就是我们的装饰器
def laughing_magic(func):
    # func 在这里就代表 xiaoming_tells_a_joke 这个函数
    
    # 我们定义一个“包装后”的新功能
    def wrapper():
        print("（哈哈哈...气氛组已就位）") # 这是“之前”要做的
        
        func() # <-- 在这里，执行原始的功能（讲笑话）
        
        print("（哈哈哈...气氛真是太棒了）") # 这是“之后”要做的
        
    # 把这个包装好的、更强的新功能返回出去
    return wrapper
```
## 2.使用包装
```python
# 先定义原始函数
def xiaoming_tells_a_joke():
    print("小明：从前有座山，山里有个庙...")

# 手动施加魔法
xiaoming_tells_a_joke = laughing_magic(xiaoming_tells_a_joke)

# 调用“被施法后”的小明
xiaoming_tells_a_joke()
```
## 3.语法糖 @
```python
@laughing_magic
def xiaohong_sings_a_song():
    print("小红：啦啦啦啦啦...")
```
