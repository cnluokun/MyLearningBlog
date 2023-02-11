---
title: MATLAB绘图入门庖丁解牛 - 绘图基本操作
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

## 设置以LaTeX格式显示图片文字

```Matlab
set(0, 'defaulttextinterpreter','latex');%设置文本解析器为LaTeX
set(0, 'defaultAxesTickLabelInterpreter','latex');%设置坐标轴说明文字格式为LaTeX
set(0, 'defaultLegendInterpreter','latex');%设置图例文字格式为LaTeX
set(groot,'defaultLineLineWidth',2); % 修改默认线宽
```

## 基本绘图函数plot

```matlab 
x = 0:pi/100:2*pi;
% x = linspace(0,2*pi,100)；

{y1,y2,y3,y4}={sin(x),cos(x),x,exp(sin(x))};
% {y{1},y{2},y{3},y{4}}={sin(x),cos(x),x,exp(sin(x))}
% z = {sin(x),cos(x),x,exp(sin(x))}

%% 绘图Plot
figure(1);
plot(x,y1,'--',...
     x,y2,':',...
     x,y3,'-.',...
     x,y4);
xlabel('X-axis');
ylabel('Y-axis');
title('一些简单函数图像');
legend('sinx','cosx','x','exp(sinx) ');
```

## 绘制子图块subplot

```matlab
x = linspace(0,2*pi,100)；
{y1,y2,y3,y4}={sin(x),cos(x),x,exp(sin(x))};

figure(2);
subplot(2,2,1);plot(x,y1,'--');
subplot(2,2,2);plot(x,y2,':');
% subplot(2,2,3);plot(x,y3,'-.');
% subplot(2,2,4);
%plot(x,y4);
subplot(2,2,[3,4]);
plot(x,y3,'-.');
xlim([0,2*pi]);
hold on;
plot(x,y4);
```