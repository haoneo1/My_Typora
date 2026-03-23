#### Train

```
python legged_lab/scripts/train.py --task=Dex_walk --headless --logger=tensorboard

```

#### Play

```
python legged_lab/scripts/play.py --task=Dex_walk --num_envs=32
```

#### SIM2SIM

**1、启动运控在xmigcs目录下运行：**

```bash
source /opt/ros/humble/setup.bash
python3 rl_control_node.py 

在新的终端运行：touch /tmp/rl_start_signal


```

**2、启动手柄在xmigcs目录下运行**

```
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
ros2 run joy joy_node --ros-args --remap joy:=xbox_data
```

**3、启动仿真在xsim_mujoco文件**

```
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
python scripts/simulator_view_asyn.py -m dex_evt_hand


```

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
