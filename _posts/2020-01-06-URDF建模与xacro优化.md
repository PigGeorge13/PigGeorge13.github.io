---
title: URDF建模与xacro优化
date: 2020-01-06
categories: tutorial
tags: [建模]
---

<center>URDF机器人建模与xacro建模</center>

<!-- more -->


## URDF

·Unified Robot Description Format,统一机器人描述格式

·ROS中一个非常重要的机器人模型描述格式

·可以解析URDF文件中使用XML格式描述的机器人模型

·ROS同时也提供URDF文件的C++解析

### URDF主标签

#### `<robot>`标签

·完整的机器人模型的最顶层标签

·`<link>`和`<joint>`标签都必须包含在`<robot>`标签内


```
<robot name="name of the robot">
	<link> ...... </link>
	<link> ...... </link>

	<joint> ...... </joint>
	<joint> ...... </joint>
</robot>
```

完整的机器人模型由一系列`<link>`和`<joint>`组成


#### `<link>`标签

·描述机器人某个刚体部分的外观和物理属性

·尺寸（size）、颜色（color），形状（shape），惯性矩阵（inertial matrix），碰撞参数（collision properties）等。

子标签：

`<visual>`：描述机器人link部分的外观参数

`<inertial>`：描述link的惯性参数

`<collision>`：描述link的碰撞参数

例程：

```
    <link name="base_link">
      <visual>
        <origin xyz=" 0 0 0" rpy="0 0 0" />
        <geometry>
          <cylinder length="0.16" radius="0.20"/>
        </geometry>
        <material name="yellow">
          <color rgba="1 0.4 0 1"/>
         </material>
    </visual>
    <collision>
      <geometry>
        <cylinder length="0.6" radius="0.2"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="10"/>
      <inertia ixx="0.4" ixy="0.0" ixz="0.0" iyy="0.4" iyz="0.0" izz="0.2"/>
    </inertial>
    </link>
```



#### `<joint>`标签

·描述关节的运动学和动力学属性

·包括关节运动的位置和速度限制

子标签：

`<type>`：关节运动形式，其可分为六种类型

|关节类型|描述|
|:--|:--|
| continuous | 旋转关节，可以绕单轴无限旋转 |
| revolute | 旋转关节，有旋转的角度限制 |
| prismatic | 滑动关节，沿某一轴线移动的关节，带位置极限 |
| planar | 平面关节，允许在平面正交方向上平移或旋转 |
| floating | 浮动关节，允许进行平移、旋转运动 |
| fixed | 固定关节，不允许运动的特殊关节 |

`<calibration>`：关节的参考位置，用来校准关节的绝对位置

`<dynamics>`：描述关节的物理属性，例如阻尼、物理静摩擦力等

`<limit>`：运动的极限值，包括关节的上下限位置、速度极限、力矩限制等

`<mimic>`：描述该关节与已有关节的关系

`<safety_controller>`：描述安全控制器参数

`<parent>`：母link

`<child>`：子link

例程：

```
  <joint name="left_gripper_joint" type="revolute">
    <axis xyz="0 0 1"/>
    <limit effort="1000.0" lower="0.0" upper="0.548" velocity="0.5"/>
    <origin rpy="0 0 0" xyz="0.2 0.01 0"/>
    <parent link="gripper_pole"/>
    <child link="left_gripper"/>
    <calibration ... />
    <dynamics damping .../>
    ....
  </joint>
```

### urdf功能包的文件结构

·urdf：存放机器人模型的URDF或xacro文件

·meshes：放置URDF中引用的模型渲染文件

·launch：保存相关启动文件

·config：保存rviz的配置文件

### launch文件的编写

```
<launch>
	<param name="robot_description" textfile="$(find mbot_description)/urdf/mbot_base.urdf" />

	<!-- 设置GUI参数，显示关节控制插件 -->
	<param name="use_gui" value="true"/>
	
	<!-- 运行joint_state_publisher节点，发布机器人的关节状态  -->
	<node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher" />
	
	<!-- 运行robot_state_publisher节点，发布tf  -->
	<node name="robot_state_publisher" pkg="robot_state_publisher" type="state_publisher" />
	
	<!-- 运行rviz可视化界面 -->
	<node name="rviz" pkg="rviz" type="rviz" args="-d $(find mbot_description)/config/mbot_urdf.rviz" required="true" />
</launch>
```

launch文件基本只需要改第一行的文件地址。

### 查看URDF整体结构

```
urdf_to_graphiz name.urdf
```

### URDF缺点

·建模代码冗长

·修改参数麻烦，不利于二次开发

·没有参数计算功能

##　URDF建模优化--xacro建模

·精简模型代码：

创建宏定义、文件包含

·可供编程接口

常量、变量、数学计算、条件语句

### 常量定义

定义：
```
<xacro:property name="M_PI" value="3.14159" />
```

使用：
```
<origin:xyz="0 0 0" rpy="${M_PI/2} 0 0" />
```

### 数学计算

```
<origin xyz="0 ${(motor_length+wheel_length)/2} 0" rpy="0 0 0"   />
```

注：所有运算都会转换为浮点计算，以保证运算精度

### 宏定义

定义：

```
<xacro:macro name="name" params="A B C">
......
</xacro:macro>
```

调用：
```
<name A="A_value" B="B_value" C="C_value" />
```

### 文件包含

```
<xacro:include filename="$(find mbot_description)/urdf/xacro/mbot_base.xacro">
```

### launch文件的编写

与urdf的.launch文件类似，需要加一个xacro的解析器

```
<launch>
	<arg name="model" default="$(find xacro)/xacro --inorder '$(find mbot_description)/urdf/xacro/mbot.xacro'" />
	<arg name="gui" default="true" />

	<param name="robot_description" command="$(arg model)" />

    <!-- 设置GUI参数，显示关节控制插件 -->
	<param name="use_gui" value="$(arg gui)"/>

    <!-- 运行joint_state_publisher节点，发布机器人的关节状态  -->
	<node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher" />

	<!-- 运行robot_state_publisher节点，发布tf  -->
	<node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" />

    <!-- 运行rviz可视化界面 -->
	<node name="rviz" pkg="rviz" type="rviz" args="-d $(find mbot_description)/config/mbot.rviz" required="true" />

</launch>
```

## wiki教程

·ROS URDF Turtorials
[(wiki.ros.org/urdf/Turtorials)](http://wiki.ros.org/urdf/Turtorials)

·ROS xacro
[(wiki.ros.org/xacro)](http://wiki.ros.org/xacro)