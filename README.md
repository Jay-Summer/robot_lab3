# 机器人雷达 ROS2 驱动

## 系统版本

- ROS2 Humble (Ubuntu 22.04)

## 使用说明
1. 获取源码：
  ```bash
  cd ~/ros2_ws/src/
  git clone https://github.com/zhangwanjie/lidar_pkg_ros2.git
  ```
  备用地址：
  ```bash
  cd ~/catkin_ws/src/
  git clone https://gitee.com/s-robot/lidar_pkg_ros2.git
  ```
2. 插上雷达，执行如下指令：  
  ```bash
  ls /dev/ttyUSB* 
  ```
  或者
  ```bash
  ls /dev/ttyACM* 
  ```
  拔掉雷达，再执行上述指令。消失的那个设备就是雷达设备。  
  3. 设置访问权限
  ```
  sudo usermod -a -G dialout $USER 
  ```
4. 修改设备参数：  
  修改文件 /config/lidar_params.yaml
  ```
  lidar_node:
  ros__parameters:
    port_name: "/dev/ttyUSB*"  #修改为具体的设备名称，有时是`ttyUSB0`，有时是`ttyACM0`
    frame_id: "laser"
  ```
5. 编译
  ```bash
  conda deactivate # 退出conda，确保是系统python路径
  cd ~/ros2_ws
  colcon build --symlink-install
  ```
6. 运行测试：
  ```bash
  source install/setup.bash 
  ros2 launch lidar_pkg monitor.launch.py 
  ```
7. 对于没有外接显示设备的情况，使用如下指令启动雷达：
  ```bash
  ros2 launch lidar_pkg lidar.launch.py 
  ```

## Q&A

### 1. 运行`colcon build --symlink-install`无法成功编译lidar_pkg并报如下错误：
```bash
execute_process(/usr/local/miniconda3/bin/python3
...
ModuleNotFoundError: No module named 'catkin_pkg'
```
- **解决方案**

当前激活了 miniconda (base) 环境，colcon build 调用的是 conda 的 python3，而不是系统 ROS2 要用的系统 python3

1. 退出 conda 环境
```bash
conda deactivate
```
2. 确认当前 python 是系统 python
```bash
which python3
```
✅ 正确输出：`/usr/bin/python3`

3. 清理旧编译缓存，重新编译
```bash
rm -rf build install log
colcon build --symlink-install
```

### 2. 运行`ros2 launch lidar_pkg monitor.launch.py`雷达不转动并报如下错误：
```bash
Package 'lidar_pkg' not found: "package 'lidar_pkg' not found, searching: ['/opt/ros/humble']"
```

- **解决方案**

终端没有加载工作空间的 `install` 环境

加载本工作空间的环境，并重新launch
```bash
source install/setup.bash
```

### 3. 运行`ros2 launch lidar_pkg monitor.launch.py`雷达不转动并报如下错误：
```bash
[lidar_node-2] [ERROR] [1791340167.129155162] [lidar_node]: Open Error: Permission denied
[lidar_node-2] [FATAL] ... Failed to open serial port: /dev/ttyACM0
```
- **解决方案**

打开串口 `/dev/ttyACM0` 权限不足！

1. 先确认当前用户是否在 `dialout` 组
```bash
groups
```
看输出列表里有没有 `dialout`，如果没有，说明`sudo usermod -a -G dialout $USER`没生效。

2. 临时解决方案：
```bash
sudo chmod 666 /dev/ttyACM0  # 仅当前终端生效
```

## 其他
1. 雷达底座的3D打印模型：[models/lidar_mount.stl](./models/lidar_mount.stl)



