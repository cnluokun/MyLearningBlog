---
title: ROS中的Gazebo仿真教程02
author: Luokun
date: '2022-03-03'
slug: []
categories:
  - ROS Integrated Tools
tags:
  - ROS
  - Gazebo
  - Robot Simulation
subtitle: 'Gazebo日志记录与回放'
description: ''
image: ''
---

通过GUI或者命令行，记录仿真过程日志，随后复盘。
<!--more-->
Gazebo的日志文件是压缩的`.log`文件，包含对整个世界的初始完整描述（从场景到展示实例）以及世界状态信息。世界状态记录的是仿真中每一次的变化，例如：

* 仿真数据：仿真时间、迭代次数
* 每个模型的目前状态以及模型中连杆和关节的状态：即时位置（instantaneous pose)、速度、加速度和力信息。
* 每个光源的当前位置

## 01 通过GUI记录日志

1. 打开一个双向摆（double pendulum)的简易世界。

2. `Ctrl+D`，打开数据日志记录器（Data Logger)，开始记录日志

   * 可以修改log文件的保存目录，比如我们可以将其保存至`~/logs/double_pendulum/`

   ![image-20220228202428043](https://s2.loli.net/2022/02/28/TbVK26er7URmZfy.png)

3. 点击红色按钮即开始记录日志，过程中可以看到log文件中的字节数量不断增加。

   > 只有随时间发生变化的models和lights会被记录，如果场景是静态场景，log文件中的字节数量不会改变，也就是说，log文件中的样本数量会合仿真中的迭代次数不同。

4. 点击红色按键，停止日志记录。

5. 展开`Recordings`折叠选项卡，可以看到生成的`state.log`文件，还会发现这个文件在一个带有时间戳的目录下。

   ![img](https://github.com/osrf/gazebo_tutorials/raw/master/logging_playback/files/recordings.png)

## 02 通过命令行记录日志

命令行方式，可以记录全部仿真过程的日志，也可以在仿真任意时间开始记录。

**记录整个仿真过程**

按下面指令启动和运行Gazebo

```bash
$ gazebo -r --record_path ~/logs/random_velocity worlds/random_velocity.world
```

> **-r[--record]**:从Gazebo打开开始到关闭结束，记录日志
>
> **--record_encoding arg**：log数据的压缩编码格式，默认为`zlib`，可选`bz2`和`txt`.

执行该命令，日志记录到Gazebo关闭。

按上述路径可以查看生成的`state.log`文件。

```bash
$ ls ~/logs/random_velocity/
state.log
```

**部分仿真过程的日志记录**

`gz log`:用来在任意时刻触发日志记录动作。Gazebo运行后，在新的终端执行以下命令就开始记录日志：

```bash
$ gz log -d 1
```

执行下面命令，终止日志记录

```bash
$ gz log -d 0
```

![image-20220228212925184](https://s2.loli.net/2022/02/28/7DmixjdvSH6qtsu.png)

## 03 日志文件的回放

replay/introspect a log file visually

**GUI中回放**

按以下指令启动和运行Gazebo，其中参数选项`-p`用于指定一个log文件，`-u`表示开始回放

```bash
$ gazebo -u -p ~/logs/double_pendulum/2016-01-25T15\:09\:49.677400/gzserver/state.log
```

然后Gazebo会在回放模式下运行，可以播放/暂停、倒带/快进等。

![img](https://github.com/osrf/gazebo_tutorials/raw/master/logging_playback/files/playback_gui.png)

## 04 log state文件的解析[^a]

`gz log`命令配合其参数选项可以查看log文件，比如

`-s`可以步进一个log文件 

```bash
$ gz log -s -f ~/logs/double_pendulum/2016-01-25T15\:09\:49.677400/gzserver/state.log
```

You'll see the full initial SDF representation of the world, something like this:

```css
<?xml version='1.0'?>
<gazebo_log>
<header>
<log_version>1.0</log_version>
<gazebo_version>7.0.0~pre1</gazebo_version>
<rand_seed>10622214</rand_seed>
<log_start>43 380000000</log_start>
<log_end>69 651000000</log_end>
</header>

<chunk encoding='txt'><![CDATA[
<sdf version ='1.6'>
<world name='default'>
  (...)
  <light name='sun' type='directional'>
    (...)
  </light>
  <model name='ground_plane'>
    (...)
  </model>
  <model name='double_pendulum_with_base'>
    (...)
  </model>
</world>
</sdf>]]></chunk>

--- Press space to continue, 'q' to quit ---
```

按下空格键，继续步进之后的状态。

```css
<chunk encoding='txt'><![CDATA[
<sdf version='1.6'>
<state world_name='default'>
<sim_time>43 380000000</sim_time>
<real_time>43 478499228</real_time>
<wall_time>1453763389 677873530</wall_time>
<iterations>43380</iterations>
<model name='double_pendulum_with_base'><pose>1.140 -1.074 -0.000 0.000 -0.000 0.000 </pose><scale>1.000 1.000 1.000</scale><link name='base'><pose>1.13998 -1.07367 -0.00000 0.00000 0.00000 -0.00042 </pose><velocity>-0.0000 0.0000 -0.0005 0.0004 0.0030 0.0001 </velocity></link><link name='lower_link'><pose>1.38969 -1.79815 1.41059 -2.45351 0.00000 -0.00042 </pose><velocity>0.0042 -0.2557 0.2659 1.9694 0.0048 0.0001 </velocity></link><link name='upper_link'><pose>1.13999 -1.07367 2.10000 2.33144 -0.00000 -0.00042 </pose><velocity>0.0063 -0.0008 -0.0005 -0.3739 0.0032 0.0001 </velocity></link></model><model name='ground_plane'><pose>0.000 0.000 0.000 0.000 -0.000 0.000 </pose><scale>1.000 1.000 1.000</scale><link name='link'><pose>0.00000 0.00000 0.00000 0.00000 -0.00000 0.00000 </pose><velocity>0.0000 0.0000 0.0000 0.0000 -0.0000 0.0000 </velocity></link></model></state></sdf>
]]></chunk>

--- Press space to continue, 'q' to quit ---
```

注意到状态信息更密集（只包含世界中发生变化的模型和视角，而没显示`sun`和`ground_plane`的信息，因为这两者是不运动变化的）。

**Reference**

1. Gazebo Tutorials:[Logging and playback](http://gazebosim.org/tutorials?cat=tools_utilities&tut=logging_playback)

**Footnotes**

[^a]: Gazebo [Log filtering](http://gazebosim.org/tutorials?tut=log_filtering&cat=tools_utilities):Introspect a log file
