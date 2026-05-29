# Dex学习



# 机器人基本知识

## Action tracker

这句话把当前领域分成两个趋势：

第一类是 **general motion tracker**：

```
大量动作数据
    ↓
训练一个通用 tracker
    ↓
可以跟踪各种人体/机器人动作
```

第二类是 **teleoperation system**：

```
人通过 VR / 动捕 / 手柄控制机器人
    ↓
收集真实任务数据
    ↓
用于移动操作、模仿学习、VLA 训练
```

## IMU

IMU 是 **Inertial Measurement Unit，惯性测量单元**。

在机器人里，它通常用来测量机器人机身的运动状态，尤其是：

1. **角速度 angular velocity**
    也就是机器人绕三个轴转动的速度：
   $$
   \omega_t = [\omega_x,\omega_y,\omega_z]
   $$
   例如机器人向左转、身体前后摇晃、左右倾斜，都会产生角速度。

2. **线加速度 linear acceleration**
    也就是机器人在 x、y、z 方向上的加速度：
   $$
   a_t = [a_x,a_y,a_z]
   $$

3. **姿态估计 orientation / attitude**
    有些 IMU 会结合陀螺仪、加速度计、磁力计，通过滤波算法估计 roll、pitch、yaw，或者四元数姿态。





##  link

在机器人模型里，**link 是机器人中的刚体部件**。

机器人通常由两类东西组成：

```
link：
    刚体部件

joint：
    连接两个 link 的关节
```

例如一个 humanoid 机器人可以有这些 link：

```
pelvis link
torso link
left thigh link
left calf link
left foot link
right upper arm link
right forearm link
right hand link
head link
...
```

简单说：

```
joint 是关节
link 是被关节连接起来的身体段
```

比如人的腿：

```
大腿骨段 = thigh link
膝盖 = knee joint
小腿骨段 = calf link
踝关节 = ankle joint
脚 = foot link
```

在仿真模型 URDF / MJCF 里面，机器人就是由 link 和 joint 组成的树形结构。















## 训练传参

**训练 Dex 机器人在 Isaac Lab / Isaac Sim 里学会稳定行走。**

通常会包含这些内容：

- command 生成：给目标线速度、角速度
- observation 设计：机器人姿态、关节状态、接触状态等
- action 设计：关节控制命令
- reward 设计：速度跟踪、姿态稳定、动作平滑、足部行为等
- termination 设计：摔倒、超界、异常状态结束

#### 最重要的三个参数

```
DexEnv
DexEnvCfg()
DexAgentCfg()
```

##### 1. `DexEnv`

它是**环境类**。

你可以把它理解成：
 **这个任务到底怎么跑，是由它决定的。**

它负责的是“行为和流程”

##### 2. `DexEnvCfg()`

它是**环境配置对象**。

你可以把它理解成：
 **给 `DexEnv` 用的一整套参数说明书。**

DexEnv 在运行时，应该使用哪一版环境设定

##### 3. `DexAgentCfg()`

它是**训练器/算法配置对象**。

你可以把它理解成：
 **给 RL 训练器和算法用的参数说明书。**

### 事件的三种触发模式

#### 1）`startup` 

环境创建后执行一次 每一次运行train 之运行一次

适合做：

- 材质随机化
- 质量随机化
- 关节参数随机化

这类是“开机级别”的随机化。

------

#### 2）`reset`

每次 episode reset 时执行

适合做：

- 重置 base 位姿
- 重置速度
- 重置关节
- 每局随机电机参数

这类是“每局开始前”的初始化。

------

#### 3）`interval`

训练过程中隔一段时间触发

适合做：

- 推一下机器人
- 外部扰动
- 周期性干扰

这类是“训练中途制造麻烦”。

## motion_loader.py

**它是一个 AMP 动作数据加载器。**
 它把不同格式的专家动作文件（`txt / npz / pkl`）统一读进来，整理成训练时能直接使用的状态序列，并进一步随机采样出 `(s, s_next)` 这样的相邻帧对，供 AMP / 模仿学习训练使用。

代码里面定义的observation是：

- 20 维关节位置
- 20 维关节速度
- 12 维末端位置

12 维末端位置其实对应的是 4 个末端点 × 3 维：

- 左手
- 右手
- 左脚
- 右脚

### 原始的npz的数据

**也就是整个机器人全身就是要有29个关节** 

#### **上肢：7*2**

##### 1. 左臂关节（7个）

- shoulder_pitch_l_joint
- shoulder_roll_l_joint
- shoulder_yaw_l_joint
- elbow_pitch_l_joint
- elbow_yaw_l_joint
- wrist_pitch_l_joint
- wrist_roll_l_joint

##### 2. 右臂关节（7个）

- shoulder_pitch_r_joint
- shoulder_roll_r_joint
- shoulder_yaw_r_joint
- elbow_pitch_r_joint
- elbow_yaw_r_joint
- wrist_pitch_r_joint
- wrist_roll_r_joint

#### **下肢：6*2**

##### 4. 左腿关节（6个）

- hip_roll_l_joint
- hip_pitch_l_joint
- hip_yaw_l_joint
- knee_pitch_l_joint
- ankle_pitch_l_joint
- ankle_roll_l_joint

##### 5. 右腿关节（6个）

- hip_roll_r_joint
- hip_pitch_r_joint
- hip_yaw_r_joint
- knee_pitch_r_joint
- ankle_pitch_r_joint
- ankle_roll_r_joint

#### **waist：3** 

##### 3. 腰部关节（3个）

- waist_yaw_joint

- waist_roll_joint

- waist_pitch_joint

  

```python
motion_data = np.load(motion_file)
dof_pos = motion_data["dof_pos"]
dof_vel = motion_data["dof_vel"]
endpoint_pos_BCS = motion_data["endpoint_pos_BCS"].reshape(...)
```

这三样最重要：

- `dof_pos`：原始 29 维关节角 
- `dof_vel`：原始 29 维关节速度
- `endpoint_pos_BCS`：末端点在 body coordinate system 下的位置。

你前面已经打印过，当前文件里确实有这些键。
 说明你的 `npz` 是“原始机器人动作数据”，而不是已经整理好的 AMP 状态。

这篇论文：

**从原始 29 个关节中，筛出 AMP 需要的 20 个关节。**



**29 维是“机器人完整原始关节空间”，20 维是“为了 AMP 提炼出来的关键运动关节空间”。**

## URDF文件

在 **URDF** 里你可以把机器人理解成一棵树：

- **link** 是“刚体本体”，也就是机器人的一段实体
- **joint** 是“连接关系/运动关系”，也就是两个刚体之间怎么连、能不能动、绕哪根轴动

### link 是什么

`link` 表示一个**刚性部件**。
 它通常负责描述这几个东西：

- **惯性参数**：质量、惯量
- **外观**：visual，给渲染器看
- **碰撞体**：collision，给物理引擎看

比如你文件里的 `pelvis` 是一个 link，里面有：

- `<inertial>`：质量和惯量
- `<visual>`：mesh 外观
- 还可以有 `<collision>`：碰撞几何 

所以你可以把 **link 理解成“骨头/零件”**。

### Link定义代码

```C
<link name="hip_pitch_l_link">   <!-- 定义一个刚体 link，名字是左髋 pitch 这一段 -->

  <inertial>   <!-- 惯性属性：给物理引擎用，决定质量、质心、转动惯量 -->
    <origin
      xyz="0.009589 0.069327 -0.024358"
      rpy="0 0 0" />
      <!-- 惯性坐标系原点（通常可理解为质心位置）在本 link 坐标系下的位置
           x=0.009589 : 稍微向前
           y=0.069327 : 向左偏一些（因为这是左侧髋部结构）
           z=-0.024358: 稍微向下 -->

    <mass value="2.221379" />
    <!-- 这个刚体的质量，单位一般是 kg -->

    <inertia
      ixx="0.004839"
      ixy="-0.00014"
      ixz="8.3e-05"
      iyy="0.003411"
      iyz="0.000185"
      izz="0.003611" />
    <!-- 惯性张量，描述该刚体绕各轴转动时的惯性大小
         ixx/iyy/izz : 分别是绕 x/y/z 主轴的转动惯量
         ixy/ixz/iyz : 惯性耦合项，说明质量分布不是完全对称的 -->
  </inertial>

  <visual>   <!-- 可视化模型：主要用于显示，不直接用于物理碰撞 -->
    <origin
      xyz="0 0 0"
      rpy="0 0 0" />
      <!-- visual 模型相对于该 link 坐标系的位姿
           这里表示可视模型和 link 坐标系完全重合 -->

    <geometry>
      <mesh filename="../meshes/hip_pitch_l_link.STL" />
      <!-- 视觉外形使用的三维网格模型文件 -->
    </geometry>

    <material name="">
      <color rgba="0.752941176470588 0.752941176470588 0.752941176470588 1" />
      <!-- 显示颜色：灰色，不透明 -->
    </material>
  </visual>

  <collision>   <!-- 当前真正启用的碰撞模型 -->
    <origin
      rpy="0 1.570 0"
      xyz="0.0 0.075 -0.025" />
      <!-- 碰撞体相对于 link 坐标系的位置和朝向
           xyz:
             x=0.0      不前后偏移
             y=0.075    向左偏 7.5 cm
             z=-0.025   向下偏 2.5 cm
           rpy:
             绕 y 轴旋转约 1.570 rad（约 90°）
             说明下面这个圆柱需要转个方向再放置 -->

    <geometry>
      <cylinder
        length="0.04"
        radius="0.065" />
      <!-- 碰撞几何体是一个圆柱：
           长度 0.04 m
           半径 0.065 m
           用来近似左髋 pitch 这一段的实体外形 -->
    </geometry>
  </collision>

</link>
```



### Joint 是什么

`joint` 表示**两个 link 之间的连接方式**。
 它定义的是：

- 父 link 是谁：`<parent link="..."/>`
- 子 link 是谁：`<child link="..."/>`
- 连接点在哪：`<origin .../>`
- 沿什么轴运动：`<axis xyz="..."/>`
- 运动范围：`<limit lower="..." upper="..."/>`
- 关节类型：`revolute / fixed / prismatic ...`

例如你这里：

```
<joint name="hip_pitch_l_joint" type="revolute">
  <parent link="pelvis" />
  <child link="hip_pitch_l_link" />
  <axis xyz="0 1 0" />
  ...
</joint>
```

这表示：

- 父刚体是 `pelvis`
- 子刚体是 `hip_pitch_l_link`
- 两者之间不是焊死的
- 而是一个 **revolute（转动）关节**
- 并且绕 **y 轴** 转动 

所以你可以把 **joint 理解成“关节/铰链/约束”**。



#### Joint的运动方式

| type         | 含义         | 自由度 |
| ------------ | ------------ | ------ |
| `fixed`      | 固定         | 0      |
| `revolute`   | 有限位旋转   | 1      |
| `continuous` | 无限位旋转   | 1      |
| `prismatic`  | 线性平移     | 1      |
| `planar`     | 平面运动     | 3      |
| `floating`   | 空间自由运动 | 6      |

------

在本urdf里只涉及fixed和revolute

#### Joint定义方式

```C
<joint
    name="hip_pitch_l_joint"      <!-- 定义一个关节，名字叫左髋 pitch 关节 -->
    type="revolute">              <!-- 关节类型是 revolute：绕一根固定轴旋转，且有角度上下限 -->

    <origin
      xyz="-0.00010305 0.076815 -0.11847"
      rpy="-0.17453 0 0.0013415" />
      <!-- 这个 joint 坐标系相对于父 link（pelvis）坐标系的位置和姿态
           xyz:
             x = -0.00010305   几乎没有前后偏移
             y =  0.076815     向左偏 7.68 cm
             z = -0.11847      向下偏 11.85 cm
           这很符合“左髋关节”在骨盆左下方的位置

           rpy:
             roll  = -0.17453 rad   大约 -10°
             pitch = 0
             yaw   = 0.0013415 rad  非常小，接近 0
           说明这个关节坐标系相对于 pelvis 不是完全正对齐的，
           而是做了一点点安装角修正 -->

    <parent link="pelvis" />
    <!-- 父 link 是 pelvis
         也就是这个关节是从骨盆发出来的 -->

    <child link="hip_pitch_l_link" />
    <!-- 子 link 是 hip_pitch_l_link
         也就是这个关节带动的下一段刚体是左髋 pitch link -->

    <axis xyz="0 1 0" />
    <!-- 关节转轴是 y 轴
         也就是说这个关节绕局部 y 轴旋转
         这就是“pitch（前后摆动/屈伸）”自由度 -->

    <limit
      lower="-3.141592653589793"
      upper="2.8797932657906435"
      effort="235.0"
      velocity="16.755160819145562" />
      <!-- 关节限制参数

           lower = -3.141592653589793
             最小关节角，大约 -pi，也就是 -180°

           upper = 2.8797932657906435
             最大关节角，大约 165°

           effort = 235.0
             关节允许的最大输出力矩/驱动力上限
             对 revolute 关节来说通常理解为最大力矩

           velocity = 16.755160819145562
             关节允许的最大角速度，单位通常是 rad/s -->
</joint>
```



## 奖励函数基本

## 数学设计





```
asset: Articulation = env.scene[asset_cfg.name]
```

这一句就是从场景里把机器人拿出来。

这里的 `asset` 一般就是你的机器人本体，里面有各种状态量，比如：

- 根部位置
- 根部姿态
- 根部线速度
- 根部角速度
- 各关节位置速度等



### 1. 根部位置

英文常见名字：

- `root_pos_w`

意思是：

> 机器人“根部”在世界坐标系里的位置。



### 2. 根部姿态

英文常见名字：

- `root_quat_w`

意思是：

> 机器人根部当前“朝向和倾斜状态”。

这个通常不是直接用欧拉角存，而是用**四元数 quaternion**表示。
 你代码里就有：

```
asset.data.root_quat_w
```

它就是根部在世界坐标系下的姿态四元数。 

因为四元数更稳定，适合三维旋转计算，不容易出万向锁问题。

### 3. 根部线速度

英文常见名字：

- `root_lin_vel_w`
- `root_lin_vel_b`

意思是：

> 机器人根部“平移得有多快”。

线速度就是位置随时间的变化率。

如果是三维向量：
$$
[v_x,\ v_y,\ v_z]
$$
表示：

- `vx`：前后方向速度
- `vy`：左右方向速度
- `vz`：上下方向速度

你代码里有：

```
asset.data.root_lin_vel_w[:, :3]
```

表示根部在**世界坐标系**下的线速度。 

还有：

```
asset.data.root_lin_vel_b[:, 2]
```

表示根部在**机体坐标系 body frame**下的 z 向速度。 



如果机器人往前走：

- 前向速度大
- 左右速度小
- 上下速度接近 0

那说明它主要在平稳向前移动。

### 4. 根部角速度

英文常见名字：

- `root_ang_vel_w`
- `root_ang_vel_b`

意思是：

> 机器人根部“转得有多快”。

它不是平移速度，而是**旋转速度**。

如果写成三维向量：
$$
[\omega_x,\ \omega_y,\ \omega_z]
$$
表示绕三个轴的旋转快慢：

- `ωx`：绕 x 轴转
- `ωy`：绕 y 轴转
- `ωz`：绕 z 轴转

你代码里有：

```
asset.data.root_ang_vel_w[:, 2]
```

这通常就是根部绕 z 轴的角速度，也就是**转向速度 yaw rate**。 

还有：

```
asset.data.root_ang_vel_b[:, :2]
```

表示机体系下绕 x、y 轴的角速度，常用于惩罚机身晃动。 

比如机器人原地左转：

- 线速度可能很小
- 但角速度 `ωz` 很大

### 5. 各关节位置

英文常见名字：

- `joint_pos`

意思是：

> 每个关节当前转到了什么角度。

例如人形机器人某些关节：

- hip_pitch
- knee_pitch
- ankle_pitch
- shoulder_pitch

它们每个都有当前角度。

你代码里有：

```
asset.data.joint_pos
```

还常常和默认姿态比较：

```
asset.data.joint_pos[:, asset_cfg.joint_ids] - asset.data.default_joint_pos[:, asset_cfg.joint_ids]
```

这表示当前关节角和默认站立角之间的偏差。 

比如膝关节：

- 0 rad：伸直
- 0.5 rad：弯曲一些
- 1.0 rad：弯得更多

所以关节位置本质上就是：

> 各个关节“摆到了哪”。

### 6. 各关节速度

英文常见名字：

- `joint_vel`

意思是：

> 每个关节转动得有多快。

也就是关节角度对时间的变化率。

比如：

- 膝盖快速弯曲
- 踝关节快速回摆

这些都会体现在 `joint_vel` 里。

你代码里有：

```
asset.data.joint_vel
```

比如在 energy reward 里：

```
asset.data.applied_torque * asset.data.joint_vel
```

这个实际上就在算关节功率相关的量。 



关节位置回答的是：

> 关节现在在哪个角度？

关节速度回答的是：

> 关节正在以多快的速度往那个方向转？



### 7. 各关节加速度

英文常见名字：

- `joint_acc`

意思是：

> 各关节速度变化得有多快。

你代码里有：

```
asset.data.joint_acc
```

并用它做惩罚：

```
torch.sum(torch.square(asset.data.joint_acc[:, asset_cfg.joint_ids]), dim=1)
```

这通常是为了避免动作太猛、太抖。 

### 直白总结

一句话说：

- **根部位置**：机器人整体在哪里
- **根部姿态**：机器人整体朝哪、歪成什么样
- **根部线速度**：机器人整体平移多快
- **根部角速度**：机器人整体旋转多快
- **关节位置**：每个关节转到哪里
- **关节速度**：每个关节转得多快
- **关节加速度**：每个关节变化得多猛

### 四元数

四元数，就是一种**表示三维旋转**的方法。

你可以先别把它想得太复杂，先抓住一句话：

**四元数 = 用 4 个数来表示“物体当前朝向”**

通常写成：
$$
q = [w, x, y, z]
$$
其中：

- `w` 是实部
- `x, y, z` 是虚部



## 奖励函数汇总





## Asset.data

### 维度解释

很好，这份输出非常完整。你这个 `asset.data` 可以理解成：每个并行环境里，机器人当前/默认的“状态总表”。
先抓核心维度：

- `N=4096`：并行环境数
- `B=24`：刚体（link/body）数量
- `D=23`：关节数

所以常见 shape 含义：

- `(4096, 3)`：每个环境一个 3D 向量（如 root 速度）
- `(4096, 24, 3)`：每个环境、每个刚体一个 3D 向量（如 body 速度）
- `(4096, 23)`：每个环境、每个关节一个标量（如 joint pos/vel/torque）





### 内容

```python
data.FORWARD_VEC_B: shape=(4096, 3) #每个环境里，机体（root）局部坐标系下的“前向”单位向量，3维。
data.GRAVITY_VEC_W: shape=(4096, 3) #每个环境里，世界系下的重力方向向量（通常归一化），3维。
data.applied_torque: shape=(4096, 23) #每个环境、每个关节当前施加的力矩（执行器输出），共23个关节。
data.body_acc_w: shape=(4096, 24, 6) #每个刚体在世界系下的“加速度”类6维量（一般为线加速度3+角加速度3的打包，具体顺序以Isaac Lab文档为准）。
data.body_ang_acc_w: shape=(4096, 24, 3) #每个刚体在世界系下的角加速度(wx, wy, wz)。
data.body_ang_vel_w: shape=(4096, 24, 3) #每个刚体在世界系下的角速度。
data.body_com_acc_w: shape=(4096, 24, 6) #以质心COM为参考的刚体加速度（6维打包）。
data.body_com_ang_acc_w: shape=(4096, 24, 3) #COM处的角加速度。
data.body_com_ang_vel_w: shape=(4096, 24, 3) #COM处的角速度。
data.body_com_lin_acc_w: shape=(4096, 24, 3) #COM处的线加速度。
data.body_com_lin_vel_w: shape=(4096, 24, 3) #COM处的线速度。
data.body_com_pos_b: shape=(4096, 24, 3) #各刚体COM在机体(root)局部系下的位置。
data.body_com_pos_w: shape=(4096, 24, 3) #各刚体COM在世界系下的位置。
data.body_com_pose_b: shape=(4096, 24, 7) #COM在机体系下的位姿：位置3 + 四元数4。
data.body_com_pose_w: shape=(4096, 24, 7) #COM在世界系下的位姿：位置3 + 四元数4。
data.body_com_quat_b: shape=(4096, 24, 4) #COM姿态四元数（机体系表达上下文）。
data.body_com_quat_w: shape=(4096, 24, 4) #COM姿态四元数（世界系）。
data.body_com_state_w: shape=(4096, 24, 13) #COM在世界系下的打包状态（通常含pos、quat、线速度、角速度等，共13维）。
data.body_com_vel_w: shape=(4096, 24, 6) #COM在世界系下的速度打包（一般线3 + 角3）。
data.body_incoming_joint_wrench_b: shape=(4096, 24, 6) #作用在刚体上的关节侧力旋量/wrench（机体系），6维（力3 + 力矩3）。
data.body_lin_acc_w: shape=(4096, 24, 3) #刚体（link frame语义下）线加速度，世界系。
data.body_lin_vel_w: shape=(4096, 24, 3) #刚体线速度，世界系（脚滑等常用）。
data.body_link_ang_vel_w: shape=(4096, 24, 3) #Link原点/坐标系上的角速度（与COM版区分）。
data.body_link_lin_vel_w: shape=(4096, 24, 3) #Link上的线速度。
data.body_link_pos_w: shape=(4096, 24, 3) #Link原点在世界系位置。
data.body_link_pose_w: shape=(4096, 24, 7) #Link位姿打包（世界系）。
data.body_link_quat_w: shape=(4096, 24, 4) #Link四元数（世界系），做相对姿态时常用。
data.body_link_state_w: shape=(4096, 24, 13) #Link打包状态（世界系，13维）。
data.body_link_vel_w: shape=(4096, 24, 6) #Link速度打包（线+角，世界系）。
data.body_names: type=list, len=24 #24个刚体名字的Python列表，下标与body_*第二维对应。
data.body_pos_w: shape=(4096, 24, 3) #刚体位置（世界系），常用接口。
data.body_pose_w: shape=(4096, 24, 7) #刚体位姿打包（世界系）。
data.body_quat_w: shape=(4096, 24, 4) #刚体四元数（世界系）。
data.body_state_w: shape=(4096, 24, 13) #刚体打包状态（世界系）。
data.body_vel_w: shape=(4096, 24, 6) #刚体速度打包（世界系）。
data.com_pos_b: shape=(4096, 24, 3) #COM相对root的位置（机体系），与body_com_pos_b类信息重叠用途。
data.com_quat_b: shape=(4096, 24, 4) #COM相对姿态（机体系上下文）。
data.computed_torque: shape=(4096, 23) #控制器/物理算出的“计算力矩”（与applied_torque可对比debug）。
data.default_fixed_tendon_damping: type=NoneType #固定肌腱（fixed tendon）默认阻尼；你模型无此项则为None。
WARNING: default_fixed_tendon_limit ... deprecated ... default_fixed_tendon_pos_limits #旧API提醒：以后用default_fixed_tendon_pos_limits。
data.default_fixed_tendon_limit: type=NoneType #旧字段，无肌腱则为None。
data.default_fixed_tendon_limit_stiffness: type=NoneType #肌腱限位刚度默认，无则为None。
data.default_fixed_tendon_offset: type=NoneType #肌腱偏移默认。
data.default_fixed_tendon_pos_limits: type=NoneType #肌腱位置限位默认。
data.default_fixed_tendon_rest_length: type=NoneType #静息长度默认。
data.default_fixed_tendon_stiffness: type=NoneType #刚度默认。
data.default_inertia: shape=(4096, 24, 9), device=cpu #每个刚体默认惯性张量相关参数（9维打包），存在CPU。
data.default_joint_armature: shape=(4096, 23) #关节默认电机/传动等效惯量（armature）。
data.default_joint_damping: shape=(4096, 23) #关节默认阻尼。
data.default_joint_dynamic_friction_coeff: shape=(4096, 23) #关节动摩擦系数默认。
WARNING: default_joint_friction ... use default_joint_friction_coeff #旧名default_joint_friction将废弃。
data.default_joint_friction: shape=(4096, 23) #旧摩擦字段（仍可读）。
data.default_joint_friction_coeff: shape=(4096, 23) #推荐用的摩擦系数默认。
WARNING: default_joint_limits ... use default_joint_pos_limits #旧限位字段将废弃。
data.default_joint_limits: shape=(4096, 23, 2) #旧：每关节一对限位（如min/max）。
data.default_joint_pos: shape=(4096, 23) #关节默认/标称位置（reset基准等）。
data.default_joint_pos_limits: shape=(4096, 23, 2) #关节位置限位默认（推荐用这个）。
data.default_joint_stiffness: shape=(4096, 23) #关节刚度默认（PD等）。
data.default_joint_vel: shape=(4096, 23) #关节默认速度。
data.default_joint_viscous_friction_coeff: shape=(4096, 23) #粘性摩擦系数默认。
data.default_mass: shape=(4096, 24), device=cpu #各刚体默认质量，在CPU。
data.default_root_state: shape=(4096, 13) #根默认打包状态（pos+quat+vel等）。
data.default_spatial_tendon_* : type=NoneType #空间肌腱相关默认，无则为None。
data.device: type=str #数据主设备描述字符串（如"cuda:0"），不是张量。
data.fixed_tendon_* : type=NoneType #当前步肌腱状态，无肌腱则为None。
WARNING: fixed_tendon_limit ... fixed_tendon_pos_limits #同上，API迁移提示。
data.gear_ratio: shape=(4096, 23) #关节传动比。
data.heading_w: shape=(4096,) #每个环境根朝向（偏航一类标量），世界系heading。
data.joint_acc: shape=(4096, 23) #关节加速度。
data.joint_armature: shape=(4096, 23) #当前关节armature（可能被随机化后变化）。
data.joint_damping: shape=(4096, 23) #当前阻尼。
data.joint_dynamic_friction_coeff: shape=(4096, 23) #当前动摩擦系数。
data.joint_effort_limits: shape=(4096, 23) #关节力矩/力限制。
data.joint_effort_target: shape=(4096, 23) #期望关节力矩目标（控制器内部）。
WARNING: joint_friction ... joint_friction_coeff #旧字段将废弃。
data.joint_friction: shape=(4096, 23) #旧摩擦。
data.joint_friction_coeff: shape=(4096, 23) #当前摩擦系数。
WARNING: joint_limits ... joint_pos_limits #旧限位将废弃。
data.joint_limits: shape=(4096, 23, 2) #旧限位。
data.joint_names: type=list, len=23 #23个关节名字列表，下标与joint_*最后一维对应。
data.joint_pos: shape=(4096, 23) #关节位置。
data.joint_pos_limits: shape=(4096, 23, 2) #关节位置限位（当前）。
data.joint_pos_target: shape=(4096, 23) #关节位置目标（PD/阻抗等）。
data.joint_stiffness: shape=(4096, 23) #当前关节刚度。
data.joint_vel: shape=(4096, 23) #关节速度。
data.joint_vel_limits: shape=(4096, 23) #关节速度限幅。
data.joint_vel_target: shape=(4096, 23) #关节速度目标。
WARNING: joint_velocity_limits ... joint_vel_limits #旧名将废弃。
data.joint_velocity_limits: shape=(4096, 23) #旧速度限幅。
data.joint_viscous_friction_coeff: shape=(4096, 23) #当前粘性摩擦。
data.projected_gravity_b: shape=(4096, 3) #重力向量在机体系下的表达；姿态稳定奖励常用。
data.root_ang_vel_b: shape=(4096, 3) #根角速度，机体系。
data.root_ang_vel_w: shape=(4096, 3) #根角速度，世界系（跟踪yaw率时常用）。
data.root_com_ang_vel_b / _w: shape=(4096, 3) #根COM处角速度，机体系 / 世界系。
data.root_com_lin_vel_b / _w: shape=(4096, 3) #根COM线速度，机体系 / 世界系。
data.root_com_pos_w: shape=(4096, 3) #根COM世界位置。
data.root_com_pose_w: shape=(4096, 7) #根COM世界位姿打包。
data.root_com_quat_w: shape=(4096, 4) #根COM世界四元数。
data.root_com_state_w: shape=(4096, 13) #根COM世界打包状态。
data.root_com_vel_w: shape=(4096, 6) #根COM世界速度打包（线+角）。
data.root_lin_vel_b: shape=(4096, 3) #根线速度机体系。
data.root_lin_vel_w: shape=(4096, 3) #根线速度世界系。
data.root_link_ang_vel_b / _w: shape=(4096, 3) #根link角速度（与COM区分）。
data.root_link_lin_vel_b / _w: shape=(4096, 3) #根link线速度。
data.root_link_pos_w: shape=(4096, 3) #根link世界位置。
data.root_link_pose_w: shape=(4096, 7) #根link世界位姿。
data.root_link_quat_w: shape=(4096, 4) #根link世界四元数。
data.root_link_state_w: shape=(4096, 13) #根link世界打包状态。
data.root_link_vel_w: shape=(4096, 6) #根link世界速度打包。
data.root_pos_w: shape=(4096, 3) #根位置世界系。
data.root_pose_w: shape=(4096, 7) #根位姿打包。
data.root_quat_w: shape=(4096, 4) #根四元数世界系。
data.root_state_w: shape=(4096, 13) #根打包状态（最常用总览之一）。
data.root_vel_w: shape=(4096, 6) #根速度打包世界系。
data.soft_joint_pos_limits: shape=(4096, 23, 2) #“软”位置限位（缓冲区 / curriculum等）。
data.soft_joint_vel_limits: shape=(4096, 23) #软速度限位。
data.spatial_tendon_* : type=NoneType #空间肌腱，无则为None。
data.update: type=method #刷新/写回仿真数据的成员方法，不是状态张量。


```



# 操作

## Train

```
CUDA_VISIBLE_DEVICES=1 PYTHONUNBUFFERED=1 
python legged_lab/scripts/train.py --task=Dex_walk --headless --logger=tensorboard
```

## Play

```
python legged_lab/scripts/play.py --task=Dex_walk --num_envs=32
```

## MUJOCO仿真

**仿真都需要启动xwalk-run conda环境**

### 启动主控制节点

在xmigcs文件下：

- 更新配置文件dex_config.yaml 调节手柄
- 更新配置文件mlp.yaml  策略
- 在model文件中加入policy

然后运行：

```bash
export ROS_DOMAIN_ID=99
source /opt/ros/humble/setup.bash
python3 rl_control_node.py  ## 同时需要运行 touch /tmp/rl_start_signal
```

###  启动XBOX手柄数据节点

在xmigcs文件下：

```bash
export ROS_DOMAIN_ID=99
source /opt/ros/humble/setup.bash
ros2 run joy joy_node --ros-args --remap joy:=xbox_data
```

### 启动仿真

在MuJoCo文件

```bash
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
python scripts/simulator_view_asyn.py -m evt2
```

### **录屏**

 直接按 `Print` 键打开截图工具，切到录屏模式后可以录整个屏幕或选区；快捷键 `Shift + Ctrl + Alt + R` 也可以直接开始/停止录屏。录好的视频默认会存到 `Videos/Screencasts`

### 控制器使用说明

**XBOX手柄键位映射**

xMIGCS支持标准XBOX手柄控制，以下是详细键位映射关系：

**单按钮状态切换**

| 按钮 | 对应状态     | 功能说明     |
| ---- | ------------ | ------------ |
| A    | gotoDHZERO   | DH零位状态   |
| B    | gotoPBHCZERO | PBHC零位状态 |
| X    | gotoZERO     | 回到零位状态 |
| Y    | gotoSTOP     | 停止状态     |

**组合按钮状态切换**

| 按钮组合            | 对应状态        | 功能说明                  |
| ------------------- | --------------- | ------------------------- |
| 左触发键(LT) + A    | gotoDH          | DH策略状态                |
| 左触发键(LT) + B    | gotoPBHC        | PBHC策略状态              |
| 左触发键(LT) + X    | gotoMLP         | MLP策略状态               |
| 左触发键(LT) + HOME | gotoBEYONDMIMIC | BOYONDMIMIC策略状态       |
| LB + X              | gotoMLPH        | MLP+H策略状态（增强模式） |

**基础运动控制**

| 控制方式  | 功能                       |
| --------- | -------------------------- |
| 左摇杆Y轴 | 前后移动控制（正向为前进） |
| 左摇杆X轴 | 左右移动控制               |
| 右摇杆X轴 | 机身旋转控制               |

**辅助功能控制**

| 控制方式                    | 功能说明                                   |
| --------------------------- | ------------------------------------------ |
| 方向键水平(D-pad H)         | 增加/减少机身高度（左键增加，右键减少）    |
| Start按钮                   | 若持续按下，可重置移动速度为0              |
| 左触发键(LT) / 右触发键(RT) | 作为组合键使用，需配合其他按钮激活特殊状态 |

**高度调整机制**

高度调整通过以下方式实现：

- D-pad水平方向键：增加/减少高度（左键增加+0.05，右键减少-0.05）
- 高度值限制在0.65到0.90之间，并通过平滑步进（0.01）逐步调整到目标值。

**云卓手柄键位映射**

xMIGCS支持标准云卓手柄控制，开始使用前先确保所有键都回中，以下是详细键位映射关系：

**状态映射关系**

**单按钮状态切换**

| 按钮 | 对应状态 | 功能说明 |
| ---- | -------- | -------- |
| C    | gotoSTOP | 停止状态 |

**组合按钮状态切换**

| 切入策略按钮组合 | 策略内使用按键 | 对应状态                     | 功能说明            |
| ---------------- | -------------- | ---------------------------- | ------------------- |
| H(拨中)          | D              | gotoZERO                     | MLP零位状态         |
| H(拨中)          | A              | gotoMLP                      | MLP策略状态         |
| H(左拨)          | D              | gotoPBHCZERO                 | PBHC零位状态        |
| H(左拨)          | A              | gotoPBHC                     | PBHC策略状态        |
| H(右拨)          | D              | gotoDHZERO                   | DH零位状态          |
| H(右拨)          | A              | gotoDH                       | DH策略状态          |
| E(上拨)          | A              | gotoBEYONDMIMIC              | BEYONDMIMIC策略状态 |
| F(上拨)          | 无             | 手柄控制失能，只有停止键可用 | 用于切换话题控制    |

**基础运动控制**

| 控制方式   | 功能                       |
| ---------- | -------------------------- |
| 左摇杆Y1轴 | 前后移动控制（正向为前进） |
| 左摇杆X1轴 | 左右移动控制               |
| 右摇杆X2轴 | 机身旋转控制               |

**高度调整机制**

高度调整通过以下方式实现：

- E键：每上/下拨一次再回中，增加/减少高度（步长0.05m）
- 高度值限制在min_height到max_height之间，并通过平滑步进（0.01）逐步调整到目标值。

## 上机

先连接网线，然后通过 SSH 登录机器人：

```
ssh ubuntu@192.168.41.1
```

密码：

```
123
```

**检查自启动**： **自启动修改后要重启**

```bash
查询自启动状态
sudo systemctl status proc_manager.service
使能自启动
sudo systemctl enable proc_manager.service
关闭自启动
sudo systemctl disable proc_manager.service
打开自启动
sudo systemctl start proc_manager.service
停止自启动
sudo systemctl stop proc_manager.servic
```

**修改**

**dex_config.yaml文件**

```bash
motor_num: 29        # 电机数量
# actions_size: 12     # action的大小
dt: 0.01

sim: true # true:仿真模式 false:真机模式，一定要修改

debug: false

control_tool: joystick # joystick, xbox, keyboard

joystick:
  max_x_plus_speed: 1.0 #上机改成1.0
  max_x_minus_speed: 0.5
  max_y_speed: 0.5
  max_yaw_speed: 1.0
```



**进入`tmux` 1:**

```bash
sudo su
source /home/ubuntu/xos/setup.bash
ros2 launch body_control body_control.launch.py

```

**进入`tmux` 2:**

```bash

source /home/ubuntu/xos/setup.bash
python3 rl_control_node.py

```

**回到当前文件的terminal:**

```
touch /tmp/rl_start_signal
```

**启动手柄：**

```bash
#进入tmux
tmux
sudo su
source /home/ubuntu/xos/setup.bash
ros2 run joystick joystick_node
```

手柄拨片回正

**pip安装**

```
pip install joblib --break-system-packages
```



# 调试记录

#### 20260316_135237

最开始的只训练走的，存在的问题是走切换到站会有抖动

#### 20260317_175919

加入了 高斯平滑的stand_still，出现问题play出现阶段的混乱， sim2sim直接崩掉

#### 20260323_110051

修改了高斯平滑参数的stand_still，出现问题play出现阶段的混乱， sim2sim直接崩掉

#### 20260323_203020

拉下来的最新的eric代码 同样出现很大的问题，和上面一样,是因为拉下来的时候merge了之前对stand still的修改所以出现问题

#### **20260324_135439**

还原到最开始的stand still

#### 20260324_170917

同步了eric的对has_command的修改，解决了在learning iteration在训练到4W+是发生的左右摇晃的问题

```python
    \# 只在有命令时惩罚

has_command = (

torch.norm(env.command_generator.command[:, :2], dim=1) + torch.abs(env.command_generator.command[:, 2])
) > 0.2 # eric周二进行了修改
```

#### 20260325_103424

应该算是最接近eric代码的，之前的代码有问题且出现崩溃

#### Original policy的数据统计

```python
Hip（髋）
位置（pos, L/R）：mean≈−0.068 / −0.058, std≈0.254 / 0.233, peak_abs≈0.420 / 0.381 rad。
速度（vel, L/R）：mean≈0.059 / −0.057, std≈1.590 / 1.447 rad/s, peak_abs≈3.79 / 3.31 rad/s。
力矩（tau, L/R）：mean≈3.09 / −0.41, std≈43.26 / 41.02, peak_abs≈77.4 / 81.1 Nm。
Knee（膝）
位置：mean≈0.231 / 0.267, std≈0.322 / 0.318, peak_abs≈1.048 / 1.015 rad。
速度：mean≈−0.142 / 0.023, std≈3.197 / 2.962, peak_abs≈9.50 / 8.10 rad/s。
力矩：mean≈5.10 / 5.22, std≈32.64 / 26.68, peak_abs≈79.94 / 62.99 Nm。
Ankle（踝）
位置：mean≈−0.0569 / −0.0703, std≈0.0973 / 0.0948, peak_abs≈0.257 / 0.287 rad。
速度：mean≈−0.0059 / −0.0046, std≈1.312 / 1.265, peak_abs≈4.32 / 3.70 rad/s。
力矩：mean≈7.19 / 10.74, std≈25.27 / 24.67, peak_abs≈55.0 / 55.0 Nm。


左右对称性（按数据类型）
位置（pos）
hip RMS 差 ≈ 0.482, 相关系数 ≈ −0.961, 滞后 ≈ −0.56 s（右侧领先约0.56 s）。
knee RMS 差 ≈ 0.564, 相关系数 ≈ −0.547, 滞后 ≈ −0.54 s。
ankle RMS 差 ≈ 0.140, 相关系数 ≈ −0.056, 滞后 ≈ −0.55 s。
速度（vel）
hip RMS 差 ≈ 2.862, 相关 ≈ −0.774, 滞后 ≈ −0.55 s。
knee RMS 差 ≈ 4.727, 相关 ≈ −0.175, 滞后 ≈ −0.55 s。
ankle RMS 差 ≈ 1.818, 相关 ≈ 0.005, 滞后 ≈ −0.59 s。
力矩（tau）
hip RMS 差 ≈ 83.01, 相关 ≈ −0.937, 滞后 ≈ +0.56 s（与位置/速度滞后方向相反）。
knee RMS 差 ≈ 49.60, 相关 ≈ −0.392, 滞后 ≈ −0.55 s。
ankle RMS 差 ≈ 46.24, 相关 ≈ −0.704, 滞后 ≈ +0.60 s。

机器人当前步态主要是靠髋关节维持表面的左右交替，所以从外形上看还像在走；但膝关节的左右动态协调已经明显变差，而踝关节在位置和速度层面几乎失去稳定对称关系，只能通过较大的补偿性力矩去维持足端接触和稳定



```

全身

**时间窗与采样**

- 时间范围：**40.011 s – 45.001 s**
- 总帧数：**500**
- 采样间隔：**0.01 s**
- 采样频率：**100 Hz**

#### 20260330_102752

在删除了上周的修改的代码之后，重新和eric联系得出的新的原始的代码

##### 出现问题

问题很大  身体晃动 手臂不能往后打开

[](/home/mig/Videos/Screencasts/20260330_102752.webm)

#### 20260330_134105

要解决顿挫问题 使用上身身体后仰

**policy_add_waist_pitch**

```C
def waist_pitch_target_exp(
    env: BaseEnv,
    target_rad: float = -0.10,   # 先假设负号是“后仰一点”
    std: float = 0.08,
    asset_cfg: SceneEntityCfg = SceneEntityCfg("robot"),
) -> torch.Tensor:
    """Reward waist_pitch_joint to stay near a small backward lean target."""
    asset: Articulation = env.scene[asset_cfg.name]

    # 取 waist_pitch_joint 的关节角
    waist_pitch = asset.data.joint_pos[:, asset_cfg.joint_ids].squeeze(-1)

    # 误差
    error = waist_pitch - target_rad

    # 高斯型奖励：越接近 target_rad 越接近 1
    reward = torch.exp(-torch.square(error) / (std ** 2))

    # 可选：只在有运动命令时启用
    has_command = (
        torch.norm(env.command_generator.command[:, :2], dim=1)
        + torch.abs(env.command_generator.command[:, 2])
    ) > 0.05

    return reward * has_command
```



```c
在你的 reward cfg 里加一项，类似这样：

waist_pitch_target = RewTerm(
    func=mdp.waist_pitch_target_exp,
    weight=0.5,
    params={
        "target_rad": -0.10,
        "std": 0.08,
        "asset_cfg": SceneEntityCfg("robot", joint_names=["waist_pitch_joint"]),
    },
)

这里几个参数的意义是：

target_rad=-0.10：目标后仰大约 5.7°
std=0.08：允许一定浮动，不要卡得太死
weight=0.5：先小一点，不然容易和速度跟踪奖励冲突
```

##### 出现问题：

出现左右腿的长短脚，行走不稳定

[](/home/mig/Videos/Screencasts/20260330_134105_39500.webm)



#### 20260331_162700

针对上面的问题，提出修改link的版本

**相对 pelvis 的版本**

```c
def waist_link_relative_pitch_exp(
    env: BaseEnv,
    target_rad: float = -0.10,
    std: float = 0.08,
    asset_cfg: SceneEntityCfg = SceneEntityCfg("robot"),
) -> torch.Tensor:
    """Reward waist_pitch_link to lean slightly backward relative to pelvis."""
    asset: Articulation = env.scene[asset_cfg.name]

   
    pelvis_quat = asset.data.body_quat_w[:, asset_cfg.body_ids[0], :]
    waist_quat  = asset.data.body_quat_w[:, asset_cfg.body_ids[1], :]

    # 相对姿态：q_rel = q_pelvis^{-1} * q_waist
    pelvis_inv = math_utils.quat_inv(pelvis_quat)
    waist_rel_quat = math_utils.quat_mul(pelvis_inv, waist_quat)

    roll, pitch, yaw = math_utils.euler_xyz_from_quat(waist_rel_quat)

    error = pitch - target_rad
    reward = torch.exp(-torch.square(error) / (std ** 2))

    has_command = (
        torch.norm(env.command_generator.command[:, :2], dim=1)
        + torch.abs(env.command_generator.command[:, 2])
    ) > 0.05

    return reward * has_command
```

对应配置：

```c
waist_link_relative_pitch = RewTerm(
    func=mdp.waist_link_relative_pitch_exp,
    weight=0.5,
    params={
        "target_rad": -0.10,
        "std": 0.08,
        "asset_cfg": SceneEntityCfg("robot", body_names=["pelvis", "waist_pitch_link"]),
    },
)
```

##### 出现问题

目前在mujoco上变现良好，等待real测试，胳膊有问题

[](/home/mig/Videos/Screencasts/20260331_162700.webm)



#### 20260401_134434

为了缓解摆臂存在的问题

```python

def arm_swing_phase_coupled_exp(
    env: BaseEnv,
    sensor_cfg: SceneEntityCfg,
    asset_cfg: SceneEntityCfg,
    std: float = 0.40,
    target_shoulder_pitch_rad: float = -0.35,
    deadzone_rad: float = 0.05,
    min_forward_cmd: float = 0.10,
) -> torch.Tensor:
    """相位耦合摆臂（对侧手在摆动脚相时肩 pitch 贴近目标）。

    约定：双脚 `body_ids` 顺序为 [左踝, 右踝]；双肩 `joint_ids` 顺序为 [左肩 pitch, 右肩 pitch]。
    左脚摆动（左离地、右支撑）→ 奖励右臂肩 pitch；右脚摆动 → 奖励左臂。
    仅在 yaw 系前进命令 `vx_cmd > min_forward_cmd` 时生效。

    调参：摆臂不够 → 增大 weight 或把 |target_shoulder_pitch_rad| 加大（向“身后”一侧调）；
    过僵或抖动 → 增大 std / deadzone，或略降 weight。

    输入shape: current_contact_time=(N,2), joint_pos=(N,D), command=(N,3)。
    """
    assert len(sensor_cfg.body_ids) == 2, "arm_swing_phase_coupled_exp: need exactly 2 foot bodies [L, R]"
    assert len(asset_cfg.joint_ids) == 2, "arm_swing_phase_coupled_exp: need 2 joints [L shoulder_pitch, R shoulder_pitch]"

    contact_sensor: ContactSensor = env.scene.sensors[sensor_cfg.name]
    asset: Articulation = env.scene[asset_cfg.name]

    contact_time = contact_sensor.data.current_contact_time[:, sensor_cfg.body_ids]
    left_on = contact_time[:, 0] > 0.0
    right_on = contact_time[:, 1] > 0.0
    left_swing = (~left_on) & right_on
    right_swing = left_on & (~right_on)

    j0, j1 = int(asset_cfg.joint_ids[0]), int(asset_cfg.joint_ids[1])
    q_l = asset.data.joint_pos[:, j0]
    q_r = asset.data.joint_pos[:, j1]

    err_l = q_l - target_shoulder_pitch_rad
    err_r = q_r - target_shoulder_pitch_rad
    abs_l = torch.abs(err_l)
    abs_r = torch.abs(err_r)
    eff_l = torch.clamp(abs_l - deadzone_rad, min=0.0)
    eff_r = torch.clamp(abs_r - deadzone_rad, min=0.0)
    r_l = torch.exp(-torch.square(eff_l) / (std**2))
    r_r = torch.exp(-torch.square(eff_r) / (std**2))

    # 右脚摆动 → 左臂向后；左脚摆动 → 右臂向后
    phase_reward = right_swing.float() * r_l + left_swing.float() * r_r

    vx_cmd = env.command_generator.command[:, 0]
    forward_gate = (vx_cmd > min_forward_cmd).float()

    return phase_reward * forward_gate

```

- 脚：`sensor_cfg.body_ids` 必须为 2 个，顺序 [左踝, 右踝]（与 `feet_slide_update` 一致：`ankle_roll_l_link`, `ankle_roll_r_link`）。
- 相位：单支撑且摆动脚离地时
  - 右脚摆动（左撑地、右离地）→ 用指数核奖励 左肩 `shoulder_pitch` 接近 `target_shoulder_pitch_rad`
  - 左脚摆动 → 奖励 右肩
- 门控：仅当 yaw 系前进指令 `vx_cmd > min_forward_cmd` 时给奖励（避免站立/小速度乱摆）。
- 平滑：对肩角误差有 `deadzone_rad`，再 `exp(-eff_err²/std²)`。

##### 出现问题：

胳膊不能往后的问题依然存在  所以修正了motion_loader 里面的数据

[](/home/mig/Videos/Screencasts/20260402_101758.webm)

#### 20260402_101758

尝试在motion_loader 里修改AMP的arm姿态

```python
elbow_offset = -0.5 #原为-0.3

left_arm_pos[:, -1]  += elbow_offset

right_arm_pos[:, -1] += elbow_offset
shoulder_pitch_offset = 0.4 #原为0.2

left_arm_pos[:, 0]  += shoulder_pitch_offset

right_arm_pos[:, 0] += shoulder_pitch_offset
```

##### 出现问题

僵硬问题明显，上臂后仰但是小臂向前弯曲，不能超过身侧

[](/home/mig/Videos/Screencasts/20260402_101758.webm)

#### 20260403_163237

解决手臂的问题

```python
def joint_torque_pair_diff_l2(
    env: BaseEnv,
    asset_cfg: SceneEntityCfg,
    sensor_cfg: SceneEntityCfg | None = None,
    require_double_support: bool = False,
) -> torch.Tensor:
    """惩罚一对关节施加力矩的差异 (tau0 - tau1)^2。
    用于左右膝 pitch 等成对关节的“负载均衡”正则。注意：单脚支撑时两膝力矩本就不对称，
    若 require_double_support=False，会对正常步态产生拉扯，权重务必很小或改为仅双支撑。
    require_double_support=True 时需传 sensor_cfg（与 feet 一致的两踝 body），仅在双脚同时接触时计惩罚。
    输入shape: applied_torque=(N,D)；可选 current_contact_time=(N,2)。
    """
    asset: Articulation = env.scene[asset_cfg.name]
    if len(asset_cfg.joint_ids) != 2:
        raise ValueError("joint_torque_pair_diff_l2: asset_cfg.joint_ids must have exactly 2 entries [L, R]")
    j0, j1 = int(asset_cfg.joint_ids[0]), int(asset_cfg.joint_ids[1])
    t0 = asset.data.applied_torque[:, j0]
    t1 = asset.data.applied_torque[:, j1]
    penalty = torch.square(t0 - t1)
    if require_double_support:
        if sensor_cfg is None:
            raise ValueError("joint_torque_pair_diff_l2: sensor_cfg required when require_double_support=True")
        contact_sensor: ContactSensor = env.scene.sensors[sensor_cfg.name]
        contact_time = contact_sensor.data.current_contact_time[:, sensor_cfg.body_ids]
        in_contact = contact_time > 0.0
        dual = torch.sum(in_contact.int(), dim=1) == 2
        penalty = penalty * dual.float()
    return penalty

```

双脚都在地时，左右膝更应对称，和步态冲突小。

##### 出现问题

[](/home/mig/Videos/Screencasts/20260403_163237.webm)

躯体僵硬， 不应该对motion_loader做大的修改， 不应该加入太多的rewards



#### 20260407_164421

取消了对arm做的改动，但是依然使用的是：

```
amp_motion_files = glob.glob("legged_lab/envs/Dex/datasets/motion_walk/*.npz")
```

[](/home/mig/Videos/Screencasts/20260407_164421.webm)







#### 20260407_182804

取消了对arm做的改动，使用的是：

```
amp_motion_files = glob.glob("legged_lab/envs/Dex/datasets/motion_47/*.npz")
```

[](/home/mig/Videos/Screencasts/20260407_182804.webm)



#### **20260408_191452**

为了缓解arm不后仰

```python

def arm_swing_phase_coupled_exp(
    env: BaseEnv,
    sensor_cfg: SceneEntityCfg,
    asset_cfg: SceneEntityCfg,
    std: float = 0.40,
    target_shoulder_pitch_rad: float = -0.45, #原始为-0.35
    deadzone_rad: float = 0.05,
    min_forward_cmd: float = 0.10,
) -> torch.Tensor:
  ....
```

[](/home/mig/Videos/Screencasts/20260408_191452.webm)

#####  出现问题

arm不后仰得到缓解

左右脚长短不一得到缓解



#### 20260409_154531

arm的target 调大

```python
    arm_swing_phase = RewTerm(
        func=mdp.arm_swing_phase_coupled_exp,
        weight=0.12,
        params={
            "sensor_cfg": SceneEntityCfg(
                "contact_sensor", body_names=["ankle_roll_l_link", "ankle_roll_r_link"]
            ),
            "asset_cfg": SceneEntityCfg(
                "robot",
                joint_names=["shoulder_pitch_l_joint", "shoulder_pitch_r_joint"],
            ),
            "std": 0.40,
            "target_shoulder_pitch_rad": -0.55,#从-0.35  到-0.45  再到-0.55
            "deadzone_rad": 0.05,
            "min_forward_cmd": 0.10,
        },
    )
```

waist_link 调大

```python
    waist_back_lean = RewTerm(
        func=mdp.waist_link_relative_pitch_exp,
        weight=0.6,
        params={
            "target_rad": -0.15,#原始为-0.1
            "std": 0.06,
            "deadzone_rad": 0.02,
            "require_command": True,
            "asset_cfg": SceneEntityCfg("robot", body_names=["pelvis", "waist_pitch.*"]),
        },
    )
```

##### 出现问题

[](/home/mig/Videos/Screencasts/20260409_154531.webm)

行走僵硬





#### 20260413_151003

锁定waist

名单上的关节：策略动作在 buffer/clip/目标里都被强制成「等价于只跟踪默认角」；其它关节照旧由 RL 控。 打开或关掉只靠 `action_locked_to_default_joint_names` 是否为空。

[](/home/mig/Videos/Screencasts/20260413_151003.webm)

##### 出现问题：

长短脚 左右脚腾空时间不一致，身体晃动明显  顿挫明显



#### 20260416_151110

解锁waist

[](/home/mig/Videos/Screencasts/20260413_151110.webm)

##### 出现问题：

和上面一样，同样有腾空时间不一样的问题  右脚走路快一点  左脚走路慢











#### 20260421_104720

修改盆骨

添加

```python
def pelvis_pitch_l2(
    env: BaseEnv,
    target_rad: float = 0.0,
    deadzone_rad: float = 0.03,
    asset_cfg: SceneEntityCfg = SceneEntityCfg("robot"),
) -> torch.Tensor:
    """惩罚 pelvis pitch 偏离目标角度，抑制骨盆前倾/后仰。
    输入shape: body_quat_w=(N,B,4)。
    """
    asset: Articulation = env.scene[asset_cfg.name]
    body_quat = asset.data.body_quat_w[:, asset_cfg.body_ids[0], :]
    _, pitch, _ = math_utils.euler_xyz_from_quat(body_quat)
    error = pitch - target_rad
    eff_error = torch.clamp(torch.abs(error) - deadzone_rad, min=0.0)
    return torch.square(eff_error)

```

效果并不明显

[](/home/mig/Videos/Screencasts/20260421_104720.webm)





#### 20260422_142159

调整了weight   下一次尝试不同的约束

```python
pelvis_pitch_l2 = RewTerm(
    func=mdp.pelvis_pitch_l2,
    weight=-2.5,#原始为-1.5
    params={
        "asset_cfg": SceneEntityCfg("robot", body_names="pelvis"),
        "target_rad": 0.0,
        "deadzone_rad": 0.03,
    },
)
```

[](/home/mig/Videos/Screencasts/20260422_142159.webm)



#### 20260423_103721



```python
    pelvis_pitch_l2 = RewTerm(
        func=mdp.pelvis_pitch_l2,
        weight=-2.5,
        params={
            "asset_cfg": SceneEntityCfg("robot", body_names="pelvis"),
            "target_rad": -0.05, #修改了后仰角度
            "deadzone_rad": 0.03,
        },
    )
```

有点长短脚



[](/home/mig/Videos/Screencasts/20260423_103721.webm)





```
在 legged_lab/mdp/rewards.py 增强了 pelvis_pitch_l2：

新增 forward_weight / backward_weight
支持前后非对称惩罚：前倾（pitch < target）可罚得更重
在 legged_lab/envs/Dex/env_cfg/dex_env_cfg.py 调了姿态相关奖励：


pelvis_pitch_l2.weight: -2.5 -> -3.5
pelvis_pitch_l2.target_rad: -0.05 -> -0.02
pelvis_pitch_l2.deadzone_rad: 0.03 -> 0.02
pelvis_pitch_l2 新增参数：forward_weight=2.2, backward_weight=0.8


waist_back_lean.weight: 0.6 -> 0.45
waist_back_lean.target_rad: -0.10 -> -0.08
waist_back_lean.std: 0.06 -> 0.08
waist_back_lean.deadzone_rad: 0.02 -> 0.03
新增 pelvis_orientation_l2（weight=-0.5）约束骨盆整体姿态
```









## 数据分析

#### 20260401_134434

| 关节对      | 位置幅值L | 位置幅值R | 速度峰值L | 速度峰值R | 力矩峰值L | 力矩峰值R |
| ----------- | --------- | --------- | --------- | --------- | --------- | --------- |
| hip_roll    | 0.139     | 0.112     | 0.803     | 0.727     | 107.060   | 101.205   |
| hip_pitch   | 0.624     | 0.646     | 3.201     | 3.312     | 87.469    | 84.317    |
| hip_yaw     | 0.084     | 0.101     | 0.761     | 0.811     | 18.468    | 15.064    |
| knee_pitch  | 1.003     | 1.031     | 7.271     | 7.472     | 145.466   | 126.125   |
| ankle_pitch | 0.449     | 0.447     | 5.713     | 5.539     | 55.000    | 55.000    |
| ankle_roll  | 0.175     | 0.191     | 1.346     | 1.418     | 12.519    | 10.488    |























