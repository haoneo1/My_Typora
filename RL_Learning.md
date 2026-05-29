# **强化学习** 



## **策略梯度算法**

**给定参数化策略**  
$$
\pi_\theta(a \mid s)
$$

**目标是最大化期望折扣回报：**
$$
J(\theta)
=
\mathbb{E}_{\tau \sim \pi_\theta}
\left[
\sum_{t=0}^{T-1} \gamma^t r_t
\right]
$$

**其中轨迹**  
$$
\tau = (s_0, a_0, r_0, s_1, a_1, r_1, \dots)
$$
**由策略与环境交互采样得到。**



### **策略梯度定理（REINFORCE 核心）**

**使用 log-derivative trick：**

$$
\nabla_\theta J(\theta)
=
\mathbb{E}
\left[
\sum_{t=0}^{T-1}
\nabla_\theta \log \pi_\theta(a_t \mid s_t)
\;
G_t
\right]
$$

**其中蒙特卡洛回报：**
$$
G_t
=
\sum_{k=t}^{T-1} \gamma^{k-t} r_k
$$

**因此可构造最小化损失函数：**
$$
\mathcal{L}(\theta)
=
- \mathbb{E}
\big[
\log \pi_\theta(a_t \mid s_t)\; G_t
\big]
$$

**对该 loss 做梯度下降  
⇔  
对 \( J(\theta) \) 做梯度上升。**



**优化对象：**
$$
\theta \leftarrow \theta + \alpha \frac{1}{T}\sum_{t=0}^{T-1} A_t \nabla_\theta \log\pi_\theta(a_t|s_t)
$$

- **注意这里是一个 乘积：**

  - $$
    \nabla_\theta \log \pi_\theta(a_t \mid s_t)
    $$

    ****

    **👉 是一个 向量（维度 = 参数个数）**

  - $$
    A_t
    $$

    **👉 是一个 标量权重**


$$

$$

## **A2C算法**









## **PPO算法**

##### **GAE**

**GAE（Generalized Advantage Estimation，广义优势估计）**
 **是一种 用 TD 残差的指数加权和来估计优势函数 A(s_t,a_t) 的方法，用来在高方差（Monte Carlo）和高偏差（TD）之间做连续可调的折中。**

###  **V、Q、A、r 的含义与关系**

**在强化学习（尤其是 PPO / Actor–Critic 方法）中，r、V、Q、A 是最核心、但也最容易混淆的四个概念。下面从定义 → 数学形式 → 直觉理解 → 代码对应四个层面统一说明。**

------

####  **r** 

**—— Reward（即时奖励）**

**数学定义**
$$
r_t = r(s_t, a_t, s_{t+1})
$$
**含义**

> **环境在当前时间步给出的即时反馈**

- **来自环境，而非模型学习**
- **只反映“当前这一步”的好坏**
- **不包含对未来的直接判断**

**直觉理解**

> **“我刚刚这一步，环境给我打了多少分？”**

**CartPole 中**

- **每一步未倒：`r = 1`**
- **杆倒或越界：episode 结束**

**PPO 代码中的位置**

```
next_obs, reward, terminated, truncated, _ = env.step(action)
```

**这里的 `reward` 即 $r_t$。**

------

#### **V** 

**—— State Value Function（状态价值函数）**

**数学定义**
$$
V^\pi(s) = \mathbb{E}_\pi\!\left[\sum_{k=0}^{\infty} \gamma^k r_{t+k} \;\middle|\; s_t = s\right]
$$
**含义**

> **站在状态 s上，在当前策略 pi下，未来还能期望获得多少累计回报**

- **只与状态有关，不指定动作**
- **是一个期望值（平均意义）**
- **由 Critic 网络近似学习**

**直觉理解**

> **“这个位置整体上值不值？”**

**PPO 代码中的位置**

```
logits, value = model(obs)
```

**其中 `value` 即 V(s_t)**

------

#### **Q** 

**—— Action-Value Function（动作价值函数）**

**数学定义**
$$
Q^\pi(s,a) = \mathbb{E}_\pi\!\left[\sum_{k=0}^{\infty} \gamma^k r_{t+k} 
\;\middle|\; s_t = s,\; a_t = a\right]
$$
**含义**

> **在状态 $s$ 下，如果当前这一步选择动作 $a$，未来期望能获得多少累计回报**

- **同时依赖状态和动作**
- **粒度比 V 更细**
- **在 DQN 等方法中会被显式建模**

**直觉理解**

> **“在这个位置，走这一步值不值？”**

**在 PPO 中如何得到 Q？**

**PPO 不显式学习 Q 网络，而是使用一步 bootstrap 近似：**
$$
Q(s_t,a_t) \;\approx\; r_t + \gamma(1-d_t)V(s_{t+1})
$$




#### **证明**

**把“未来回报”拆成“第一步 + 剩余部分”**

**对无穷和做代数拆分：**
$$
\sum_{k=0}^{\infty}\gamma^k r_{t+k}
=
r_t
+
\gamma \sum_{k=0}^{\infty}\gamma^k r_{t+1+k}
$$
**这是纯代数恒等式。**

**代回 Q 的定义：**
$$
Q^\pi(s_t,a_t)
=
\mathbb{E}_\pi
\!\left[
r_t
+
\gamma \sum_{k=0}^{\infty}\gamma^k r_{t+1+k}
\;\middle|\;
s_t,a_t
\right]
$$
**现在关注后半项：**

**通过公式可以证明**
$$
\mathbb{E}_\pi
\!\left[
\sum_{k=0}^{\infty}\gamma^k r_{t+1+k}
\;\middle|\;
s_{t+1}
\right]
=
V^\pi(s_{t+1})
$$


**理论层面（严格）**
$$
Q^\pi(s,a)
=
\mathbb{E}\big[
r + \gamma V^\pi(s')
\big]
\qquad \text{（严格成立）}
$$

------

**算法层面（PPO / Actor–Critic）**

**在实际算法中：**

- **真实的 $V^\pi$ 不可得**
- **只能使用一个学习到的 Critic 网络：**

$$
V_\phi(s) \approx V^\pi(s)
$$

**于是，在样本层面使用：**
$$
Q(s_t,a_t)
\;\approx\;
r_t + \gamma V_\phi(s_{t+1})
$$










------

#### **A** 

**—— Advantage Function（优势函数）**

**数学定义**
$$
A^\pi(s,a) = Q^\pi(s,a) - V^\pi(s)
$$
**含义（PPO 的核心）**

> **在当前状态下，选这个动作比“平均动作”好多少**

- **是一个相对量**
- **$A > 0$：好于平均，应该提高概率**
- **$A < 0$：差于平均，应该降低概率**

**为什么策略梯度用 A？**

- **直接用 Q → 方差大**
- **减去 V 作为 baseline → 不改变期望梯度，但显著降低方差**

**PPO 代码中的位置**

**优势由 TD 残差 + GAE 计算：**

```
buffer.compute_gae(...)
adv_b = buffer.advantages
```

------

#### **四者之间的因果关系**

```
环境反馈：        r_t
                   ↓
Critic 估计：   V(s_t), V(s_{t+1})
                   ↓
一步 Q 近似：   Q(s_t,a_t) ≈ r_t + γ V(s_{t+1})
                   ↓
优势定义：     A(s_t,a_t) = Q - V(s_t)
```

------

#### **在 PPO 中各自的角色分工**

| **量** | **来源**        | **作用**                 |
| ------ | --------------- | ------------------------ |
| **r**  | **环境**        | **提供真实即时反馈**     |
| **V**  | **Critic 网络** | **baseline + bootstrap** |
| **Q**  | **隐式近似**    | **构造优势**             |
| **A**  | **GAE 计算**    | **Actor 更新方向**       |

------

#### **一句话终极记忆法**

> **r 是当下发生的事，
> V 是站在这里的总体期望，
> Q 是走这一步后的长期价值，
> A 是“这一步比平均好多少”。**



在**右手坐标系**下：

| 旋转轴   | 名称（工程/机器人） | 常用记号 | 含义            |
| -------- | ------------------- | -------- | --------------- |
| **X 轴** | Roll（滚转）        | $\phi$   | 左右翻滚        |
| **Y 轴** | Pitch（俯仰）       | $\theta$ | 前后抬头 / 低头 |
| **Z 轴** | Yaw（偏航）         | $\psi$   | 水平面内转向    |









# 代码





### ManagerBasedRLEnv 



传统强化学习环境里，通常会写一个很大的：

```
obs, reward, done, info = env.step(action)
```

然后所有逻辑都塞在 `step()` 里面：

```
def step(action):
    apply_action(action)
    sim.step()
    compute_observation()
    compute_reward()
    check_done()
    reset_if_needed()
    return obs, reward, done, info
```

但 Isaac Lab 的 `ManagerBasedRLEnv` 不鼓励你把所有逻辑堆在一起，而是拆成多个 manager。

你可以把它理解成：

```
ManagerBasedRLEnv
│
├── ActionManager        负责动作怎么进入机器人
├── CommandManager       负责当前任务目标是什么
├── ObservationManager   负责构造网络输入
├── RewardManager        负责计算奖励
├── TerminationManager   负责判断是否结束/重置
└── EventManager         负责随机化、初始化、扰动等事件
```

也就是说，`ManagerBasedRLEnv.step(action)` 仍然存在，但它更像一个“总控流程”，真正的细节分散在各个 manager 里。



































