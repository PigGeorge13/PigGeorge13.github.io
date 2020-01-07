---
title: Arbotix+rviz仿真
date: 2020-01-07
categories: tutorial
tags: [仿真]
---

<center>Arbotix+rviz仿真环境的搭建</center>

<!-- more -->


## Arbotix

·一款控制电机、舵机的硬件控制板

·提供了相应的ROS包

·提供了一个差速控制器，通过接收速度控制指令，更新机器人里程计状态

## Arbotix安装

Kinetic:
```
git clone https://github.com/vanadiumlabs/arbotix_ros.git
catkin_make
```

注：arbotix_ros中的python文件需要添加可执行权限

## 环境的搭建

### 创建launch文件

需要在上节中的.launch文件中加入：

```
	<node name="arbotix" pkg="arbotix_python" type="arbotix_driver" output="screen">
        <rosparam file="$(find mbot_description)/config/fake_mbot_arbotix.yaml" command="load" />
        <param name="sim" value="true"/>
    </node>
```


### 创建配置文件

fake_mbot_arbotix.yaml:


```
controllers:{
	base_controller:{
		type: diff_controller,
		base_frame_id: base_footprint,
		base_width: 0.26,
		ticks_meter: 4100,
		Kp: 12,
		Kd: 12,
		Ki: 0,
		Ko: 50,
		accel_limit: 1.0
	}
}
```


### 运行

运行.launch文件和键盘控制


## wiki教程
