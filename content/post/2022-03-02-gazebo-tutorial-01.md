---
title: ROS中的Gazebo仿真教程01
author: lowkeng
date: '2022-03-02'
slug: []
categories:
  - ROS Integrated Tools
tags:
  - ROS
  - Gazebo
  - Robot Simulation
subtitle: 'Gazebo入门基本介绍'
description: ''
image: 'img/pexels-photo-5011647.jpeg'
---

Gazebo仿真器使用具有高保真度（degree of fidelity）的物理引擎，可以在加载机器人模型后，添加传感器（比如IMU传感器和摄像机）和物体模型，搭建各种复杂的室内或室外的环境场景。Gazebo除了加载机器人等模型，还可以检测碰撞、测量位置等。
<!--more-->

## 01 Gazebo基本特点[^a]

通过一个简单的机器人demo对Gazebo的特点和功能进行展示。

**Typical uses of Gazebo:**

* 机器人设计

* 测试机器人算法
* 执行带有真实场景的回归测试[^b]（regression testing)

**Key Features of Gazebo:**

* 具有多种物理引擎
* 丰富的机器人模型库和环境库
* 支持各种传感器
* 方便的编程和图形界面

## 02 Ubuntu安装Gazebo

1. Download the [installer](http://get.gazebosim.org/)：右键保存shell文件`gazebo.sh`到磁盘目录，比如`~/Downloads`
2. 运行`chmod +x ~/Downloads/gazebo.sh`
3. 运行`gnome-terminal --working-directory="~" -e "./Downloads/gazebo.sh"`

## 03 界面说明

运行Gazebo：ALT+F2在弹出的对话框输入要运行的命令`gazebo`

![image-20220228122851508](https://s2.loli.net/2022/02/28/p9Xkeav3DNMIYhF.png)

**场景部分Scene**：仿真物体动画展示、与环境交互

**面板Panel**: 通过拖动面板与场景的分隔条可以display/hide/resieze

* **左面板**有3个Tag

  * **World**:显示当前场景中的模型，可以修改模型参数，比如改动模型的位置。可以展开`GUI`选项调整相机视角。

    ![image-20220228142428452](https://s2.loli.net/2022/02/28/CvBGwRWcD9LYSUl.png)

  * **Insert:**顾名思义，就是添加新物体和新模型的标签选项。

  * **Layers:** 组织和显示一个或多个模型构成的group，更多相关参考[Visibility Layers](http://gazebosim.org/tutorials?tut=visual_layers&cat=build_robot)教程。

* **右面板**

  * 默认关闭，通过拖动面板与场景的分隔条可以拉出。
  * 关节交互面板，与选定模型的可动作组件进行交互。

**工具条Toobars**

* **上方工具条**

  交互工具：选择、移动、旋转、缩放按键

  创建简单形状（例如方块、球体和圆柱体等)

​	![img](https://github.com/osrf/gazebo_tutorials/raw/master/guided_b/files/ftu3-top-toolbar.png)

* **底部工具条**

  显示仿真数据：仿真时间Simulation Time(仿真中时间可以比实际时间快或慢，这取决于仿真的计算任务的繁重)。

  Gazebosh中世界的状态迭代一次计算更新一次。默认迭代时间步长是1ms.

​	![img](https://github.com/osrf/gazebo_tutorials/raw/master/guided_b/files/ftu3-bottom-toolbar.png)

**菜单栏Menu**

![img](https://github.com/osrf/gazebo_tutorials/raw/master/guided_b/files/ftu3-menu-options.png)

**鼠标使用Mouse Control**

用鼠标进行场景导览与视角变更

![img](https://github.com/osrf/gazebo_tutorials/raw/master/guided_b/files/ftu3-mouse-controls.png)

## 04 模型编辑

构建模型

* 简单模型可以通过用户界面创建
* 复杂的机器人模型等需要写[SDF](http://sdformat.org/)(Simulation Description Format)文件，更多相关可以参考[Gazebo教程：Build a Robot](http://gazebosim.org/tutorials?cat=build_robot)

菜单栏选择`Edit/Model Editor`打开模型编辑器（Ctrl+M）

![img](https://github.com/osrf/gazebo_tutorials/raw/master/guided_b/files/gazebo8_model_editor_ui.png)

1. 工具条：用于模型编辑
2. 左面板，包括
   * `Insert Tag`:添加关节和嵌套模型
   * `Model Tag`:修改模型属性和内容

**左面板/Insert Tag**

主要用于添加新的关节和模型，包含以下三部分：

* **简单形状**：三种原型的几何形状，to form a *link* in the model
* **自定义形状**：导入自定义的`meshes`(支持COLLADA`.dae`, 3D Systems`.stl`, Wavefront `.obj` and W3C SVG `.svg`文件格式）
* **模型数据库**：添加模型库中各种模型，to form a *link* in the model，称作`嵌套模型Nested Models`

**左面板/Model Tag**

可以设置模型名称和基本参数，此时面板可以看到众多的links、joints、nested models and plugins.

可以通过右击模型打开`链接观测器Link Inspector`修改模型参数

![image-20220228165409555](https://s2.loli.net/2022/02/28/e1bJsrqFIf2o4dG.png)

模型编辑器目前

* 不能在嵌套模型中编辑嵌套模型和连杆
* 不能添加包括平面、折线等几何原型
* 不支持高度图（height maps）
* 不带有CAD功能

**Reference** 

1. CSDN | [ROS学习笔记之——gazebo仿真](https://blog.csdn.net/gwplovekimi/article/details/104255826)

**Footnotes**

[^a]: Gazebo Tutorial | [Beginner Overview](http://gazebosim.org/tutorials?cat=guided_b&tut=guided_b1)
[^b]: 运行所有的测试用例（test case），以验证没有退化情况发生，这一过程就是回归测试。

