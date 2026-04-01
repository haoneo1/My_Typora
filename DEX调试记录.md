## 学习记录

**训练 Dex 机器人在 Isaac Lab / Isaac Sim 里学会稳定行走。**

通常会包含这些内容：

- command 生成：给目标线速度、角速度
- observation 设计：机器人姿态、关节状态、接触状态等
- action 设计：关节控制命令
- reward 设计：速度跟踪、姿态稳定、动作平滑、足部行为等
- termination 设计：摔倒、超界、异常状态结束



### 训练参数传递

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





## 基础代码

#### Train

```
PYTHONNOUSERSITE=1  python legged_lab/scripts/train.py --task=Dex_walk --headless --logger=tensorboard

CUDA_VISIBLE_DEVICES=1 PYTHONUNBUFFERED=1 \
python legged_lab/scripts/train.py --task=Dex_walk --headless --logger=tensorboard
  --device=cuda:0 --info
```

#### Play

```
python legged_lab/scripts/play.py --task=Dex_walk --num_envs=32
```

#### SIM2SIM

**1、启动运控在xmigcs目录下运行：**

```bash
新策略要放到
/home/mig/Documents/xmigcs/policy/mlp/model 
然后到
/home/mig/Documents/xmigcs/policy/mlp/model/mlp.yaml 修改model_path

source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
python3 rl_control_node.py 

在新的终端运行：touch /tmp/rl_start_signal


```

**2、启动手柄在xmigcs目录下运行**

```
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
ros2 run joy joy_node --ros-args --remap joy:=xbox_data
```

**3、启动仿真在xsim_mujoco文件** /home/mig/Documents/xsim_mujoco

```bash
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
# python scripts/simulator_view_asyn.py -m dex_evt_hand
python scripts/simulator_view_asyn.py -m evt2


/home/mig/Documents/xmigcs/policy/mlp/model
```

4、**Ubuntu / GNOME 自带录屏**
 直接按 `Print` 键打开截图工具，切到录屏模式后可以录整个屏幕或选区；快捷键 `Shift + Ctrl + Alt + R` 也可以直接开始/停止录屏。录好的视频默认会存到 `Videos/Screencasts`



**4、具体手柄操作**

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

## 调试记录

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

#### Original policy

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

#### 20260330_134105







------

# 位置数据 `pos`（mean / std / peak_abs, 单位：rad）

## 左上肢

- `shoulder_pitch_l_joint`：**-0.022 / 0.279 / 0.436**
- `shoulder_roll_l_joint`：**0.134 / 0.023 / 0.172**
- `shoulder_yaw_l_joint`：**-0.241 / 0.164 / 0.465**
- `elbow_pitch_l_joint`：**-0.349 / 0.192 / 0.653**
- `elbow_yaw_l_joint`：**≈0 / ≈0 / ≈0**
- `wrist_pitch_l_joint`：**0.003 / 0.004 / 0.011**
- `wrist_roll_l_joint`：**≈0 / ≈0 / ≈0**

## 右上肢

- `shoulder_pitch_r_joint`：**0.107 / 0.284 / 0.507**
- `shoulder_roll_r_joint`：**-0.110 / 0.027 / 0.149**
- `shoulder_yaw_r_joint`：**0.052 / 0.172 / 0.311**
- `elbow_pitch_r_joint`：**-0.149 / 0.182 / 0.459**
- `elbow_yaw_r_joint`：**≈0 / ≈0 / ≈0**
- `wrist_pitch_r_joint`：**0.001 / 0.003 / 0.009**
- `wrist_roll_r_joint`：**≈0 / ≈0 / ≈0**

## 腰部

- `waist_yaw_joint`：**0.022 / 0.063 / 0.126**
- `waist_roll_joint`：**0.010 / 0.044 / 0.087**
- `waist_pitch_joint`：**0.016 / 0.030 / 0.083**

## 左下肢

- `hip_roll_l_joint`：**-0.019 / 0.029 / 0.076**
- `hip_pitch_l_joint`：**-0.047 / 0.247 / 0.413**
- `hip_yaw_l_joint`：**0.005 / 0.039 / 0.080**
- `knee_pitch_l_joint`：**0.265 / 0.334 / 1.043**
- `ankle_pitch_l_joint`：**-0.066 / 0.096 / 0.272**
- `ankle_roll_l_joint`：**0.034 / 0.056 / 0.155**

## 右下肢

- `hip_roll_r_joint`：**0.012 / 0.031 / 0.069**
- `hip_pitch_r_joint`：**-0.070 / 0.233 / 0.408**
- `hip_yaw_r_joint`：**-0.001 / 0.044 / 0.082**
- `knee_pitch_r_joint`：**0.234 / 0.305 / 1.029**
- `ankle_pitch_r_joint`：**-0.065 / 0.088 / 0.281**
- `ankle_roll_r_joint`：**-0.076 / 0.051 / 0.209**

------

# 速度数据 `vel`（std / peak_abs, 单位：rad/s）

## 左上肢

- `shoulder_pitch_l_joint`：**1.629 / 2.707**
- `shoulder_roll_l_joint`：**0.257 / 0.566**
- `shoulder_yaw_l_joint`：**0.984 / 2.357**
- `elbow_pitch_l_joint`：**1.141 / 1.974**
- `elbow_yaw_l_joint`：**≈0 / ≈0**
- `wrist_pitch_l_joint`：**0.027 / 0.068**
- `wrist_roll_l_joint`：**≈0 / ≈0**

## 右上肢

- `shoulder_pitch_r_joint`：**1.658 / 2.743**
- `shoulder_roll_r_joint`：**0.287 / 0.531**
- `shoulder_yaw_r_joint`：**1.068 / 2.460**
- `elbow_pitch_r_joint`：**1.092 / 1.927**
- `elbow_yaw_r_joint`：**≈0 / ≈0**
- `wrist_pitch_r_joint`：**0.021 / 0.059**
- `wrist_roll_r_joint`：**≈0 / ≈0**

## 腰部

- `waist_yaw_joint`：**0.483 / 1.016**
- `waist_roll_joint`：**0.448 / 0.749**
- `waist_pitch_joint`：**0.343 / 0.669**

## 左下肢

- `hip_roll_l_joint`：**0.345 / 0.687**
- `hip_pitch_l_joint`：**1.589 / 3.765**
- `hip_yaw_l_joint`：**0.309 / 0.812**
- `knee_pitch_l_joint`：**3.217 / 9.254**
- `ankle_pitch_l_joint`：**1.303 / 4.008**
- `ankle_roll_l_joint`：**0.589 / 1.522**

## 右下肢

- `hip_roll_r_joint`：**0.353 / 0.740**
- `hip_pitch_r_joint`：**1.451 / 3.497**
- `hip_yaw_r_joint`：**0.309 / 0.698**
- `knee_pitch_r_joint`：**2.810 / 8.738**
- `ankle_pitch_r_joint`：**1.224 / 3.769**
- `ankle_roll_r_joint`：**0.561 / 1.464**

------

# 力矩数据 `torque`（mean / std / peak_abs, 单位：Nm）

## 左上肢

- `shoulder_pitch_l_joint`：**-1.289 / 7.471 / 18.395**
- `shoulder_roll_l_joint`：**2.599 / 2.656 / 8.199**
- `shoulder_yaw_l_joint`：**0.428 / 1.767 / 4.586**
- `elbow_pitch_l_joint`：**-0.575 / 3.533 / 7.402**
- `elbow_yaw_l_joint`：**≈0 / ≈0 / ≈0**
- `wrist_pitch_l_joint`：**≈0 / ≈0 / ≈0**
- `wrist_roll_l_joint`：**≈0 / ≈0 / ≈0**

## 右上肢

- `shoulder_pitch_r_joint`：**0.305 / 7.274 / 14.911**
- `shoulder_roll_r_joint`：**-2.335 / 2.661 / 8.621**
- `shoulder_yaw_r_joint`：**-0.028 / 1.846 / 4.730**
- `elbow_pitch_r_joint`：**0.065 / 3.398 / 7.113**
- `elbow_yaw_r_joint`：**≈0 / ≈0 / ≈0**
- `wrist_pitch_r_joint`：**≈0 / ≈0 / ≈0**
- `wrist_roll_r_joint`：**≈0 / ≈0 / ≈0**

## 腰部

- `waist_yaw_joint`：**-0.253 / 14.305 / 32.712**
- `waist_roll_joint`：**-0.728 / 7.407 / 18.417**
- `waist_pitch_joint`：**-1.531 / 3.160 / 12.337**

## 左下肢

- `hip_roll_l_joint`：**22.637 / 27.301 / 100.055**
- `hip_pitch_l_joint`：**-0.617 / 41.492 / 78.495**
- `hip_yaw_l_joint`：**-1.057 / 5.757 / 14.011**
- `knee_pitch_l_joint`：**4.445 / 30.864 / 76.756**
- `ankle_pitch_l_joint`：**9.482 / 25.605 / 55.000**
- `ankle_roll_l_joint`：**2.578 / 4.114 / 10.788**

## 右下肢

- `hip_roll_r_joint`：**-23.169 / 29.491 / 103.712**
- `hip_pitch_r_joint`：**3.565 / 40.763 / 81.504**
- `hip_yaw_r_joint`：**1.355 / 5.606 / 14.410**
- `knee_pitch_r_joint`：**5.611 / 26.310 / 70.200**
- `ankle_pitch_r_joint`：**7.656 / 24.759 / 55.000**
- `ankle_roll_r_joint`：**-3.923 / 5.689 / 15.386**

------

# 左右对称性统计（RMS差 / corr0 / best_lag_s / best_corr）

## 位置 `pos`

- `shoulder_pitch`：**0.577 / -0.996 / -0.57 / 0.997**
- `shoulder_roll`：**0.249 / -0.824 / -0.57 / 0.955**
- `shoulder_yaw`：**0.300 / 0.934 / -0.57 / 0.980**
- `elbow_pitch`：**0.419 / -0.949 / -0.57 / 0.990**
- `wrist_pitch`：**0.007 / -0.816 / -0.57 / 0.910**
- `wrist_roll`：**≈0 / ≈0 / ≈0 / ≈0**
- `hip_roll`：**0.034 / 0.882 / -0.01 / 0.882**
- `hip_pitch`：**0.475 / -0.957 / -0.57 / 0.996**
- `hip_yaw`：**0.031 / 0.738 / -0.01 / 0.738**
- `knee_pitch`：**0.564 / -0.551 / 0.55 / 0.995**
- `ankle_pitch`：**0.131 / -0.005 / 0.56 / 0.946**
- `ankle_roll`：**0.121 / 0.587 / -0.01 / 0.587**

------

## 速度 `vel`

- `shoulder_pitch`：**3.287 / -0.982 / -0.57 / 0.993**
- `shoulder_roll`：**0.557 / -0.871 / -0.57 / 0.966**
- `shoulder_yaw`：**0.745 / 0.740 / -0.57 / 0.915**
- `elbow_pitch`：**2.142 / -0.825 / -0.57 / 0.958**
- `wrist_pitch`：**0.105 / -0.861 / -0.57 / 0.930**
- `wrist_roll`：**≈0 / ≈0 / ≈0 / ≈0**
- `hip_roll`：**0.139 / 0.922 / -0.01 / 0.922**
- `hip_pitch`：**2.873 / -0.774 / -0.58 / 0.988**
- `hip_yaw`：**0.399 / 0.168 / -0.01 / 0.168**
- `knee_pitch`：**4.632 / -0.177 / -0.58 / 0.985**
- `ankle_pitch`：**1.742 / 0.051 / 0.58 / 0.829**
- `ankle_roll`：**0.625 / 0.409 / -0.01 / 0.409**

------

## 力矩 `torque`

- `shoulder_pitch`：**14.170 / -0.824 / -0.57 / 0.940**
- `shoulder_roll`：**6.688 / -0.442 / -0.57 / 0.796**
- `shoulder_yaw`：**1.895 / 0.482 / -0.57 / 0.667**
- `elbow_pitch`：**6.521 / -0.753 / -0.57 / 0.903**
- `wrist_pitch`：**0.763 / -0.832 / -0.57 / 0.908**
- `wrist_roll`：**≈0 / ≈0 / ≈0 / ≈0**
- `hip_roll`：**52.508 / 0.594 / -0.01 / 0.594**
- `hip_pitch`：**80.936 / -0.931 / -0.56 / 0.979**
- `hip_yaw`：**7.968 / 0.107 / -0.01 / 0.107**
- `knee_pitch`：**47.697 / -0.387 / -0.57 / 0.944**
- `ankle_pitch`：**46.413 / -0.696 / 0.56 / 0.942**
- `ankle_roll`：**8.501 / 0.412 / -0.01 / 0.412**

------

# 主要极值（按峰值绝对值）

## 位置峰值较大关节（rad）

- `knee_pitch_l_joint`：**1.043**
- `knee_pitch_r_joint`：**1.029**
- `elbow_pitch_l_joint`：**0.653**
- `shoulder_pitch_r_joint`：**0.507**
- `shoulder_yaw_l_joint`：**0.465**
- `elbow_pitch_r_joint`：**0.459**

## 速度峰值较大关节（rad/s）

- `knee_pitch_l_joint`：**9.254**
- `knee_pitch_r_joint`：**8.738**
- `ankle_pitch_l_joint`：**4.008**
- `hip_pitch_l_joint`：**3.765**
- `ankle_pitch_r_joint`：**3.769**
- `hip_pitch_r_joint`：**3.497**

## 力矩峰值较大关节（Nm）

- `hip_roll_r_joint`：**103.712**
- `hip_roll_l_joint`：**100.055**
- `hip_pitch_r_joint`：**81.504**
- `hip_pitch_l_joint`：**78.495**
- `knee_pitch_l_joint`：**76.756**
- `knee_pitch_r_joint`：**70.200**
- `ankle_pitch_l_joint`：**55.000**
- `ankle_pitch_r_joint`：**55.000**



































