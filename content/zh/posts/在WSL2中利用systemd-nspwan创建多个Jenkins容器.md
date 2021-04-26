---
title: "在WSL2中利用Systemd-nspwan创建多个Jenkins容器"
date: 2021-04-26T19:37:06+02:00
type: post
---

## 0x01 简介

本教程将会介绍如何利用Systemd-nspwan容器技术在WSL2中创建多个相互隔离的容器并在其中运行多个Jenkins。

本教程具有相当大的普适性。虽然本文名中与WSL2有关，但接下来的大部分操作在任何使用Systemd的Linux发行版上也可以完全使用（并且不需要进行下一节“准备”中的任何操作）。仅需要根据实际发行版情况稍作修改（e.g. 包管理器的不同，容器系统的不同）。

<!--more-->

## 0x02 准备

### 1. 安装WSL2

相较于第一代WSL，WSL2中使用的虚拟化技术使得我们使用Systemd工具链变得可能。WSL2与Windows 10的交互整合也是一般虚拟机所不能及的。这也是为什么本文推荐在Windows 10系统中优先尝试使用WSL2来满足开发或运维对Linux环境的需求。

本教程中你可以选择任意WSL2发行版，比如官方支持的Ubuntu，Debian，openSUSE或Alpine等，又或者其他开发者开源支持的项目如ArchWSL。

作为Arch Linux的实际使用者以及爱好者，**接下来的我将选择[ArchWSL项目](https://github.com/yuk7/ArchWSL)作为我的WSL2发行版。**

而具体如何安装WSL2，请阅读[微软的官网文档](https://docs.microsoft.com/en-us/windows/wsl/install-win10)。

### 2. 安装Genie

由于WSL2的设计，Systemd在WSL2中并不是默认以PID 1为启动的。这将导致我们基本上无法正常使用Systemd的功能。然而有一个小工具可以帮助我们非常容易的修复之。

这就是Genie，在[Genie的项目仓库](https://github.com/arkane-systems/genie#installation)中你可以轻松找到针对各种包管理器的安装包。

一旦在WSL2中安装后你可以在Powershell中直接启动一个“Genie模式”的WSL shell

`wsl genie -c bash`

或者在WSL中切换

`genie -c bash`

### 3. 安装Windows Terminal（可选）

在微软商店中就可以直接安装，开箱即用。强烈推荐。

## 0x03 创建我们的第一个Systemd-nspwan容器

### 0. **进入Genie模式下的WSL shell**

* 在Powershell中

>```Powershell  
>wsl genie -c bash
>```

* 或，在WSL中

>```Bash  
>genie -c bash
>```

### 1. 为容器创建一个文件夹

从主机的视角上来看，Systemd-nspwan的容器就是一个一个的文件夹。这是因为Systemd-nspwan是轻量命名空间虚拟化技术。和chroot命令相似，但是更加强大，它完全虚拟化了文件系统层次结构、进程树、各种 IPC 子系统以及主机和域名。

接下来我们在家目录下创建一个文件夹用来存放我们稍后建立的容器以及为第一个容器建立一个文件夹：

```Bash
mkdir -p ~/MyContainers/Container1
```

然后进入*MyContainers*文件夹

```Bash
cd ~/MyContainers/
```

### 2. 装配容器

这一步我们要在容器中安装基础的linux文件系统。这里我推荐并演示安装Arch Linux至容器中，因为它非常的简单简洁，不包含任何多余的软件。可以极大的缩小一个容器的体积。当然你也可以选择安装Debian或者Ubuntu，但是方法有所不同，请自己寻找安装教程。

#### 2.1 获取arch-install-scripts安装脚本

首先在WSL中安装Arch Linux社区维护的安装脚本

```Bash
sudo pacman -S arch-install-scripts
```

#### 2.2 安装Arch Linux至容器

```Bash
sudo pacstrap -i -c ./Container1/ base
```

> 如果在安装过程出现错误“Detected unsafe path transition / → ...”，可以忽略，这并不影响接下来的操作。这是由于WSL中根目录所有权与正常Linux环境有所不同。

#### 2.3 创建容器的符号链接

针对不同使用情况，Systemd为nspwan容器设计了多种不同的管理工具。我们将使用*machinectl*工具，但这要求我们要在/var/lib/machines/中创建容器，但是更好的解决办法是使用符号链接。
> 注意：创建符号链接不要使用相对路径，而是绝对路径。

`sudo ln -s /home/usename/MyContainers/Container1/ /var/lib/machines/`
> 这里*usename*请替换成你的linux用户名。  
> 如果你使用的是root身份，那么家目录的路径为/root。

检查是否成功链接

```bash
machinectl list-images
```

输出结果应该类似于

```bash
NAME       TYPE      RO USAGE CREATED                      MODIFIED
Container1 directory no   n/a Fri 2021-04-09 18:18:39 CEST n/a
```

#### 2.4 设置容器网络

受*machinectl*管理的容器默认使用的是私有网络，容器的网络是完全隔离的（与外部网络以及其他容器）。这将不适合我们管理多个Jenkins，因此我们将其改成主机网络，使得主机可以访问容器内的服务。

容器的配置文件路径为`/etc/systemd/nspawn/container-name.nspawn`。配置文件的名字与容器名对应。

我们编辑配置文件（使用nano或者vim）。

```bash
sudo nano /etc/systemd/nspawn/Container1.nspawn
```

向文件中加入如下两行：

```cfg
[Network]
VirtualEthernet=no
```

#### 2.5 启动容器，root密码

通过`machinectl`启动容器：

```bash
sudo machinectl start Container1
```

查看容器状态

```bash
machinectl list
```

通过容器的交互式shell会话，这一步不会调用登陆进程。

```bash
sudo machinectl shell Container1
# When you in the shell
passwd
```

紧接着删除两个文件，否则你接下来将无法正常登录容器

```bash
rm /etc/securetty
rm /usr/share/factory/etc/securetty
exit
```

退出容器的shell后，你就可以用密码登录进容器了

```bash
sudo machinectl login Container1
```

> 如果你忘记密码，要从容器内终止session，请按住 Ctrl 并快速地按 ] 三下。非美国键盘用户应该使用 % 而不是 ]。  
> 当你用这种方法终止session时，别忘了重新进入geine。

### 3. 安装Jenkins

当你登录进入容器中，就可以使用包管理器安装Jenkins和一些必要的程序了，比如：

* Jenkins
* OpenJDK
* Nano or Vim
* Xorg （这是必要的，如果不安装xorg，jenkins就会报有关GUI的错，虽然事实上我们完全没有安装任何GUI程序。）

```bash
# Update all packages
pacman -Syu
# Install packages
pacman -S nano bash-completion jdk-openjdk jenkins xorg
```

在启动jenkins之前，要赋予容器内的/var/cache/jenkins足够权限

```bash
mkdir /var/cache/jenkins
chmod 777 /var/cache/jenkins
```

### 4. 启动并配置Jenkins

因为接下来我们会创建第二个Jenkins容器，所以我们最好修改第一个Jenkins端口。

Jenkins的配置文件在`/etc/conf.d/jenkins`

更改`JENKINS_PORT=--httpPort=8090`中的端口号至`8091`

现在在容器内启动Jenkins就可以了

```Bash
systemctl start jenkins
```

然后通过你Windows上的浏览器（比如Microsoft Edge）打开`http://localhost:8091`配置你的第一个Jenkins就可以了。

你可以放心的`logout`，你的容器会持续运行，你可以随时用如下命令检查容器的状态以及关闭容器：

```bash
machinectl list
machinectl poweroff
```

## 0x04 第二个容器

还记得我之前说过，容器就是一个文件夹吗。如果要创建相似的容器，那么我们只需要复制容器所在的文件夹并生成新的符号链接至`/var/lib/machine`中就可以了。哦对了，别忘了新的容器需要新的配置文件。

复制容器之前别忘了关闭容器。

```bash
# 关闭容器
sudo machinectl poweroff Container1
# 复制容器
sudo cp -r ~/MyContainers/Container1/ ~/MyContainers/Container2/
sudo ln -s /home/amao/MyContainers/Container2/ /var/lib/machines/
sudo cp /etc/systemd/nspawn/Container1.nspawn /etc/systemd/nspawn/Container2.nspawn
```

然后检查一下

```bash
$ machinectl list-images
NAME       TYPE      RO USAGE CREATED                      MODIFIED
Container1 directory no   n/a Fri 2021-04-09 18:18:39 CEST n/a
Container2 directory no   n/a Fri 2021-04-09 18:18:39 CEST n/a

2 images listed.
```

同时启动两个容器

```bash
sudo machinectl start Container1
sudo machinectl start Container2
sudo machinectl list
MACHINE    CLASS     SERVICE        OS   VERSION ADDRESSES
Container1 container systemd-nspawn arch -       -
Container2 container systemd-nspawn arch -       -

2 machines listed.
```

然后根据不同的需要分别进入不同容器进行不同的配置就可以了。比如修改Container2中Jenkins的端口号。之后你就可以通过你的Windows上的浏览器访问不同端口来控制不同的Jenkins。

## 0x04 打包容器

如果你想将容器转移到你的另一台主机上，你当然也可以随时用`machinectl`命令打包导出容器。

```bash
# Export
sudo machinectl export-tar Container1 /home/username/Container1.gz
# Import
sudo machinectl import-tar /home/username/Container1.gz ContainerX
```

你可以在导入导出过程中随时用`machinectl list-transfers`查看进度，用`machinectl cancel-transfer`取消。
