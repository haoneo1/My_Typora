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

## pytorch加载模型
- JIT 路线 vs state_dict 路线，本质差异
1) TorchScript / JIT（torch.jit.load）

你拿到的是：一个可执行的 TorchScript 模型
优点：

更接近部署（图已固化，依赖更少）

某些场景推理更稳定/可移植
缺点：

不好改结构、不好插模块

训练/微调时可控性差（尤其要改 forward、加 loss、加 hook 等）

2) state_dict（torch.load + Python build）

你拿到的是：Python 模型 + 一份参数字典
优点：

最适合研究/训练/微调：能改结构、加模块、插 LoRA、加 prompt、加 adapter

易 debug：可以 print、hook 中间特征
缺点：

依赖代码实现（build_model 必须和权重匹配）

结构变化会导致 key 不匹配


## 线性代数

### 数组

#### 遍历

**遍历列表时，同时拿到“下标”和“元素”。**

和它类似的方法有几种。

------

**1. `range(len(nums))`**

这是最常见的替代写法。

```
nums = [10, 20, 30]

for i in range(len(nums)):
    print(i, nums[i])
```

输出：

```
0 10
1 20
2 30
```

这里的意思是：

- `range(len(nums))` 先产生下标 `0, 1, 2`
- 再用 `nums[i]` 取出对应元素

**和 `enumerate(nums)` 对比**

`enumerate(nums)` 写起来更自然：

```
for i, num in enumerate(nums):
    print(i, num)
```

所以一般更推荐 `enumerate()`。



### 字符串

```python
len(s)          # 长度
s[i]            # 取字符
s[a:b]          # 切片
s[::-1]         # 反转
s.lower()       # 转小写
s.upper()       # 转大写
s.split()       # 分割
" ".join(lst)   # 拼接
s.strip()       # 去空白
s.replace(a,b)  # 替换
x in s          # 判断是否包含
s.count(x)      # 统计次数
s.startswith()  # 判断开头
s.endswith()    # 判断结尾
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





## 小的知识点

### 列表和数组区别

**列表**

更灵活。

- 可以放不同类型的数据
- 长度通常更容易动态变化
- 用起来更方便

例如：

```
a = [1, "hello", 3.14, True]
```

------

**数组**

更规整。

- 通常要求元素类型一致
- 更适合数值计算
- 往往内存更紧凑，效率更高

例如：

```
[1, 2, 3, 4, 5]
```



### 运算符

```python
# 算术
+  -  *  /  //  %  **
//是整除 向下取整 38 // 10 = 3

# 比较
==  !=  >  <  >=  <=

# 赋值
=  +=  -=  *=  /=  //=  %=  **=  ^=

# 逻辑
and  or  not

# 位运算
&  |  ^  ~  <<  >>

# 成员
in  not in

# 身份
is  is not
```





























