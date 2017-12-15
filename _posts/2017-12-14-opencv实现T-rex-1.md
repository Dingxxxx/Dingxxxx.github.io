---
layout: post
author: Ding
title: 用opencv实现chrome浏览器离线小游戏T-rex(一)
date: 2017-12-14
categories: Python
tags:
- Python
- OpenCV
---

* content
{:toc}

> T-rex是Chrome浏览器自带的离线小游戏，研究了一下它的源码，并尝试用Opencv复现。
>
> 参考了[博客](http://www.cnblogs.com/undefined000/p/trex_1.html)。



## 概览

![游戏截图](/images/T-rex/demo.png)

游戏截面为 600X150 像素，主要包括五个构造函数：

- 游戏逻辑控制 Runner
- 游戏界面管理 Horizon
  + 地面 HorizonLine
  + 云朵 Cloud
  + 昼夜模式 NightMode
  + 障碍物 Obstacle
- 霸王龙 T-rex
- 分数记录 DistanceMeter
- 游戏结束面板 GameOverPanel

游戏所有的资源都在 1233X68 的Sprite图中。

![sprite图](/images/T-rex/sprite.png)

## 地面 HorizonLine

HorizonLine在sprite图中的位置为 2:1202, 54:66 的矩形范围，可以分为 2:602, 602:1202两种不同的地形。

绘制思路为记录2个600长度的路段范围，判断当前窗口在路段范围的位置，根据绘制两帧的时间间隔更新移动范围。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Wed Dec 13 16:33:19 2017

@author: ding
"""

import cv2
import numpy as np
import matplotlib.pyplot as plt
import random
import time


class horizonLine:
    def __init__(self):
        self.spriteXPos = 2
        self.spriteYPos = 54
        self.width = 600
        self.height = 12
        self.xPos = [0, self.width]
        self.yPos = 127
        self.bumpThreshold = 0.5
        self.sourceXPosPresent = [2, 602]
        self.sourceXPosStart = [2, 602]


    def draw(self, canvas):
        width1 = self.sourceXPosStart[0] + 600 - self.sourceXPosPresent[0]

        canvas[self.yPos: self.yPos+self.height, \
               self.xPos[0]: self.xPos[0]+width1] = \
               sprite[self.spriteYPos: self.spriteYPos+self.height, \
               self.sourceXPosPresent[0]: self.sourceXPosStart[0]+600]

        canvas[self.yPos: self.yPos+self.height, \
               self.xPos[0]+width1: self.xPos[0]+self.width] = \
               sprite[self.spriteYPos: self.spriteYPos+self.height, \
               self.sourceXPosStart[1]: self.sourceXPosPresent[1]]          
        return canvas

    def update(self, deltaTime, speed):
        movement = int(speed * deltaTime * 1000 * speed)

        self.sourceXPosPresent[0] += movement
        self.sourceXPosPresent[1] += movement

        if self.sourceXPosPresent[0] >= self.sourceXPosStart[0]+600 :
            delta = self.sourceXPosPresent[0] - self.sourceXPosStart[0] - 600

            self.sourceXPosStart[0] = self.sourceXPosStart[1]
            self.sourceXPosPresent[0] = self.sourceXPosStart[0] + delta

            self.sourceXPosStart[1] = self.getRandom() + self.spriteXPos
            self.sourceXPosPresent[1]  = self.sourceXPosStart[1] + delta

    def getRandom(self):
        if random.random() > self.bumpThreshold:
            return 600
        else:
            return 0

if __name__ == '__main__'
    sprite = cv2.imread('t-rex.png')
    canvas = np.ones((150,600,3), np.uint8) * 255

    ground = horizonLine()

    while True:
        st = time.time()
        canvas = ground.draw(canvas)
        cv2.imshow('t-rex',canvas)
        time.sleep(0.01)
        speed = 1/2
        ground.update(time.time()-st, speed)
        ch = cv2.waitKey(1)
        if ch == 27:
            break
    cv2.destroyAllWindows()
```

## 另一种实现 ---- 移动地面

上一种直观的实现方式采用了移动画面的方法，但是考虑到后面其他部件的绘制，在此实现一个通用的绘制移动部件的函数接口。

```python
def drawCanvas(canvas, sprite, xPos, yPos, sourceXPos, sourceYPos, width, height):
    """
    canvas: 600X150的画布，x坐标范围0-600
    sprite: 1233X68的sprite图
    xPos: 部件在canvas坐标系下的坐标
    sourceXPos: 部件在sprite中的位置
    width: 部件的长度
    height: 部件的高度
    """
    if xPos < 0:
        canvas[yPos: yPos+height, 0:xPos+width] = \
            sprite[sourceYPos: sourceYPos+height, sourceXPos-xPos: sourceXPos+width]
        return canvas
    if xPos + width > 600:
        canvas[yPos: yPos+height, xPos:600] = \
            sprite[sourceYPos: sourceYPos+height, sourceXPos: sourceXPos+600-xPos]
        return canvas
    canvas[yPos: yPos+height, xPos:xPos+width] = \
        sprite[sourceYPos: sourceYPos+height, sourceXPos: sourceXPos+width]
    return canvas
```

修改后的HorizonLine程序。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Wed Dec 13 16:33:19 2017

@author: ding
"""

import cv2
import numpy as np
import matplotlib.pyplot as plt
import random
import time

WIDTH = 600
class horizonLine:
    def __init__(self):
        self.spriteXPos = 2
        self.spriteYPos = 54
        self.width = 600
        self.height = 12
        self.xPos = [0, 600]
        self.yPos = 127
        self.bumpThreshold = 0.5
        self.sourceXPos = [2, 602]


    def draw(self, canvas, sprite):
        canvas = drawCanvas(canvas, sprite, self.xPos[0], self.yPos,
                            self.sourceXPos[0], self.spriteYPos,
                            self.width, self.height)
        canvas = drawCanvas(canvas, sprite, self.xPos[1], self.yPos,
                            self.sourceXPos[1], self.spriteYPos,
                            self.width, self.height)
        return canvas

    def update(self, deltaTime, speed):
        movement = int(speed * deltaTime * 1000 * speed)

        self.xPos[0] -= movement
        self.xPos[1] -= movement

        if self.xPos[0] < -600:
            self.xPos[0] = self.xPos[1]
            self.sourceXPos[0] = self.sourceXPos[1]
            self.xPos[1] = self.xPos[0] + 600
            self.sourceXPos[1] = self.getRandom() + self.spriteXPos


    def getRandom(self):
        if random.random() > self.bumpThreshold:
            return 600
        else:
            return 0

def drawCanvas(canvas, sprite, xPos, yPos, sourceXPos, sourceYPos, width, height):
    """
    canvas: 600X150的画布，x坐标范围0-600
    sprite: 1233X68的sprite图
    xPos: 部件在canvas坐标系下的坐标
    sourceXPos: 部件在sprite中的位置
    width: 部件的长度
    height: 部件的高度
    """
    if xPos < 0:
        canvas[yPos: yPos+height, 0:xPos+width] = \
            sprite[sourceYPos: sourceYPos+height, sourceXPos-xPos: sourceXPos+width]
        return canvas
    if xPos + width > 600:
        canvas[yPos: yPos+height, xPos:600] = \
            sprite[sourceYPos: sourceYPos+height, sourceXPos: sourceXPos+600-xPos]
        return canvas
    canvas[yPos: yPos+height, xPos:xPos+width] = \
        sprite[sourceYPos: sourceYPos+height, sourceXPos: sourceXPos+width]
    return canvas      

if __name__ == '__main__':

    sprite = cv2.imread('t-rex.png')
    canvas = np.ones((150,600,3), np.uint8) * 255

    #cv2.namedWindow('b', cv2.WINDOW_NORMAL)
    #cv2.imshow('b',sprite)

    ground = horizonLine()

    canvas = ground.draw(canvas, sprite)
    cv2.imshow('t-rex',canvas)

    while True:
        st = time.time()
        canvas = ground.draw(canvas, sprite)
        cv2.imshow('t-rex',canvas)
        time.sleep(0.01)
        speed = 1/2
        ground.update(time.time()-st, speed)
        ch = cv2.waitKey(1)
        if ch == 27:
            break
    cv2.destroyAllWindows()

``````
