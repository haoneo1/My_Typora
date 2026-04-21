## 上机

先连接网线，然后通过 SSH 登录机器人：

```
ssh ubuntu@192.168.41.1
```

密码：

```
123
```

**检查自启动： 自启动修改后要重启**

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

**修改 `dex_config.yaml`文件**

```bash
motor_num: 29        # 电机数量
# actions_size: 12     # action的大小
dt: 0.01

sim: true # true:仿真模式 false:真机模式，一定要修改

debug: false

control_tool: joystick # joystick, xbox, keyboard

joystick:
  initial_height: 0.87
  max_height: 0.87
  min_height: 0.65
  x_command_offset: 0.0
  y_command_offset: 0.0
  yaw_command_offset: 0.0
  max_x_plus_speed: 1.0 #上机改成1.0
  max_x_minus_speed: 0.5
  max_y_speed: 0.5
  max_yaw_speed: 1.0
```



进入`tmux` 1:

```bash
sudo su
source /home/ubuntu/xos/setup.bash
ros2 launch body_control body_control.launch.py

```

进入`tmux` 2:

```bash
source /home/ubuntu/xos/setup.bash
python3 rl_control_node.py

```

回到当前文件的terminal:

```
touch /tmp/rl_start_signal
```

启动手柄：

```bash
#进入tmux
tmux
sudo su
source /home/ubuntu/xos/setup.bash
ros2 run joystick joystick_node
```

**手柄拨片回正**

pip安装

```
pip install joblib --break-system-packages
```

