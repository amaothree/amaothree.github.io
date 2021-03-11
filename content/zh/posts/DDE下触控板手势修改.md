---
title: DDE下触控板手势修改
date: 2019-08-04 19:42:22
tags:
- dde
- Geek
- Arch
---

这个post比较短。只是起到备忘的操作。

因为我一直是arch+dde的使用环境，deepin还是有不少能吸引我的地方，就比方说开发组的坚持，以及良好的社区。

因为我一直习惯是触控板自然滚动，即两指向上为页面向下这样的。这一点可以再设置面板里面改，但是却仅限两指滚动。而四指左右滑切换桌面却还是正规的左右，想要变成自然滑动，需要额外做一下配置。

<!--more-->

我找到了配置文件目录。
```
/usr/share/dde-daemon/gesture.json 
```
我们用nano打开（当然，你习惯用什么编辑器用什么）。定位到这两段，
```json
    {
        "Name": "swipe",
        "Direction": "left",
        "Fingers": 4,
        "Action": {
            "Type": "built-in",
            "Action": "ReverseSwitchWorkspace"
        }
    },
    {
        "Name": "swipe",
        "Direction": "right",
        "Fingers": 4,
        "Action": {
            "Type": "built-in",
            "Action": "SwitchWorkspace"
        }
    },
```

这里将left，right互换就好了，如果你想改其他的手势也在这个文件里。

改完重启就好了。