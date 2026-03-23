#### Train

```
python legged_lab/scripts/train.py --task=Dex_walk --headless --logger=tensorboard
```

#### Play

```
python legged_lab/scripts/play.py --task=Dex_walk --num_envs=32
```

#### MUJOCO仿真

**仿真都需要启动xwalk-run conda环境**

##### 启动主控制节点

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

#####  启动XBOX手柄数据节点

在xmigcs文件下：

```bash
export ROS_DOMAIN_ID=99
source /opt/ros/humble/setup.bash
ros2 run joy joy_node --ros-args --remap joy:=xbox_data
```

##### 启动仿真

在MuJoCo文件

```bash
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=99
python scripts/simulator_view_asyn.py -m dex_evt_hand
```

##### 控制器使用说明

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
