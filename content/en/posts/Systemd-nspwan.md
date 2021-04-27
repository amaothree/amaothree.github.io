---
title: "Creat multi Jenkins containers in Windows Subsystem for Linux based on systemd-nspawn"
date: 2021-04-16T11:31:05+01:00
type: post
tags: 
- Linux
- DevOps
- WSL
---

## 0x01 Introduction

This tutorial will describe how to create multiple isolated containers in WSL2 and run multiple Jenkins in them using Systemd-nspwan container technology.

This tutorial is quite generic. Although the name of this article relates to WSL2, most of the operations which follow will work perfectly well on any Linux distribution that uses Systemd ( and will not require any of the operations in the next section, "Preparation" ). Only minor modifications are required depending on the specific distribution (e.g. the difference in package managers and container systems).

<!--more-->

## 0x02 Preparation

### 1. Install WSL2

Compared to the first generation of WSL, the virtualization technology used in WSL2 makes it possible to use the Systemd toolchain, and the interactive integration of WSL2 with Windows 10 is beyond the usual virtual machines. Therefore, this article recommends trying WSL2 on Windows 10 systems as a first priority to meet the needs of Dev or Ops for Linux environments.

In this tutorial you can choose any WSL2 distribution, such as the officially supported Linux distributions: Ubuntu, Debian, openSUSE or Alpine, or other open source projects supported by developers such as ArchWSL.

As a practical user and lover of Arch Linux, **I will choose the [ArchWSL project](https://github.com/yuk7/ArchWSL) as my next WSL2 distribution**.

And for details on how to install WSL2, please read [Microsoft's official documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

### 2. Install Genie

Due to the design of WSL2, Systemd is not started with PID 1 by default in WSL2. This will cause that we basically can't use Systemd's functions properly. However, there is a small tool that can help us fix it very easily.

This is Genie, and you can easily find installation packages for various package managers in [Genie's project repository](https://github.com/arkane-systems/genie#installation).

Once installed in WSL2 you can start a "Genie mode" WSL shell directly in Powershell

`wsl genie -c bash`

Or switch in the WSL

`genie -c bash`

### 3. Install Windows Terminal (optional)

It can be installed in the Microsoft Store, and works right out of the box. Highly recommended.

## 0x03 Create our first Systemd-nspwan container

### 0. **Enter the WSL shell in Genie mode**

* In Powershell

>```Powershell  
>wsl genie -c bash
>```

* Or，In WSL

>```Bash  
>genie -c bash
>```

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

First install the Arch Linux community maintained installation script in WSL

```Bash
sudo pacman -S arch-install-scripts
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

## 0x04 Packing containers

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
