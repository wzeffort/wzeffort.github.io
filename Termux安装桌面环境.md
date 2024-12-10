+++
date = '2024-12-10T00:22:09+08:00'
draft = false
title = 'Termux 安装桌面环境'
+++

# Termux 安装桌面环境

> **前言**：安卓平板使用Termux安装Linux桌面折腾记录



> 目前的平板电脑屏幕尺寸都变得很大，并且**8+256**的配置也很常见。
> 
> 如果你也和我一样拥有一台屏幕为😁**13英寸**😁的平板电脑，且有**键盘**和**蓝牙鼠标**，那么在上面利用安装`Termux`安装一个带有桌面的`Linux`环境实在是太方便了🧨！！！

## 知识点速览

**1. 图型化实现方式**

- [ ] VNC

- [x] X11（termux-x11）

> 目前`Termux-X11`这个项目已经开发的很完善了，启动方式比起VNC更简单丝滑

**2. Linux桌面**

- [x] XFCE

- [x] LXQt

> `XFCE`和`LXQt`都很不错，个人感觉来说`LXQt`更轻量更流畅一些，`XFCE`则是主题更多更漂亮

**3. 桌面环境**

- [ ] Termux

- [x] Proot-distro

> 本文撰写时`Termux`还没能支持`locale`以及`Fcitx5`包，所以没办法进行系统中文化（LXQt可以给桌面组件单独设置，XFCE不行），更严重的是没有`Fcitx5`或者其他能够支持中文输入法的包对中文用户太致命了，没有实用性。所以本文本着实用原则选择`Proot-distro`容器这种方式。

**备注**：如果仅仅是用来跑一些exe格式的应用，例如在平板电脑上玩儿Windows下的小游戏那么选取`Termux`也未尝不可，如果你采用这种方式，这里强烈推荐[**termux-desktop** @by sabamdarif](https://github.com/sabamdarif/termux-desktop)这个项目。

**备注**：此外撰写这篇笔记的时候[GitHub - termux/termux-packages](https://github.com/termux/termux-packages)仓库中正有人尝试添加`Fcitx5`支持，一旦这项工作完成那么Termux原生桌面也可以列入待选。

**4. Linux发行版**

- [ ] `alpine`: Alpine Linux (edge)

- [ ] `archlinux`: Arch Linux ARM

- [ ] `artix`: Artix Linux (AArch64 only)

- [ ] `chimera`: Chimera Linux (20240707)

- [x] `debian`: Debian (stable)

- [ ] `debian-oldstable`: Debian (old stable)

- [ ] `deepin`: Deepin (beige)

- [ ] `fedora`: Fedora 40 (64bit only)

- [ ] `manjaro`: Manjaro (AArch64 only)

- [ ] `openkylin`: OpenKylin (Yangtze)

- [ ] `opensuse`: OpenSUSE (Tumbleweed)

- [ ] `pardus`: Pardus (yirmibir)

- [ ] `ubuntu`: Ubuntu (24.04)

- [ ] `ubuntu-oldlts`: Ubuntu (22.04)

- [ ] `void`: Void Linux

> `Deepin`系统的dde桌面很适合中国用户，但是由于`Proot-distro` 是基于`SysVinit` 而dde桌面确必须依赖于`Systemd`所以没必要选择deepin，有关这个发行版的教程文章太少了。



> 上述发行版中教程最多的应该是`Ubuntu`，但是`Ubuntu 22.04`之后开始强制使用`snap`安装应用，而`snap`，`Appimage`以及`Flatpak`等工具在PRoot环境下不能使用。由于`Ubuntu`和`Debian`二者的配置大致一致，且其他有关`Ubuntu`的参考资料也可以轻松的类比到`Debian`，所以本教程最终采用`Debian`替代`Ubuntu`。



> 至于其他发行版在`Termux`中尝试使用的人较少，避免踩坑不考虑。

## Termux 配置

启用`X11`仓库和`root`仓库

```shell

pkg install x11-repo root-repo -y
```

实际使用中我们常需要访问安卓目录下的`Downloads`，`Video`等文件夹，所以需要给Termux存储权限，方便我们在`Android`和`Linux`之间共享文件。

```bash
# 获取存储权限
termux-setup-storage## proot-distro安装Linux发行版
```

需要安装相关包来支持基础功能

- **termux-x11-nightly**：`Texmux-X11`软件的通信配合包

- **pulseaudio**：提供在`Linux`中播放声音支持

- **virglrenderer-android**：在 `Termux` 上使用硬件`GPU `加速的 `PRoot` 容器

- **termux-am**：安装后可以使用am命令打开设备上的其他软件，这里用来给脚本提供自动跳转到X11界面的功能

- **tmux**：一个强大的终端复用器，这里用来在`Termux终端`和`Proot-distro终端`之间切换

- **proot-distro**：无需Root提供一个相较`Termux环境`更加完整的`Linux环境`，并且只需要消耗极少的性能

```bash
# 安装桌面相关软件
pkg install termux-x11-nightly pulseaudio virglrenderer-android termux-am tmux proot-distro -y 
```

## Proot-distro 容器配置

### 安装Linux发行版

使用以下命令安装Debian

```bash
proot-distro install debian
```

接下来需要进入debian容器进行一些配置(此小结之后内容都在容器内进行)

```bash
proot-distro login debian
```

### apt换源

后续安装桌面环境的包比较大，先给apt进行换源



采用 [中国科学技术大学](https://www.ustc.edu.cn/)的镜像，这里给出Debian和Ubuntu Ports（如果你使用Ubuntu）镜像使用帮助网址，如果下方信息失效，或者需要查看详细信息可以使用



[Debian - USTC Mirror Help](https://mirrors.ustc.edu.cn/help/debian.html)

[Ubuntu Ports - USTC Mirror Help](https://mirrors.ustc.edu.cn/help/ubuntu-ports.html)

```bash
# 国内源采用https，安装必要的包支持https
sudo apt install apt-transport-https ca-certificates

# 编辑sources.list文件
nano /etc/apt/sources.list

```

使用"`#`"注释掉sources.list之前存在的内容，重新写入新内容，这里使用镜像

```bash
# /etc/apt/sources.list
# 默认注释了源码仓库，如有需要可自行取消注释
deb http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware

# backports 软件源，请按需启用
# deb http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-fian.html
```

apt更新包

```bash
apt update; apt-get update
```



### 中文环境配置

首先配置系统中文，安装语言环境以及文泉驿字体（中文字体）

```
sudo apt install locales fonts-wqy-zenhei -y
```

 生成地区设置

```
sudo dpkg-reconfigure locales
# 选择97. en_US.UTF-8 UTF-8
# 回车后选择3. en_US.UTF-8
# 相当于nano /etc/locale.gen取消注释en_US.UTF-8 UTF-8后,locale-gen
```

### 配置输入法

安装Fcitx5以及中文输入法

```bash
apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk4 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 fcitx5-frontend-qt5
```

配置使用Fcitx5

打开配置文件`/etc/profile`(也可以写入`~/.bashrc`或者`~/.profile`,区别见[附录-Fcitx5配置](#Fcitx5配置)

```bash
nano /etc/profile
```

在文件末尾添加以下内容

```
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx 
export QT_IM_MODULE=fcitx
```

### 配置普通用户并给予root权限

默认情况下会使用root账号登录，但是一些软件会出现提示或者问题，所以我们创建一个普通账户并赋予root权限作为日常使用的账户



新增用户

```bash
adduser <user-name>
```

赋予Root权限

```bash
nano /etc/sudoers
```

在 `root ALL=(ALL:ALL) ALL` 之后添加 `<user-name>  ALL=(ALL:ALL) ALL` , 就像下面这样，请注意这里添加了`NOPASSWD`这个选项以避免频繁使用sudo命令

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
<user-name>   ALL=(ALL:ALL) NOPASSWD:ALL
```

### 安装桌面环境

#### XFCE

```bash
apt install xfce
```

#### LXQt

```bash
apt install lxqt
```

**注意**：如果弹出要选择地区的话请选择**5.Asia** --> **69.ShangHai**，键盘布局可以选择**English**或者**Chinese**没有区别，语言选项可以选择**zh_CN.UTF-8**

安装需要较长一段时间（我的设备耗时30min左右）在解压`plasma`资源的时候格外费时，感觉就像进度条卡了一样。

### 添加常用软件包提升舒适度

我推荐的应用有：

- fish：更舒适的终端（LXQt的默认终端都不支持全彩色以及 ↑ ↓切换执行历史！！！）

- zsh：具有很多特性的shell

- geany：编辑器

- code-oss：Linux版的VsCode

- synaptic：可视化的包管理工具（自动创建的快捷方式可能有误，出现打不开情况需要按照下文进行更改）

- firefox-esr：使用firefox浏览网页

- wps-office：PC端的WPS

**注意**：建议安装`synaptic`后登录桌面使用`synaptic`安装firefox、wps这些应用

这里给出最小安装，其他的自选

```bash
apt install fish
```

### 配置X11显示尺寸

默认情况下显示的窗口比例较小，可以在**termux-x11 App**里进行更改

1. 打开应用

2. 选择Preferences --> Output --> Display resolution更改为custom

3. 在新出现的选项Display resolution输入你希望的显示尺寸，例如我的是1600x1000

此外，如果你希望操作时可以全屏，勾选下方的Fullscreen选项



### 进入桌面环境

**注意**：如果你按照教程一步步进行操作，那么在继续之前你需要先退出容器回到termux环境后继续，使用`exit`命令退出，你可以输入`uname -r`来检查是否成功，即输出内容中没有Proot-distro即可



累，终于可以松口气了！接下来你就可以尝试启动桌面环境了。



不着急使用自动化脚本，我们先手动输入命令进入



在termux执行以下命令启动X11服务

```bash
XDG_RUNTIME_DIR=\${TMPDIR} termux-x11 :1.0 &
```

登录Debian环境，并启用`LXQt桌面`

> 这里我遇到的问题是如果直接在`proot-distro login`的后面使用`-- bash -c`参数在登录后自动执行命令的话Fcitx5需要的环境变量无法成功配置，个人猜测是没有加载配置文件之前就启动了Fcitx5导致缺少环境变量，所以在启动桌面前添加了命令来重载配置文件

```bash
proot-distro login debian --shared-tmp  -- bash -c 'source /etc/profile; source ~/.barhrc; export PULSE_SERVER=tcp:127.0.0.1; env DISPLAY=:1.0 dbus-launch --exit-with-session startlxqt'
```

接下来切换到**termux x11 App界面** 应该就可以看到对应的桌面环境了

**注意**：如果需要选择窗口管理器的话请选择**xfwm4**，这个选项的配置文件在`~/.config/lxqt/session.conf`你可以手动更改`window_manager=xfwm4`



你可以测试一下你的桌面是否能够正常工作，如果他是英文的你可以在LXQt的配置中心的区域设置中选择简体中文（注意此更改重新登录后才能生效），最重要的是你应该测试你的输入法是否如期工作，你可以使用`Ctrl + 空格`来切换中英文输入法。



如果一切正常，那么回到`Termux`输入`Ctrl + C`终止桌面进程，之后使用`exit`命令退出容器



## 自动化脚本

启动桌面的指令实在是太长了，如果每次输入显然是不合适，接下来创建脚本来简化工作

我们将会创建两个脚本

- **btx11**：begin Termux X11 启动Termux X11桌面

- **stx11**：stop Termux X11 停止Termux X11桌面

### btx11

创建文件

```bash
nano ../usr/bin/btx11
```

写入以下内容，注意你需要更改USER_HOME="/home/<your user name>"的内容为你之前创建的普通用户名，`tmux new-session -d -s desktop-server "proot-distro login --user`处也是一样，如果你需要以Root启动，那么配置是用户名为root, USER_HOME="/root"

```bash
#!/data/data/com.termux/files/usr/bin/bash

USER_HOME="/home/<your user name>"

# check -o
open_existing=false
while getopts "o" opt; do
  case $opt in
    o)
     open_existing=true
     ;;
    *)
     ;;
  esac
done

# env DISPLAY=:1.0 dbus-launch --exit-with-session startlxqt > /dev/null 2>&1 &
# nohup proot-distro login debian --shared-tmp --bind $HOME/storage/shared/Music:/root/Music --bind $HOME/storage/shared/Download:/root/Download --bind $HOME/storage/shared/Pictures:/root/Pictures --bind $HOME/storage/shared/Videos:/root/Videos --bind $HOME/storage/shared:/root/storage -- bash -c "env DISPLAY=:1.0 dbus-launch --exit-with-session startlxqt" > /dev/null 2>&1 &

if [ "$open_existing" = true ]; then
  # check "desktop-server" tmux session
  if tmux has-session -t desktopserver 2>/dev/null; then
    tmux attach-session -t desktop-server
    exit 0
  else
    echo "no desktop-server opend, open new session now."
    processes=$(pgrep -f com.termux.x11)
    for pid in $processes; do
        echo "killing x11 server: $pid"
        kill $pid
    done
    echo "open new x11 server"
    XDG_RUNTIME_DIR=\${TMPDIR} termux-x11 :1.0 &
    sleep 1
    echo "jump to x11 app"
    am start --user 0 -n com.termux.x11/com.termux.x11.MainActivity > /dev/null 2>&1
    sleep 1
  fi
else
  echo "close old desktop-server tmux session"
  tmux kill-session -t desktop-server

  processes=$(pgrep -f com.termux.x11)
  for pid in $processes; do
      echo "killing x11 server: $pid"
      kill $pid
  done
  echo "open new x11 server"
  XDG_RUNTIME_DIR=\${TMPDIR} termux-x11 :1.0 &
  sleep 1
  echo "jump to x11 app"
  am start --user 0 -n com.termux.x11/com.termux.x11.MainActivity > /dev/null 2>&1
  sleep 1
fi

# close pulseaudio
processes=$(pgrep -f pulseaudio)
for pid in $processes; do
    echo "killing pulseaudio: $pid"
    kill $pid
done

# close virgl renderer
processes=$(grep -f virglrenderer-android)
for pid in $process; do
    echo "killing virglrenderer-android $pid"
    kill $pid
done

echo "Starting pulseaudio server"
pulseaudio --start --load="moudle-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" --exit-idle-time=-1

echo "Starting Virgl Renderer"
virgl_test_server_android &

echo "open proot-distro desktop"
tmux new-session -d -s desktop-server "proot-distro login --user <your user name> debian --shared-tmp --bind /data/data/com.termux/files/usr:/termux --bind $HOME/storage/shared/Music:$USER_HOME/Music --bind $HOME/storage/shared/Download:$USER_HOME/Downloads --bind $HOME/storage/shared/Pictures:$USER_HOME/Pictures --bind $HOME/storage/shared/Videos:$USER_HOME/Videos --bind $HOME/storage/shared:$USER_HOME/storage -- bash -c 'source /etc/profile; source ~/.barhrc; export PULSE_SERVER=tcp:127.0.0.1; env DISPLAY=:1.0 dbus-launch --exit-with-session startlxqt'"
# --bind /data/data/com.termux/files/usr:/termux 

if [ "$open_existing" = true ]; then
  tmux attach-session -t desktop-server
fi
```

给于可执行权限

```bash
chmod +x ../usr/bin/btx11
```

### stx11

创建文件

```bash
nano ../usr/bin/stx11
```

写入以下内容

```bash
#!/data/data/com.termux/files/usr/bin/bash

processes=$(pgrep -f com.termux.x11)
for pid in $processes; do
    echo "killing x11 server: $pid"
    kill $pid
done
if tmux has-session -t desktop-server 2>/dev/null; then
  echo "killing existing "desktop-server" tmux session"
  tmux kill-session -t desktop-server
fi
```

给于可执行权限

```bash
chmod +x ../usr/bin/stx11
```

进行测试

启动桌面

```bash
btx11
```

关闭桌面

```bash
stx11
```

提示：

1. **btx11**命令启动桌面后默认情况下会在后台的`tmux`终端中运行，你可以输入`btx11 -o`来保持容器终端在前台运行，使用`Ctrl + b`再按下`d`可以回到`Termux`终端（详见tmux教程）。

2. 在桌面已经启动的情况下，`btx11 -o`会直接进入开启的容器终端，`btx11`会关闭所有已有的桌面服务并重新启动。

3. **btx11**命令启动桌面会自动将安卓目录下的Download，Video，Music，Pictures映射到用户目录下，此外storage映射的是安卓文件系统的根目录，/termux映射的是Termux下`$PERFIX/usr`目录，在其中可以看到`Termux`的文件，安装在`Termux`中的**应用**可以在`/termux/bin`目录下找到，这些应用在容器内也可以启动，但是各别方面会出现问题，例如这些应用中都无法使用`Fcitx5`输入法。



## 备份和还原

Linux与Window相比用户的权限就大了很多，所以日常使用可能会出现某些意外导致系统无法正确运行。为了避免意外我们可以将某些状态备份，以待将来恢复。



proot-distro可以将整个容器的内容打包为一个gz压缩包，之后可以从这个备份文件中恢复状态。

备份的命令为

```bash
proot-distro backup debian --output ~/storage/download/debian_back.tar.gz
```

**注意**：`~/storage/download`是映射到Android下的Download目录，你可以打开文件管理器在对应位置找到这个文件，之后妥善保存它。



恢复的命令为

```bash
proot-distro restore ~/storage/download/debian_back.tar.gz
```



## 附录

参考了许多文章，由于有些博客的服务器在国外可能存在打不开情况，以及为了避免链接丢失，我将重要的几篇摘录到了附录中，各附录均标明了原文章链接。

# Systemd 和 SysVInit对比

> 摘录自[Systemd基础篇：systemd vs SysVinit_systemd sysvinit-CSDN博客](https://blog.csdn.net/z69183787/article/details/113868288)

Systemd已经基本取代了SysV的Init，这篇文章从几个方面整理一下Systemd与Init的使用上的区别。

## 命令比较: SysVinit vs Systemd

| 命令用途         | SysVInit命令            | Systemd命令                                    |
| ------------ | --------------------- | -------------------------------------------- |
| 服务启动         | service 服务名 start     | systemctl start 服务名.service (.service可省略，后同） |
| 服务停止         | service 服务名 stop      | systemctl stop 服务名                           |
| 服务重启         | service 服务名 restart   | systemctl restart 服务名                        |
| 服务重新加载       | service 服务名 reload    | systemctl reload 服务名                         |
| 服务状态确认       | service 服务名 status    | systemctl status 服务名                         |
| 服务开机启动设定     | chkconfig 服务名 on      | systemctl enable 服务名                         |
| 取消服务开机启动设定   | chkconfig service off | systemctl disable 服务名                        |
| 确认服务开机启动设定状态 | chkconfig 服务名         | systemctl is-enabled 服务名                     |
| 加载服务配置文件     | chkconfig 服务名 -add    | systemctl deamon-reload                      |
| 关机           | halt                  | systemctl halt                               |
| 关机（电源）       | poweroff              | systemctl poweroff                           |
| 重启           | reboot                | systemctl reboot                             |
| 休眠           | pm-hibernate          | systemctl hibernate                          |
| 挂起           | pm-suspend            | systemctl suspend                            |

## RunLevel: SysVinit vs Systemd

| RunLevel                                | SysVInit | Systemd                             |
| --------------------------------------- | -------- | ----------------------------------- |
| System halt                             | 0        | runlevel0.target, poweroff.target   |
| Single user mode                        | 1        | runlevel1.target, rescure.target    |
| Multi user                              | 2        | runlevel02target, multi-user.target |
| Multi user with network                 | 3        | runlevel3.target, multi-user.target |
| Experimental                            | 4        | runlevel4.target, multi-user.target |
| Multi user with network, graphical mode | 5        | runlevel5.target, graphical.target  |
| Reboot                                  | 6        | runlevel6.target, reboot.target     |

## 日志确认方式

### SysVinit方式

- 确认系统日志文件

> 文件名： /var/log/message  
> 文件名：/var/log/syslog

### Systemd

- 确认系统日志信息

> 使用命令： journalctl -f

- 确认某一时间点之后的日志信息

> 使用命令：journalctl -since=xxx

## Systemd特殊命令

### 确认启动时间

> 使用命令：systemd-analyze 或者 systemd-analyze time

```bash
[root@host ~]# systemd-analyze
Startup finished in 1.638s (kernel) + 1.951s (initrd) + 14.177s (userspace) = 17.767s
[root@host ~]# 
[root@host ~]# systemd-analyze time
Startup finished in 1.638s (kernel) + 1.951s (initrd) + 14.177s (userspace) = 17.767s
[root@host ~]#
```

### 停止服务相关进程

> 使用命令：systemctl kill 服务名

### hostname设定

> 使用命令：hostnamectl

```bash
[root@host shell]# hostnamectl
   Static hostname: host.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: d27a659b15fc379d24204584d5c051bd
           Boot ID: 3fe84ed9506248b9a41c492ca543748a
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 4.10.4-1.el7.elrepo.x86_64
      Architecture: x86-64
[root@host shell]# 
[root@host shell]# 
[root@host shell]# hostnamectl set-hostname liumiaocn
[root@host shell]# 
```

再次登录之后，hostname的变化即可看到

```bash
[root@liumiaocn ~]#
```

### 时间设定

> 使用命令： timedatectl

注：使用方法可参看：

- [Linux基础：timedatectl命令使用介绍-CSDN博客](https://liumiaocn.blog.csdn.net/article/details/88408155)

### 管理login：loginctrl

> 使用例：查看当前登录用户的Session状况

```bash
[root@liumiaocn ~]# loginctl
   SESSION        UID USER             SEAT            
        10          0 root                             

1 sessions listed.
[root@liumiaocn ~]#
```

## 管理locale：localectl

> 使用例：查看当前本地化设定信息

```bash
[root@liumiaocn ~]# localectl
   System Locale: LANG=en_US.UTF-8
       VC Keymap: us
      X11 Layout: us
[root@liumiaocn ~]#
```





[Termux:从0到1安装桌面系统(proot-distro)-CSDN博客](https://blog.csdn.net/m0_75196987/article/details/137704117)

[📱 在 Termux 上使用硬件 GPU 加速的 proot 容器 - 風雪城](https://blog.chyk.ink/2023/03/19/termux-hw-accelerated-proot/)

[使用Termux安装xfce4桌面, Android Studio, Code Server(VSCode) - Alain&#039;s Blog](https://www.alainlam.cn/?p=859)

[Termux如何安裝Debian系統 (圖形界面＋中文化＋音訊＋一鍵啟動指令稿) &#183; Ivon的部落格](https://ivonblog.com/posts/termux-proot-distro-debian/)

# Fcitx5配置

> 摘录自 [设置 Fcitx 5 - Fcitx](https://fcitx-im.org/wiki/Setup_Fcitx_5/zh-cn)

## 开机自启动

### 特定发行版中的工具

特定的发行版可能会提供一些用于自动启动 Fcitx 的工具，并且这些工具通常也会设置环境变量。

#### im-config (Debian/Debian-based/Ubuntu)

这是一个用于 Debian 和 Debian-based 发行版的工具。在登录到 GUI 之后，从命令行执行 `im-config`，应该会弹出一个向导程序，在其中选择 fcitx5 即可。

#### imsettings (Fedora)

这是一个与 im-config 类似的程序，它也提供了 GUI 来选择要使用的输入法框架。imsettings 
应该是被默认安装的，如果没有，你可以手动安装它。imsettings 
可以设置环境变量并且启动相应的输入法，它还提供了一个图形化的前端用于修改配置。你需要做的就是简单地执行`im-chooser`，log-out 然后再次 log-in。

[针对 Fedora 36 KDE 的操作说明](https://www.youtube.com/watch?v=FwqTtGEN4vQ)。 这个操作说明应该适用于除 GNOME 外的 XDG 兼容桌面。

#### fcitx5-autostart (Fedora)

这是一个 [fedora package]，打包了一个用于设置环境变量和 XDG autostart file 的 /etc/profile.d 脚本，可用于自启动。

### XDG Autostart

特定的发行版可能没有提供这个文件，如未提供，你可以直接复制 `/usr/share/applications/org.fcitx.Fcitx5.desktop` 到 `~/.config/autostart`

mkdir -p ~/.config/autostart && cp /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart

### KWin Wayland 5.24+

如果你只使用 Gtk/Qt/Xwayland 应用，那么你不需要这里的操作。如果你希望使用支持 text-input-v3 的原生 wayland 应用，则需要让 KWin 将输入法作为一个特殊的客户端启动。

打开 systemsettings，转到 "Virtual Keyboard" 部分，将输入法从 "None" 改为 "Fcitx 5"

### 非 XDG 兼容的窗口管理器/Wayland Compoistor

在不支持 XDG Autostart 的场景中，请检查你的窗口管理器的手册中关于如何在系统启动时自动运行应用程序的方法。

#### Weston

Weston 是一个 wayland compositor 的参考实现，并不是普通用户的常规配置。

如果你希望使用 westons zwp_input_method_v1 实现，你需要确保以下内容存在 ~/.config/weston.ini 文件中（如果路径不是 /usr/bin/fcitx5 请做相应修改）。

[input-method]
path=/usr/bin/fcitx5

如果你已经在同一个会话中运行 fcitx5，当你为了调试和 fcitx5 尝试在 nested mode 中使用 weston 时，会存在特定的问题。

如果你出于调试目的只在 X11 中运行 weston，最简单的方法是在启动 weston 前退出 fcitx5.

另请注意，weston 有一个 bug，在首次运行时不会正确设置 DISPLAY 为输入法。您可能需要终止 fcitx5 一次才能使其正确设置 DISPLAY，或使用 OpenX11Connection dbus 调用来连接 fcitx。

## 环境变量

由于许多地方都处于过渡阶段，因此没有适合所有人的完美解决方案。请根据您的环境选择适合您的解决方案。基本上，您想要做的是为桌面会话设置以下环境变量。

 XMODIFIERS=@im=fcitx
 GTK_IM_MODULE=fcitx
 QT_IM_MODULE=fcitx
虽然它看起来像有效的 shell 脚本，但请 *注意* 上面的代码片段只是为了演示这些值是什么。请检查以下部分以了解不同方法的具体语法。

### 登录 shell 配置文件

如果您正在使用Bash作为您的登录shell，`~/.bash_profile` 是您可以信赖的最好的用户级东西。它受到不同 DM 的广泛支持，如果您从 TTY 启动图形，它也可以工作。

- 支持主流显示管理器，包括GDM/SDDM/LightDM
- TTY 登录

如果您不使用 bash，您可能需要仔细检查您的 shell 配置文件是否可以用作设置环境变量的位置，尤其是当您正在使用一些不常见的登录 shell时。

您需要添加到 `~/.bash_profile` 的代码片段如下：

export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx

有些人可能会认为 `~/.profile` 是一个与 shell 无关的解决方案，这是错误的。虽然 GDM 始终获取此文件，但如果 `~/.bash_profile` 存在，SDDM/Bash 将不会获取此文件。这使得 `~/.bash_profile` 成为更好的解决方案，因为 bash 的使用相当广泛。但在继续之前请检查您的登录 shell，某些发行版可能不使用 bash 作为默认 shell。

此[视频](https://youtu.be/8XDmLr6wb4M)演示了如何在 Archlinux 上手动设置环境变量

### /etc/profile

如果您不关心使用 root 修改文件，这是最好的选择。所有发行版通常都支持此文件。您需要附加到 `/etc/profile` 末尾的代码片段与 [登录shell配置文件](https://fcitx-im.org/wiki/Special:MyLanguage/Setup_Fcitx_5#Login_in_shell_profile "Special:MyLanguage/Setup Fcitx 5") 相同。

### ~/.xprofile

如果您使用 X11 和显示管理器，这是一个古老的完美选择。但 Wayland 没有对应的环境变量，因此如果要为 Wayland 设置环境变量并不理想。您要添加的代码与[登录shell配置文件](https://fcitx-im.org/wiki/Special:MyLanguage/Setup_Fcitx_5#Login_in_shell_profile "Special:MyLanguage/Setup Fcitx 5") 相同。

### environment.d

这是 system.d 引入的新配置，但并未得到桌面环境或显示管理器支持的广泛使用。它基本上是 systemd 
用户单元的环境配置。目前，仅 GDM 或 Plasma 5.22+ 支持。作为 GDM，这意味着使用 GDM 登录的任何会话都可以工作。至于 
Plasma，这意味着无论您使用什么 DM，它都适用于 Plasma。

此配置将在您首次用户会话登录时应用，并在之后持续存在，除非您手动停止 systemd 用户。因此，修改此配置后，使其生效的最简单方法是重新启动系统。

语法与 shell 类似，但不需要 `export`。例如，您可以创建一个包含以下内容的文件 `~/.config/environment.d/im.conf`：

XMODIFIERS=@im=fcitx
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx

### pam_env.so

由于以下原因，这是一个过时的解决方案：

* pam 自 1.5.0 起弃用用户级别配置 `~/.pam_environment`。
* 某些发行版未在其 pam 配置中启用 pam_env。

如果您知道它适用于您的系统，您可以将以下代码片段添加到您的 `~/.pam_environment` 中。

XMODIFIERS DEFAULT=\@im=fcitx
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE DEFAULT=fcitx.

请**注意**，其语法与 shell 脚本不同。

### ~/.config/plasma-workspace/env/*.sh

仅适用于 Plasma 桌面的 env 脚本位置，您需要创建自己的 .sh 文件，例如 `~/.config/plasma-workspace/env/im.sh` 并将代码片段与[登录shell配置文件](https://fcitx-im.org/index.php?title=Special%EF%BC%9AmyLanguage/Setup_Fcitx_5&action=edit&redlink=1 "Special：myLanguage/Setup Fcitx 5 (page does not exist)")。

### 其他不太常见的设置

还有一些其他变量可能对某些应用程序有用。

#### SDL_IM_MODULE

将值设置为 fcitx。只有 SDL2 需要这个。SDL1 使用 XIM。

#### GLFW_IM_MODULE

这是仅由[[1]](https://github.com/kovidgoyal/kitty/kitty)使用的变量。您需要将其设置为“GLFW_IM_MODULE=ibus”。

#### Binary Qt application

由于 Qt 5 不支持 XIM，并且它仅捆绑 ibus im 模块，因此您可能需要为不使用系统 Qt 库的 Qt 应用程序设置“QT_IM_MODULE=ibus”。 （它可能仍然无法工作，因为某些 Qt 应用程序甚至没有捆绑任何 IM 模块）。

### DBus

在大多数附带 systemd 的发行版上，这应该不再是问题。但是如果您使用一些所谓的“systemd”free 
的发行版，您可能需要自己启动DBus并设置相关的环境变量。通常，这可以通过在启动脚本中添加如下行来完成。例如，如果您使用的是 X11，则为 
~/.xprofile。此外，您还需要确保此语法适用于您的登录 shell。

eval `dbus-launch --sh-syntax --exit-with-session`
