robot_pose_ekf [![Build Status](https://travis-ci.com/ros-planning/robot_pose_ekf.svg?branch=master)](https://travis-ci.org/ros-planning/robot_pose_ekf)
========================================================================================================================================================
Package Summary
 Released Continuous Integration: 3 / 3   Documented
The Robot Pose EKF package is used to estimate the 3D pose of a robot, based on (partial) pose measurements coming from different sources. It uses an extended Kalman filter with a 6D model (3D position and 3D orientation) to combine measurements from wheel odometry, IMU sensor and visual odometry. The basic idea is to offer loosely coupled integration with different sensors, where sensor signals are received as ROS messages.

Maintainer status: unmaintained
Maintainer: ROS Orphaned Package Maintainers <ros-orphaned-packages AT googlegroups DOT com>
Author: Wim Meeussen
License: BSD
Source: git https://github.com/ros-planning/robot_pose_ekf.git (branch: master)

目录

How to use the Robot Pose EKF
Configuration
Running
Nodes
robot_pose_ekf
Subscribed Topics
Published Topics
Provided tf Transforms
How Robot Pose EKF works
Pose interpretation
Covariance interpretation
Timing
Package Status
Stability
Roadmap
Tutorials
 

How to use the Robot Pose EKF
Configuration
A default launch file for the EKF node can be found in the robot_pose_ekf package directory. The launch file contains a number of configurable parameters:

freq: the update and publishing frequency of the filter. Note that a higher frequency will give you more robot poses over time, but it will not increase the accuracy of each estimated robot pose.
sensor_timeout: when a sensor stops sending information to the filter, how long should the filter wait before moving on without that sensor.
odom_used, imu_used, vo_used: enable or disable inputs.
The configuration can be modified in the launch file, which looks something like this:

 <launch>
  <node pkg="robot_pose_ekf" type="robot_pose_ekf" name="robot_pose_ekf">
    <param name="output_frame" value="odom"/>
    <param name="freq" value="30.0"/>
    <param name="sensor_timeout" value="1.0"/>
    <param name="odom_used" value="true"/>
    <param name="imu_used" value="true"/>
    <param name="vo_used" value="true"/>
    <param name="debug" value="false"/>
    <param name="self_diagnose" value="false"/>
  </node>
 </launch>
Running
Build the package

 $ rosdep install robot_pose_ekf
 $ roscd robot_pose_ekf
 $ rosmake
Run the robot pose ekf

 $ roslaunch robot_pose_ekf.launch
Nodes

robot_pose_ekf
robot_pose_ekf implements an extended Kalman filter for determining the robot pose.
Subscribed Topics
odom (nav_msgs/Odometry)
2D pose (used by wheel odometry): The 2D pose contains the position and orientation of the robot in the ground plane and the covariance on this pose. The message to send this 2D pose actually represents a 3D pose, but the z, roll and pitch are simply ignored.
imu_data (sensor_msgs/Imu)
3D orientation (used by the IMU): The 3D orientation provides information about the Roll, Pitch and Yaw angles of the robot base frame relative to a world reference frame. The Roll and Pitch angles are interpreted as absolute angles (because an IMU sensor has a gravity reference), and the Yaw angle is interpreted as a relative angle. A covariance matrix specifies the uncertainty on the orientation measurement. The robot pose ekf will not start when it only receives messages on this topic; it also expects messages on either the 'vo' or the 'odom' topic.
vo (nav_msgs/Odometry)
3D pose (used by Visual Odometry): The 3D pose represents the full position and orientation of the robot and the covariance on this pose. When a sensor only measures part of a 3D pose (e.g. the wheel odometry only measures a 2D pose), simply specify a large covariance on the parts of the 3D pose that were not actually measured.
The robot_pose_ekf node does not require all three sensor sources to be available all the time. Each source gives a pose estimate and a covariance. The sources operate at different rates and with different latencies. A source can appear and disappear over time, and the node will automatically detect and use the available sensors.To add your own sensor inputs, check out the Adding a GPS sensor tutorial


Published Topics
robot_pose_ekf/odom_combined (geometry_msgs/PoseWithCovarianceStamped)
The output of the filter (the estimated 3D robot pose).

Provided tf Transforms
odom_combined → base_footprint
How Robot Pose EKF works
Pose interpretation
All the sensor sources that send information to the filter node can have their own world reference frame, and each of these world reference frames can drift arbitrary over time. Therefore, the absolute poses sent by the different sensors cannot be compared to each other. The node uses the relative pose differences of each sensor to update the extended Kalman filter.

Covariance interpretation
As a robot moves around, the uncertainty on its pose in a world reference continues to grow larger and larger. Over time, the covariance would grow without bounds. Therefore it is not useful to publish the covariance on the pose itself, instead the sensor sources publish how the covariance changes over time, i.e. the covariance on the velocity. Note that using observations of the world (e.g. measuring the distance to a known wall) will reduce the uncertainty on the robot pose; this however is localization, not odometry.

Timing
Imagine the robot pose filter was last updated at time t_0. The node will not update the robot pose filter until at least one measurement of each sensor arrived with a timestamp later than t_0. When e.g. a message was received on the odom topic with timestamp t_1 > t_0, and on the imu_data topic with timestamp t_2 > t_1 > t_0, the filter will now update to the latest time at which information about all sensors is available, in this case to time t_1. The odom pose at t_1 is directly given, and the imu pose at t_1 is obtained by linear interpolation of the imu pose between t_0 and t_2. The robot pose filter is updated with the relative poses of the odom and imu, between t_0 and t_1.

robot_pose_ekf.png

The above figure shows experimental results when the PR2 robot started from a given initial position (green dot), driven around, and returned to the initial position. A perfect odometry x-y plot should show an exact loop closure. The blue line shows the input from the wheel odometry, with the blue dot the estimated end position. The red line shows the output of the robot_pose_ekf, which combined information of wheel odometry and imu, with the red dot the estimated end position.

Package Status
Stability
The code base of this package has been well tested and has been stable for a long time. The ROS API however has been changing as message types have evolved over time. In future versions, the ROS API is likely to change again, to a simplified single-topic interface (see Roadmap below).

Roadmap
The filter is currently designed for the three sensor signals (wheel odometry, imu and vo) that we use on the PR2 robot. We plan to make this package more generic: future versions will be able to listen to 'n' sensor sources, all publishing a (nav_msgs/Odometry) message. Each source will set the covariance of the 3D pose in the Odometry message to specify which part of the 3D pose it actually measured.

We would like to add velocity to the state of the extended Kalman filter.
Tutorials
robot_pose_ekf/Tutorials/AddingGpsSensor
