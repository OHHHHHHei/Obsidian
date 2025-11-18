# Linked Lists Class

### 1. 链表的递归定义 (Recursive Definition)

CS61A 中链表的实现方式非常巧妙，它本身就是递归的：

- 一个 `Link` 对象代表链表中的一个“节点”（Node）。
    
- 每个 `Link` 对象（节点）都有两个属性：
    
    1. `self.first`: 存储当前节点的值（例如 3, 4, 5...）。
        
    2. `self.rest`: 存储指向 **列表剩余部分** 的引用。
        

最关键的一点是，`self.rest` **本身也是一个 `Link` 对象**。这就是递归的体现。

### 2. 如何表示“空列表”（Base Case）

如果 `rest` 总是另一个 `Link` 对象，那么这个链表要如何结束呢？

- 这就是 `Link.empty = ()` 的作用。
    
- `Link.empty` 被定义为一个**空元组 `()`**。它是一个特殊的、全局共享的值，用来代表“**空列表**”，即链表的末尾。
    
- 在 `__init__` 构造函数中，`rest` 参数的默认值被设为 `Link.empty`。
    
- 这意味着，当你创建链表的最后一个节点时，比如 `Link(5)`，它实际上会被创建为 `Link(5, Link.empty)`，表示“这个节点的值是 5，它的后面是空列表”。
    

### 3. `assert`：保证数据结构的正确性

`assert rest is Link.empty or isinstance(rest, Link)`

这一行是**非常重要**的检查。它强制规定：

- 一个 `Link` 对象的 `rest` 属性（即“列表的剩余部分”）**必须是**以下两者之一：
    
    1. `Link.empty` （表示列表到此结束）
        
    2. `isinstance(rest, Link)`（表示列表还有后续，且后续部分是另一个 `Link` 对象）
        
- 这可以防止你创建出“坏”的链表，比如 `Link(1, 2)`。`2` 不是一个 `Link` 对象也不是 `Link.empty`，所以 `assert` 会报错，保证了链表的结构完整性。

# Linked List Processing




# Linked List Mutation

![[Pasted image 20251028015547.png]]

### 知识点一：链表突变 (Mutation)

- `>>> s = Link(1, Link(2, Link(3)))`
    
    - **初始状态**: 创建一个普通的链表 `s`，它在逻辑上代表 `(1, 2, 3)`。
        
    - `s` 指向 `[1, rest]` -> `[2, rest]` -> `[3, rest]` -> `()` (空)。
        
- `>>> s.first = 5`
    
    - **发生突变**: 这是一个**值突变 (value mutation)**。我们找到了 `s` 指向的那个 `Link` 对象（即第一个节点），并将其 `first` 属性从 `1` 修改为 `5`。
        
    - **结果**: 链表 `s` 现在逻辑上代表 `(5, 2, 3)`。
        
    - **对应图示**: 这就是为什么图中的第一个框 `First` 字段是 `5` 而不是 `1`。
        

### 2. 知识点二：别名 (Aliasing)

- `>>> t = s.rest`
    
    - **创建别名**: 这**不是**在创建新链表。它创建了一个新变量 `t`，让它指向 `s` 的 `rest` 属性所指向的对象。
        
    - `s` 指向 `[5, rest]` (我们称之为 **Node A**)。
        
    - `s.rest` 指向 `[2, rest]` (我们称之为 **Node B**)。
        
    - 所以，`t` 现在也指向 **Node B**。
        
    - **对应图示**: 你可以看到，`Global frame` 里的 `s` 指向第一个框 (Node A)，而 `t` 指向第二个框 (Node B)。
        

### 3. 知识点三：创建环形链表 (Creating a Cycle)

- `>>> t.rest = s`
    
    - **发生突变**: 这是最关键的一步，一次**结构突变 (structural mutation)**。
        
    - `t` 指向 **Node B** (`[2, rest]`)。
        
    - `t.rest` 原本指向 `Link(3)` 那个节点。
        
    - 这行代码把 `t.rest` (即 Node B 的 `rest` 指针) **修改为指向 `s`** (即 **Node A**)。
        
    - **结果**:
        
        1. `Link(3)` 节点“丢失”了。没有变量再指向它，它会被垃圾回收。
            
        2. `s` 和 `t` 现在指向一个**环形结构**。
            
            - 从 `s` (Node A) 出发，`s.rest` 指向 `t` (Node B)。
                
            - 从 `t` (Node B) 出发，`t.rest` 指向 `s` (Node A)。
                
    - **对应图示**: 这就是图中那条从第二个框 (Node B) 指回第一个框 (Node A) 的长长的弧线。
        

### 4. 知识点四：在环中追踪指针

现在，这个结构变成了一个无限循环： `s -> t -> s -> t -> ...`

- `>>> s.first`
    
    - `s` 指向 Node A (`[5, rest]`)。
        
    - `s.first` 就是 Node A 的 `first` 属性，即 `5`。
        
    - (输出 `5`，正确)
        
- `>>> s.rest.rest.rest.rest.rest.first`
    
    - 让我们来“顺着指针”走：
        
    - `s` -> Node A
        
    - `s.rest` -> Node B
        
    - `s.rest.rest` (即 `t.rest`) -> Node A
        
    - `s.rest.rest.rest` (即 `s.rest`) -> Node B
        
    - `s.rest.rest.rest.rest` (即 `t.rest`) -> Node A
        
    - `s.rest.rest.rest.rest.rest` (即 `s.rest`) -> Node B
        
    - 因此，`s.rest.rest.rest.rest.rest.first` 应该等于 `Node B.first`，也就是 `2`。


## Linked List Mutation Example

**如何通过突变 (mutation) 在一个有序链表的开头插入一个新元素**。

### 基本情况 (Base Cases)：递归停止

基本情况是指我们**不再需要**往下（`s.rest`）递归，在当前节点 `s` 就能做出决定的情况。

#### 🔹 Base Case 1: `v` 应该插入在最前面

- **代码:** `if s.first > v:`
    
- **示例:** `add(s, 0)` （当 `s` 是 `Link(1, Link(3, ...))` 时）
    
- **逻辑:**
    
    - 当前节点的值 `s.first`（例如 `1`）已经**大于**我们要插入的值 `v`（例如 `0`）。
        
    - 因为链表是有序的，这意味着 `v` 必须是这个链表（或子链表）的新头部。
        
- **操作 (Node Hijacking):**
    
    - `s.first, s.rest = v, Link(s.first, s.rest)`
        
    - 我们“劫持”了当前节点 `s`。
        
    - 把 `s` 的值 `s.first` 改为 `v` (即 `0`)。
        
    - 把 `s` 原来的值 (`1`) 和 `s` 原来的剩余部分 (`Link(3, ...)`) 打包成一个**新**的 `Link` 对象。
        
    - 让 `s` 的 `rest` 指向这个新创建的 `Link(1, Link(3, ...))`。
        
    - **结果:** `s` 现在变成了 `Link(0, Link(1, Link(3, ...)))`。
        
    - 函数结束，返回 `s`。
        

#### 🔹 Base Case 2: `v` 应该插入在最后面

- **代码:** `elif s.first < v and empty(s.rest):` (在 CS61A 中，`empty(s.rest)` 通常指 `s.rest is Link.empty`)
    
- **示例:** `add(s, 6)` （当递归到 `s` 是 `Link(5)` 时）
    
- **逻辑:**
    
    - 当前节点的值 `s.first`（例如 `5`）**小于** `v`（例如 `6`）。
        
    - **并且**，`s` 已经是最后一个节点了 (`s.rest` 是空的)。
        
    - 这意味着 `v` 应该被添加到 `s` 的后面。
        
- **操作:**
    
    - `s.rest = Link(v)`
        
    - 我们创建了一个新节点 `Link(6)`，并将其“挂”在 `s` 节点的 `rest` 属性上。
        
    - **结果:** `s` 现在变成了 `Link(5, Link(6))`。
        
    - 函数结束，返回 `s`。
        
#### 🔹 Base Case 3: `v` 已经存在 (找到重复)

- **代码:** `elif s.first == v:` (这个条件在幻灯片的代码中是隐含的)
    
- **示例:** `add(s, 3)` （当递归到 `s` 是 `Link(3, ...)` 时）
    
- **逻辑:**
    
    - 当前节点的值 `s.first`（例如 `3`）**等于** `v`（例如 `3`）。
        
    - 由于我们把它当作一个“集合”，重复的元素不需要被添加。
        
- **操作:**
    
    - 什么也不做 (`pass`)。
        
    - 函数直接进行到最后的 `return s`。

---

### 2. 递归步骤 (Recursive Step)：继续向下

递归步骤是指在当前节点 `s` 无法做出最终决定，需要**把问题交给 `s.rest`**（链表的剩余部分）去处理。

#### 🔹 Recursive Step 1: `v` 属于列表的“剩余部分”

- **代码:** `elif s.first < v:` (这个条件隐含了 `s.rest` _不是_ `Link.empty`，因为它没有被前面的 `Base Case 2` 捕获)
    
- **示例:** `add(s, 4)` （当 `s` 是 `Link(1, Link(3, ...))` 时
    
- **逻辑:**
    
    - 当前节点的值 `s.first`（例如 `1`）**小于** `v`（例如 `4`）。
        
    - 这意味着 `v` **不**属于当前位置 `s`。它可能在 `s.rest` 的某个地方。
        
    - 我们**信任**递归，把“将 `4` 添加到 `Link(3, Link(5, ...))`”这个更小的任务交给 `add(s.rest, v)` 去完成。
        
- **操作:**
    
    - `add(s.rest, v)`
        
    - **让我们追踪 `add(s, 4)`:**
        
        1. `add(Link(1, ...), 4)` 被调用。`1 < 4`。进入递归。
            
        2. 调用 `add(Link(3, ...), 4)`。`3 < 4`。继续递归。
            
        3. 调用 `add(Link(5), 4)`。`5 > 4`。**触发 Base Case 1**。
            
        4. `Link(5)` 节点被“劫持”，它变成了 `Link(4, Link(5))`。
            
        5. `add(Link(5), 4)` 返回指向 `Link(4, ...)` 的指针。
            
        6. `add(Link(3, ...), 4)` 的调用结束，返回 `s` (即 `Link(3, ...)`，但它的 `rest` 已经被修改了)。
            
        7. `add(Link(1, ...), 4)` 的调用结束，返回 `s` (即 `Link(1, ...)`，它的 `rest` 指向 `Link(3, ...)`，而 `Link(3, ...)` 的 `rest` 最终指向 `Link(4, ...)` )。

# Tree Class

通过面向对象变成方法来实现树的数据结构

这是我们通常（在 Python 中）更熟悉的方式。
- **`class Tree:`**:
    - 我们定义了一个 `Tree` 类。
- **`__init__(self, label, branches=[])`**:
    - 这是构造函数。一个 `Tree` 对象有两个**属性 (attributes)**：
        - `self.label`: 存储节点的值。
        - `self.branches`: 存储一个包含**其他 `Tree` 对象**的列表。
    - `assert isinstance(branch, Tree)`: 这是一个检查，确保所有分支确实都是 `Tree` 对象，这维护了数据的递归定义。

 数据抽象 (Data Abstraction) 的实现 

这是 CS61A 早期强调的一种编程范式，**它将“数据如何被使用”与“数据如何被存储”分离开**。
- **实现方式**:
    - 我们不使用 `class`。我们**约定**用一个**Python 列表 (list)** 来表示一棵树。
    - **约定**：列表的**第一个元素 (`[0]`) 是 `label`**，**剩余的元素 (`[1:]`) 是 `branches`**。
- **“抽象屏障” (Abstraction Barrier)**:
    - **构造函数 (Constructor):** `tree(label, branches=[])`
        - 它**不**返回一个类实例，而是返回一个列表：`[label] + list(branches)`。
    - **选择器 (Selectors):**
        - `label(tree)`: 返回 `tree[0]`。它“知道”标签存储在索引 0 处。
        - `branches(tree)`: 返回 `tree[1:]`。它“知道”分支存储在索引 1 之后。
    - `is_tree(branch)` (在 `tree` 函数中断言) 负责检查一个东西是不是合法的“树”（即一个列表）。

**灵活与解耦**：在右侧（数据抽象）的方法中，`fib_tree` 函数**完全不知道**树是用 `list` 实现的。它只知道 `tree()`、`label()` 和 `branches()` 这些函数。如果我们将来想把树的实现从 `list` 改成 `tuple` 或 `dict`，**我们只需要修改这三个函数，而 `fib_tree` 函数一行代码都不用改**。这就是数据抽象的强大之处。

## OOP的好处

### 封装 (Encapsulation)：将数据和行为捆绑在一起

这是最核心的好处之一。

- **OOP 方式**: `class Tree` 将**数据**（`self.label`, `self.branches`）和**操作这些数据的行为**（如 `__init__`）捆绑在一个叫做“对象”的单元中。
- **对比 (数据抽象)**: 在右侧，数据（一个 `list`）和行为（`tree()`, `label()`, `branches()` 函数）是**分离**的。

**好处在于：** 假设你想给“树”增加一个新功能，比如“计算树叶的数量”(count leaves)。

- **OOP**: 你可以在 `Tree` 类**内部**添加一个**方法 (method)**，比如 `def count_leaves(self): ...`。调用时非常自然：`my_tree.count_leaves()`。所有与“树”相关的功能都被整洁地组织在 `Tree` 类里。
- **数据抽象**: 你必须在**外部**定义一个**新函数** `def count_leaves(t): ...`。当你有很多数据类型（树、链表、图...）和很多功能时，你的全局空间会充斥着大量零散的函数 (`count_tree_leaves(t)`, `count_list_nodes(l)`...)，难以管理。
### 2. 继承 (Inheritance)：轻松实现代码复用和扩展

这是 OOP 独有的一个“超能力”。

- **问题**: 假设你学完了 `Tree`，现在想实现一个 `BinaryTree` (二叉树)。二叉树是树的一种_特例_，它最多只有两个分支（left 和 right）。
- **OOP**: 你可以轻松地让 `BinaryTree` **继承** `Tree`：
```python
    class BinaryTree(Tree):
        # 也许 __init__ 需要修改，只接受 left 和 right
        # 但其他方法，比如 count_leaves，也许可以直接重用！
        def __init__(self, label, left=None, right=None):
            # ... 构造逻辑 ...
            super().__init__(label, [left, right]) 
    ```
你不需要重写所有代码，只需在 `Tree` 的基础上进行修改和扩展。
- **对比 (数据抽象)**: 你**无法“继承”**。你必须从头开始，编写一套全新的函数：`binary_tree()`, `left_branch(t)`, `right_branch(t)` 等等，几乎无法复用 `tree()` 的逻辑。

### 3. 可读性和清晰的“命名空间” (Readability & Namespace)

- **可读性**: `my_tree.label` 的意图非常明确：“获取 `my_tree` 对象的 `label` 属性”。而 `label(my_tree)` 只是“一个名为 `label` 的函数作用于 `my_tree`”，`label` 和 `my_tree` 的关系不那么紧密。
- **命名空间**: 在 `class` 内部定义的方法（如 `__init__`）不会污染全局命名空间。你可以有 `Tree.__init__` 和 `LinkList.__init__`，它们不会冲突。但在数据抽象中，你必须小心命名你的全局函数，`tree()` 和 `link_list()` 不能重名。
### 4. 明确的类型 (Explicit Type)

使用 `class Tree`，你就创造了一个**新的、明确的数据类型**。

- `isinstance(my_tree, Tree)` 这样的检查非常可靠且符合 Python 的习惯。
- 而在右侧，`is_tree(t)` 只是一个我们自己定义的辅助函数，它可能只是简单地检查 `isinstance(t, list)`，但这并不精确（例如 `[1, 2, 3]` 也会通过，但它不是我们定义的树）。
# Tree Mutation

### 突变 (Mutation) —— 修剪当前层的分支

- **代码:** `t.branches = [ __b__ for b in t.branches if __b.label != n__ ]`
    
- **这是做什么的？**
    - 这是一个列表推导式 (list comprehension)，它会**创建一个新列表**。
    - 它会遍历 `t.branches` (树 `t` 的所有_直接_子分支 `b`)。
    - **`if b.label != n`**: 这是一个过滤器。只有当一个分支 `b` 的_标签_ (label) **不等于** `n` 时，它才会被保留在**新列表**中。
    - **`t.branches = ...`**: 这是关键的**突变**步骤。我们将 `t.branches` 属性重新赋值为这个过滤后的新列表。
- **效果:** 这一行代码**移除了 `t` 的所有 `label` 为 `n` 的直接子分支**。

### 知识点二：递归 (Recursion) —— 修剪所有后代分支

- **问题:** 步骤一（列表推导式）只检查了 `t` 的直接孩子。但是 `t` 的孙子、曾孙子... 也可能有 `label` 为 `n` 的子树需要被修剪。
- **解决方案:**
    - **`for b in t.branches:`**
        - **注意！** 这里的 `t.branches` 已经是被步骤一**修剪过后的**列表了。
    - **`prune( __b__ , __n__ )`**
        - 我们对_所有幸存下来_的分支 `b`，**递归地**调用 `prune` 函数。
        - 也就是告诉 `b`：“现在轮到你了，请你检查并修剪_你自己的_所有 `label` 为 `n` 的分支”。
- **效果:** 这个递归调用确保了修剪操作会一直传递到树的“叶子”为止，将所有层级中 `label` 为 `n` 的分支全部移除。




