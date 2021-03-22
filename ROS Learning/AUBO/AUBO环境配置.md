# kinetic下AUBO环境配置
资源和教程参见：[aubo官方github资源](https://github.com/lg609/aubo_robot)

1. 首先建立工作空间（a catkin workspace）。
```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin_make
```

2. 进入工作空间下的src目录，克隆github资源：
```
cd src
git clone https://github.com/lg609/aubo_robot.git
```

或手动打开网址下载该资源，解压到上述目录中。

最终资源目录看起来应该是：~/catkin_ws/src/aubo_robot

3. 安装依赖包
```
sudo apt-get install ros-kinetic-moveit
sudo apt-get install ros-kinetic-moveit-visual-tools
sudo apt-get install ros-kinetic-industrial-core
```

4. 进行编译
```
cd ~/catkin_ws
catkin_make
```

5. 刷新环境
```
source ~/catkin_ws/devel/setup.bash
```

6. 要使用gazebo仿真，需要安装gazebo依赖

```
sudo apt-get install ros-kinetic-desktop-full

sudo apt-get install ros-kinetic-transmission-interface

sudo apt-get install ros-kinetic-gazebo-ros-control

sudo apt-get install ros-kinetic-joint-state-controller

sudo apt-get install ros-kinetic-effort-controllers

sudo apt-get install ros-kinetic-position-controllers

sudo apt-get install ros-kinetic-control-manager
```

7. 仿真测试

rviz

```
roslaunch aubo_i5_moveit_config moveit_planning_execution.launch robot_ip:=127.0.0.1
```

新打开一个终端，运行控制demo

```
source ~/catkin_ws/devel/setup.bash
roslaunch aubo_demo MoveGroupInterface_To_Kinetic.launch
```

新打开一个终端，运行gazebo

```
source ~/catkin_ws/devel/setup.bash
roslaunch aubo_gazebo aubo_i5_gazebo_control.launch
```

