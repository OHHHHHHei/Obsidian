# 查看GPU

首先，我们需要知道怎么在代码里“称呼”我们的硬件。
- **查看显卡信息**： 在终端输入 `nvidia-smi`（Notebook 中加感叹号）。
    - **关注点**：`Memory-Usage`（显存占用），`GPU-Util`（GPU 利用率）。如果显存满了，你的程序就会报 OOM (Out Of Memory) 错误。
- **代码中的设备表示（以 PyTorch 为主）**：
    - `torch.device('cpu')`：代表 CPU。
    - `torch.device('cuda')` 或 `torch.device('cuda:0')`：代表第 0 块 GPU。
    - `torch.device('cuda:1')`：代表第 1 块 GPU。
    - **查询可用 GPU 数量**：`torch.cuda.device_count()`。
- **实用的封装函数**： 课件中定义了 `try_gpu(i)`。这在工程中很常见，用来写出**可移植**的代码：如果机器有 GPU 就用 GPU，没有就自动退化到 CPU，防止代码报错。
```Python
def try_gpu(i=0):
	if torch.cuda.device_count() >= i + 1:
		return torch.device(f'cuda:{i}')
	return torch.device('cpu')
```
# 张量与 GPU

## **1. 张量的存储设备**

默认情况下，你创建一个张量 `x = torch.tensor([1, 2, 3])`，它是存在 **CPU 内存**里的。 你可以通过 `x.device` 属性来查看它在哪里。
## **2. 在 GPU 上创建张量**

- **方式一（创建时指定）**：
```Python
X = torch.ones(2, 3, device=try_gpu()) 
# X 就直接生在显存里了
```
- **方式二（事后移动，重点！）**： 在 PyTorch 中，最常用的方法是 `.to()`。
```Python
Z = X.to(device) 
# 如果 X 在 CPU，Z 就会在 GPU；如果 X 已经在该 GPU，Z 就是 X 本身（不复制，零开销）。
```
## **3. 跨设备运算的禁忌**

这是初学者最容易踩的坑：

```Python
# 假设 X 在 GPU0，Y 在 CPU 或 GPU1
Z = X + Y  # 报错！
```

- **原因**：CPU 和 GPU 是物理隔离的，内存不互通。没办法直接把 CPU 里的数和 GPU 里的数相加。
    
- **解决**：必须先把 `Y` 搬运到 `X` 所在的设备：`Y = Y.to(X.device)`，然后再相加。
    

**4. 旁注：性能开销**

- **数据传输是昂贵的**：把数据从 CPU 内存拷贝到 GPU 显存（通过 PCIe 总线）非常慢。
- **最佳实践**：不要频繁地在 CPU 和 GPU 之间来回倒腾数据。通常的做法是：**先把一批数据（Batch）一次性发给 GPU，在 GPU 内部做完几十层网络的复杂运算，最后只把那个小小的 Loss 值或者预测结果传回 CPU**。
# 神经网络与 GPU

不仅数据要在 GPU 上，负责运算的“工人”——神经网络模型，也得去 GPU 上上班。
```Python
net = nn.Sequential(nn.Linear(3, 1))
net = net.to(device=try_gpu()) # 将模型的所有参数（权重和偏置）移动到 GPU
```
- **验证**：你可以检查 `net[0].weight.device`，会发现它变成了 `cuda:0`。
- **前向传播**：
```Python
net(X) # 这里的 X 必须也在同一个 GPU 上
```