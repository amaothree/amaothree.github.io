---
title: Arch Linux 下两种推荐的bootloader（下）：rEFInd
date: 2019-03-24 19:39:53
tags:
- Arch
- EFI
- Geek
---

[上一篇](https://amao.run/zh/posts/arch-linux-%E4%B8%8B%E4%B8%A4%E7%A7%8D%E6%8E%A8%E8%8D%90%E7%9A%84bootloader%E4%B8%8Asystemd-boot/)介绍了如何安装systemd-boot，这一篇给大家介绍另个非热门的热门bootloader，也更好看的refind。

![](https://camo.githubusercontent.com/999cff82d4bea54f222e165d647b5df597f45b86/687474703a2f2f692e696d6775722e636f6d2f33624d473655372e706e67)

> rEFInd is a UEFI boot manager capable of launching EFISTUB kernels. It is a fork of the no-longer-maintained rEFIt and fixes many issues with respect to non-Mac UEFI booting. It is designed to be platform-neutral and to simplify booting multiple OSes. 

refind给我的感觉就是不仅好看而且更实用，配置起来章法可依，参数详细，定制化更简单。好看的主题虽然数量应该不及GRUB多，但是质量均为不俗。

最重要的是即使不做配置，它的全盘搜索efi可以检测到你所有的硬盘上拥有的安装的所有启动项！非常强大，非常方便，可以直接开箱即用！

正因如此，它开机会慢一丢丢，但是因为直接读取了UEFI，所以也不会像是搜索文件那么夸张。仅仅是慢一丢丢。

<!--more-->

-------------

## 哇！那我要怎么安装呢？

1. 安装refind包
```shell
sudo pacman -S refind-efi
```
2. 执行安装脚本
```shell
refind-install
```
3. archlinux下会默认生成两个配置文件，一个是/boot/refind_linux.conf，另一个/boot/EFI/refind/refind.conf，本篇教程只针对后者，前者删除即可。当然这里不是说前者无用，具体差距请看[wiki](https://wiki.archlinux.org/index.php/REFInd#Passing_kernel_parameters)

4. 这里不推荐直接在refind.conf上修改，最好新建一个对照的有所依的添加功能。
```shell
cd /boot/EFI/refind/
sudo mv ./refind.conf ./refind.conf.backup
sudo nano ./refind.conf
```

5. 还是要看一下根分区的partuuid
```shell
ls -l /dev/disk/by-partuuid/
```

6. 编辑config文件
```shell
sudo nano ./refind.conf
```
配置文件内容如下：
```conf
timeout 4

resolution 1920 1080

dont_scan_files vmlinuz-linux,systemd-bootx64.efi

default_selection "Arch Linux"

menuentry "Arch Linux" {
    icon     /EFI/refind/icons/os_arch.png
    volume   "Arch Linux"
    loader   /vmlinuz-linux
    initrd   /initramfs-linux.img
    options  "rw root=PARTUUID=3b877628-b582-3749-b36b-b999b39f315c radeon.si_support=0 amdgpu.si_support amdgpu.dc=1 initrd=/intel-ucode.img"
    submenuentry "Boot using fallback initramfs" {
        initrd /initramfs-linux-fallback.img
    }
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
    submenuentry "Boot to single user mode" {
                add_options "single"
    }
}
```

这里对配置文件做个说明：
1. 第一行是timeuot，不多说了
2. 第二行是分辨率
3. 第三行是你不希望refind扫出来哪几个启动项，因为我们下面手写了一个archlinux的entry，因此如果不禁止它去扫，开机菜单就会出来重复的启动项！难看！于是我这里就禁止了linux和之前建立的systemd-boot（这个也会被扫出来哦）。这个参数有很多用法，具体看之前生成的他的config文件，很详细。
4. 第四行，默认启动项。
5. 第五行以下，entry的详细配置，重点是第一个闭包，这里和其他的bootloader写法大同小异，ucode的调用要写到options里，我这个例子还有一些我用到的内核参数，都是统统写到这里。根分区的partuuid别忘了写！

具体entry如何写，其他os的写法，可以参照wiki里的，也可以直接看原来的config里，里面还有其他的很多设置参数。我这里用不上，大家依需自取。

------------------
## 如何美化

你重启后就会发现，默认主题实在是丑爆了！

github上有非常好看的主题，这里举例推荐[rEFInd Minimal](https://evanpurkhiser.com/rEFInd-minimal/),我们来安装它。

```shell
1. cd /boot/EFI/refind/
2. mkdir ./themes
3. cd ./themes
4. git clone https://github.com/EvanPurkhiser/rEFInd-minimal.git
5. cd ../
6. nano ./refind.conf
```
这里修改两个地方，一个是修改原来arch的entry配置里的icon位置，另一个是在conf末尾加入include。
```shell
timeout 4

…………………………
    icon /EFI/refind/themes/rEFInd-minimal/icons/os_arch.png
…………………………

include themes/rEFInd-minimal/theme.conf
#EOF
```
好了，保存后重启看看！！
