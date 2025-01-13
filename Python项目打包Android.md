---
title: "Python项目打包Android"
description: "关于Python打包运行在Android上的讨论，囊括简单的原理，Python-for-Android项目的使用介绍以及一个简单的案例"
keywords: 
- python
- android
- Python-for-Android
- Xterm
- Python打包Android App
date: 2025-01-13T18:02:05+08:00
lastmod: 2025-01-13T18:02:05+08:00
---

## 内容概述

这篇文章用来记录我自己在Android上折腾Python后的一点小小心得，大致包含前置理论，相关工具使用介绍，以及最后一些打包实例，三个部分。

前置理论部分主要是记录一些“为什么”，但这部分我也只是浅尝罢了给不出太过深奥的讲解，为此我在文末收集了很多相关的链接方便有需要的人，也方便我自己后续学习。

相关工具板块包括了Python-for-Android以及Buildozer这两个工具的使用心得，点击[这里](#Python-for-Android)跳转

至于实例部分，则是将Github项目[超星学习通自动化完成任务点(命令行版)](https://github.com/Samueli924/chaoxing)打包成App运行在Android上的过程记录，单击[这里](#案例实录)跳转

## 在Android上使用Python比较复杂的原因

Python 是一门**解释型**语言，这意味着只要我们有对应的**解释器**就能执行我们的代码，而不用关注平台差异。在大部分开发情况下我们都是这样做的，Python官方团队提供了包含Windows、Linux、macOS等众多系统上可用的Python解释器。那么问题来了，为什么没有Android这个主流平台呢？这就要从Android自身特性说起了。

首先，Android 系统采用了沙盒机制，每个应用在自己的独立进程和用户空间中运行，互不干扰。这种机制提高了安全性，但也意味着应用不能轻易共享代码或资源。实际开发过程中，我们将每个 Android 应用程序都打包成一个 APK 文件，包含了应用运行所需的**所有**资源和代码。但实际上还是有一些诸如系统库和Android API是共用的，不过Android原生支持Java/Kotlin开发，这显然和Python没有关系。

其次，Android是专门为移动设备设计的，移动设备的性能和资源（如内存和电池）相对有限，要想让Python解释器在Android上运行就需要对源码稍加修改。

总而言之，Android因为考虑安全性，设备性能差异等进行了一些独有的优化设计，这些设计使得Android平台与Windows等为PC设计的系统环境有所差异。而想要在这种特殊的系统环境下运行Python也不是没有办法。

## 系统层面探究实现原理

上述我们提到Python运行的关键就是它的解释器，这其实也相当于一个软件，而我们常用的（即官网下载）是所谓的CPython，那么C体现在哪里？其实就是这个解释器是使用C语言编写的。下面这句话摘抄自[《维基百科》](https://zh.wikipedia.org/wiki/CPython)

```
CPython是用C语言实现的Python解释器。
作为官方实现，它是最广泛使用的Python解释器。
除了CPython以外，还有用Java实现的Jython，用.NET实现的IronPython，
使Python方便地和Java程序、.NET程序集成。
另外还有一些实验性的Python解释器比如PyPy。[1] 
```

了解到这里，你还需要知道的是在Android开发过程中开发人员可以打包调用使用C语言编译的动态链接库(通常为.so后缀结尾的文件)。那么思路就很简单了，既然Python解释器使用C语言所写的，而Android又可以调用C程序，我们就可以将CPython打包进我们的App然后进而动态解析执行我们的Python代码。

> 有关Android使用C代码开发的原理可以查看Android系统架构，文末参考中也给出了一些包括官方在内的链接

使用PyInstaller等工具打包过Python程序的同学一定知道一件事，我们只能在对应的平台上打包相应的软件包，例如在Windows x64系统上就只能打包Windows x64系统上可用的软件包，要想为Linux打包就需要我们使用相应的Linux系统来操作。这个道理很简单，所谓打包就是将程序执行所需要的一系列依赖项目整合到一起方便发布与使用的过程，那么在Windows上肯定只能找到Windows相关的系统库和依赖，Linux上也是同理。那么理所应当的，为Android环境打包的最直接的想法就是在Android上进行操作。实则不然，鉴于Android平台硬件算力的不足（某些ARM也是同理），以及开发环境的局限性，直接在 Android 设备上进行打包并不是一个理想的选择。为了解决这个问题，可以使用交叉编译的方法。在桌面操作系统上设置交叉编译环境，生成适用于 Android 平台的可执行文件和依赖项。

什么是交叉编译？上面其实已经阐述过了，下面给出总结：交叉编译是相对本地编译来说的，即在当前编译平台下，编译出来的程序能运行在体系结构不同的另一种目标平台上，但是编译平台本身却不能运行该程序。

实际操作中，对于Android环境下的交叉编译，我们有官方提供的工具NDK，有关交叉编译的事情这里不再深入叙述，原因当然是我自己也不太了解，想要了解的可以自行搜索或者查看本文最后附上的参考链接中相关内容。

好了，截止目前我们已经梳理清楚了**在Android平台上运行Python解释器**的相关思路，有没有发现我说的只是**解释器**，事实上Python的强大除了自身语法简单易用外，众多的第三方库是一个很重要的原因。我们使用Python编程大概率是离不开第三方库的，那么如果实际想要在Android上使用Python，我们在交叉编译时就不仅仅要考虑到解释器，还需要考虑使用的库。

这里我们简单将所有的库分为三大类：

1. Python标准库（官方内置，提供最基础的功能）

2. 纯粹使用Python语言开发的第三方库（相当于我们自己用Python写的模块，这类代表是Requests）

3. 其他设计如C代码的Python库（典型例子为NumPy）

对于第一类与第二类库，我们都不需要考虑跨平台运行的问题，但对于第三类来说，如果我们需要使用他们就需要使用NDK交叉编译对应的包，日常我们使用时都是直接使用pip进行管理，这其实是隐藏了编译相关的细节让我们觉得安装第三方库是一件非常轻松的事情，实则不然，如果需要自己编译（甚至要为了适配Android环境进行打包）是一件非常麻烦的事情，这需要我们去深入了解对应包的源码，然后一个个去改造编译。

这里不得不引出另一种在Android平台上使用Python的方式——Termux。它的实现和我们上述将所有依赖都单独拿出来为Android进行构建不同，而是借助Android底层就是Linux内核的特点以及Proot能够在非根目录下运行Linux系统的能力，在Android环境下提供了一个类似ARM Linux的系统环境，这个环境很多包都会直接支持，从而省去了交叉编译的很多麻烦。

言归正传，这里我们主要介绍的还是重新为Android进行编译的方法，一个个编译太过于复杂且难度太大，而这种在Android上运行Python，或者是使用Python为Android开发的想法虽然说小众但肯定不是只有我们能够想到，所以我们不难找到一些工具来帮助我们完成这项工作。

## 实际操作过程中使用的工具

这里我列出我搜到的一些能够完成Python开发Android的工具或是项目：

1. QPython：QPython 是一个在 Android 上运行 Python 的应用，提供了完整的 Python 环境。它支持 Python 2 和 Python 3，并且内置了许多常用的库和模块。

2. BeeWare：BeeWare 是一个跨平台的工具包，允许使用 Python 编写 Android 应用。它提供了一套工具和库，使得开发者可以在多个平台上编写和运行 Python 应用。

3. Chaquopy：Chaquopy 是一个 Android Studio 插件，允许在 Android 应用中使用 Python。它集成了 Python 解释器和标准库，使得开发者可以在 Java 和 Python 之间无缝切换。

4. Kivy：Kivy 是一个用于开发多点触控应用的开源 Python 库。它支持在 Android 上运行，并且提供了丰富的 UI 组件和工具。

5. SL4A (Scripting Layer for Android)：SL4A 是一个允许在 Android 上运行脚本语言的项目，包括 Python。它提供了一组 API，使得脚本可以访问 Android 的功能和服务。

6. Buildozer：Buildozer 是一个用于将 Python 应用打包成 Android APK 文件的工具。它支持 Kivy 和其他框架，并且可以自动处理依赖项和配置。

上述这些工具中，我要介绍的就是Kivy团队开发维护的Python-for-Android项目，这个团队维护了一些列关于Android相关的Python项目，而且实用性都比较高。kivy是能够打包Android App的Python图形界面库，Buildozer是用来简化Python-for-Android项目一系列配置的自动化工具。要想使用Python-for-Android你需要自己配置Java SDK，NDK等等，而Buildozer能够帮助你自动完成这个过程。

## Python-for-Android
<span id="Python-for-Android"></span>

首先给出官网，这是第一手的学习资料[python-for-android](https://python-for-android.readthedocs.io/en/latest/)

简单提一下，这个工具的打包方式有三种，第一种是Kivy or SDL2，即使用特定的GUI工具开发然后进行打包（Android环境下均需要配置一个界面），第二章则是WebView，即Python使用Fastapi或者Flask这样的框架开发后端服务，然后App使用WebView展示本地的网页来和用户交互，第三种则是Service library archive，即只是导出半成品，剩下的需要自己使用Android Studio等进行开发，适合高级用户。

项目中的Recipes则是相当于用来编译上述第三方依赖的脚本，用来描述某个包的资源如何组织，Bootstrap则是用来描述整个应用如何组织的脚本，完成组织图形化界面或者是WebView这样的工作。

下面着重介绍开发环境的搭建流程，这个在官网也是有的。

> 如果不是为了研究的话，实际使用还是建议直接使用Buildozer，当初我尝试Python-for-Android的原因是我发现Buildozer的速度明显会很慢，有了了解之后我发现其实是Buildozer默认会开启调试模式，进而输出大量的日志，拖慢了速度，所以如果不是调试的话在配置文件中进行设置就好，我会在Buildozer的章节叙述具体做法。

接下来的步骤会有一些繁琐，大概需要你多半天的时间，你需要保持耐心去操作，我的记录的很详细，希望能帮到你

> 下面的记录是我在尝试开发项目ChaoxingLite的时候写的，有提到的地方直接替换成你需要的打包的项目即可

### 系统环境
操作系统我这里使用的Windows11，在此基础上我使用了wsl2安装了Ubuntu-24.04
这里假设你已经安装并配置好了wsl2

1. 安装Ubuntu-24.04
```shell
wsl --install Ubuntu-24.04
```
2. 新系统先更换apt的源，这里选择[中科大的镜像](https://mirrors.ustc.edu.cn/help/ubuntu.html)
```shell
# 国内镜像使用https协议所以需要先安装两个https有关的包
sudo apt install apt-transport-https ca-certificates
# 接下来打开配置文件
nano /etc/apt/sources.list
```
在文件的后面写入下面内容,你可以参考中科大的镜像使用帮助
https://mirrors.ustc.edu.cn/help/ubuntu.html
```
# 默认注释了源码仓库，如有需要可自行取消注释
deb https://mirrors.ustc.edu.cn/ubuntu/ noble main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ noble-security main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-security main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ noble-updates main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-updates main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ noble-backports main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.ustc.edu.cn/ubuntu/ noble-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-proposed main restricted universe multiverse
```
换源完毕，更新apt源
```shell
apt update
```

系统层面的准备到此结束

### python 环境
这时你查看python的版本一般为python12，我们需要切换到python10，这里使用pyenv

```shell
# 查看当前python版本
python --version
```

安装pyenv的步骤很简单，如下
```shell
# 执行以下命令安装依赖库
sudo apt-get install make build-essential libssl-dev zlib1g-dev 
sudo apt-get install libbz2-dev libreadline-dev libsqlite3-dev wget curl 
sudo apt-get install llvm libncurses5-dev libncursesw5-dev 

# 安装pyenv，作者提供了一键安装脚本，直接执行
curl https://pyenv.run | bash
```
配置下面内容到`~/.bashrc`的后面
```shell
export PATH="~/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

立即重新加载`~/.bashrc`
```shell
source ~/.bashrc
```

pyenv到此安装完毕，接下来安装python10
```shell
pyenv install 3.10 -v
```

安装好后，配置默认使用python10，这里我们之后选择在用户目录下执行操作，所以我选择将用户目录下默认python配置为python10

```shell
cd /home/你的用户名
# 或者使用
cd ~

# 配置当前目录下默认python
pyenv local python10

# 验证
python --version
```
到这里pyenv配置结束，有关他的其他信息你可以查看
https://www.cnblogs.com/safe-rabbit/p/17130336.html

有一些python包是后续可能需要的，我们在这里现将他们一次性都装好

```shell
pip install cython kivy pyjnius six

# Cython用于某些包的编译
# jnius 是一个用于在 Python 中调用 Java 代码的库
# six 是一个用于编写兼容 Python 2 和 3 代码的库，某些依赖项可能需要它。
# 如果您的项目使用了 kivy，安装他，kivy是python-for-android的首选图形库
```

现在我们有了合适版本的python，可以开始其他工作了！

### python-for-android 环境配置

接下来的内容你可以参考官方说明

https://python-for-android.readthedocs.io/en/latest/quickstart.html

此外，我还找到了一个使用了python-for-android的有趣项目，他对我的帮助不浅，这份资料也可以作为参考

https://github.com/NaitLee/Cat-Printer/blob/main/build-android/manual-steps.md

开始正题，我们需要两个文件夹：

    - git-repo 用来存放我们从github克隆下来的项目代码
    - android 用来存放android-sdk等构建工具

进入用户主目录下创建文件夹

```shell
cd ~
mkdir android
mkdir git-repo
```

安装python-for-android项目库，直接pip就可以安装
```shell
pip install python-for-android

# 此外还需要一些其他的包，按照官方文档给出的安装一下

sudo apt-get install -y \
    ant \
    autoconf \
    automake \
    ccache \
    cmake \
    g++ \
    gcc \
    git \
    lbzip2 \
    libffi-dev \
    libltdl-dev \
    libtool \
    libssl-dev \
    make \
    openjdk-17-jdk \
    patch \
    pkg-config \
    python3 \
    python3-dev \
    python3-pip \
    python3-venv \
    sudo \
    unzip \
    wget \
    zip
```

这里安装了openjdk-17-jdk，但是有时候你会发现他不是默认选项，使用下面命令切换jdk版本
```shell
sudo update-alternatives --config java
```

接下来，我们需要配置python-for-android需要的Android构建工具

```shell
# 先切换到android 目录
cd ~/android
```

进入 https://developer.android.com/studio?hl=zh-cn
下载commandlinetools-linux-*_latest.zip（一直往下翻可以看到）

进入 https://github.com/android/ndk/wiki/Unsupported-Downloads 下载旧版本的Android ndk 这里选择r25c

你可以直接使用Windows浏览器下载得到两个文件
    - android-ndk-r25c-linux.zip
    - commandlinetools-linux-*_latest.zip
  
打开Windows资源管理器将他们放到当前linux的用户文件夹下的android目录

解压两个文件
```shell
# 先安装unzip

sudo apt install unzip

# 解压
sudo unzip android-ndk-r25c-linux.zip
sudo unzip commandlinetools-linux-*_latest.zip（这里注意更换成你真正下载得到的文件名）
```

将解压得到的cmdline-tools文件夹内创建一个latest文件，其内部的文件都复制到这个latest文件夹内部，之后再将cmdline-tools文件夹放入android-sdk目录下，最终你的目录结构是这样的

```
android
    android-sdk
        cmdling-tools
            latest
                解压得到原cmdling-tools文件内的文件
```

这么做是为了让工具能够正确识别根目录

然后还有android-ndk，这个好说，将解压后的android-ndk-r25c文件直接放入android-ndk下，你的目录结构变为

```
android
    android-sdk
        cmdling-tools
            latest
                解压得到原cmdling-tools文件内的文件
    android-ndk
        android-ndk-r25c
```

进入~/android/android-sdk/cmdling-tools/latest/bin/下执行

```shell
cd ~/android/android-sdk/cmdling-tools/latest/bin/

# 安装平台工具
sudo ./sdkmanager "platform-tools"

# 安装指定的 Android 平台
# 不同于官方文档我们选择最新的标准30
sudo ./sdkmanager "platforms;android-30" 

# 安装构建工具
# 同样这里选择33.0.2
sudo ./sdkmanager "build-tools;33.0.2"

# 安装 tools 目录： 运行以下命令安装 tools 目录
sudo ./sdkmanager "tools"
```

执行上述命令后android-sdk的目录变成了

```
android
    android-sdk
        build-tools
        cmdline-tools
        emulator
        licneses
        platforms
        platform-tools
        tools
```

接下来配置环境变量让python-for-android能够找到他们

在`~/.bashrc`中写入

```bash
export DIR_BUILD=/home/你的用户名
export ANDROIDSDK=$DIR_BUILD/android/android-sdk
export ANDROIDNDK=$DIR_BUILD/android/android-ndk/android-ndk-r25c
export ANDROIDAPI="30"  # 目标 API 版本
export NDKAPI="21"  # 最低支持的 API 版本
export ANDROIDNDKVER="r25c"  # 安装的 NDK 版本

# 将 SDK 和 NDK 路径添加到 PATH
export PATH="$ANDROIDSDK/tools:$ANDROIDSDK/platform-tools:$ANDROIDNDK:$PATH"
```

记得更新
```shell
source ~/.bashrc
```

到这里，我们基本配置完毕，我们回到`~/git-repo`进行测试构建，这一步需要较长的时间，在此期间你可以先休息一下了。

```shell
cd ~/git-repo
cd ChaoxingLite/build-android/

# 执行测试构建
./0-build-android.sh
```

到这里如果你已经正常打包，这很好，如果你有一些问题，那也不必担心。

首先，你遇到的问题可能是进度卡在了自动下载`gradle-8.0.2-all.zip`这个文件的过程中了，这时候你需要强制终止构建进程(同时按下`Ctrl`和字母`c`键),然后手动下载这个文件，将他放在日志输出的期望位置，我这里是`\\wsl.localhost\Ubuntu-24.04\home\用户名\.gradle\wrapper\dists\gradle-8.0.2-all\14bt34ptcsg1ikmfn78tdh1keu`下

之后你可以重新开始构建

```shell
./0-build-android.sh
```

这一次你还有可能遇到`Java heap space`错误，这里我没有找到其他较好的有效办法，我选择了简单更改一下`python-for-android`项目的源代码

```shell
# 首先查看一下这个包的安装位置
pip show python-for-android
```
进入到那个文件（你可以使用文件浏览器打开）,你会看到一个`toolchain.py`代码文件，打开编辑他

你需要做的是定位到`_build_package`这个函数

将函数最后的

`
output = shprint(gradlew, "clean", gradle_task, _tail=20,
                             _critical=True, _env=env)
`
改为
`output = shprint(gradlew, "clean", gradle_task, "-Dorg.gradle.jvmargs=\"-Xmx6g\"", _tail=20,
                             _critical=True, _env=env)`


好吧，现在请你再重新构建一次

```shell
./0-build-android.sh
```

这次应该能正常成功了，如果还是不行的话，你可以先按照上述步骤进行检查，确认没有遗漏后，可以提出issue或者直接到社区寻求帮助，官方的Discord是https://discord.com/invite/eT3cuQp


## Buildozer

Buildozer的官网在这里[buildozer](https://buildozer.readthedocs.io/en/latest/)，同样的官方文档依然是最重要的学习资料，而且上面一节我已经记录了很详细的过程，所以buildozer这里不在赘述。

值得一提的是可以通过修改下面的参数(在buildoze.spec)，减少Buildozer的输入，从而提升打包速度
```spec
[buildozer]

# (int) Log level (0 = error only, 1 = info, 2 = debug (with command output))
log_level = 2
```



## Python-for-Android 源码分析

这里记录我对Python-for-Android项目中PythonActivity的代码分析，本人是一个Java新手，所以写的很细，同时因为水平问题内容有些杂乱，当我自己的胡言乱语就好。

### getAppRoot
```java
public String getAppRoot() {
        String app_root =  getFilesDir().getAbsolutePath() + "/app";
        return app_root;
    }
```
用于获取应用程序（python）的根路径

**getFilesDir()**：这是 `Context` 类中的一个方法，返回应用程序的内部存储文件目录的 `File` 对象。这个目录是应用程序专用的，其他应用程序无法访问。通常，这个目录位于 `/data/data/<package_name>/files`

**getFilesDir().getAbsolutePath()**:获取该目录的绝对路径

综上python脚本会被放在`/data/data/<package_name>/files/app`目录下

### getEntryPoint
```java
public String getEntryPoint(String search_dir) {
    /* Get the main file (.pyc|.py) depending on if we
     * have a compiled version or not.
    */
    List<String> entryPoints = new ArrayList<String>();
    entryPoints.add("main.pyc");  // python 3 compiled files
    for (String value : entryPoints) {
        File mainFile = new File(search_dir + "/" + value);
        if (mainFile.exists()) {
            return value;
        }
    }
    return "main.py";
}
```
用于获取应用程序的入口点文件（.pyc 或 .py）

### onCreate
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    Log.v(TAG, "My oncreate running");
    resourceManager = new ResourceManager(this);
    super.onCreate(savedInstanceState);

    this.mActivity = this;
    this.showLoadingScreen();
    new UnpackFilesTask().execute(getAppRoot());
}
```

这段代码定义了 onCreate 方法，这是 Android 应用程序中 Activity 类的一个生命周期方法。onCreate 方法在活动被创建时调用，用于初始化活动的状态。

[showLoadingScreen 方法见此](#showloadingscreen)

`new UnpackFilesTask().execute(getAppRoot())`：创建一个 UnpackFilesTask 类的实例调用 UnpackFilesTask 实例的 execute 方法，并传递 getAppRoot() 方法的返回值作为参数。execute 方法会启动异步任务，并依次调用 onPreExecute、doInBackground 和 onPostExecute 方法。

[UnpackFilesTask 类定义见此](#unpackfilestask)

### showLoadingScreen

```java
protected void showLoadingScreen() {
    // load the bitmap
    // 1. if the image is valid and we don't have layout yet, assign this bitmap
    // as main view.
    // 2. if we have a layout, just set it in the layout.
    // 3. If we have an mImageView already, then do nothing because it will have
    // already been made the content view or added to the layout.

    if (mImageView == null) {
        int presplashId = this.resourceManager.getIdentifier("presplash", "drawable");
        InputStream is = this.getResources().openRawResource(presplashId);
        Bitmap bitmap = null;
        try {
            bitmap = BitmapFactory.decodeStream(is);
        } finally {
            try {
                is.close();
            } catch (IOException e) {};
        }

        mImageView = new ImageView(this);
        mImageView.setImageBitmap(bitmap);

        /*
         * Set the presplash loading screen background color
         * https://developer.android.com/reference/android/graphics/Color.html
         * Parse the color string, and return the corresponding color-int.
         * If the string cannot be parsed, throws an IllegalArgumentException exception.
         * Supported formats are: #RRGGBB #AARRGGBB or one of the following names:
         * 'red', 'blue', 'green', 'black', 'white', 'gray', 'cyan', 'magenta', 'yellow',
         * 'lightgray', 'darkgray', 'grey', 'lightgrey', 'darkgrey', 'aqua', 'fuchsia',
         * 'lime', 'maroon', 'navy', 'olive', 'purple', 'silver', 'teal'.
         */
        String backgroundColor = resourceManager.getString("presplash_color");
        if (backgroundColor != null) {
            try {
                mImageView.setBackgroundColor(Color.parseColor(backgroundColor));
            } catch (IllegalArgumentException e) {}
        }
        mImageView.setLayoutParams(new ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.FILL_PARENT,
            ViewGroup.LayoutParams.FILL_PARENT));
        mImageView.setScaleType(ImageView.ScaleType.FIT_CENTER);
    }

    if (mLayout == null) {
        setContentView(mImageView);
    } else if (PythonActivity.mImageView.getParent() == null){
        mLayout.addView(mImageView);
    }
}
```

这个方法用于在应用程序启动时显示加载屏幕。加载屏幕通常用于在应用程序初始化或加载资源时向用户显示一个过渡界面，以提高用户体验。

### UnpackFilesTask

```java
private class UnpackFilesTask extends AsyncTask<String, Void, String> {
    @Override
    protected String doInBackground(String... params) {
        File app_root_file = new File(params[0]);
        Log.v(TAG, "Ready to unpack");
        PythonUtil.unpackAsset(mActivity, "private", app_root_file, true);
        PythonUtil.unpackPyBundle(mActivity, getApplicationInfo().nativeLibraryDir + "/" + "libpybundle", app_root_file, false);
        return null;
    }

    @Override
    protected void onPostExecute(String result) {
        Log.v("Python", "Device: " + android.os.Build.DEVICE);
        Log.v("Python", "Model: " + android.os.Build.MODEL);

        PythonActivity.initialize();

        // Load shared libraries
        String errorMsgBrokenLib = "";
        try {
            loadLibraries();
        } catch(UnsatisfiedLinkError e) {
            System.err.println(e.getMessage());
            mBrokenLibraries = true;
            errorMsgBrokenLib = e.getMessage();
        } catch(Exception e) {
            System.err.println(e.getMessage());
            mBrokenLibraries = true;
            errorMsgBrokenLib = e.getMessage();
        }

        if (mBrokenLibraries) {
            AlertDialog.Builder dlgAlert  = new AlertDialog.Builder(PythonActivity.mActivity);
            dlgAlert.setMessage("An error occurred while trying to load the application libraries. Please try again and/or reinstall."
                  + System.getProperty("line.separator")
                  + System.getProperty("line.separator")
                  + "Error: " + errorMsgBrokenLib);
            dlgAlert.setTitle("Python Error");
            dlgAlert.setPositiveButton("Exit",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog,int id) {
                        // if this button is clicked, close current activity
                        PythonActivity.mActivity.finish();
                    }
                });
           dlgAlert.setCancelable(false);
           dlgAlert.create().show();

           return;
        }

        // Set up the webview
        String app_root_dir = getAppRoot();

        mWebView = new WebView(PythonActivity.mActivity);
        mWebView.getSettings().setJavaScriptEnabled(true);
        mWebView.getSettings().setDomStorageEnabled(true);
        mWebView.loadUrl("file:///android_asset/_load.html");

        mWebView.setLayoutParams(new LayoutParams(LayoutParams.FILL_PARENT, LayoutParams.FILL_PARENT));
        mWebView.setWebViewClient(new WebViewClient() {
                @Override
                public boolean shouldOverrideUrlLoading(WebView view, String url) {
                    Uri u = Uri.parse(url);
                    if (mOpenExternalLinksInBrowser) {
                        if (!(u.getScheme().equals("file") || u.getHost().equals("127.0.0.1"))) {
                            Intent i = new Intent(Intent.ACTION_VIEW, u);
                            startActivity(i);
                            return true;
                        }
                    }
                    return false;
                }

                @Override
                public void onPageFinished(WebView view, String url) {
                    CookieManager.getInstance().flush();
                }
            });
        mLayout = new AbsoluteLayout(PythonActivity.mActivity);
        mLayout.addView(mWebView);

        setContentView(mLayout);

        String mFilesDirectory = mActivity.getFilesDir().getAbsolutePath();
        String entry_point = getEntryPoint(app_root_dir);

        Log.v(TAG, "Setting env vars for start.c and Python to use");
        PythonActivity.nativeSetenv("ANDROID_ENTRYPOINT", entry_point);
        PythonActivity.nativeSetenv("ANDROID_ARGUMENT", app_root_dir);
        PythonActivity.nativeSetenv("ANDROID_APP_PATH", app_root_dir);
        PythonActivity.nativeSetenv("ANDROID_PRIVATE", mFilesDirectory);
        PythonActivity.nativeSetenv("ANDROID_UNPACK", app_root_dir);
        PythonActivity.nativeSetenv("PYTHONHOME", app_root_dir);
        PythonActivity.nativeSetenv("PYTHONPATH", app_root_dir + ":" + app_root_dir + "/lib");
        PythonActivity.nativeSetenv("PYTHONOPTIMIZE", "2");

        try {
            Log.v(TAG, "Access to our meta-data...");
            mActivity.mMetaData = mActivity.getPackageManager().getApplicationInfo(
                    mActivity.getPackageName(), PackageManager.GET_META_DATA).metaData;

            PowerManager pm = (PowerManager) mActivity.getSystemService(Context.POWER_SERVICE);
            if ( mActivity.mMetaData.getInt("wakelock") == 1 ) {
                mActivity.mWakeLock = pm.newWakeLock(PowerManager.SCREEN_BRIGHT_WAKE_LOCK, "Screen On");
                mActivity.mWakeLock.acquire();
            }
        } catch (PackageManager.NameNotFoundException e) {
        }

        final Thread pythonThread = new Thread(new PythonMain(), "PythonThread");
        PythonActivity.mPythonThread = pythonThread;
        pythonThread.start();

        final Thread wvThread = new Thread(new WebViewLoaderMain(), "WvThread");
        wvThread.start();
    }
}
```

`AsyncTask` 是 Android 提供的一个类，用于简化在后台线程中执行任务并在主线程中更新 UI 的过程。AsyncTask 类主要用于在后台线程中执行耗时操作，并在操作完成后在主线程中更新 UI。它提供了一种简单的方式来避免在主线程中执行耗时操作，从而防止应用程序的 UI 卡顿。

- doInBackground：
  这是一个抽象方法，必须在子类中实现。在后台线程中执行耗时操作。接受一个参数数组，返回一个结果。不能直接更新 UI。
- onPostExecute：
  在 doInBackground 方法完成后在主线程中执行。接受 doInBackground 方法的返回值作为参数。用于更新 UI。
- onPreExecute：
  在 doInBackground 方法完成后在主线程中执行。接受 doInBackground 方法的返回值作为参数。用于更新 UI。在主线程中执行，用于更新任务的进度。需要在 doInBackground 方法中调用 publishProgress 方法来触发。
- onCancelled：
  当任务被取消时在主线程中执行。用于处理任务取消后的清理工作。

`UnpackFilesTask`类实现了上述的`doInBackground`和`onPostExecute`两个方法。

```java
@Override
protected String doInBackground(String... params) {
    File app_root_file = new File(params[0]);
    Log.v(TAG, "Ready to unpack");
    PythonUtil.unpackAsset(mActivity, "private", app_root_file, true);
    PythonUtil.unpackPyBundle(mActivity, getApplicationInfo().nativeLibraryDir + "/" + "libpybundle", app_root_file, false);
    return null;
}
```

于在后台线程中执行解压和初始化应用程序所需的文件。

`File app_root_file = new File(params[0]);`：使用传递的参数创建一个 File 对象，表示应用程序的根目录。

`PythonUtil.unpackAsset(mActivity, "private", app_root_file, true);`：调用 PythonUtil 类的 unpackAsset 方法，将 private 目录中的资产文件解压到应用程序根目录。

参数解释如下：
- mActivity：当前活动的上下文。
- "private"：要解压的资产目录。
- app_root_file：解压的目标目录。
- true：表示覆盖现有文件。

`PythonUtil.unpackPyBundle(mActivity, getApplicationInfo().nativeLibraryDir + "/" + "libpybundle", app_root_file, false);`：调用 PythonUtil 类的 unpackPyBundle 方法，将 libpybundle.so 文件解压到应用程序根目录。

参数解释如下：
- mActivity：当前活动的上下文。
- getApplicationInfo().nativeLibraryDir + "/" + "libpybundle"：libpybundle.so 文件的路径。
- app_root_file：解压的目标目录。
- false：表示不覆盖现有文件。

```java
// Load shared libraries
String errorMsgBrokenLib = "";
try {
    loadLibraries();
} catch(UnsatisfiedLinkError e) {
    System.err.println(e.getMessage());
    mBrokenLibraries = true;
    errorMsgBrokenLib = e.getMessage();
} catch(Exception e) {
    System.err.println(e.getMessage());
    mBrokenLibraries = true;
    errorMsgBrokenLib = e.getMessage();
}

if (mBrokenLibraries) {
    AlertDialog.Builder dlgAlert  = new AlertDialog.Builder(PythonActivity.mActivity);
    dlgAlert.setMessage("An error occurred while trying to load the application libraries. Please try again and/or reinstall."
            + System.getProperty("line.separator")
            + System.getProperty("line.separator")
            + "Error: " + errorMsgBrokenLib);
    dlgAlert.setTitle("Python Error");
    dlgAlert.setPositiveButton("Exit",
        new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog,int id) {
                // if this button is clicked, close current activity
                PythonActivity.mActivity.finish();
            }
        });
    dlgAlert.setCancelable(false);
    dlgAlert.create().show();

    return;
}
```

用于加载共享库文件，如果发生错误那就显示一个错误弹框之后退出程序。

```java
// Set up the webview
String app_root_dir = getAppRoot();

mWebView = new WebView(PythonActivity.mActivity);
mWebView.getSettings().setJavaScriptEnabled(true);
mWebView.getSettings().setDomStorageEnabled(true);
mWebView.loadUrl("file:///android_asset/_load.html");

mWebView.setLayoutParams(new LayoutParams(LayoutParams.FILL_PARENT, LayoutParams.FILL_PARENT));
mWebView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        Uri u = Uri.parse(url);
        if (mOpenExternalLinksInBrowser) {
            if (!(u.getScheme().equals("file") || u.getHost().equals("127.0.0.1"))) {
                Intent i = new Intent(Intent.ACTION_VIEW, u);
                startActivity(i);
                return true;
            }
        }
        return false;
    }

    @Override
    public void onPageFinished(WebView view, String url) {
        CookieManager.getInstance().flush();
    }
});
mLayout = new AbsoluteLayout(PythonActivity.mActivity);
mLayout.addView(mWebView);

setContentView(mLayout);
```
这段代码用于在应用程序中显示一个 WebView，并加载指定的 HTML 文件。通过设置 WebViewClient，可以自定义 URL 加载行为和页面加载完成后的操作。将 WebView 添加到布局中，并设置为活动的内容视图，以便在应用程序中显示。



`mWebView = new WebView(PythonActivity.mActivity);`: 创建一个新的 WebView 实例，并传递当前活动的上下文。

`mWebView.setLayoutParams(new LayoutParams(LayoutParams.FILL_PARENT, LayoutParams.FILL_PARENT));`: 设置 WebView 的布局参数，使其填满父布局。


`setWebViewClient`方法设置一个自定义的 WebViewClient，用于处理 URL 加载和页面加载完成事件。
`shouldOverrideUrlLoading` 方法用于拦截 URL 加载请求。如果 mOpenExternalLinksInBrowser 为 true，并且 URL 不是本地文件或 127.0.0.1，则在外部浏览器中打开该 URL。
`onPageFinished` 方法在页面加载完成时调用，刷新 CookieManager。

```java
mLayout = new AbsoluteLayout(PythonActivity.mActivity);
mLayout.addView(mWebView);
setContentView(mLayout);
```
创建一个新的 AbsoluteLayout 实例，并将 WebView 添加到布局中。
将布局设置为活动的内容视图。

```java
try {
    Log.v(TAG, "Access to our meta-data...");
    mActivity.mMetaData = mActivity.getPackageManager().getApplicationInfo(
            mActivity.getPackageName(), PackageManager.GET_META_DATA).metaData;

    PowerManager pm = (PowerManager) mActivity.getSystemService(Context.POWER_SERVICE);
    if (mActivity.mMetaData.getInt("wakelock") == 1) {
        mActivity.mWakeLock = pm.newWakeLock(PowerManager.SCREEN_BRIGHT_WAKE_LOCK, "Screen On");
        mActivity.mWakeLock.acquire();
    }
} catch (PackageManager.NameNotFoundException e) {
}
```
获取应用程序的元数据。
如果元数据中包含 wakelock 设置，则获取屏幕常亮锁并保持屏幕常亮。

```java
Log.v(TAG, "Setting env vars for start.c and Python to use");
PythonActivity.nativeSetenv("ANDROID_ENTRYPOINT", entry_point);
PythonActivity.nativeSetenv("ANDROID_ARGUMENT", app_root_dir);
PythonActivity.nativeSetenv("ANDROID_APP_PATH", app_root_dir);
PythonActivity.nativeSetenv("ANDROID_PRIVATE", mFilesDirectory);
PythonActivity.nativeSetenv("ANDROID_UNPACK", app_root_dir);
PythonActivity.nativeSetenv("PYTHONHOME", app_root_dir);
PythonActivity.nativeSetenv("PYTHONPATH", app_root_dir + ":" + app_root_dir + "/lib");
PythonActivity.nativeSetenv("PYTHONOPTIMIZE", "2");
```

PythonActivity.nativeSetenv是来自start.c中定义的方法，

```java
final Thread pythonThread = new Thread(new PythonMain(), "PythonThread");
PythonActivity.mPythonThread = pythonThread;
pythonThread.start();

final Thread wvThread = new Thread(new WebViewLoaderMain(), "WvThread");
wvThread.start();
```
创建并启动 PythonThread 线程，用于运行 Python 代码。
创建并启动 WvThread 线程，用于加载 WebView。


### PythonMain
```java
class PythonMain implements Runnable {
    @Override
    public void run() {
        PythonActivity.nativeInit(new String[0]);
    }
}
```

`Runnable` 接口是 Java 中用于定义一个任务的接口。它只有一个方法 run，需要在实现该接口的类中重写。实现 Runnable 接口的类可以被传递给 Thread 对象，以便在新线程中执行任务。
`nativeInit`是调用的C代码，它的定义如下

`public static native int nativeInit(Object arguments);`

### WebViewLoaderMain
```java
class WebViewLoaderMain implements Runnable {
    @Override
    public void run() {
        WebViewLoader.testConnection();
    }
}
```

通过实现 Runnable 接口，WebViewLoaderMain 类可以被用来创建一个新线程，并在该线程中执行 testConnection 方法。这样可以避免在主线程中执行耗时操作，从而防止应用程序的 UI 卡顿。


## 案例实录

<span id="案例实录"></span>

源码仓库如下：
[前端项目 - Vue](https://github.com/CXLite/termnal_web)
[修改后的chaoxing项目](https://github.com/CXLite/CXLite_Server)
[项目打包工具](https://github.com/CXLite/CXLite)

CXLite这个项目的打包其实是讨巧了的，原项目是一个命令行程序，但是Android App却是需要
界面才行，读到这儿你肯定已经知道了Python-for-Android所支持的几种bootstrap。

而我的思路就来源于Termux那样的伪终端界面。展开了说，其实自从开始学编程起，我们敲下第一行HelloWord代码，讲的比较细致的教程就告诉我们print函数与input函数的作用就是输出到标准输出设备以及从标准输入设备获取输入。在了解的深入一些，这两个标准是我们是可以自己指定的，你也或许尝试过使用print输出内容到文件。而CXLite利用的则是自定义标准输入输出。

CXLite首先重定向系统标准输入输出到自己编写工具类，然后调用运行原项目的Python代码。用于处理输入输出的类则是一方面接收代码的输入输出请求，另一方面通过Websocket连接到前端页面的伪终端，这里前端网页中使用的是Xterm。

具体的源码也比较简单，只是实现的思路比较奇怪，需要看源码的链接我已经放在最开始了。
你只需要先克隆[项目打包工具](https://github.com/CXLite/CXLite)这个项目到本地，之后根据平台选择执行`setup.sh`或者是`setup.bat`就会自动配置好前端和改造后的chaoxing项目，至于打包app使用Buildozer即可。

然后需要注意的就是，原项目使用了lxml以及fonttools这两个库，我的改动主要就是删去了他们，lxml是因为我在打包过程中Kivy团队提供的对应Recipe貌似有问题，加之BeautifulSoup没有它任然可以使用，所以直接删除，而fonttools这库则是因为Kivy团队还没有支持。


## 文章参考

**安卓系统架构**

- [官方（中文）](https://developer.android.google.cn/guide/platform?hl=zh-cn)
- [Android 操作系统架构开篇 - 袁辉辉](http://gityuan.com/android/)

**Termux相关**

- [Termux原理 - 虚空之地](https://lixing48.github.io/archives/2021-12-22-Termux%E5%8E%9F%E7%90%86/)
- [Termux 原理探究 - HumphreyIO](https://blog.dang8080.cn/2020/03/31/22/)

**Python交叉编译**

- [深入理解交叉编译(Cross Compile) - Kernel-Team](https://kernel-team.github.io/2019-03-19/Cross-Compile)
- [python及第三方库交叉编译 - 侯哥](https://www.cnblogs.com/Se7eN-HOU/p/16736164.html)
