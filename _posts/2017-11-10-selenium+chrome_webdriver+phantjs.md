---
layout: post
author: Ding
title: 配置selenium和chromeDriver
date: 2017-11-10
categories: Python
tags:
- Python
- scrapy
---

* content
{:toc}


> opencv-python中使用cv2.destroyAllWindows()来关闭窗口时会出现程序崩溃的问题，找了很多方法，目前最好的解决策略是这样的。








## 安装selenium

```
conda install selenium
```

安装完成后测试：

```python
from selenium import webdriver

url='https://dingxxxx.github.io/'
driver = webdriver.Chrome()
driver.maximize_window()
driver.get(url)  
```

发现报错，没有chromedriver，去[官网](https://sites.google.com/a/chromium.org/chromedriver/home)下载.

根据chrome版本号`google-chrome --version`下载对应的chromedriver.

将下载后的ChromeDriver解压后，移到`/usr/bin`中.

在python中使用时,用ChromeDriver所在路径初始化webdriver。

```python
import time
from selenium import webdriver

driver = webdriver.Chrome('/usr/bin/chromedriver')  # Optional argument, if not specified will search path.
driver.get('https://dingxxxx.github.io/');
time.sleep(5) # Let the user actually see something!
driver.quit()
```
