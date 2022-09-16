---
layout: post
title: "Generals 将军棋"
author: "Toseic"
tags: AI game github
comments: true
excerpt_separator: <!--more-->
actions:
  - label: "Github"
    icon: github
    url: "https://github.com/Toseic/Magic-generals"
---


## 游戏介绍 



[generals](http://generals.io)游戏，又名将军棋，此游戏以调兵遣将作为主要玩点，可以体验多方对抗中的心理博弈以及战术思想。<!--more-->

游戏规则，在x*y(16<=x,y<=23)个方格构成的的地图上，每个地图上面可以是山地，王城，古城，空白的土地。

山地是不可占领或者路过的土地，王城即为玩家的大本营，游戏初始时玩家从大本营出发，初始时大本营里面有1个士兵。

时间单位：1`turn`即为现实生活中的1s, 已经被攻占的城市（王城或者古城）每一turn会增加一个士兵，在有士兵的土地上（包括城市），每25turn增加一个士兵。

古城即上图深灰色标有数字的格子，古城上的数字代表城内守卫的兵力。你可以花费 数量的士兵占领古城，占领后古城将变为你的城池。你的城池与你的王城古城守卫兵力+1一样，每 1 turn 会为你生产 1 个兵，所以拥有自己的城池至关重要。

迷雾

<img src="https://dev.generals.io/replay.gif" style="zoom:50%;" />

## AI部分

### 数据集

#### 获取

此网站提供了通过api接口接入玩家的bot程序的服务，

<img width="934" alt="img2" src="https://user-images.githubusercontent.com/97432569/171527249-b5595391-871e-4d81-8f9e-6f900b8090c7.png">

另外还提供了收集的对局数据供训练bot使用

<img width="785" alt="img3" src="https://user-images.githubusercontent.com/97432569/171527351-265a0013-de67-4622-bb96-e2af565b970c.png">



但是，当我们试着下载对局时（链接已经过期）:

<img width="748" alt="img4" src="https://user-images.githubusercontent.com/97432569/171527398-3c5bc07d-c035-4e74-93fb-e18693fe2f44.png">


但是网站的管理员并没有修复这个链接（坏了就坏了吧...）：

<img width="531" alt="img5" src="https://user-images.githubusercontent.com/97432569/171528387-512f83f4-317b-4900-a83f-8e255f53e1b8.png">

但是好在这个网站另外还提供了下载单次对局回放的功能：

<img width="440" alt="img6" src="https://user-images.githubusercontent.com/97432569/171528503-ba4cb952-3bb9-4a6d-9c9c-a27086e42756.png">

于是写了一个网络爬虫，爬取了排行榜上前三十的玩家，大约八万局数据：[link](https://github.com/Toseic/Generals.io_crawl)

<img width="614" alt="img7" src="https://user-images.githubusercontent.com/97432569/171528605-c3ce6f31-169d-46c5-b4e3-9001e8f69152.png">



### 数据结构

显而易见：直接拿一个二维的向量储存就行了

问题：数据集里面的地图的尺寸是不确定的，那怎么办呢？

| width | times | height | times |
| ----- | ----- | ------ | ----- |
| 16    | 4     | 17     | 9     |
| 17    | 20    | 18     | 13327 |
| 18    | 13359 | 19     | 9519  |
| 19    | 9527  | 20     | 6424  |
| 20    | 6488  | 21     | 5187  |
| 21    | 5070  | 22     | 4164  |
| 22    | 4279  | 23     | 1893  |
| 23    | 1786  |        |       |

将地图放大到25*25，相较于原地图多出来的部分放置山峰，也就是士兵不能到达的地方，原地图嵌在新地图的左上方，。
![img8](https://user-images.githubusercontent.com/97432569/171543245-73593ba7-ad3f-4b0b-8688-05cdecf95a44.png)



#### 输入：

包含十个通道，内容均为没被迷雾遮挡的部分

| 内容                                            | 通道数 |
| ----------------------------------------------- | ------ |
| 自己以及敌人的土地                              | 2      |
| 自己以及敌人的城市                              | 2      |
| 自己以及敌人的大本营                            | 2      |
| 尚未被人攻占的城市                              | 1      |
| 山峰                                            | 1      |
| 自己距离下一个城市增加士兵（per 1 turn）的时间  | 1      |
| 自己距离下一个土地增加士兵（per 25 turn）的时间 | 1      |



思考：由于地图的右方和下方固定会有大量的一片山峰，这会不会对AI的学习产生影响呢？ -> 数据增强

### 数据增强

由于这个游戏的结构就是一张地图，数据增强的方式也显而易见，将地图以按一定概率旋转，翻转之后，生成新的数据。



### 模型结构：

使用了二维卷积，12层resblock

![model](https://user-images.githubusercontent.com/97432569/171543260-9eb8c905-708c-4259-b8fa-29247bf9a04a.svg)
<img width="775" alt="model2" src="https://user-images.githubusercontent.com/97432569/171543273-8a51da14-f9db-46eb-8d13-1020e01c1eba.png">



## UI部分

众所周知，UI对于一个游戏至关重要。

### 简约线条：

这个版本的UI仅仅只是用于调试，可以拿来复现已有的对局，赛博朋克风格（X

不支持人类输入
![img1](https://user-images.githubusercontent.com/97432569/171473506-ad83a3e1-8dab-4e61-add6-134b78cc78ce.gif)

### 极致色彩：

使用pygame框架，可以实现基本与网站上类似的体验，包括鼠标点击事件，键盘输入事件等等，支持人机对局，AI对局，暂时不支持双人对局

缺点：画质较低（屏幕兼容问题）

图片
<img width="601" alt="img9" src="https://user-images.githubusercontent.com/97432569/171543289-e972619c-379b-4534-acbc-d67dbd71542f.png">
<img width="602" alt="img10" src="https://user-images.githubusercontent.com/97432569/171543298-bbcefc73-9544-4dbf-bf01-1b70efc87a39.png">
![img2](https://user-images.githubusercontent.com/97432569/171551458-3c3ab83d-b849-406d-bb70-4c0840e9e637.gif)



## future:

1.  让我们的AI加入到网站里面进行对战
2. 设计本地对战平台，可以短时间进行大量对局，以用于比较AI的水平
3. 写搜索算法，跟AI对战，查看效果
4. 开发双人对局功能（网络双端连接？）