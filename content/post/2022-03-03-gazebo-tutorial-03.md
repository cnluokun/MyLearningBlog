---
title: ROS中的Gazebo仿真教程03
author: Luokun
date: '2022-03-03'
slug: []
categories:
  - ROS Integrated Tools
tags:
  - Gazebo
  - ROS
  - Robot Simulation
subtitle: 'Gazebo的world文件、server/client、plugin、环境变量和架构'
description: ''
image: ''
---

本文主要介绍一些Gazebo组织架构相关的文件以及库类。
<!--more-->
## 01 文件和插件

运行一个带有移动小车(pioneer2dx)的Gazebo环境

```bash
$ gazebo worlds/pioneer2dx.world
```

![image-20220301110711124](https://s2.loli.net/2022/03/01/VIoS81HsvA7BCX3.png)

环境文件通常位于版本化的系统路径下，比如`/usr/share/gazeb0-7/worlds`

查看worlds

```bash
$ ls /usr/share/gazeb0-7/worlds
```

Gazebo启动运行命令`gazebo`实际上运行了两个可执行程序，分别是客户端`gzclient`和服务器`gzserver`

* 服务器`gzserver`:物理循环更新，生成传感器数据，读取world文件生成和输入环境

  启动和调用

  ```bash
  $ gzserver <world_name>
  ```

  > `world_name`:
  >
  > * 相对当前路径，或相对`GAZEBO_RESOURCE_PATH`变量中的某个路径
  > * 绝对路径，`/usr/share/gazeb0-7/worlds/empty_sky.world`

* 客户端gzclient:运行一个基于QT的用户接口，仿真的可视化，控制仿真属性

**world文件**一般包含机器人、光源、静态物体等的属性信息，属于格式化的仿真描述格式SDF文件。

![image-20220301140010505](https://s2.loli.net/2022/03/01/XmIE7htCelgJa4P.png)

**模型文件**也是一种格式化的仿真描述格式SDF文件，它有且只有一组`<model>...</model>`标签。模型文件可以在world文件中调用。调用语法如下（SDF语法）：

```xml
<include>
  <uri>model://model_file_name</uri>
</include>
```

Github上的Gazebo在线模型库：[osrf](https://github.com/osrf)/[gazebo_models](https://github.com/osrf/gazebo_models)，支持在Gazebo运行过程中加载在线模型。

**插件Plugins**提供了一种简单与Gazebo对接的机制，可以通过写命令行加载，也可以用SDF文件定义，加载过程中前者优先级更高。

* **Server插件**

```bash
$ gzserver -s <plugin_filename>
```

> `-s`参数选项表示系统插件(a system plugin)
> `<plugin_name>`是`GAZEBO_PLUGIN_PATH`中插件共享库的名称
> 比如，运行`RestWebPlugin`插件
>
> ```bash
> $ gzserver --verbose -s libRestWebPlugin.so
> ```

* **Client插件**

```bash
$ gzclient -g <plugin_filename>
```

> Gezebo7及以前版本：用`-g`参数加载GUI Plugin
>
> Gazebo8及以后版本：用参数`--gui-client-plugin`参数加载GUI插件
>
> 比如，要运行`TimeGUIPlugin`
>
> ```bash
> $ gzclient --gui-client-plugin libTimerGUIPlugin.so
> ```
>
> 更多相关内容参考For more information refer to the [plugins overview](http://gazebosim.org/tutorials/?tut=plugins_hello_world) page.



## 02 环境变量

环境变量可以定位文件，建立客户端和服务器之间的联系。

`GAZEBO_MODEL_PATH`: 逗号分隔的路径目录集合，用于Gazebo搜索模型

`GAZEBO_RESOURCE_PATH`: 逗号分隔的路径目录集合，用于Gazebo搜索world环境或媒体文件 

`GAZEBO_MASTER_URI`: URI of the Gazebo master，用于给定服务器入口IP和端口，以供客户端连接。

`GAZEBO_PLUGIN_PATH`: 逗号分隔的路径目录集合，用于Gazebo搜索插件共享库 

`GAZEBO_MODEL_DATABASE_URI`: 在线模型数据库的URI，用于Gazebo下载在线模型

以上环境变量的默认值可以在`<install_path>/share/gazebo/setup.bash`这个脚本文件中看到

![image-20220301142621796](https://s2.loli.net/2022/03/01/lGmVKqZwUH3zCr4.png)

如果需要作出修改，比如扩展搜索模型的路径，首先应该`source`环境变量（`source <install_path>/share/gazebo/setup.sh`)，然后再修改`setup.bash`中的默认变量值。

> Gazebo8开始使用[Ignition Transpor](https://ignitionrobotics.org/libs/transport)[^a]library进行进程间通信，一些功能是基于该开源通信库实现的，比如标志参照(markers)和绘图功能，这些功能还与下面的环境变量相关：
>
> * `IGN_PARTITION`: Ignition Transport nodes节点的分区名(partition name) 
> * `IGN_IP`: Similar to `GAZEBO_MASTER_URI`，但是面向的是Ignition Transport nodes
> * `IGN_VERBOSE`: Show debug information from Ignition Transport
>
> 更多相关内容参考[Ignition Transport Tutorials](https://ignitionrobotics.org/tutorials/transport/4.0/md__data_ignition_ign-transport_tutorials_20_env_variables.html)

## 03 Gazebo架构介绍

Gazebo是分布式架构，各种单独的文件库，可用于物理仿真、模型渲染、用户接口和通信感知等。

Gazebo服务器和客户端通过通信库进行通信。

### 3.1 进程间通信

采用开源的`Google Protobuf`[^b]数据编解码协议，用于将消息序列化

采用`boost::ASIO`异步输入输出的传输机制

支持发布/订阅的通信范式

![Gazebo PS Communication](https://s2.loli.net/2022/03/01/b4YSVxLitEsyNvT.png)

### 3.2 系统组成

**Gazebo Master**：本质是一个话题（a topic name server)，名称查询和话题管理

**通信库Communication Library**：

* 依赖项dependencies: Protobuf和boost::ASIO
* 外部API：支持在命名话题上的Gazebo节点同通信
* 内部API：None
* Advertized Topics
* Subscribed Topics

目前只支持发布/订阅的通信方式。

**物理库Physics Library**:

* 依赖项：动力引擎（带有内部碰撞检测）
* 外部API：提供一个简单而又通用的物理仿真接口
* 内部API：定义了可以给第三方动态引擎使用的物理库的基本接口

物理库提供一个简单而又通用的物理仿真接口，包括刚体、碰撞形状、用于表征链式约束的关节。该接口集成了4个开源物理引擎：

* ODE（[Open Dynamics Engine](http://ode.org/)）
* BUllet([Bullet](http://bulletphysics.org/))
* Simbody([Simbody](https://simtk.org/home/simbody))
* DART([Dynamic Animation and Robotics Toolkit (DART)](http://dartsim.github.io/))

通过这些物理引擎，用户可以加载XML格式的SDF模型文件，可以方便的使用集成的算法和仿真功能。

**渲染库**：

* 依赖：OGRE
* 外部API：创建和初始化场景
* 内部API：存储可视化元数据，调用OGRE API渲染3D scenes（包括GUI和传感器库）

渲染库包括光线、纹理和天空仿真，可以为渲染引擎写插件。

**传感器生成库**

* 依赖项：渲染库、物理库
* 外部API：初始化和运行传感器的功能
* 内部API：TBD

传感器生成库可以运行各种传感器，从物理仿真器中监听世界状态更新，产生输出。

**GUI库**

* 依赖项：渲染库、Qt
* 外部API：None
* 内部API：None

GUI基于Qt创建用于与仿真进行交互的图形组件，提供传感器数据可视化和日志记录工具。

**插件库**：物理库、渲染库和传感器生成库都支持插件，这些插件可以让用户不通过使用通信系统就可以访问这些库(物理库、渲染库和传感器生成库)。

**Reference**

1. Gazebo's [worlds and Client/Server Separation](http://gazebosim.org/tutorials?tut=quick_start&cat=get_started)

**Footnotes**

[^a]: Ignition Transport，一款开源通信库，是[Ignition Robotics](https://ignitionrobotics.org/)的一款组件，提供快速有效的异步消息传递、服务和数据日志等功能。
[^b]: Google *Protobuf*是一种平台无关,语言无关的结构化数据编解码协议,类似JSON或者XML,与这两者不同的是*Protobuf*采用的是二进制编解码方式,二JSON和XML是文本编解码
