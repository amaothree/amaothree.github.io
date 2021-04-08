---
title: 回到Windows，最全教程让你的Powershell更好看且好用
date: 2020-06-05 15:18:02
type: post
tags:
- Geek
- Windows
- Powershell
---

![](/images/回到Windows，让你的powershell更好看且好用/screenshot.png)

## 0x00 前言
从ArchLinux退烧（现在是低烧状态，还是非常非常喜欢Arch的）回坑Windows 10后，因为有了两年多的Gnome以及Kde的使用经验后，回来其实发现Windows的UI设计并没有那么的不堪，甚至是有很多可取之处的。可能只是之前用了太久反而会唾弃之吧。再加上这几年微软开始拥抱开源并且回馈开源，有很多好的项目都提出来了并且已经release了，因此我对微软的态度也来了个大转变，现在日常生活生态和撸代码环境也都往微软生态上转移了（365真香！）。我先用了半年的win10，从1903到现在的2004，对微软的设计稍微有了些自己的见解后，便将一些能够极大提升使用体验的小方法写了出来。正好最近入手了一个二手Surface Pro买来学习用和轻办公，正好借着这个机会从头到尾的写一篇美化Powershell的教程，也算是备忘用。最终效果应当如头图（头图是我的主力机上之前截下来的旧图）。

<!--more-->

本文基于1903后的Windows 10系统进行教程，1903之前的用户本人不确定是否完全有效。（另外提醒，用系统就要保持系统持续最新，更新系统永远是利大于弊，一次系统更新，远比什么傻逼360的建议要有效且强上数百倍。为了安全，推荐每一位用户开启Windows更新。）

*本文作者不对您的不当操作导致的电脑数据丢失，机箱爆炸，系统蓝屏死机等所有的遗憾以及其带来的损失负责*

## 0x01 官方且好用的终端：Windows Terminal的安装以及基础配色优化。
从Linux转移回来最不适应的就是每次敲指令时面对这超大的刺眼深蓝色配色的Powershell窗口，每每开启不只是眼睛，我头都要疼一下。垃圾的字体渲染，垃圾的配色，垃圾的字体，虽然设置里能改一部分，但那个是在是鸡肋，而且这东西也没有Tab，全是一个一个windows实在是不太方便。

面对这个痛点，尤其是微软推出WSL之后，发现开发者们确实应当有一个更好的终端体验，于是便开源做出了Windows Terminal。截止笔者写这篇文章的时候正好这玩意也进入1.0版本了，使用体验提升不少。我们先来安装它。这很简单。

1. 打开系统自带的商店“Microsoft Store”，搜索Windows Terminal，安装之就可。

    ![](/images/回到Windows，让你的powershell更好看且好用/1.1.png)

    （题外话，本来微软是想通过微软商店作为软件的官方分发以及包管理器的，然而前期实在是做的太烂了，开发者都不愿意来，恶性循环下微软自己也半放弃这个商店了，但是已经上架的产品都在持续更新，而且伴随着微软很多自己开源产品逐渐上架，这个东西还算苟活着。主要是UWP应用必须通过这下商店下载，在UWP框架没有大的功能性升级之前，应该还不会火。不过.net 5出了，而且确实很强大，我还是很看好未来的。）

2. 点击上方Tab栏右侧下箭头，点击设置，这里会默认打开Windows Terminal的默认配置json文件。
   ![](/images/回到Windows，让你的powershell更好看且好用/1.2.png)

3. 关于这个配置文件的详细描述以及解释请看[官方中文文档](https://aka.ms/terminal-documentation)，在这一步我就简单介绍一下怎么配置字体，配色。如果你对配置文件敏感，你一眼就能看出，Windows Terminal是可以对不同的shell单独进行配置的，你可以给powershell，bash，cmd分别设置不同的配色字体等等。在`profiles`这一list下有不同的profile块，往往默认第一块应当针对Powershell的。这里放出我头图中的针对powershell的配置：
   ```json
    {
            // Make changes here to the powershell.exe profile
            "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
            "name": "Windows PowerShell",
            "commandline": "powershell.exe",
            "fontFace": "Source Code Pro",
            "colorScheme" : "One Half Dark",
            "fontSize": 10,
            "hidden": false
        },
   ```
   以我头图那样为例。我使用的是[官方提供的9大配色方案](https://docs.microsoft.com/zh-cn/windows/terminal/customize-settings/color-schemes)之一的“One Half Dark”，在linux时，我是Solarized配色的忠实粉丝，但是回到windows时，我发现Solarized的灰色字体有点儿看不清了，于是换成更好一点儿的One Half配色了。字体和字号，按照我的样例，分别修改fontFace和fontSize的值就好了。这个因人而异。

4. 由于Window Terminal是动态的读取配置文件，所以你的每一次修改是及时生效的，无需重启应用，这样非常方便你去修改字号这样的配置。你可以一边vscode一遍terminal的去调整。

## 0x02 安装包管理器Scoop!

用过Linux甚至是mac后的开发者，再回来用windows最不爽的一点可能就是开发环境的配置问题了。在linux各个发行版都有自己的包管理器，提供最稳定或者最新的软件包，开发者大多数情况下只需要一行命令install一下就行，然后就可以在ide或者工具里使用刚刚安装好的库了。然后这一切在windows下边的有点儿尴尬，在winget项目还没有立项之前，windows官方是希望每一家公司自己负责自己程序的安装和更新的，因此每个人都需要自己去每一家的官网上下载msi或者exe文件自己安装，而且一般商业性质的软件还会常驻一个updater在后台，久而久之，安装的东西多了，系统免不齐会运行的越来越慢。

因此，Windows的开发者们就产生了自己做包管理器的想法，于是社区两大著名的windows包管理器：Chocolatey和Scoop就诞生了，相比之下，Chocolatey臃肿庞大，自带GUI界面，相当于是一个进化版的windows商店……

[Scoop](https://scoop.sh/)是后起之秀，特点是轻量绿色化，scoop是按照aptget或者zypper这样的linux包管理理念设计的一个包管理器。本身不存储安装程序，而是将安装逻辑和安城程序的获取url写进配置文件再上传至github的repo上（有点儿像AUR的那种感觉）。这样社区就可以给scoop方便的进行贡献，只需要提交PR就好了。而且scoop多半是绿色安装，安装在每一个用户的用户文件夹里，大多数收录的程序安装时不会触发UAC获取管理员权限。软件的安装位置固定，卸载彻底，软件环境自动写进环境变量，不会对系统造成破坏。（总之就是好处很多）

因为后续的优化插件我们都将通过scoop获取，所以现在我们来安装它，

1. 右键windows terminal，用管理员身份启动程序。
2. 输入下面两行
   ```powershell
    Set-ExecutionPolicy RemoteSigned -scope CurrentUser
    iwr -useb get.scoop.sh | iex
   ```
   等安装成功后我们安装git（因为正如上文所说，scoop将自己的软件仓库存在了github上，所以我们需要git来更新仓库）
   ```
    scoop install git
   ```
   等安装完成后就可以进行第一次仓库更新了
   ```
    scoop update
   ```
   如果更新很慢，说明你家网络环境访问github不太理想，请尝试科学上网。
3. 默认下，scoop是只开启了main桶，scoop里把软件仓库称之为bucket（桶），你可以通过`scoop bucket known`查看官方收录的“桶”们，并且通过`scoop bucket add xxx`来启用添加这个桶，这里我个人推荐至少添加上extras桶，如果你有java开发的需求，更推荐你添加java桶。举例：
   ```
    scoop bucket known #查看官方支持的桶
    scoop bucket add extras #添加桶
    scoop bucket list #查看本地已经添加的桶
   ```
    然后你就可以像在linux上一样方便地安装jdk或者jre
   ```
    scoop search openjdk
    scoop install openjdk
   ```
4. 同时也有很多人在github上维护了自己的第三方桶，你也可以add他们。正好我们需要安装[scoop-completion](https://github.com/Moeologist/scoop-completion)这个帮助我们在powershell下自动补全scoop参数的小工具，他收录在开发者自己的桶中，我们来安装它：
   ```
    # 通过GitHub链接添加桶
    scoop bucket add scoop-completion https://github.com/Moeologist/scoop-completion
    scoop update
    # 安装
    scoop install scoop-completion
   ```
   为了让自动补全工具能每次打开powershell都有效，我们需要将其写进powershell的配置文件中（有点儿像bash的.bashrc文件。不是Windows Terminal的配置文件），
   ```
    # 首先检测有没有配置文件并生成profile文件
    if (!(Test-Path $profile)) { New-Item -Path $profile -ItemType "file" -Force }
    # 查看配置文件路径（在哪儿）
    $profile
   ```
   然后通过路径**打开这个文件**，然后通过你喜欢的编辑器打开这个文件，**加入**下面这一行。
   ```
    Import-Module "$($(Get-Item $(Get-Command scoop).Path).Directory.Parent.FullName)\modules\scoop-completion"
   ```
   保存重进一下terminal就成。

## 0x03 安装Powershell下的zsh, Posh!

很多在Linux或者mac下使用过zsh的开发者都对其赞不绝口甚至再也无法没有他了，那么powershell作为一款设计上远超shell的命令行，有没有类似的产品呢？

有！那就是[Posh](https://github.com/dahlbyk/posh-git)!

![](/images/回到Windows，让你的powershell更好看且好用/3.1.png)

但是实际上Posh打不过Zsh。Posh的设计初衷只是为了设计出能像zsh（oh-my-zsh）那样的能够反应git信息的一个工具。而且你看看上图，真的是有点儿丑哈。不过你zsh没了oh-my-zsh不也是丑嘛，所以说，便有另一位好心人设计出了posh的美化方案，[oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)!

![](/images/回到Windows，让你的powershell更好看且好用/3.2.png)

那现在我们就通过scoop安装posh和oh-my-posh，并且进行配置。

1. 通过scoop安装posh以及oh-my-posh.
   ```
   # 因为posh是oh-my-posh的依赖，因此安装oh-my-posh会自动安装posh
   scoop install oh-my-posh
   ```

2. 打开[官网](https://github.com/JanDeDobbeleer/oh-my-posh#themes)，选择一款自己喜欢的主题，下面以Agnoster为例子。
3. 打开powershell配置文件(通过命令行输入`$profile`查找配置文件所在目录)，**加入**下面几行：
   ```powershell
    Import-Module posh-git
    Import-Module oh-my-posh
    Set-Theme Agnoster #这里对应你选好的主题名
   ```
4. 重启terminal，安装完成。

## 0x04 细节继续优化

完成上面三大步，我们就有了好用的terminal，美观的shell和高效的包管理器。接下来我们再继续往细节上优化。

### 类似zsh的历史命令补全

如果你有用过zsh或者posh，你想回到某一次pacman的安装命令，你就可以只敲`sudo pacman -S`后通过上下方向键浏览以已经输入的字符串为前缀的历史命令，这样就可以方便快速的寻找我们之前执行过的命令。

![](/images/回到Windows，让你的powershell更好看且好用/4.1.png)

这个功能随着powershell的更新已经在5.0版本后内置了（通过`$PSVersionTable`查看当前powershell版本），但是没有默认开启，我们只需要往配置文件中**加入**如下信息开启它就好了。

```powershell
Import-Module PSReadLine

Set-PSReadlineKeyHandler -Key Tab -Function Complete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
```

自带的psreadline有可能是老版本，如果你想更新，请执行：
```powershell
sudo Install-Module -Name PowerShellGet -Force
exit 
#重新打开terminal
sudo Install-Module PSReadLine
```

### sudo命令

sudo大家都熟，linux下的临时提权命令。由于windows的UAC设计问题，在Windows下很多命令都需要管理员权限运行。这也是为什么很多教程以前都是推荐大家右键打开powershell或者cmd的管理员窗口执行命令。

然而windows terminal作为一个terminal是没办法直接在terminal里单独开一个管理员权限的tab的。如果我们只是临时需要一下admin权限还得右键admin启动terminal。这就很烦了。

于是推荐大家安装powershell里的sudo。

```
scoop install sudo
```

安装完开箱即用，无需额外配置。以后terminal里不够权限的操作，sudo一下。

### ls命令根据文件类型彩色显示菜单

powershell下ls默认是一样的颜色，也是不好看直接，最主要的是，如果你这个文件夹下如果有很多问价，不好分辨不好找。

开启彩色显示后，就能更直观的查看当下文件夹里的文件了

![](/images/回到Windows，让你的powershell更好看且好用/4.3.png)

```powershell
sudo Install-Module -AllowClobber Get-ChildItemColor
```

然后往powershell的配置文件里**添加**：
```powershell
Import-Module Get-ChildItemColor
```

重新启动terminal就可以了。

### 安装screenfetch（头图）用来装逼

```
scoop install screenfetch
```

