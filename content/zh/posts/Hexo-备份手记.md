---
title: Hexo 备份手记
date: 2019-02-15 10:11:34
tags:
- Blog
- Hexo
- Git
---

由于hexo只是将生成的静态页面推到github的io仓库里，这就导致了，如果你有一天想要异地更新博客，或者换机后也能更新博客，就必须要将本地的框架做数据备份，否则一旦丢失，是非常头疼的问题，99%是不能恢复成以前的博客了。

做数据备份，往往都是物理上的或者云上的。物理上的便是复制整个文件夹到移动硬盘或者U盘随身带着。这些东西我并不少，但是总是个累赘，还有物理上的可能性损伤造成的数据丢失。或者就放到某个云盘上，还可以自动同步，只可惜作为ArchLinux为主系统的用户，本人常用的onedrive并不在该系统上提供官方服务，第三方的bin也无法做的实时检测文件更新并同步。

那么既然自动化无望，无论如何都要定期手动备份，那我还是将整体作为项目放到github的仓库里吧。下边便是我做成之后写下的记录，也算是分享（虽然网上也是一大把这种教程）。

<!--more-->

首先正如很多同志们说的那样，新建一个repo来备份实在是有所浪费，还是在原有io的repo建立一个分支最佳。

### **需要注意的是**
> * 我的教程是写在一种 "已经开始写博客了,中途需要备份" 的情况下,因此有些步骤或许会和“从0开始建立博客并备份”的情况有所不同。但是稍微有所经验的同志们应该会自我调整步骤顺序，都是大同小异的东西。
> * 本文命令环境皆在linux->zsh 下。其中多数命令多平台通用，少数类似cd命令，请根据自己情况进行更改。

### 步骤如下

1. 将你的XXXX.github.io仓库先clone下来到一个位置(比如Blog文件夹)。
```zsh
mkdir ./Blog
cd ./Blog
git clone xxxxxxxxxxxxxxxxxx
```
2. 创建并切换进新分支，名字随意，下文以hexo为例
```zsh
git checkout -b hexo
```
3. （这一步可以进入文件管理器进行）现在删除除.git文件夹外，所有clone下来的文件。**注意千万不要删除了.git文件夹！否则请回到第一步。**

4. 将之前的Hexo文件夹中的_config.yml，themes/，source，scffolds/，package.json，.gitignore复制到你克隆下来的仓库文件夹。

5. 删除你所用的themes里的.git文件夹，比如themes/next/.git，请细细检查每一个主题的目录，里面的.git统统不要！

6. 执行下面命令
```zsh
npm install hexo-cli -g #如果之前没有执行过这个全局命令，比如你是在webstorm里面管理你的博客的，请执行这一步。
npm install
npm install hexo-deployer-git
```
7. 测试新目录博客是否可用
```zsh
hexo g
hexo s
hexo clean
```
8. push仓库到github上
```zsh
git add -A
git commit -m "Backup"
git push origin hexo
```
9. 打开github检查是否hexo分支推成功了，并且将hexo分支设置为默认分支（在repo的setting里修改）。

10. 结束，以后就在这个目录下写博客就好了。

{% img https://i0.hdslb.com/bfs/album/db44bbad92b92f66f378bb2b8de97ff6b99ccef0.jpg Aqua&Mea%}