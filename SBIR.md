### **LeJEPA** 

**2.26：**接下来我想采取两阶段的训练方法

两阶段训练（让 JEPA 发挥“自监督预训练”的本职）

**Stage 1：Photo-only JEPA + SIGReg（预训练）** 我觉得与训练的方法是有问题的，在VIEW过程中要有改变，是图片变得想sketch才行

- encoder：**同一套 CLIP visual**
- 训练参数建议：`img_prompt + LN(+ predictor)`，先别放开更多 backbone
- 数据：Sketchy photo +（最好）额外无标签 photo
- 产出：保存 `clip视觉参数(或LN)+img_prompt`

**Stage 2：Triplet 微调（ZS-SBIR）**

- 加载 Stage1 权重
- `sk_prompt` 初始化为：
  - 方案1：随机
  - 方案2（更稳）：`sk_prompt = img_prompt.clone()`
- 只用 triplet/contrastive 做检索优化
- 监控 best mAP early stop



**你真正应该做的，是“控制变量实验矩阵”**

如果你要证明 JEPA 有用，我建议你至少做下面 4 组：

------

实验 1：Baseline

- 不做 JEPA
- 直接用 base classes 做 triplet 微调
- 在 novel classes 上测 mAP

这是一切比较的基准。

------

实验 2：In-domain JEPA

- 只用 **base classes 的 photo** 做 photo-only JEPA 预训练
- 再用 base classes 做 triplet 微调
- 在 novel classes 上测

这个实验回答的是：

> **不用额外数据，只利用原始训练集内部的无标签结构，JEPA 有没有帮助？**

这是你现在最应该先做的。

------

实验 3：Extra-data JEPA

- 用额外数据集 photo 做 JEPA 预训练
- 再用原始 base classes 做 triplet 微调
- 在 novel classes 上测

这个实验回答的是：

> **加入额外数据后，整体 pipeline 能不能更强？**

但它不能单独证明 JEPA 方法本身有多强，因为混入了数据规模因素。

------

实验 4：Data-scale control

这是最重要但最容易被忽视的一组。

如果你用了额外数据集做 JEPA 预训练，最好还要有一个“数据规模对照”：

例如：

- 用相同数量的额外数据
- 但不用 JEPA，换成：
  - 普通 continued pretraining
  - MAE / SimCLR / BYOL 之类另一种 SSL
  - 或者至少做一个“只训练 image prompt/LN，不加 JEPA predictor”的对照

这样你才能回答：

> “提升到底来自额外数据本身，还是来自 JEPA 这种预训练方式？”





### **LeJEPA** 训练记录

```
CUDA_VISIBLE_DEVICES=1 python -m experiments.train_pretrain
CUDA_VISIBLE_DEVICES=1 python -m experiments.train_finetune
```

### ZS_SBIR Baseline

**Triplet**

```
最高：0.626  种子是0.639
```

**triplet +class**

```
0.685
```

baseline既然这么高就不应该出现这个问题，我得想办法改一下



### FG_SBIR baseline

**triplet**

ChairV2 目前原始版本的最佳结果：

```
mAP=0.633732, rank1=0.479669, top10=0.919614
```

ShoeV2 目前原始版本的最佳结果：

```
mAP=0.369188, rank1=0.220721, top10=0.704198
```

**triplet +class**

ChairV2 目前原始版本的最佳结果：

```
mAP=0.675732, rank1=0.543469, top10=0.938914
```

ShoeV2 目前原始版本的最佳结果：

```
mAP=0.445188, rank1=0.297721, top10=0.752198
```



#### 1、初步尝试

```bash
CUDA_VISIBLE_DEVICES=0 python -m experiments.train_finetune
CUDA_VISIBLE_DEVICES=0 python -m experiments.train_pretrain

```

**LN_prompt/version_10：** 最初始冲github拉下来的数据

**LN_prompt/version_13：**使用最开始的lejepa的数据

**LN_prompt/version_16**：使用种子和warm-up之后的数据目前baseline，但是还是有很多问题的！

```python
self.jepa_warmup_start_epoch= int(getattr(self.opts,"jepa_warmup_start_epoch",3)
self.jepa_warmup_ramp_epochs = int(getattr(self.opts, "jepa_warmup_ramp_epochs", 3))

# 可选：SIGReg 也支持单独延迟（默认如果lambda_sigreg=0就不生效
self.sigreg_warmup_start_epoch = int(getattr(self.opts, "sigreg_warmup_start_epoch", 6))
self.sigreg_warmup_ramp_epochs = int(getattr(self.opts, "sigreg_warmup_ramp_epochs", 3))
```

#### 2、SSL_photo_jepa

**SSL_same_class/version_0：**使用分阶段的相同数据集的photo-only-jepa  +  只有triplet的fine-tune

```
最高：0.7011
```

**SSL_same_class_finetune/version_1** 使用分阶段的相同数据集的photo-only-jepa +（triplet+text-class）

**SSL_same_class_finetune/version_3**是上面的续训没有经过任何的超参数修改

```
最高：0.677
```

#### 3、SSL_ph_jepa+reg

**使用相同数据集的photo-(jepa+REG)_(triplet+text-class）**

```
最高：0.667
```

**使用相同数据集的photo-(jepa+REG)_(triplet）**

```
最高：0.695
```



#### 4、SSL_ph+sk_jepa

使用相同数据集的ph+sk-(jepa)_(triplet+class）

```
0.688
```



使用相同数据集的ph+sk-(jepa)_(triplet）

```
0.703
```

使用224bitch-size相同数据集的ph+sk-(jepa_reg)_(triplet）

```
0.698
```

使用相同数据集的ph+sk-(jepa_reg)_(triplet）

```
0.699
```

#### **5、SSL_tri-ph+tri-sk_jepa**

目的是恢复之前自己写的那个， 所以自己还使用用ph+sk的预训练的，但是在fine-tune的时候要加上ph和sk内部的treiplet损失

```
0.672
```

#### 6、SSL_cross-ph+sk_dino+jepa

使用思路2的相互约束的方法

```
0.687
```

#### 7、SSL_cross_jepa_sk+ph





```
/home/bingxing2/home/scx9951/SBIR_Data/saved_models
```

```7、

```

#### 6、尝试硬样本

#### 7、尝试方案一下面的

#### 8、清洗代码







不确定性

基于clip的蒸馏减少参数量

针对FG-SBIR任务   数据：Sketch Me That Shoe

用**强化学习**来做一下非可微的工作，

计算效率



半监督方式的预处理



基于图片生成草图, 有一篇文章在做这个 但是只局限在chair和shoe像个数据集

能不能作伪草图和真实草图之间的度量学习



距离加强可以放到老师那里：



全局的一个rl决定：

图片使用灰度行不行



用RL决定蒸馏的层数，多老师的选择，多学生的选择，



# 思路2

## 1. 预训练阶段

用两类自监督约束去学 sketch-photo 的共享表征：

### （1）DINO-like 约束

负责**全局对齐**
 也就是让配对的 sketch 和 photo 在整体语义上靠近。

你可以把它理解成：

- sketch 的全局表示去对齐对应 photo 的全局表示
- 或者双向对齐：sketch→photo，photo→sketch

它解决的是：
 **“这张 sketch 和这张 photo 本质上是不是同一个对象/类别/语义实例”**

------

### （2）JEPA-like 约束

负责**局部预测 / 结构对应**
 也就是不是只看全图，而是让一部分上下文去预测另一部分的潜表示。

它解决的是：
 **“sketch 里的局部线条结构，能不能和 photo 里的局部语义区域建立对应关系”**

这一步比单纯 DINO 更像是在学：

- 局部结构
- 部件关系
- 稀疏 sketch 和真实 photo 之间的细粒度对应

------

## 2. 微调阶段

再用 **Triplet Loss** 做检索任务收尾。

### （3）Triplet Loss

负责**检索判别性**
 即：

- 正样本 sketch-photo 拉近
- 负样本拉远

它解决的是：
 **“最后检索排序要正确”**

因为前面的 DINO-like 和 JEPA-like 更像是在学表示，
 但真正做 SBIR 检索时，你还需要一个明确的 retrieval objective。







## 如果你想在类别级 ZS-SBIR 里用，我建议这样改

### 方案 1：只对 hard-negative 生效

不要用所有异类负样本来算 $\delta$，而是只用**语义接近**或**视觉上容易混淆**的负类。

例如：

- dog vs wolf
- chair vs bench
- cup vs mug

这样 $L_\delta$ 约束的是“困难类别边界”的稳定性，而不是把所有类别都硬拉成一样。

可以写成：
$$
\delta_c^{hard}=d(s,p^+)-d(s,p^-_{hard})
$$
然后只对这些 hard-negative 的类别分布做 KL 对齐。

这个版本比原始版更适合 category-level。

### 方案 2：不对齐整条分布，只对齐均值和方差

原论文是对齐类别间 $\delta$ 分布的 KL。
 在类别级任务里，你可以改成更温和的版本，比如只约束：
$$
\mu_c = \mathbb{E}[\delta_c], \qquad \sigma_c^2 = \mathrm{Var}(\delta_c)
$$
再最小化类别间的均值/方差差异，而不是整条分布。
 这样不容易把类别特有的几何关系压扁。

------

### 方案 3：做成 semantic-aware 的 $L_\delta$

不要要求所有类别都共享同一个“稳定 margin”，而是让**语义相近类别共享相近分布**。

比如按 WordNet / CLIP text embedding / 类别语义树分组：

- 动物一组
- 交通工具一组
- 家具一组

然后只在组内做 $\delta$ 对齐。

## RL实现了什么：

不同草图的“最优像素尺寸分布”

# FG-SBIR

ChairV2 目前原始版本的最佳结果：

```
mAP=0.633732, rank1=0.479669, top10=0.919614
```

ShoeV2 目前原始版本的最佳结果：

```
mAP=0.369188, rank1=0.220721, top10=0.704198
```



# ZS_FG_SBIR

