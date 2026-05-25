# 面向 Dex / 人形机器人 Locomotion 的论文阅读表

## 0. 阅读目标

本文档整理近两年顶会和顶刊中与机器人 locomotion，尤其是 **humanoid locomotion / Dex 项目**高度相关的论文。阅读目标不是泛泛了解 locomotion，而是帮助你判断 Dex 后续应该从哪些方向切入：

- PPO / RL locomotion baseline
- history-based adaptation
- Transformer / sequence modeling
- world model for locomotion
- rough terrain locomotion
- sim-to-real / sim-to-sim alignment
- whole-body control
- human motion imitation
- embodied locomotion and contextual skills

---

## 1. 总体阅读策略

建议采用 **4 周阅读法**，每周聚焦一个主题：

```text
第 1 周：humanoid RL locomotion baseline
第 2 周：复杂地形与鲁棒 locomotion
第 3 周：sim-to-real 与 whole-body control
第 4 周：人类数据、模仿学习与 embodied locomotion
```

如果时间有限，优先精读以下 5 篇：

1. **Real-world Humanoid Locomotion with Reinforcement Learning**
2. **Advancing Humanoid Locomotion with Denoising World Model Learning**
3. **ASAP: Aligning Simulation and Real-World Physics for Learning Agile Humanoid Whole-Body Skills**
4. **HugWBC: A Unified and General Humanoid Whole-Body Controller for Fine-Grained Locomotion**
5. **Humanoid Locomotion as Next Token Prediction**

---

# 2. 四周论文阅读表

## 第 1 周：建立 humanoid RL locomotion baseline

### 论文 1：Real-world Humanoid Locomotion with Reinforcement Learning

| 项目 | 内容 |
|---|---|
| 年份/来源 | Science Robotics 2024 |
| 方向 | 真实人形机器人 RL locomotion |
| 核心关键词 | reinforcement learning, humanoid locomotion, observation history, action history, Transformer policy, sim-to-real |
| 重点阅读 | observation history、action history、policy structure、domain randomization、真实机器人部署 |
| 阅读问题 | 为什么它不用很复杂的模型，也能让真实 humanoid 稳定行走？history 到底提供了什么信息？ |
| 对 Dex 的启发 | Dex 的 policy 不应该只依赖当前 observation，可以重点研究 observation history 和 action history 对鲁棒性的作用。 |

#### 你需要重点理解

这篇论文是现代 humanoid RL locomotion 的基础论文之一。它说明，对于真实人形机器人，**历史状态序列**本身能够隐式编码地面属性、身体扰动、负载变化和动力学不确定性。

对 Dex 来说，最值得借鉴的是：

- 是否在 observation 中加入历史 proprioception；
- 是否加入 action history；
- 是否用 Transformer / RNN 替代简单 MLP；
- 是否通过 domain randomization 提高 sim-to-real 鲁棒性；
- policy 输出是 position target、velocity target，还是 torque/action correction。

---

### 论文 2：Humanoid Locomotion as Next Token Prediction

| 项目 | 内容 |
|---|---|
| 年份/来源 | NeurIPS 2024 |
| 方向 | Transformer / 序列建模式 humanoid control |
| 核心关键词 | next-token prediction, causal Transformer, sensorimotor sequence, autoregressive policy |
| 重点阅读 | 如何把 locomotion 建模为 sensorimotor token 的序列预测问题 |
| 阅读问题 | locomotion 能不能被看成“动作序列预测”？这和 world model / JEPA 有什么关系？ |
| 对 Dex 的启发 | 可以把 Dex 的历史 observation-action 序列作为输入，用序列预测思想建模动作生成。 |

#### 你需要重点理解

这篇文章的思想是：**机器人 locomotion 可以被看成类似语言模型的 next-token prediction 问题**。

传统 RL policy 往往是：

```text
当前 observation -> 当前 action
```

而 sequence modeling 的思想是：

```text
过去 observation-action 序列 -> 下一个 action / 下一个 sensorimotor token
```

这对 Dex 的启发是：如果机器人在复杂地形或者扰动下不稳定，问题可能不是单个 reward 项没调好，而是 policy 缺少对历史动态的建模能力。

---

## 第 2 周：复杂地形与鲁棒 locomotion

### 论文 3：Advancing Humanoid Locomotion: Mastering Challenging Terrains with Denoising World Model Learning

| 项目 | 内容 |
|---|---|
| 简称 | DWL |
| 年份/来源 | RSS 2024 |
| 方向 | 世界模型 + 复杂地形 humanoid locomotion |
| 核心关键词 | denoising world model, challenging terrains, rough terrain, zero-shot sim-to-real |
| 重点阅读 | denoising world model 学了什么、如何帮助复杂地形泛化 |
| 阅读问题 | 它的 world model 到底预测什么？为什么 denoising 对真实复杂地形有效？ |
| 对 Dex 的启发 | 如果 Dex 后续要走 gravel、slope、stairs、rough terrain，可以考虑 world model 或 terrain-aware policy。 |

#### 你需要重点理解

DWL 的核心不只是让 humanoid 在平地稳定走，而是让同一个 policy 面对复杂地形仍然能够泛化。它关注的问题包括：

- 雪地；
- 斜坡；
- 楼梯；
- 不平整地面；
- sim-to-real zero-shot transfer。

对 Dex 来说，这篇适合作为 **world model + locomotion** 的主要参考。你可以重点思考：

```text
Dex 是否需要显式地形输入？
Dex 是否需要 height scanner / depth / elevation map？
Dex 是否需要从历史状态中隐式估计地形？
Dex 的 terrain randomization 是否足够？
```

---

### 论文 4：BeamDojo: Learning Agile Humanoid Locomotion on Sparse Footholds

| 项目 | 内容 |
|---|---|
| 年份/来源 | RSS 2025 |
| 方向 | 稀疏落脚点 / 精准 foothold / agile humanoid locomotion |
| 核心关键词 | sparse footholds, stepping stones, balancing beam, foothold precision, LiDAR elevation map |
| 重点阅读 | foothold reward、two-stage RL、double critic、terrain perception |
| 阅读问题 | 如果 Dex 要走离散落脚点，reward 和 terrain perception 应该怎么设计？ |
| 对 Dex 的启发 | 复杂地形不能只靠速度跟踪 reward，还需要精确落脚、足端规划和地形感知。 |

#### 你需要重点理解

BeamDojo 的关键问题是：机器人不是在连续平面上随便落脚，而是必须踩到有限区域，例如：

- stepping stones；
- narrow beam；
- sparse footholds；
- irregular foothold regions。

这类任务对 Dex 的启发很直接：如果你未来要做复杂地形，不能只调：

```text
track_lin_vel_xy_yaw_frame_exp
track_ang_vel_z_exp
feet_slide
foot_clearance
```

还需要引入：

- foothold precision reward；
- foot placement constraint；
- terrain height map；
- contact timing；
- swing trajectory control；
- perception-conditioned policy。

---

## 第 3 周：sim-to-real 与 whole-body control

### 论文 5：ASAP: Aligning Simulation and Real-World Physics for Learning Agile Humanoid Whole-Body Skills

| 项目 | 内容 |
|---|---|
| 年份/来源 | RSS 2025 |
| 方向 | sim-to-real / 真实物理对齐 / agile whole-body skills |
| 核心关键词 | simulation-to-real, residual action model, delta action, motion tracking, real-world data |
| 重点阅读 | 两阶段训练、真实数据校正、residual action model |
| 阅读问题 | 如果 Dex 仿真能走但真实/Sim2Sim 不稳，除了 domain randomization 还能怎么做？ |
| 对 Dex 的启发 | 可以考虑学习 residual action correction，而不是只靠 domain randomization。 |

#### 你需要重点理解

ASAP 的关键思想是：不要只依赖仿真随机化，而是让真实机器人数据反过来修正仿真和真实之间的动力学差异。

传统 sim-to-real 常用路线：

```text
仿真训练 policy
↓
domain randomization
↓
真实部署
```

ASAP 更进一步：

```text
仿真中训练 motion tracking policy
↓
真实机器人收集数据
↓
学习 residual / delta action model
↓
补偿仿真与真实动力学差异
↓
真实机器人更稳定执行 agile whole-body skills
```

对 Dex 来说，如果你发现：

```text
Isaac Lab 训练中走得很好
↓
换仿真器 / 换真实机器人后不稳定
```

那么可以考虑 ASAP 这类 residual alignment 思路。

---

### 论文 6：HugWBC: A Unified and General Humanoid Whole-Body Controller for Fine-Grained Locomotion

| 项目 | 内容 |
|---|---|
| 年份/来源 | RSS 2025 |
| 方向 | unified whole-body control / fine-grained locomotion |
| 核心关键词 | whole-body control, walking, running, jumping, hopping, waist rotation, body pitch, foot height |
| 重点阅读 | command space、全身姿态控制、多 gait 统一 policy |
| 阅读问题 | 如何把 Dex 从“只会走”扩展成“可调步态 + 全身协调”？ |
| 对 Dex 的启发 | 你的 waist、arm、shoulder posture reward 可以上升为 whole-body locomotion 的研究问题。 |

#### 你需要重点理解

HugWBC 的价值在于：它不是只让机器人跟踪 x/y/yaw 速度，而是设计了更丰富的 command space，使同一个 policy 支持多种行为：

- walking；
- running；
- jumping；
- hopping；
- standing；
- body height adjustment；
- waist rotation；
- body pitch control；
- foot swing height control；
- upper-body intervention。

这对 Dex 特别重要，因为你之前调过：

- `target_shoulder_pitch_rad`；
- `waist_back_lean`；
- stand-still reward；
- feet slide penalty；
- foot air/contact time；
- gait period；
- upper-body posture。

这些不只是工程调参，而可以被总结成一个更高级的问题：

> 上半身姿态、腰部控制和手臂摆动如何影响 humanoid locomotion 的稳定性、速度跟踪与复杂地形泛化？

---

## 第 4 周：人类数据、模仿学习与 embodied locomotion

### 论文 7：HumanPlus: Humanoid Shadowing and Imitation from Humans

| 项目 | 内容 |
|---|---|
| 年份/来源 | CoRL 2024 |
| 方向 | 人类动作数据到人形机器人控制 |
| 核心关键词 | human motion data, humanoid shadowing, RGB camera, imitation learning, whole-body skill |
| 重点阅读 | human motion retargeting、low-level policy、shadowing、真实示教数据 |
| 阅读问题 | 人类动作数据如何转成 humanoid policy？ |
| 对 Dex 的启发 | 如果 Dex 要做人形 imitation，可以从 mocap / AMASS / human motion retargeting 入手。 |

#### 你需要重点理解

HumanPlus 说明 humanoid locomotion 可以借助大规模人类动作数据，而不是完全靠人工 reward。它的核心流程可以理解为：

```text
人类动作数据
↓
retarget 到 humanoid 机器人
↓
仿真中训练 low-level tracking policy
↓
RGB camera 进行 humanoid shadowing
↓
真实机器人学习和执行 whole-body skills
```

对 Dex 来说，这篇论文适合在你准备从 locomotion 走向 imitation 或 whole-body skill 时阅读。

---

### 论文 8：VideoMimic: Visual Imitation Enables Contextual Humanoid Control

| 项目 | 内容 |
|---|---|
| 年份/来源 | CoRL 2025 |
| 方向 | 单目视频模仿 / contextual humanoid control |
| 核心关键词 | monocular video, real-to-sim-to-real, visual imitation, contextual control, whole-body skill |
| 重点阅读 | 从视频重建人体和环境、上下文动作学习、real-to-sim-to-real pipeline |
| 阅读问题 | locomotion 如何从速度跟踪变成“看环境做动作”？ |
| 对 Dex 的启发 | 这是把 locomotion 推向 embodied intelligence 的重要方向。 |

#### 你需要重点理解

VideoMimic 的重点不是单纯让机器人走，而是让机器人根据环境上下文完成动作，例如：

- 上楼梯；
- 坐到椅子上；
- 从椅子上站起来；
- 在具体环境中模仿人类动作。

这说明 locomotion 的研究正在从：

```text
给定速度 command -> 机器人跟踪速度
```

转向：

```text
给定视觉环境 + 人类演示 -> 机器人理解上下文并完成动作
```

对 Dex 来说，这属于更长期的 embodied locomotion 方向。

---

### 论文 9：ALMI: Adversarial Locomotion and Motion Imitation for Humanoid Policy Learning

| 项目 | 内容 |
|---|---|
| 年份/来源 | NeurIPS 2025 |
| 方向 | adversarial imitation / locomotion + upper-body motion tracking |
| 核心关键词 | adversarial training, lower-body stability, upper-body imitation, whole-body humanoid policy |
| 重点阅读 | 为什么要区分 lower-body locomotion 和 upper-body imitation |
| 阅读问题 | Dex 如果以后做 whole-body imitation，为什么不能全身一起硬模仿？ |
| 对 Dex 的启发 | 下半身负责稳定运动，上半身负责动作表达，两者应当分工建模。 |

#### 你需要重点理解

ALMI 的核心启发是：humanoid whole-body imitation 不应该简单地把全身所有关节都作为同等重要的 tracking target。

更合理的分工是：

```text
lower body：保证 locomotion stability
upper body：保证 motion imitation / expressiveness
```

这对 Dex 很重要。因为如果你直接让所有关节都强 tracking 人类动作，可能会导致：

- 下肢稳定性下降；
- foot contact timing 混乱；
- pelvis / waist 姿态不稳定；
- 上半身动作干扰行走；
- policy 学到不自然的 compensation。

---

# 3. 扩展阅读

## 3.1 Gait-Net-augmented Implicit Kino-dynamic MPC for Dynamic Variable-frequency Humanoid Locomotion over Discrete Terrains

| 项目 | 内容 |
|---|---|
| 年份/来源 | RSS 2025 |
| 方向 | MPC + learning / variable-frequency gait / discrete terrains |
| 核心关键词 | kino-dynamic MPC, Gait-Net, step location, step duration, contact force, discrete terrain |
| 适合什么时候读 | 当你想把 RL 和 MPC 结合时 |
| 对 Dex 的启发 | 纯 RL 不是唯一方向，可以用 MPC 提供约束和可解释性，用 network 预测 gait timing 或局部参数。 |

### 重点理解

这篇论文说明，在 humanoid locomotion 中，RL 和 MPC 可以结合：

```text
MPC：负责动力学约束、接触约束、稳定性和可解释性
Gait-Net：负责预测步态参数、步频、step timing、step location
```

如果 Dex 后续对安全性、可解释性和物理约束要求更高，可以考虑这条路线。

---

# 4. 建议精读的 5 篇

如果时间有限，最建议先精读这 5 篇：

| 必读顺序 | 论文 | 对 Dex 的价值 |
|---|---|---|
| 1 | Real-world Humanoid Locomotion with Reinforcement Learning | 建立 humanoid RL locomotion 的基本范式 |
| 2 | Advancing Humanoid Locomotion with Denoising World Model Learning | 学复杂地形、去噪 world model、zero-shot sim-to-real |
| 3 | ASAP | 学真实物理对齐和 residual action correction |
| 4 | HugWBC | 学 whole-body command space 和全身步态控制 |
| 5 | Humanoid Locomotion as Next Token Prediction | 学 Transformer / 序列预测式 locomotion |

---

# 5. 每篇论文的固定阅读模板

建议你每读一篇论文，都按下面 8 个问题整理。

## 5.1 论文解决的问题

这篇论文主要解决什么问题？

- 平地行走？
- 复杂地形？
- 稀疏落脚点？
- sim-to-real？
- whole-body control？
- motion imitation？
- 视觉引导的 contextual skill？

---

## 5.2 核心输入

policy 或 controller 的输入是什么？

- proprioception；
- joint position；
- joint velocity；
- IMU；
- base angular velocity；
- projected gravity；
- command；
- observation history；
- action history；
- height map；
- RGB / depth；
- human motion data；
- reference motion；
- contact state。

---

## 5.3 核心输出

policy 输出什么？

- joint position target；
- joint velocity target；
- torque；
- residual action；
- footstep location；
- contact force；
- gait timing；
- latent action token。

---

## 5.4 Policy / Controller 结构

使用了什么模型结构？

- MLP；
- RNN / LSTM / GRU；
- Transformer；
- causal Transformer；
- teacher-student；
- world model；
- diffusion model；
- MPC + neural network；
- adversarial discriminator。

---

## 5.5 Reward 设计

reward 包括哪些项？

- velocity tracking；
- angular velocity tracking；
- height tracking；
- orientation reward；
- foot clearance；
- feet slide penalty；
- contact timing；
- gait phase；
- action smoothness；
- torque penalty；
- energy penalty；
- joint limit penalty；
- imitation reward；
- adversarial imitation reward；
- termination penalty。

---

## 5.6 Sim-to-real 方法

它如何处理仿真到真实的差距？

- domain randomization；
- friction randomization；
- mass randomization；
- motor strength randomization；
- latency randomization；
- observation noise；
- action noise；
- system identification；
- residual action model；
- real-world data correction；
- teacher-student distillation。

---

## 5.7 实验平台

实验用的是什么机器人？

- Unitree H1；
- Unitree G1；
- Berkeley Humanoid；
- Booster T1；
- Digit；
- Cassie；
- custom humanoid；
- quadruped baseline。

需要注意：不同机器人自由度、脚掌结构、关节力矩、传感器配置差异很大，不能直接照搬 reward 和超参数。

---

## 5.8 能否迁移到 Dex

最后一定要回答：

```text
这篇论文中哪些东西可以直接迁移到 Dex？
哪些东西需要改？
哪些东西目前不适合 Dex？
如果我要复现，第一步应该做什么？
```

---

# 6. 面向 Dex 的推荐路线

结合你现在的 Dex 项目，我建议路线如下：

```text
第一阶段：PPO humanoid locomotion baseline
Real-world Humanoid Locomotion with RL
↓
第二阶段：加入 history / Transformer / sequence modeling
Humanoid Locomotion as Next Token Prediction
↓
第三阶段：复杂地形与鲁棒性
DWL + BeamDojo
↓
第四阶段：sim-to-real / sim-to-sim 对齐
ASAP
↓
第五阶段：全身控制
HugWBC + ALMI
↓
第六阶段：具身智能方向
HumanPlus + VideoMimic
```

---

# 7. 对 Dex 项目的具体启发

## 7.1 近期最现实的方向

你现在最现实的方向不是马上做 VideoMimic 或 HumanPlus，而是先把 Dex 的 locomotion baseline 做扎实：

```text
PPO baseline
+ observation history
+ action history
+ whole-body posture reward
+ rough terrain randomization
+ feet slide / foot clearance / contact timing reward
```

也就是说，近期可以重点做：

1. 平地速度跟踪稳定性；
2. 站立时抖动控制；
3. 行走时 feet slide 控制；
4. 腰部和上半身姿态约束；
5. rough terrain 适应；
6. observation/action history 对鲁棒性的影响；
7. 不同 reward 权重对步态的影响。

---

## 7.2 中期可以做的方向

中期可以考虑：

```text
Dex humanoid locomotion
+ terrain perception
+ history-based adaptation
+ sim-to-sim / sim-to-real residual correction
```

对应论文：

- DWL；
- BeamDojo；
- ASAP；
- Real-world Humanoid Locomotion with RL。

---

## 7.3 长期可以做的方向

长期如果你想往 embodied intelligence 靠，可以考虑：

```text
Dex whole-body skill learning
+ human motion imitation
+ video-based imitation
+ contextual locomotion
```

对应论文：

- HumanPlus；
- VideoMimic；
- ALMI；
- Humanoid Locomotion as Next Token Prediction。

---

# 8. 最终总结

近两年 locomotion 顶会论文的核心变化是：

> 从“调 reward 让机器人走起来”，转向“用数据、世界模型、Transformer、MPC+RL、sim-to-real 对齐，让人形机器人在真实复杂环境中完成全身、泛化、可交互的运动技能”。

对 Dex 来说，最合理的切入顺序是：

```text
PPO 速度跟踪 baseline
↓
加入 history-based adaptation
↓
加入 whole-body posture control
↓
扩展到 rough terrain locomotion
↓
研究 sim-to-real / sim-to-sim residual correction
↓
再考虑 human imitation / video imitation / embodied locomotion
```

最优先精读：

```text
1. Real-world Humanoid Locomotion with Reinforcement Learning
2. Advancing Humanoid Locomotion with Denoising World Model Learning
3. ASAP
4. HugWBC
5. Humanoid Locomotion as Next Token Prediction
```

