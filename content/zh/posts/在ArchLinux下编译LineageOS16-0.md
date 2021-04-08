---
title: 在ArchLinux下编译LineageOS16.0
date: 2019-09-15 13:53:02
type: post
tags:
- Android
- LineageOS
- Geek
---

## 9/17更新

你依然可以通过本文进行其他类似机型的编译

  ~~本人实测小米6刷入编译出来的包会卡米，也就是卡在开机MI logo上，和一些dalao讨论貌似是小米部分机型与lineage自带的clang版本冲突。可能被波及了。等我XDA问完，这边再给可能的解决方案。~~

就在我痛苦不堪的时候，发现LOS团队[悄悄把小米部分机型恢复了。](https://github.com/LineageOS/hudson/commit/9e0206569670a2566562e000b9f1e3492eacca83)

怕折腾的，还是刷官方包吧。

---
## 0x00 背景事件

8月9日，北京商汤在github上提交了[DMCA投诉](https://github.com/github/dmca/blob/master/2019/08/2019-08-09-SenseTime-5.md)，因为自家授权给小米的面部解锁专利被“开源”，故请求github删除“涉案”的454个repo，这其中主要提及到了TheMuppets团队一直维护的[小米vendor仓库](https://github.com/TheMuppets/proprietary_vendor_xiaomi)。这个仓库也是LineageOS团队（以下简称为LOS）一直依赖（合作）的vendor源。因为这个事件，LOS团队停止了8月8号之后的xiaomi机型的所有offical的ROM编译工作。截止本post编写时，工作依然没有恢复。官网版本一直停留在0808-nightly上。

作者正是LOS16的用户，这个类原生OS一直给我带来很好的用户体验。但是不幸的是0808版本遗留下了微信QQ等网络电话APP无法获取录音权限的问题，虽然社区很快接收到了反馈并做了patch。但是因为DMCA事件导致我们现在无法收到OTA更新了。现在和父母以及同学交流非常不便，只能回到宿舍使用ipad打电话。1个月后感觉DMCA事件没有什么解决进展，被逼无奈的我只好自己动手编译最新代码。

<!--more-->

## 0x01 本文参考

|名称|网址|
|:------:|:------:|
| 安卓环境搭建与编译 for Arch Linux | https://blog.firerain.me/article/13 |
| Build for sagit | https://wiki.lineageos.org/devices/sagit/build |
|Archwiki-Setting up the build environment|https://wiki.archlinux.org/index.php/Android#Setting_up_the_build_environment|
| Magisk Hide on ADB | https://github.com/topjohnwu/Magisk/issues/688 |

## 0x02 本文编译环境

OS：Arch Linux  
AUR Helper：yay （其他helper用户请自觉更改下文相关代码）  
机型：sagit （Mi6 小米6）（其他机型用户请在下文相关代码自觉替换成自己使用的设备名，不知道的请去LOS官网上查看）

## 0x03 编译前准备工作

**在开始前，还是建议有丰富linux操作经验的人进行编译，如果你是新手，并希望从中学到东西，请结合的看0x01部分的文章，因为本文不会对每一步进行太多缘由上的解释，因为大多数经验者一看便知。**

**每一步都至关重要，但是加粗步骤尤其需要谨慎对待。**

**本文及本文作者不对您的数据丢失，系统崩溃，手机变砖等事故负责**

1. 请先确保编译机最低有170GB以上剩余空间，如果你要开cache，你需要更多。有条件者请在服务器与虚拟机上跑。（我穷只能物理机）
2. 建议8G内存起，并且最好开启swap，这个不是刚需，但是more memory，less wasted time。
3. **推荐全程使用bash而非zsh或者fish。** 如果你还是想用zsh，请设置一下环境变量。
   ```bash
    setopt shwordsplit
    export LC_ALL=C
   ```
4. 请先yay更新所有包至最新版本。
5. 安装LOS编译环境，安装AUR包lineageos-devel
   ```bash
    yay -S lineageos-devel
   ```
6. 创建一个编译环境目录
   ```bash
    mkdir ~/lineageos
    cd lineageos
   ```
7. 准备python2环境，由于安卓编译只支持python2，但是arch默认为python3，建议通过venv来创建python2环境并避免对系统做大的改动。
   ```bash
    yay -S python2-virtualenv
    virtualenv2 --system-site-packages venv
    source venv/bin/activate
   ```



## 0x04 同步LOS源码
```bash
    repo init -u git://github.com/LineageOS/android.git -b lineage-16.0 --depth=1
    repo sync -c -j4 --force-sync --no-clone-bundle --no-tags
```
> 注: 以上的 --depth=1 参数代表只保留本地仓库中的最后一个commit, 可减少空间占用, 你也可以不加，如果空间有限，建议下文中的每一步git clone都加上  
最后一句的 -j4 参数中的4代表4线程, 可自行修改

## 0x05 准备相关tree
安卓编译需要三大Tree：Device Tree , Kernel Tree , Vendor Tree
我们都将通过脚本获取，但其中vendor tree由于上文DMCA原因，已经无法正常获取。我们将手动clone[其他repo](https://gitlab.com/the-muppets/proprietary_vendor_xiaomi)。(先暂时不要过多宣传该repo为好)，如果你clone vendor失败请看下一节0x06
```bash
source build/envsetup.sh
git clone https://gitlab.com/the-muppets/proprietary_vendor_xiaomi vendor/xiaomi --depth=1
breakfast sagit
```

## 0x06 提取专有blobs

>上一节vendor获取成功请跳过本节  
>建议手机已root或者装有magisk，未root用户看相关部分plan B

### 1. PLAN A 若你的手机已经root (Recommanded) 
   > Magisk 用户需要额外的设置，请去Magisk Manager设置中关闭Magisk Hide，并在手机开发者选项中开启ADB root的权限，最后重启。

   开发者选项中开启ADB后通过有线连接电脑

   ```bash
    yay -S patchelf android-tools
    cd device/xiaomi/sagit
    adb root #非常重要
    ./extract-files.sh
   ```

### 2. PLAN B 若你的手机没有root
请阅读这篇官方教程了解如何从OTA zip包中提取文件  
https://wiki.lineageos.org/extracting_blobs_from_zips.html

## 0x07 开启cache加速（可选）

**需要对剩余空间足够自信，我觉得至少200G吧，所谓牺牲空间换时间**
```bash
 export USE_CCACHE=1
 ccache -M 50G #参数为最大cache大小，可调
 export CCACHE_COMPRESS=1 #可选命令 压缩cache，节约空间，牺牲性能
```

## 0x08 开始编译
```bash
 croot
 brunch sagit
```
> 进行到99%开始打包有可能会失败，提示“zip I/O error: No space left on device”。这里并不是指硬盘空间不足，而是/tmp 占满了，重新挂载后继续编译就好。
```bash
 sudo mount -o remount,size=15G,noatime /tmp
 brunch sagit
```

## 0x09 漫长的等待
打开SWICH，异度之刃2启动！！！

## 0x10 获取输出结果
```bash
 cd $OUT
```
目录下应该有两个文件，一个是我们需要的zip，另一个是recovery image。