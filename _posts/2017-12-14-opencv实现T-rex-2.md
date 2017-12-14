---
layout: post
author: Ding
title: 用opencv实现chrome浏览器离线小游戏T-rex(二)
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



## 云朵Clouds

```python
class Clouds:
    def __init__(self):
        self.width = 46
        self.height = 14        
        self.maxGap = 400
        self.minGap = 100
        self.maxLevel = 30
        self.minLevel = 71
        self.maxClouds = 6
        self.cloudFrequncy = 0.5
        self.clouds = []
        self.spriteXPos = 86
        self.spriteYPos = 2
        self.numClouds = 0
        self.cloudGap = 0

    def update(self, deltaTime, speed):
        # 如果当前有云朵，更新云朵，判断是否存在超出范围的
        if self.numClouds != 0:
            movement = int(speed * deltaTime * 500)
            for cloud in self.clouds:
                cloud[0] -= movement
            if self.clouds[0][0] + self.width <= 0:
                self.clouds.pop(0)
                self.numClouds -= 1
        # 如果当前云朵数量少于上限并且云朵位置大于间隔，随机添加云朵
        if (self.numClouds < self.maxClouds and
            (600 - self.clouds[self.numClouds-1][0]) > self.cloudGap and
            self.cloudFrequncy > random.random()):
            self.addCloud()
        return

    def addCloud(self):
        """
        在画面最后增加一个云朵，更新一个随机的间隔
        """
        yPos = random.randint(30, 71)
        xPos = 600
        self.clouds.append([xPos, yPos])
        self.numClouds += 1
        self.cloudGap = random.randint(self.minGap, self.maxGap)
        return

    def draw(self, canvas, sprite):
        for cloud in self.clouds:
            canvas = drawCanvas(canvas, sprite, cloud[0], cloud[1],
                    self.spriteXPos, self.spriteYPos,
                    self.width, self.height)
        return canvas
```

用一个列表维护云朵，两个云朵之间有最小间隔和最大间隔，云朵的高度也是随机的，更新云朵时，如果第一个
超出canvas范围则删除。


## 问题

![云朵错误](/images/T-rex/cloud_error.png)

```python
if __name__ == '__main__':

    sprite = cv2.imread('/home/ding/Downloads/t-rex.png')
    canvas = np.ones((150,600,3), np.uint8) * 255

    #cv2.namedWindow('b', cv2.WINDOW_NORMAL)
    #cv2.imshow('b',sprite)
    speed = 1/2

    ground = HorizonLine()
    clouds = Clouds()
    clouds.addCloud()
    canvas = clouds.draw(canvas,sprite)
    canvas = ground.draw(canvas, sprite)
    cv2.imshow('t-rex',canvas)

    while True:
        st = time.time()
        canvas = ground.draw(canvas, sprite)
        canvas = clouds.draw(canvas, sprite)
        cv2.imshow('t-rex',canvas)
        time.sleep(0.01)
        ground.update(time.time()-st, speed)
        clouds.update(time.time()-st, speed)
        ch = cv2.waitKey(1)
        if ch == 27:
            break
    cv2.destroyAllWindows()
```

测试函数中，每次画布是在上一张的基础上进行更新，所以造成了云朵轨迹残留的问题，可以每次更新画面是新生成一个画布即可解决问题。

```python
while True:
    canvas = np.ones((150,600,3), np.uint8) * 255
    st = time.time()
    canvas = ground.draw(canvas, sprite)
    canvas = clouds.draw(canvas, sprite)
    cv2.imshow('t-rex',canvas)
    time.sleep(0.01)
    ground.update(time.time()-st, speed)
    clouds.update(time.time()-st, speed)
    ch = cv2.waitKey(1)
    if ch == 27:
        break
```

![正确云朵](/images/T-rex/clouds.png)
