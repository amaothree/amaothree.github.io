---
title: "Creat multi Jenkins containers in Linux based on systemd-nspawn"
date: 2021-04-16T11:31:05+01:00
type: post
tags: 
- Linux
- DevOps
- WSL
---

## 0x00 Introduction

This tutorial will describe how to create multiple isolated containers in Linux and run multiple Jenkins in them using Systemd-nspwan container technology.

<!--more-->

I also write an executable script to help you learning the totural.
You can download and run it:

```bash
curl -o totural.sh https://gist.githubusercontent.com/amaothree/d8bac64e5225b15db84aaf8e3aa6e08d/raw/fdff92ffddac8626584ad823a3b01ce0795c9f4a/totural.
chmod +x ./totural.sh
./totural.sh
```

## 0x01 Background

This section will explain the terms that appear in this tutorial. This will help you to have a better understanding of this tutorial.

### Linux

Broadly refers to a free and open source operating system. In a narrow sense, it refers to the kernel of the Linux operating system. It is popular in computer science and other data processing fields because it can be freely used and modified and is supported by a complete set of runtime libraries.

Different communities distribute versions of the Linux operating system based on the Linux kernel and self-selected software libraries. We call them Linux distributions. Famous distributions include Ubuntu, Debian, CentOS (discontinued), Arch Linux, etc.

### WSL

Windows Subsystem for Linux (WSL) is a compatibility layer for running Linux binary executables natively on Windows 10. And in 2019, WSL 2 was announced, it is based on a a highly optimized subset of Hyper-V features. WSL2 introducing important changes such as a real Linux kernel.
>If you're still confused, think of WSL2 as a small Linux virtual machine developed by Microsoft that is highly optimised for Windows 10 systems.

### Systemd

Systemd is the name of a set of system components under the Linux operating system, and it provides the functionality to manage systems and services.

### Systemd-nspwan

Systemd-nspawn is a container tool in systemd. It can be used to run commands or operating systems in a lightweight namespace container.

Compared to Docker, Systemd-nspawn can only do process isolation (the feature of the namespace) but not resource isolation. Host-to-container or container-to-container processes do not affect each other. But without restrictions, containers and hosts are sharing the same resources (e.g. CPU processing power). Docker uses cgroups to enforce resource isolation (like namespace, cgroups are a part of the linux kernel).

### Jenkins

An famours open source automation server. With the help of many plugins, Jenkins can help developers automate the building and deploying of software projects.

## 0x02 Preparation

Systemd is a toolset unique to linux systems. You must use the linux operating system to complete this operation.

### If you are a Linux user

Please check that the distribution you are using uses systemd to manage your system by default and that the PID of systemd is 1.

```bash
# Run this command to check.
$ ps -A | grep systemd
# The output should include this line.
1 ?        00:00:00 systemd
```

### If you are a macOS user

You can run a modern linux distribution such as Ubuntu, Fedora or Arch Linux in a virtual machine.

### If you are a Windows user

You can run a modern linux distribution such as Ubuntu, Fedora or Arch Linux in a virtual machine.

#### Or If you using Windows 10 Version 1903 or higher

You can install **WSL2** (WSL 1 is not work in this totural).

And for details on how to install WSL2, please read [Microsoft's official documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

Due to the design of WSL2, Systemd is not started with PID 1 by default in WSL2. This will cause that we basically can't use Systemd's functions properly. However, there is a small tool that can help us fix it very easily.

This is Genie, to install it please follow this guide. [Genie's project repository](https://github.com/arkane-systems/genie#installation).

Once installed in WSL2 you can start a "Genie mode" WSL shell directly in Powershell. **Please note that all the commands that follow you should be executed within Genie mode shell.**

* In Powershell

>```Powershell  
>wsl genie -c bash
>```

* Or，In WSL bash

>```Bash  
>genie -c bash
>```

## 0x03 Create our first Systemd-nspwan container

### 1. Create a folder for the container

From the host's view, Systemd-nspwan's containers are just folders by folders. This is because Systemd-nspwan is a lightweight namespace virtualization technology. Similar to the chroot command, but more powerful, it completely virtualizes the file system hierarchy, process tree, various IPC subsystems, and hosts and domains.

Next we create a folder in our home directory to hold the containers we will create later and a folder for the first container.

```Bash
mkdir -p ~/MyContainers/Container1
```

Then change into the *MyContainers* folder

```Bash
cd ~/MyContainers/
```

### 2. Assembling container

In this step we have to install the base linux filesystem in the container. Here I recommend and will demonstrate the installation of Arch Linux into a container, because it is very simple and clean without including any extra software. It can greatly reduce the size of a container. Of course you can also choose to install Debian or Ubuntu, but the method is different, please find the installation tutorial by yourself.

#### 2.1 Get the installation script

First install the Arch Linux community maintained installation script in WSL. Different package managers use different installation commands.

```Bash
# Arch Linux
sudo pacman -Syyu arch-install-scripts
# Debian buster or higher/Ubuntu 18.04LTS or higher
sudo apt update
sudo apt install arch-install-scripts
sudo pacman-key --init
sudo pacman-key --populate archlinux
# Fedora
sudo dnf install arch-install-scripts
sudo pacman-key --init
sudo pacman-key --populate archlinux
# 
```

#### 2.2 Installing Arch Linux to container

```Bash
sudo pacstrap -i -c ./Container1/ base
```

> If you get the error "Detected unsafe path transition / → ..." during the installation, you can ignore it. It will not affect the next operations. This is due to the fact that the root directory ownership in WSL is different from the normal Linux environment.

#### 2.3 Creating symbolic links for container

For different use cases, Systemd has designed several different management tools for nspwan containers. We will use the *machinectl* tool, which requires us to create containers in /var/lib/machines/, but a better solution is to use symbolic links.
> Note: Do not use relative paths to create symbolic links, use absolute paths instead.

`sudo ln -s /home/usename/MyContainers/Container1/ /var/lib/machines/`
> Here *usename* should be replaced with your linux username.  
> If you are using the root login, then the path to the home directory is /root.

Check if linked successfully

```bash
machinectl list-images
```

The output should be similar to

```bash
NAME       TYPE      RO USAGE CREATED                      MODIFIED
Container1 directory no   n/a Fri 2021-04-09 18:18:39 CEST n/a
```

#### 2.4 Setting up the container network

Containers managed by *machinectl* use a private network by default, and the container's network is completely isolated (from the external network as well as from other containers). This would not be suitable for us to manage multiple Jenkins, so we change it to a host networking that allows the host to access the services within the container.

The path to the container configuration file is `/etc/systemd/nspawn/container-name.nspawn`. The name of the configuration file corresponds to the container name.

We edit the configuration file (using nano or vim).

```bash
# If you have not intalled nano
sudo pacman -Syu
sudo pacman -S nano

sudo nano /etc/systemd/nspawn/Container1.nspawn
```

Add the following two lines to the file.

```cfg
[Network]
VirtualEthernet=no
```

#### 2.5 Boot container, change root password

Start the container via `machinectl`:

```bash
sudo machinectl start Container1
```

Checking container status

```bash
machinectl list
```

Through the container's interactive shell session, this step does not invoke the login process.

```bash
sudo machinectl shell Container1
# When you in the shell
passwd
```

Delete the two files immediately after, otherwise you will not be able to log into the container properly next

```bash
rm /etc/securetty
rm /usr/share/factory/etc/securetty
exit
```

After exiting the container's shell, you can log in to the container with the password

```bash
sudo machinectl login Container1
```

> 1. If you forget your password, to terminate the session from inside the container, hold Ctrl and quickly press ] three times. Non-US keyboard users should use % instead of ].  
> 2. When you terminate the session in this way, don't forget to re-enter the geine.

### 3. Installing Jenkins

Once you are logged into the container, you can use the package manager to install Jenkins and some necessary programs, such as

* Jenkins
* OpenJDK
* Nano or Vim
* Xorg (This is necessary, if xorg is not installed, jenkins will report errors about the GUI, although in fact we don't have any GUI programs installed at all.)

```bash
# Update all packages
pacman -Syu
# Install packages
pacman -S nano bash-completion jdk-openjdk jenkins xorg sl
```

Before starting jenkins, give enough permissions on /var/cache/jenkins in the container

```bash
mkdir /var/cache/jenkins
chmod 777 /var/cache/jenkins
```

### 4. Start and configure Jenkins

Since we'll create the second Jenkins container next, we'd better modify the first Jenkins port.

The configuration file for Jenkins is in `/etc/conf.d/jenkins`

Change the port number in the line `JENKINS_PORT=--httpPort=8090` to `8091`

Now just launch Jenkins inside the container

```Bash
systemctl start jenkins
```

Then just open `http://localhost:8091` through your browser on Windows (such as Microsoft Edge) to configure your first Jenkins.

You can safely `logout` and your container will keep running, you can check the status of the container and shut it down at any time with the following command:

```bash
machinectl list
machinectl poweroff
```

## 0x04 Second container

Do you remember what I said before that a container is a folder? If we want to create similar containers, then we just need to copy the folder and generate a new symbolic link to `/var/lib/machine`. Oh yeah, and don't forget that the new container needs a new configuration file.

Don't forget to close the container before copying it.

```bash
# Turn off container
sudo machinectl poweroff Container1
# Copy container
sudo cp -r ~/MyContainers/Container1/ ~/MyContainers/Container2/
sudo ln -s /home/amao/MyContainers/Container2/ /var/lib/machines/
sudo cp /etc/systemd/nspawn/Container1.nspawn /etc/systemd/nspawn/Container2.nspawn
```

Then check it out

```bash
$ machinectl list-images
NAME       TYPE      RO USAGE CREATED                      MODIFIED
Container1 directory no   n/a Fri 2021-04-09 18:18:39 CEST n/a
Container2 directory no   n/a Fri 2021-04-09 18:18:39 CEST n/a

2 images listed.
```

Start two containers at the same time

```bash
sudo machinectl start Container1
sudo machinectl start Container2
sudo machinectl list
MACHINE    CLASS     SERVICE        OS   VERSION ADDRESSES
Container1 container systemd-nspawn arch -       -
Container2 container systemd-nspawn arch -       -

2 machines listed.
```

Then just go into different containers for different configurations according to different needs. For example, change the port number of Jenkins in Container2. After that you can access different ports to control different Jenkins through your browser on Windows.

## 0x05 Packing containers

If you want to move the container to your other host, you can of course always package and export the container with the `machinectl` command.

```bash
# Export
sudo machinectl export-tar Container1 /home/username/Container1.gz
# Import
sudo machinectl import-tar /home/username/Container1.gz ContainerX
```

You can check the progress with `machinectl list-transfers` and cancel with `machinectl cancel-transfer` at any time during the import and export process.

Thanks for your reading.
> Easter egg for you. If you really follow the commands. Try to run `sl` in Container1.
