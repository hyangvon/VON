# 在Ubuntu中安装ROS Kinetic
参考[官方教程](http://wiki.ros.org/cn/kinetic/Installation/Ubuntu)

## 1. 安装
ROS Kinetic 只 支持Wily (Ubuntu 15.10), Xenial (Ubuntu 16.04) 和Jessie (Debian 8) 的debian包。

### 1.1 配置Ubuntu软件仓库
配置你的Ubuntu软件仓库（repositories）以允许使用“restricted”“universe”和“multiverse”存储库。你可以根据Ubuntu软件仓库指南来完成这项工作。

### 1.2 设置sources.list
设置电脑以安装来自packages.ros.org的软件。
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

### 1.3 设置密钥
```
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```
若无法连接到密钥服务器，可以尝试替换上面命令中的 hkp://keyserver.ubuntu.com:80 为 hkp://pgp.mit.edu:80 。

### 1.4 安装
首先，确保你的Debian软件包索引是最新的：
```
sudo apt-get update
```
在ROS中，有很多不同的库和工具。我们提供了四种默认的配置来帮助你开始。你也可以单独安装ROS包。

​	**桌面完整版: (推荐)** : 包含ROS、rqt、rviz、机器人通用库、2D/3D 模拟器、导航以及2D/3D感知
```
sudo apt-get install ros-kinetic-desktop-full
```

​	**桌面版安装**: 包含ROS、rqt、rviz以及通用机器人函数库。
```
sudo apt-get install ros-kinetic-desktop
```

​	**基础版安装: (简版)**  包含ROS核心软件包、构建工具以及通信相关的程序库，无GUI工具。
```
sudo apt-get install ros-kinetic-ros-base
```

​	**单个软件包安装**: 你也可以安装某个指定的ROS软件包（使用软件包名称替换掉下面的PACKAGE）:
```
sudo apt-get install ros-kinetic-PACKAGE
```
例如：`sudo apt-get install ros-kinetic-slam-gmapping`

要查找可用软件包，请运行：
```
apt-cache search ros-kinetic
```

### 1.5 初始化 rosdep
在开始使用ROS之前你还需要初始化rosdep。rosdep可以方便在你需要编译某些源码的时候为其安装一些系统依赖，同时也是某些ROS核心功能组件所必需用到的工具。
```
sudo rosdep init
rosdep update
```

**如果这一步报错，显示超时，则需开启代理，并设置好终端的代理。**

### 1.6 环境配置
如果每次打开一个新的终端时ROS环境变量都能够自动配置好（即添加到bash会话中），那将会方便很多：
```
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## 2. ros测试
### 2.1 运行roscore
roscore是你在运行所有ROS程序前首先要运行的命令。

请运行：
```
roscore
```

然后你会看到类似下面的输出信息：
```
... logging to ~/.ros/log/9cf88ce4-b14d-11df-8a75-00251148e8cf/roslaunch-machine_name-13039.log
Checking log directory for disk usage. This may take awhile.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://machine_name:33919/
ros_comm version 1.4.7

SUMMARY
========

PARAMETERS
 * /rosversion
 * /rosdistro

NODES

auto-starting new master
process[master]: started with pid [13054]
ROS_MASTER_URI=http://machine_name:11311/

setting /run_id to 9cf88ce4-b14d-11df-8a75-00251148e8cf
process[rosout-1]: started with pid [13067]
started core service [/rosout]
```

### 2.2 打开小乌龟
ctrl+alt+T打开独立新终端，或ctrl+shift+T打开新终端选项卡，运行：
```
rosrun turtlesim turtlesim_node
```

你会看到turtlesim窗口：
![](_v_images/20210322145148697_10850.png =382x)

### 2.3 控制小乌龟
打开新终端，运行：
```
rosrun turtlesim turtle_teleop_key
```

终端显示：
```
[ INFO] 1254264546.878445000: Started node [/teleop_turtle], pid [5528], bound on [aqy], xmlrpc port [43918], tcpros port [55936], logging to [~/ros/ros/log/teleop_turtle_5528.log], using [real] time
Reading from keyboard
---------------------------
Use arrow keys to move the turtle.
```

现在你可以使用键盘上的方向键来控制turtle运动了。
![](_v_images/20210322145535001_28014.png =384x)