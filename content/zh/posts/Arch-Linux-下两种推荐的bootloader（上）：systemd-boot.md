---
title: Arch Linux 下两种推荐的bootloader（上）：systemd-boot
date: 2019-03-24 18:43:46
type: post
tags: 
- Arch
- EFI
- Geek
---

维持一个博客还是蛮难的，并不是说服务器的维护，而是说更新博文。

周更的Flag很快就倒了，一是说这一段时间学校的B事实在是太多，自己也不太争气。二是说，自己也不知道要写一些什么好，我还是太弱了，什么都不精通，了解的也少。

不过最近正好安装了rEFInd用来替代之前一直用的systemd-boot，就一并介绍了。

先说说为什么不说更常用的GRUB，首先是之前一大段时间一直是archlinux单系统的，没有必要用bootloader，所以GRUB这种较为“庞大”的我就放弃掉了。其次是本人一直以来非主流思想较为严重，对待主流事物一直有不健康的偏见思想……所以就更爱用一些“非主流”的东西。当然这里是相对的。实际上今天介绍的两个Bootloader还是很流行的。

<!--more-->

## 先说说systemd-boot：
> **systemd-boot**, previously called gummiboot (German for: 'rubber dinghy'), is a simple UEFI boot manager which executes configured EFI images. The default entry is selected by a configured pattern (glob) or an on-screen menu to be navigated via arrow-keys. It is included with systemd, which is installed on an Arch system by default.  
It is simple to configure but it can only start EFI executables such as the Linux kernel EFISTUB, UEFI Shell, GRUB, or the Windows Boot Manager. （FROM Arch Wiki）

systemd-boot给我的最直观感受是，它应该是最符合极客标准的bootloader。结合自身使用经历后，总结一下几个特点：
* 极度简洁，没有任何花里胡哨的东西，纯CLI启动。
* 配置及其简单，配置config文件只需要三行。
* 启动迅速，没有卡顿感。

配合SSD，从按下开机键开始计时，进入DM不到10s（当然我是因为把菜单等待时间设为1s了）。

-----

## 下面来说说咋安装。

**（教程中默认ESP挂载点是在/boot,请大家注意结合自身情况修改相应部分）**

1. 首先这个模块是归在systemd里的，所以不需要安装任何其他的包的。

2. 执行安装脚本：
```sh
sudo bootctl --path=/boot install
```

3. 编辑配置文件
```sh
sudo nano /boot/loader/loader.conf
```

```config
timeout 1       #菜单超时时长，单位为秒
default arch    #默认的entry文件名（下一步介绍）
```

是的，没有看错，这个文件两行就可以了。

4. 编辑entry文件
```sh
sudo nano /boot/loader/entries/arch.conf 
```
文件名啥都行，但是得注意的是，默认的entry的文件名修改后，也要修改loader.conf对应的部分。
```config
title   Amao Linux  #启动菜单内显示的改启动项的标题
linux   /vmlinuz-linux
initrd  /intel-ucode.img    #amd cpu的修改为amd-ucode
initrd  /initramfs-linux.img
options root=PARTUUID=3b877628-b582-3749-b36b-b999b39f315c rw

# 根分区的partuuid可以通过下面这一行命令查看
# ls -l /dev/disk/by-partuuid
```

5. 完成了！是不是超简单！不过正因为是CLI，所以没有办法美化，但说不定通过写entries的标题可以搞出来字符画（大雾

6. 进UEFI修改boot启动顺序就好了


下一篇介绍另一个，rEFInd