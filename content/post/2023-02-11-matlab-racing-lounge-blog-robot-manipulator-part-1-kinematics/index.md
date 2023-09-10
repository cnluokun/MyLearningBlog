---
title: MATLAB Racing Lounge Blog - Robot Manipulator Part 1:Kinematics
author: lowkeng
date: '2023-02-11'
slug: []
categories:
  - MATLAB机器人仿真
tags:
  - 转载或翻译
  - MATLAB
  - Robotics
  - Kinematics
subtitle: ''
description: ''
image: 'img/toppic/star-sky.jpg'
---

本博客由MATLAB and Simulink Robotics Arena系列视频的主持人[Sebastian Castro](https://www.mathworks.com/matlabcentral/profile/authors/3069683-sebastian-castro)于2018年4月11日发布。

## 机器人操作臂概述

Let’s start with a quick comparison of kinematics and dynamics.

* **Kinematics**运动学
  分析不考虑力作用下的运动，一般只需要操作臂的几何特征，比如长度和自由度
* **Dynamics**动力学
  分析由力作用引起的运动，除了几何特征，还需要质量和惯性参数用来计算操作臂连杆的速度和运动等

机器人操作臂通常由关节结构组成，而关节结构又可以分为旋转关节**revolute** (rotating) 和移动关节 **prismatic** (linear) 。因此，可以在三维空间中通过控制关节位置得到末端执行器的位置。

假设已知机器人的几何特征和全部的关节位置，通过数学计算可以得出机器人上任意一点的位置和姿态。这就是正向运动学**Forward Kinematics**.它的逆问题，往往需要求得使末端执行器到达某个具体位姿的关节角度。这也称作逆向运动学**Inverse Kinematics**。

## 逆向运动学求解

Depending on your robot geometry, IK can either be solved analytically or numerically.

* **解析解**Analytical solutions 
  意思是可以得到给定末端执行器位置下关节位置的闭式表达式。这种方法属于离线方法，求解IK速度很快
* **数值解**Numerical solutions
  一般而言，数值解比解析解求解要慢，而且更难以预测，但是数值方法可以求解更难更复杂的问题。然而，这些方法引入了初始状态的不确定性、优化算法的选择性甚至带有随机性（However, these solutions introduce uncertainty in the form of initial conditions, optimization algorithm choice, or even random chance），所以有时候求解的结果并不是我们想要的结果。

末端执行器的位姿可以由6个参数定义：其中3个定义其位置，另外3个定义其姿态。通常，如果操作臂至多有6个自由度并且期望位置是可达的，那么我们可以得到它的解析解。

机器人设计者对高于6个自由度机械臂保证仍有解析解的设计变得越来越灵活。例如，在我学习优达学城的机器人软件工程这门纳米学位课程 [Udacity Robotics Software Engineer Nanodegree](https://www.udacity.com/course/robotics-software-engineer--nd209)中，有一个求解库卡KR210 6自由度机械臂 [KUKA KR210](https://www.kuka.com/en-us/products/robotics-systems/industrial-robots/kr-quantec-ultra) 逆运动学解析解的project。这款机器人有一个球形手腕，可以解耦位置和姿态的逆运动学解析解。 [You can find my writeup on GitHub](https://github.com/sea-bass/RoboND-Kinematics-Project).

那么，你为何会想到选择数值求解方法呢？原因有以下几点

* 操作臂有冗余自由度（always in the case with 7 or more)
* 不想求解数学表达，而且有求解数值解的计算资源
* 目标位姿无效，但仍显尽可能地到达靠近该位姿地效果
* 解析解可能有多种解，甚至时无穷解
* 想要引入多个复杂的约束

![img](https://blogs.mathworks.com/racing-lounge/files/2018/04/kinematics_multi_soln-e1523359052865.png)

**多种解析解的情况**（图中是处理解析解相对容易的情况）
左图IK有两个解，上部分解和下部分解
右图IK有无穷个解，因为绕着base的任意水平旋转都是有效的

![img](https://blogs.mathworks.com/racing-lounge/files/2018/04/kinematics_complex_soln-e1523359075123.png)

**复杂的操作臂更可能选择数值方法求解IK**
左图7自由度操作臂可以有多个关节角方式达到同样的末端执行器位姿
右图操作臂上两个连杆坐标系间的位置约束示例

总结而言，IK解析求解通常更快，更准确和更可靠。但是，当遇到复杂的操作臂结构，数值解往往更容易执行，或者有时是别无选择的。

## 在MATLAB和Simulink中演示操作臂

在MATLAB和Simulink中有两种固定的添加操作臂的方法。

**MATLAB**

* 创建刚体结构树对象的方法 [rigid body tree object](https://www.mathworks.com/help/robotics/ug/rigid-body-tree-robot-model.html)
* 使用场合：用于求解运动学和动力学，提取机械特征（例如雅可比，质量矩阵，重力力矩等）

![image-20210419150941229](https://i.loli.net/2021/04/19/RxQoYBumj6eWIwc.png)

**Simulink**

* 创建Simscape Multibody模型（a [Simscape Multibody](https://www.mathworks.com/products/simmechanics.html) model）

  > Simscape Multibody™（前身为 SimMechanics™）提供了适用于 3D 机械系统（例如机器人、汽车悬架、建筑设备和飞机起落架）的多体仿真环境。可以使用表示刚体、关节、约束、力元件和传感器的模块对多体系统进行建模。可以使用 MATLAB® 变量和表达式对模型进行参数赋值，在 Simulink® 中设计多体系统的控制算法。

* 使用场合：用于系统级别的动力仿真，加入驱动器的物理模型和接触机制等

  ![img](https://blogs.mathworks.com/racing-lounge/files/2018/04/simulink_representation-e1523359134210.png)

  

上述两种方法，无论是刚体结构树对象还是Simscape Multibody模型，都可以从零开始搭建，也可以通过URDF文件直接加载。另外，Simscape Multibody还可以从CAD软件导入3D模型。关于如何导入CAD模型导入Simscape Multibody，我的同事 Christoph Hahn写了一篇[相关博客](https://blogs.mathworks.com/racing-lounge/2017/05/03/import-cad-assemblies-to-simscape-multibody/)。

从MATLAB 2018a开始，机器人系统工具箱RST包括操作臂算法仿真功能块库`Manipulator Algorithms Simulink block library`。通过这些功能块我们可以在Simulink中对刚体结构树对象进行运动学和动力学分析，这使得可以联合使用上述两种（MATLAB和Simulink）添加操作臂的方法，进行系统建模和控制系统设计。这些功能块可以生成C/C++代码，所以我们不用MATLAB和Simulink也可以部署独立的算法。

## 在MATLAB和Simulink中求解IK

Robotics System Toolbox为操作臂提供了两种逆向运动学数值求解器。

* [**Inverse Kinematics**](https://www.mathworks.com/help/robotics/ref/robotics.inversekinematics-system-object.html): 考虑了关节极限，允许对每个位置和姿态施加相关权重
* [**Generalized Inverse Kinematics**](https://www.mathworks.com/help/robotics/ref/robotics.generalizedinversekinematics-system-object.html)**:** 允许添加多个复杂的额外约束，例如坐标系之间的相对位置、末端执行器朝向或者时变的关节极限角度等

下面是在MATLAB中求Sawyer逆运动学解的`GIK Solver`求解器。此处，对末端执行器的位置设置了约束——让末端执行器的一直朝向空间中的一个点。

```matlab
sawyer=importrobot('sawyer.urdf','MeshPath',fullfile(fileparts(which('sawyer.urdf')),'..','meshes','sawyer_pv'))
gik = robotics.GeneralizedInverseKinematics('RigidBodyTree',sawyer,'ConstraintInputs',{'position','aiming'});

% Target Position constraint
targetPos = [0.5 0.5 0];
handPosTgt = robotics.PositionTarget('right_hand','TargetPosition',targetPos);
% posTgt = constraintPositionTarget('right_hand');
% posTgt.TargetPosition=[0.5 0.5 0];

% Target Aiming constraint
targetPoint = [1, 0, -0.5];
handAimTgt = robotics.AimingConstraint('right_hand','TargetPoint',targetPoint);
% aimCon = constraintAiming('right_hand');
% aimCon.TargetPoint = [1 0 -0.5];

% Solve Generalized IK
[gikSoln,solnInfo] = gik(sawyer.homeConfiguration,handPosTgt,handAimTgt)
show(sawyer,gikSoln);
```

![img](https://blogs.mathworks.com/racing-lounge/files/2018/04/sawyer_gik.gif)

## Inverse Kinematics in the Bigger Picture

测试了IK求解器后，可以应用MATLAB和Simulink搭建更完整的机器人操作臂系统，例如：

* 在IK中加入机器人动力学仿真
* 添加其他算法，例如监控逻辑，感知和路径规划
* 自动生成标准的C/C++代码，部署到硬件上或者类似ROS的中间件上

We discuss this in our video "**[Designing Robot Manipulator Algorithms](https://www.mathworks.com/videos/matlab-and-simulink-robotics-arena-designing-robot-manipulator-algorithms-1515776491590.html?s_tid=srchtitle_Designing%20Robot%20Manipulator%20Algorithms_1)**"

## 总结

很多人可能正在研究已经带有固有关节扭矩控制器的机器人算法。从这个角度出发，你可以假设机器人关节能够充分地跟踪到你所提供的任何设置点。

单独使用运动学也可以有效地设计运动规划算法，以及分析机器人几何学，例如工作空间分析或者检测碰撞

In the [next part](https://blogs.mathworks.com/racing-lounge/2018/04/25/robot-manipulation-part-2-dynamics-and-control/), we’ll talk more about manipulator dynamics and how this facilitates lower-level control design applications with MATLAB and Simulink.下一部分，我们会讨论操作臂地动力学以及它是如何使MATLAB和Simulink低级控制设计应用变得容易的。

Feel free to leave us a comment or email us at [roboticsarena@mathworks.com](mailto:roboticsarena@mathworks.com). I hope you enjoyed reading!

– Sebastian

## Reference

MATLAB Racing Lounge Blog - [Robot Manipulator,Part1:Kinematics](https://blogs.mathworks.com/racing-lounge/2018/04/11/robot-manipulation-part-1-kinematics/)
