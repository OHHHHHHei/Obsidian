# 初始化模型参数

```python
net = nn.Sequential()
net.add(nn.Dense(10))
net.initialize(init.Normal(sigma=0.01))
```
- **`nn.Sequential`**：这是一个有序的容器。数据进去后，会按顺序经过 `Flatten`，其输出作为 `Linear` 的输入。这就像自动化流水线一样。
- **`nn.Flatten()`**：
    - 作用：将输入张量的第 1 维到最后一维“拍扁”。
    - 例子：输入 `(256, 1, 28, 28)` $\to$ 输出 `(256, 784)`。它保留了第 0 维（批量大小），符合我们的预期。
    - 对比：上一节我们要手动写 `X.reshape((-1, W.shape[0]))`，现在全自动了。
- **`nn.Linear(784, 10)`**：
    - 作用：全连接层（Fully Connected Layer），对应公式 $O = XW + b$。
    - 参数：`784` 是输入特征数（上一步展平后的长度），`10` 是输出类别数。
    - **关键点**：你不需要自己创建 $W$ 和 $b$，$W$ 的形状 `(10, 784)`（注意 PyTorch 内部转置）和 $b$ 的形状 `(10)` 已经由这一层自动管理了。
```python
def init_weights(m):
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01) # 均值默认为0

net.apply(init_weights)
```
- **`net.apply(fn)`**：这是一个递归方法。它会遍历 `net` 中的每一层（包括 `Flatten` 和 `Linear`），把该层作为参数传给 `fn`（即 `init_weights`）。
- **`if type(m) == nn.Linear`**：我们只关心全连接层，`Flatten` 层没有参数，不需要初始化。
- **`nn.init.normal_`**：这是 PyTorch 提供的初始化工具。注意末尾的下划线 `_`，在 PyTorch 中表示**原地操作（In-place Operation）**，即直接修改 `m.weight` 的内存，不产生新变量。
# 重新审视Softmax的实现
Softmax 公式：
$$\hat{y}_j = \frac{\exp(o_j)}{\sum_k \exp(o_k)}$$

- 上溢风险（Overflow）：
    假设神经网络输出的 $o_k$（logits）非常大，比如 $o_k = 1000$。
    计算机计算 $e^{1000}$ 会发生什么？它会远远超出 float32 能表示的最大范围。
    结果就是 inf（无穷大）。
    如果分子分母都是 inf，那么 $\frac{\inf}{\inf}$ 就会变成 NaN（Not a Number）。模型训练直接崩溃，这就是上溢。

## **第一招：减去最大值（解决上溢）**
为了防止指数爆炸，我们可以利用 Softmax 的平移不变性。
不管我们给输入 $o$ 加上或减去什么常数 $C$，结果都不变：

$$\frac{\exp(o_j - C)}{\sum_k \exp(o_k - C)} = \frac{\exp(o_j) \cdot e^{-C}}{\sum_k (\exp(o_k) \cdot e^{-C})} = \frac{\exp(o_j) \cdot e^{-C}}{e^{-C} \cdot \sum_k \exp(o_k)} = \frac{\exp(o_j)}{\sum_k \exp(o_k)}$$
上下同时约掉了 $e^{-C}$。
我们通常取 $C = \max(o_k)$。
这样做的妙处在于：
- 指数部分变成了 $o_k - \max(o)$。
- 最大的一项变成了 $e^0 = 1$。
- 其他项都是 $e^{\text{负数}}$，其值在 $[0, 1]$ 之间。
    结果：我们成功避免了无穷大，解决了上溢问题。

## **LogSumExp 技巧（解决下溢）**
虽然解决了上溢，但如果 $o_j$ 比最大值小很多（例如小 100），那么 $\exp(o_j - \max)$ 就会变成一个极小的数（例如 $10^{-40}$）。计算机可能会因为精度不足直接把它当成 **0**。这就是**下溢**。
如果我们拿着这个 0 去算交叉熵损失：$-\log(\hat{y})$，也就是 $-\log(0)$，结果是 $-\infty$（负无穷）。这同样会导致训练崩溃。
**解决方案：不要分开算！** 
我们要计算的是 $\log(\text{softmax}(o))$。既然数学上 $\log$ 和 $\exp$ 是逆运算，能不能让它们互相抵消？
$$\begin{aligned} \log(\hat{y}_j) &= \log \left( \frac{\exp(o_j - \max)}{\sum_k \exp(o_k - \max)} \right) \\ &= \log(\exp(o_j - \max)) - \log\left( \sum_k \exp(o_k - \max) \right) \\ &= (o_j - \max) - \log\left( \sum_k \exp(o_k - \max) \right) \end{aligned}$$

**这一步推导非常关键**：
1. 利用 $\log(\frac{a}{b}) = \log(a) - \log(b)$。
2. 第一项 $\log(\exp(\dots))$ 直接抵消了！原本可能下溢的 $\exp$ 不用算了，直接变成了线性的 $(o_j - \max)$。
3. 第二项虽然还有 $\log(\sum \exp(\dots))$，但因为里面有一项是 $e^0=1$，所以求和结果肯定大于 1，取对数绝对安全，不会出现 $\log(0)$
这就是著名的 **LogSumExp** 技巧。
```python
# 直接使用 CrossEntropyLoss，它内部封装了
LogSumExp loss = nn.CrossEntropyLoss(reduction='none')
```
注意，我们在定义 `net` 时没有加 Softmax 层，`net` 输出的是 $o$ (Logits)。我们将 $o$ 直接喂给 `loss` 函数，它会在内部“悄悄”进行 Softmax 和 Log 的合并运算，既快又稳。

