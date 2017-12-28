---
layout: post
author: Ding
title: Ubuntu下的环境变量文件
date: 2017-12-28
categories: Linux
tags:
- Ubuntu
---

* content
{:toc}

> 参考[博客](http://www.cnblogs.com/hzhida/archive/2012/08/06/2624998.html)






在 Ubuntu 系统中有两种设置环境变量 PATH 的方法。第一种适用于为单一用户设置 PATH，第二种是为全局设置 PATH。

## 设置环境变量

+ 第一种方法：
在用户主目录下有一个 .bashrc 文件，可以在此文件中加入 PATH 的设置如下：

```
export PATH=”$PATH:/your path1/:/your path2/…..”
```

注意：每一个 path 之间要用 “:“ 分隔。
注销重启 X 就可以了。

+ 第二种方法：
在 /etc/profile 中增加。

```
PATH="$PATH:/home/zhengb66/bin"
export PATH
```

## 环境变量详解
环境变量是和 Shell 紧密相关的，用户登录系统后就启动了一个 Shell。对于 Linux 来说一般是 bash，但也可以重新设定或切换到其它的 Shell。对于 UNIX，可能是 CShelll。环境变量是通过 Shell 命令来设置的，设置好的环境变量又可以被所有当前用户所运行的程序所使用。对 于 bash 这个 Shell 程序来说，可以通过变量名来访问相应的环境变量，通过 export 来设置环境变量。下面通过几个实例来说明。

### etc/profile:
此文件为系统的每个用户设置环境信息, 当用户第一次登录时, 该文件被执行.

并从 `/etc/profile.d` 目录的配置文件中搜集 shell 的设置.

注：在这里我们设定是为所有用户可使用的全局变量。

### /etc/bashrc:
为每一个运行 bash shell 的用户执行此文件. 当 bash shell 被打开时, 该文件被读取.

### ~/.bash_profile:
每个用户都可使用该文件输入专用于自己使用的 shell 信息, 当用户登录时, 该文件仅仅执行一次! 默认情况下, 他设置一些环境变量, 执行用户的. bashrc 文件.

注：~ 在 LINUX 下面是代表 HOME 这个变量的。

另外在不同的 LINUX 操作系统下，这个文件可能是不同的，可能是 ~/.bash_profile； ~/.bash_login 或 ~/.profile 其中的一种或几种，如果存在几种的话，那么执行的顺序便是：~/.bash_profile、 ~/.bash_login、 ~/.profile。比如我用的是 Ubuntu，我的用户文件夹下默认的就只有~/.profile 文件。

### ~/.bashrc:

该文件包含专用于你的 bash shell 的 bash 信息, 当登录时以及每次打开新的 shell 时, 该文件被读取.
(注：这个文件是 . 开头的，所以在文件夹中被隐藏了)

那么我们如何添加自己定义的环境变量呢？

用记事本打开这个文件，然后在里面最后写上:

```
xiaokang=kangkang
```
然后保存，这样每次打开一个新的 terminal 的时候，我们这个变量就生效了。记住，如果你已经打开一个 terminal，然后你修改了这个文件，那么在这个 terminal 下是不会生效的。

一般情况用户最好在这里进行修改，但是有时候会覆盖父级的变量，比如 PATH 是 ROOT 设定的，但是如果你在这个文件里面写了 PATH=xx, 那么将来所有的 PATH 都成了 xx 了，所以我们应该在这个文件中写为：

```
PATH＝$PATH:xx
```
这样就把原来的和你自己的一起加上了。而且注意在 LINUX 系统下用：分割，而不是 windows的；

3 和 4 都是在用户目录下的，他们唯一的不同是: `.bash_profile` 只能在登录的时候启动一次。在我的 Ubuntu 里面这个 3 文件似乎没有。

### ~/.bash_logout:

当每次退出系统 (退出 bash shell) 时, 执行该文件.

## 总结
另外,`/etc/profile`中设定的变量 (全局) 的可以作用于任何用户, 而`~/.bashrc` 等中设定的变量 (局部) 只能继承 `/etc/profile` 中的变量, 他们是 \"父子 \" 关系.

`~/.bash_profile` 是交互式、login 方式进入 bash 运行的

`~/.bashrc` 是交互式 non-login 方式进入 bash 运行的

通常二者设置大致相同，所以通常前者会调用后者。

好的，总结一下他们的执行方式：

当你登录并且登录 shell 是 bash 时, bash 首先执行 `/etc/profile` 文件中的命令 (如果该文件存在), 然后它顺序寻找`~/.bash_profile``,~/.bash_login` 或`~/.profil`e 文件, 并执行找到的第一个可读文件中的命令. 当登录 bash 退出时, 它 将执行`~/.bash_logout` 文件中的命令.

当启动一个交互的 bash 时, 它将执行`~/.bashrc` 文件中的命令 (如果该文件存在并且可读). 当非交互地启动以运行一个 shell 脚本时, bash 将查找 bash_env 环境变量, 确定执行文件的名称.
