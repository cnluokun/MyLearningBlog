---
title: MATLAB绘图入门庖丁解牛 - 图例位置和风格修改
author: lowkeng
date: '2023-02-11'
slug: []
categories:
  - MATLAB Basis
tags:
  - MATLAB
  - 科研绘图
subtitle: ''
description: ''
image: 'img/lake-1920.jpg'
---

## 设置图例位置

默认位置是`NorthEast`
除了可以更改图例东西南北（图框的上下左右侧）位置，还可以设置图例显示在图片之外

```maTLAB
clc
clear 
close all
Npoint = 101;
x = linspace(0,10,Npoint);
y1 = besselj(1,x);
y2 = besselj(2,x);
y3 = besselj(3,x);
y4 = besselj(4,x);
y5 = besselj(5,x);
H = plot(x,y1,x,y2,x,y3,x,y4,x,y5);
legend('First','Second','Third','Fourth','Fifth','Location','NorthEastOutside')
```

![image-20230210145355771](https://raw.githubusercontent.com/lowkeng/ImageBed/master/image-20230210145355771.png)

## 横向排列图例

```matlab
%Matlab中legend的横排，注意，Location位置改变为North

clc
clear 
close all
Npoint = 101;
x = linspace(0,10,Npoint);
y1 = besselj(1,x);
y2 = besselj(2,x);
y3 = besselj(3,x);
y4 = besselj(4,x);
y5 = besselj(5,x);
H = plot(x,y1,x,y2,x,y3,x,y4,x,y5);
h1 = legend(H([1 3 5]),'First','Third','Fifthth','Location','North');
set(h1,'Orientation','horizon')
% set(h1,'Orientation','horizon','Box','off')；%不显示图例方框
```

![image-20230210145515282](https://raw.githubusercontent.com/lowkeng/ImageBed/master/image-20230210145515282.png)

![image-20230210145612690](https://raw.githubusercontent.com/lowkeng/ImageBed/master/image-20230210145612690.png)