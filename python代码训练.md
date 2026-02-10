# python代码训练

## Checkpoint



### 保存

**pytorch.Lightning 的方法**

```python
    checkpoint_callback = ModelCheckpoint(
        monitor="val_loss", #监控指标名称，以这个来判断checkpoint好坏
        dirpath=ckpt_dir,
        filename="epoch{epoch:02d}-val_loss{val_loss:.4f}",
        mode="min", #越小越好
        save_last=True,
        save_top_k=1,
        auto_insert_metric_name=False,
    )	
```

在训练过程中，Lightning 会周期性地跑验证（val），并把你记录的指标（metric）汇总成一个字典。
 `ModelCheckpoint` 会从这些指标里取出 `val_loss`，然后决定是否保存 checkpoint。

你这份配置会保存两类 checkpoint（非常常用的组合）：

- **last checkpoint**：永远保存“最新状态”（方便断点续训）
- **best checkpoint**：保存“验证集 val_loss 最低”的那一次（用于最终测试/部署）



## PyTorch Lightning



### 主训练循环（重复到 max_epochs 或你手动停止）

每个 epoch：

**训练阶段**

- `on_train_epoch_start()`
- 对每个 batch：
  - `training_step(batch, batch_idx)`
  - Lightning 自动执行 backward + optimizer.step + zero_grad
- `on_train_epoch_end()`

**验证阶段**（你设置 `check_val_every_n_epoch=1`，每个 epoch 都验证）

- `on_validation_epoch_start()`
- 对每个 val batch：
  - `validation_step(batch, batch_idx)`
- `on_validation_epoch_end()`

**回调阶段（checkpoint 等）**

- `ModelCheckpoint` 在验证结束后读取你 log 的 `val_loss`，保存：
  - `last.ckpt`（始终最新）
  - best ckpt（`val_loss` 最低的一次，`save_top_k=1`）



## 线性代数

### 数组

#### numpy和python

##### **创建**

**1）对象本质不同**

**纯 Python（list）**

- `a = [1,2,...,10]` 是 **一维序列**
- `row = [[1,2,...,10]]` 是 **嵌套序列**（用来“模拟”1 行 10 列）
- **没有 shape / ndim / 矩阵运算语义**，只有 `len()` 和索引

**NumPy（ndarray）**

- `x = np.array([...])` / `reshape(1,10)` 得到的是 **真正的数组**
- 有严格的 **维度**：`shape`、`ndim`
- 支持矢量化、广播、矩阵乘法等数值运算语义

**2）“1 行 10 列”在两者中的表达差异**

**纯 Python**

```
row = [[1,2,3,4,5,6,7,8,9,10]]
len(row)    # 1  (行)
len(row[0]) # 10 (列)
```

这是“结构上像 1×10”，但它只是 list 嵌套。

**NumPy**

```
x = np.arange(1,11).reshape(1,10)
x.shape  # (1, 10)
x.ndim   # 2
```

这是数学意义上**真正的 1×10 矩阵/行向量**。



**np.arange 总结**

**基本形式：**

```
np.arange([start,] stop[, step], dtype=None)
```

**含义：**

- `start`：起始值（默认 0）
- `stop`：终止值（**不包含 stop**）
- `step`：步长（默认 1，可为负数）
- `dtype`：指定数组的数据类型（可选）

```
[start, stop)
```

- 即：**左闭右开**

------

✅ 常见用法

```
np.arange(5)
# [0 1 2 3 4]

np.arange(2, 7)
# [2 3 4 5 6]

np.arange(0, 10, 2)
# [0 2 4 6 8]
```

------

✅ 支持负步长（倒序）

```
np.arange(10, 0, -2)
# [10 8 6 4 2]
```

注意：
 **终点 stop 永远不包含。**

------

✅ 可指定数据类型

```
np.arange(5, dtype=np.float32)
```

------

✅ 一句话记住

> `np.arange` 生成一个等差序列数组，区间为 **[start, stop)**，步长为 `step`，支持负步长和指定类型。







##### 操作

**Python list 与 NumPy ndarray 常用数组操作对照表（简化版）**



| 操作语义                   | Python list 写法   | NumPy ndarray 写法                                      |
| :------------------------- | ------------------ | ------------------------------------------------------- |
| 修改指定元素               | `a[1] = 99`        | `a[1] = 99`                                             |
| 末尾添加一个元素           | `a.append(4)`      | `a = np.append(a, 4)`                                   |
| 末尾添加多个元素           | `a.extend([5, 6])` | `a = np.append(a, [5, 6])`                              |
| 指定位置插入元素           | `a.insert(i, x)`   | `a = np.insert(a, i, x)`                                |
| 删除数组并返回最后一个元素 | `x = a.pop()`      | `x = a[-1]`；`a = a[:-1]`                               |
| 删除数组并返回指定位置元素 | `x = a.pop(i)`     | `x = a[i]`；`a = np.delete(a, i)`                       |
| 按值删除第一个匹配元素     | `a.remove(x)`      | `idx = np.where(a == x)[0][0]`；`a = np.delete(a, idx)` |
| 按值删除所有匹配元素       | （需自己写循环）   | `a = a[a != x]`                                         |
| 按索引删除元素             | `del a[i]`         | `a = np.delete(a, i)`                                   |
|                            |                    | np.linspace(start, stop, num)                           |



| 函数                   | 作用     | 典型用法                                 |
| ---------------------- | -------- | ---------------------------------------- |
| `np.linspace(a, b, n)` | 区间等分 | `np.linspace(0, 1, 100)`  从0到1 分100份 |
| `np.zeros(shape)`      | 全 0     | `np.zeros((3,4))` **两个括号**           |
| `np.ones(shape)`       | 全 1     | `np.ones((3,4))`                         |



**np.random**

| 函数                                           | 典型用法                       | 说明                                          |
| ---------------------------------------------- | ------------------------------ | --------------------------------------------- |
| `np.random.seed(seed)`                         | `np.random.seed(42)`           | 设置全局随机种子，保证结果可复现              |
| `np.random.randint(low, high=None, size=None)` | `np.random.randint(0, 10, 5)`  | 生成区间 `[low, high)` 的5个随机整数          |
| `np.random.rand(d0, d1, ...)`                  | `np.random.rand(2, 3)`         | 生成 2 行 3 列的、服从 `[0,1)` 的随机浮点数组 |
| `np.random.randn(d0, d1, ...)`                 | `np.random.randn(5)`           | 生成服从正态 `N(0,1)` 的随机数                |
| `np.random.choice(a, size=None, replace=True)` | `np.random.choice([1,2,3], 2)` | 从数组/列表中随机选2个元素                    |
| `np.random.shuffle(a)`                         | `np.random.shuffle(a)`         | 直接在原数组上打乱顺序                        |



##### **布尔索引**

定义一维 NumPy 数组 `numpy.array([1, 2, 3, 4, 5, 6])`，分别取出其中
 👉 奇数
 👉 偶数

直接用**布尔索引**最标准。

```
import numpy as np

a = np.array([1, 2, 3, 4, 5, 6])

odd  = a[a % 2 == 1]   # 奇数
even = a[a % 2 == 0]   # 偶数

print("odd:", odd)
print("even:", even)

```

能使用布尔索引，是因为
 NumPy 的算术/比较运算是逐元素向量化的，
 并且返回一个同形状的布尔数组，
 这个布尔数组可以作为索引使用。

**对比一下 Python list 会发生什么？**

```
a = [1, 2, 3, 4, 5, 6]

a % 2
```

直接报错：

```
TypeError
```

因为：

- list 没有定义逐元素运算

你只能：

```
[x % 2 for x in a]

#[1, 0, 1, 0, 1]
```

##### **转置**

```
import numpy as np

a = np.array([1, 2, 3, 4, 5, 6])

a_T = a.T
print(a_T)
print(a.shape, a_T.shape)

```

### 矩阵

先准备：

```python
import numpy as np

A = np.array([[1, 2, 3],
              [4, 5, 6]])
# A.shape == (2, 3)
```

------

**1) `numpy.flatten()`**

**作用**：把数组“拉平”为 **一维数组**（返回的是副本 copy）。

```python
A_flat = A.flatten()
# [1 2 3 4 5 6]
```

常见点：

- 一定返回 **1D**
- 通常会复制数据（改 `A_flat` 不影响 `A`）

------

**2) numpy.column_stack()**

**作用**：按“列”把一维数组拼成二维，或把二维数组按列拼接（本质沿 axis=1）。

示例（最典型：把多个 1D 变成矩阵的列）：

```python
x = np.array([1, 2, 3])
y = np.array([10, 20, 30])

C = np.column_stack([x, y])
# [[ 1 10]
#  [ 2 20]
#  [ 3 30]]
```

要点：

- 传入的 1D 会被当成 **列向量**。

------

**3) `numpy.row_stack()`**  

**作用**：按“行”堆叠（沿 axis=0），把 1D 变成一行，再往下堆。

```
x = np.array([1, 2, 3])
y = np.array([10, 20, 30])

R = np.row_stack([x, y])
# [[ 1  2  3]
#  [10 20 30]]
```

要点：

- 传入 1D 会被当成 **行向量**。

------

**4) `numpy.flip()`**

**作用**：沿指定轴翻转（反转顺序）。最通用。

```
np.flip(A)          # 不写 axis：把所有轴都翻转（等价于先上下再左右）
np.flip(A, axis=0)  # 上下翻（行顺序反转）
np.flip(A, axis=1)  # 左右翻（列顺序反转）
```

以 `A` 为例：

- `np.flip(A, axis=0)`：

```
[[4 5 6]
 [1 2 3]]
```

- `np.flip(A, axis=1)`：

```
[[3 2 1]
 [6 5 4]]
```

------

**5) `numpy.fliplr()`**

**作用**：只做 **左右翻转**（flip left-right），等价于 `np.flip(A, axis=1)`，要求数组至少 2D。

```
np.fliplr(A)
# [[3 2 1]
#  [6 5 4]]
```

------

**6) `numpy.flipud()`**

**作用**：只做 **上下翻转**（flip up-down），等价于 `np.flip(A, axis=0)`，要求数组至少 2D。

```
np.flipud(A)
# [[4 5 6]
#  [1 2 3]]
```

------

**7) `numpy.reshape()**`

**作用**：改变数组形状（shape），在元素总数不变的前提下重排。

```
B = np.arange(6)      # [0 1 2 3 4 5]
B2 = B.reshape(2, 3)  # 变成 2x3
```

结果：

```
[[0 1 2]
 [3 4 5]]
```

常用技巧：

- `-1` 表示“自动推断该维度”

```
B.reshape(-1, 3)  # 自动算行数
B.reshape(2, -1)  # 自动算列数
```

------

**一张速查表（记忆用）**

| 函数             | 核心作用     | 常见等价/备注              |
| ---------------- | ------------ | -------------------------- |
| `flatten()`      | 拉平到 1D    | 通常返回副本               |
| `column_stack()` | 按列拼接     | 1D 会当列向量              |
| `row_stack()`    | 按行堆叠     | 等价 `vstack`，1D 当行向量 |
| `flip()`         | 沿指定轴翻转 | 最通用：axis=0/1/...       |
| `fliplr()`       | 左右翻转     | 等价 `flip(axis=1)`        |
| `flipud()`       | 上下翻转     | 等价 `flip(axis=0)`        |
| `reshape()`      | 改形状       | 元素数必须一致，支持 `-1`  |

