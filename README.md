# Kugle-ROS
ROS workspace for the Kugle robot including packages for high-level navigation tasks including localization/SLAM with LiDAR, path planning and obstacle avoidance

The repository is structured according to general ROS practices, see e.g. https://github.com/leggedrobotics/ros_best_practices/wiki

# Install tool
```bash
sudo apt-get install python-catkin-tools
sudo apt-get install python-rosdep
```

# Cloning
To set up the simulation environment you need to clone the necessary repositories into an existing or new catkin workspace.
Follow the steps below to set up a new catkin workspace and clone:
```bash
mkdir -p ~/kugle_ws/src
cd ~/kugle_ws/src
catkin_init_workspace
git clone https://github.com/mindThomas/Kugle-Gazebo
git clone https://github.com/mindThomas/Kugle-ROS
git clone https://github.com/mindThomas/realsense_gazebo_plugin
cd ..
rosdep install --from-paths src --ignore-src -r -y
```

# Building
Build the project with catkin build
```bash
cd ~/kugle_simulation_ws
catkin build
source devel/setup.bash
```

# Notes
Descriptions and guides how to use this ROS project can be found in the notes below.

## USB rules file for automatic device detection
The MCU can automatically be detected and assigned to `/dev/kugle` when connecting it over USB if the rules file, `99-kugle.rules`, is installed.

To install the rules file, copy `99-kugle.rules` to `/etc/udev/rules.d/`.

## Minimal bringup launch
The bringup launch includes both the driver (as described below) and the SICK LiDAR drivers. This launch script should only be used on the onboard computer with the full system connected.
```bash
roslaunch kugle_bringup minimal.launch 
```

## Install as service on boot
A startup script and corresponding service (for starting at boot) for launching the minimal bringup launch file has been made.

Copy the `startup_launch.sh` and `PrepareHostROS.sh` to the home folder. Copy the file `kugle.service` into `/lib/systemd/system` and modify the path to the `startup_launch.sh` script accordingly and rename the user and group to the username on the device if different. Enable the service on boot by running:
```bash
sudo systemctl daemon-reload
sudo systemctl enable kugle.service
```

After the service has been installed the driver can be started, stopped or restarted by using:
```bash
sudo service kugle start
sudo service kugle stop
sudo service kugle restart
```

## Launching the driver
The driver which communicates with the MCU over USB can be launched either on a laptop connected through USB to the MCU or on the onboard connected over USB. Launch the driver by running
```bash
roslaunch kugle_driver kugle_driver.launch
```

# Usage
## Connecting to onboard computer ROS
The onboard computer automatically launches the minimal bringup launch file at boot and creates a WiFi hotspot named 'kugle'. Connecting to this hotspot gives access to the ROS Master created by the robot. To link a user computer to the ROS Master source the `ConnectROS.sh` script:
```bash
source ConnectROS.sh
```

## Reconfigure GUI (change parameters)
To open the reconfigure GUI on a local computer connected to the ROS Master run the command
```bash
rosrun rqt_reconfigure rqt_reconfigure kugle_driver
```

## RVIZ display
An RVIZ visualization of the robot can be launched by running
```bash
roslaunch kugle_launch rviz.launch
```

## kugle_driver MCU RTOS load
When the `kugle_driver` is running the MCU load is published on the topic `mcu_load` which can be displayed by running
```bash
rostopic echo /mcu_load -p
```

## Soft restart
The controller and estimators running inside the embedded firmware on the microprocessor can be restarted by running:
```bash
rosservice call /kugle/restart_controller
```

## Reboot MCU 
The microprocessor can be rebooted by calling:
```bash
rosservice call /kugle/reboot
```

## Calibrate IMU
Calibration of the IMU requires the robot to be positioned/aligned such that its' center of mass is located above the center of the ball. Hold the robot and find the balancing point and the run the calibration by calling:
```bash
rosservice call /kugle/calibrate_imu
```
The calibration computes an alignment rotation matrix to align the z-axis with the downward-pointing gravity direction. Since the calibration also involves calibrating the gyroscope bias, the robot should be held still during calibration.

When finished the calibration is automatically stored inside non-volatile memory of the microprocessor.

## Over-the-air MCU update
The MCU can be programmed either using the onboard USB programmer/debugger or using the built-in USB bootloader. If just the Client USB connection is connected to the STM32H7 board, only the built-in USB bootloader can be used. To enter the bootloader the system should be in OFF state (motors not running) whereafter the bootloader is entered by calling:
```bash
rosservice call /kugle/enter_bootloader
```
The firmware can now be updated using the functions described in https://github.com/mindThomas/Kugle-Embedded#dfu-bootloader-over-usb
