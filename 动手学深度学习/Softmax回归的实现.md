# 初始化模型参数

- **原始输入**：Fashion-MNIST 的图像是 $28 \times 28$ 像素。
- **展平（Flatten）**：我们要把每个像素看作一个特征。目前我们暂时忽略像素之间的空间关系（比如相邻像素的关系），直接把图像拉直。
- 计算：$28 \times 28 = 784$。
- 所以，输入特征数 `num_inputs = 784`。
- **输出类别**：数据集有 10 类（衣服、鞋子等）
- 所以，输出特征数 `num_outputs = 10`。
## 参数形状推导：
我们的线性变换公式是 $O = XW + b$。
- $X$ 的形状是 `(batch_size, 784)`。
- $O$（输出）的形状应该是 `(batch_size, 10)`，因为我们要对每个样本输出10个类别的得分。
- 根据矩阵乘法规则 $(N, d) \times (d, k) = (N, k)$：
    - 权重 $W$ 的形状必须是 `(784, 10)`。
    - 偏置 $b$ 的形状是 `(1, 10)`（或者长度为10的向量，通过广播加到每一行）。

我们使用正态分布初始化 $W$（均值0，标准差0.01），将 $b$ 初始化为0。**注意：一定要开启梯度追踪（`requires_grad=True`）**，否则无法反向传播更新参数。
```python
num_inputs = 784
num_outputs = 10

# W: (784, 10), b: (10,)
W = torch.normal(0, 0.01, size=(num_inputs, num_outputs), requires_grad=True)
b = torch.zeros(num_outputs, requires_grad=True)
```

# 定义softmax操作
Softmax 的定义：
$$\text{softmax}(\mathbf{X})_{ij} = \frac{\exp(\mathbf{X}_{ij})}{\sum_k \exp(\mathbf{X}_{ik})}$$
线性层的输出 $\mathbf{X}_{ij}$（这里指代Logits，即未归一化的预测值）可能是负数，也可能非常大（如1000）。这不符合“概率”的定义。概率必须满足两个条件：
1. 所有值非负。
2. 所有可能结果的概率之和为 1。

## **Softmax 的三个步骤：**

1. **求幂（Exponentiation）**：对每个元素做 $e^x$。
    - 原因：指数函数能将负数映射为正数（$e^{-5} > 0$），同时拉大数值之间的差异（大的更大）。
    - 代码：`X_exp = torch.exp(X)`
2. **求和（Partition Function）**：对每一行（每个样本）求和。
    - **关键点**：我们需要对**同一行**的所有类别得分求和。
    - 代码：`partition = X_exp.sum(1, keepdim=True)`。
    - **解释 `keepdim=True`**：假设 `X` 是 `(2, 5)`（2个样本，5类）。如果不加 `keepdim`，求和后变成 `(2,)` 的向量。加上后，变成 `(2, 1)` 的矩阵。这对于下一步的除法至关重要。
3. **归一化（Normalization）**：将每一行除以该行的和。
    - 代码：`return X_exp / partition`。
    - 这里用到了**广播机制（Broadcasting）**。`(2, 5)` 的矩阵除以 `(2, 1)` 的矩阵，Python 会自动把 `(2, 1)` 的列复制 5 次，使其变成 `(2, 5)`，然后逐元素相除。

```python
def softmax(X):
    X_exp = torch.exp(X)  # 1. 逐元素求幂
    partition = X_exp.sum(1, keepdim=True) # 2. 按行求和，保持二维结构
    return X_exp / partition  # 3. 广播除法
```

# 定义模型

```python
def net(X):
    return softmax(torch.matmul(X.reshape((-1, W.shape[0])), W) + b)
```

# 定义损失函数

- **数学回顾**：交叉熵损失公式简化后是 $-\log(p_{y})$，其中 $p_{y}$ 是模型预测**真实类别**发生的概率。
    - 举例：如果一张图是“猫”（类别索引为1），模型预测它是猫的概率是 0.6，那么损失就是 $-\log(0.6)$。我们只关心正确类别的那个概率，其他类别的概率不参与计算（因为标签是 One-hot 的，其他位置是0）。
- 代码实现挑战：
    如果用 for 循环一个一个样本去找对应类别的概率，效率太低。Python 提供了非常强大的高级索引功能。

```python
def cross_entropy(y_hat, y):
    # y_hat: 预测概率矩阵 (batch_size, num_classes)
    # y: 真实标签索引 (batch_size,)
    return -torch.log(y_hat[range(len(y_hat)), y])
```
- **`range(len(y_hat))`**：生成一个序列 `[0, 1, 2, ...]`，代表样本的行号。
- **`y`**：代表每个样本真实的类别列号。
- **`y_hat[行号, 列号]`**：这行代码的意思是，“对于第0行，取出第 `y[0]` 列的数；对于第1行，取出第 `y[1]` 列的数……”。
- **效果**：一次性把所有样本中**对应真实标签位置的预测概率**都取出来了，组成一个向量。然后直接求对数取负。这完全避免了 One-hot 编码的繁琐转换和矩阵乘法。

# 分类精度

型训练的目标是最小化损失，但我们需要给人看的是**精度**——到底猜对了多少？

- **硬预测（Hard Prediction）**： Softmax 吐出的是一堆概率，比如 `[0.1, 0.8, 0.1]`。我们认为概率最大的那个就是预测结果。
    - **`y_hat.argmax(axis=1)`**：沿着列的方向（横向）找最大值的索引。比如上面的例子，索引是 1。
- **精度计算函数 `accuracy`**
```python
    def accuracy(y_hat, y):
        if len(y_hat.shape) > 1 and y_hat.shape[1] > 1:
            y_hat = y_hat.argmax(axis=1) # 1. 转为类别索引
        cmp = y_hat.type(y.dtype) == y   # 2. 比较预测与真实标签
        return float(cmp.type(y.dtype).sum()) # 3. 求和得到正确个数
    ```
- **注意数据类型**：`y_hat` 出来的索引可能是整型，但有时候为了安全，会显式转换类型与 `y` 一致再比较。
- **返回值**：这里返回的是“猜对的总个数”，不是比例。

**评估工具 `Accumulator`**：
在验证集上跑的时候，我们是一批一批跑的。我们需要一个“累加器”来记录跑过的所有批次的总正确数和总样本数。
    - 代码中定义了一个 `Accumulator` 类，本质上就是一个列表，分别存 `[总正确数, 总样本数]`。每次跑完一个 batch，就往里加一次。最后相除得到整体精度。

## 通用计算精度

```python
def evaluate_accuracy(net, data_iter):  #@save
    """计算在指定数据集上模型的精度"""
    if isinstance(net, torch.nn.Module):
        net.eval()  # 将模型设置为评估模式
    metric = Accumulator(2)  # 正确预测数、预测总数
    with torch.no_grad():
        for X, y in data_iter:
            metric.add(accuracy(net(X), y), y.numel())
    return metric[0] / metric[1]
```
# 训练
**逻辑梳理**：
1. **单轮训练（Epoch Loop）**：定义一个函数 `train_epoch_ch3`，负责遍历一遍训练集。在每一批数据上，执行“前向传播 $\to$ 计算损失 $\to$ 反向传播求导 $\to$ 更新参数”的标准流程。
2. **全流程训练（Main Loop）**：定义主函数 `train_ch3`，负责控制训练的总轮数（`num_epochs`）。每训练完一轮，就在测试集上评估一次精度，并实时画图展示训练集损失、训练集精度和测试集精度的变化曲线。
3. **预测与可视化**：训练结束后，使用 `predict_ch3` 函数从测试集中随机抽取几张图片，对比模型的预测标签与真实标签，直观验证模型效果。
```python
def train_ch3(net, train_iter, test_iter, loss, num_epochs, updater):  #@save
    """训练模型（定义见第3章）"""
    animator = Animator(xlabel='epoch', xlim=[1, num_epochs], ylim=[0.3, 0.9],
                        legend=['train loss', 'train acc', 'test acc'])
    for epoch in range(num_epochs):
        train_metrics = train_epoch_ch3(net, train_iter, loss, updater)
        test_acc = evaluate_accuracy(net, test_iter)
        animator.add(epoch + 1, train_metrics + (test_acc,))
    train_loss, train_acc = train_metrics
    assert train_loss < 0.5, train_loss
    assert train_acc <= 1 and train_acc > 0.7, train_acc
    assert test_acc <= 1 and test_acc > 0.7, test_acc
```

# 预测

```python
def predict_ch3(net, test_iter, n=6):
    for X, y in test_iter:
        break # 只取一个 batch
    trues = d2l.get_fashion_mnist_labels(y) # 获取真实标签名称
    preds = d2l.get_fashion_mnist_labels(net(X).argmax(axis=1)) # 获取预测标签名称
    
    #以此格式显示：
    # 真实标签
    # 预测标签
    titles = [true + '\n' + pred for true, pred in zip(trues, preds)]
    d2l.show_images(X[0:n].reshape((n, 28, 28)), 1, n, titles=titles[0:n])
```

- `net(X).argmax(axis=1)`： 这里的逻辑和计算精度时一样，取概率最大的下标作为预测结果。
- `d2l.get_fashion_mnist_labels`：把数字索引（如 0）转换成文本（如 "T-shirt"）