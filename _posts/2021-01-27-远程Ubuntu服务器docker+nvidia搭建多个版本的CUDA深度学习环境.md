---
title: 远程配置Ubuntu深度学习服务器GPU驱动+Docker+CUDA多个版本（包含NVIDIA驱动安装、NVIDIA环境下docker的安装和使用、基本的docker使用）
tags:
  - 环境搭建
---

## 前言

- 首先说一下为什么建议使用docker来使用搭建深度学习服务器。其实网上很多人都写了如何搭建CUDA10.0 + cudnn8.5 , CUDA9.0 + cudnn7.5 等等，并且从TensorFlow的官网你还可以了解到，不同版本的CUDA只能使用指定的几个TensorFlow版本（Pytorch更是只能下载指定CUDA版本的库），特别是在多个人同时使用的环境下，远程的深度学习服务器经常会崩掉，有可能是别人装了其他版本的CUDA，或者说有人想使用更高版本的CUDA就升级了GPU驱动，结果升级失败；还有可能是你自己想要用其他版本的深度学习框架来复现前人的code，结果发现要换个版本好难，甚至换个版本又把linux内核都升级了，恭喜你难受升级了。

- 因此，本文的目的就是为了让大家能够快速的从这种难受走出来，直接使用docker这个及其优美的工具来搭建深度学习服务器，你会发现，yyds啊。

首先，当然是安装GPU驱动了，本人使用的是学校的远程ubuntu系统，因此直接上手远程配置，如果是ubuntu桌面版的话，基本操作一样，在终端运行下面的代码就行。下面介绍两种安装方式：

##一、安装NVIDIA驱动

### 1. 直接安装

这种安装方式会比较快，但是很多不是刚装的系统会很难一次性装上，我之前也是有很多次部署GPU驱动都没办法用这种方式安装上，因此，建议刚装上的系统可以直接使用这个方法，只要源没有问题，基本上就可以，如果使用这种方法无法装上，那么就卸载了之后重新手动安装。


**1.1 查看显卡对应的驱动型号** 

在终端输入：

`ubuntu-drivers devices`

然后系统会花一定时间来检测你的对应驱动型号（要等一下才会出来）：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/ckqdxh.jpg"/></div>


这里看到系统推荐我安装的是460的型号，我们可以直接按照系统推荐的安装，也可以自己自己安装自己想要的几个版本，只要上面图片中展示了的都行。

**1.2 开始安装驱动**

按照推荐安装的代码：

`sudo ubuntu-drivers autoinstall`

安装自己制定的版本（**<font color='red'> 不是很推荐 </font>**）：

`sudo apt install nvidia-*`

*号代表你选择的型号，比如450,440

**1.3 检验安装结果**

上述安装完毕之后需要重启电脑：

`sudo reboot`

重启之后，我们可以直接在终端输入：

`nvidia-smi`

如果出现以下界面，那么恭喜你，驱动安装成功了：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/nvidia-smi.jpg"/></div>


### 2. 手动安装

**2.1 查询显卡型号**

首先就是按照自己的GPU版本去NVIDIA官网下载对应的驱动，如果不知道远程的电脑是什么型号的话，可以输入下面代码查询显卡的PCI设备号：

`lshw -numeric -C display`

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/sbh.jpg"/></div>

查到自己的版本号之后 [点击进入PCI对应显卡型号](http://pci-ids.ucw.cz/mods/PC/10de?action=help?help=pci) 查询，输入我上面画红圈的部分，就可以直接查询了：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/dybb.jpg"/></div>

我的是RTX 2080 Ti

**2.2 下载对应驱动**

[点击官网地址进入](https://www.nvidia.cn/Download/index.aspx?lang=cn)

点击进去之后就是这个界面：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/nvidiadownload.jpg"/></div>


接下来就是按照你的GPU版本来选择驱动程序，然后进行搜索，之后就会出来最适合你版本的驱动选项：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/nvidiapage.jpg"/></div>

接下来就是右键DOWNLOAD选项，点击复制链接地址。然后在你的远程服务器的连接页面输入：

```
sudo wget 你刚刚复制的链接地址（我的是https://www.nvidia.com/content/DriverDownload-March2009/confirmation.php?url=/XFree86/Linux-x86_64/460.32.03/NVIDIA-Linux-x86_64-460.32.03.run&lang=us&type=TITAN）
```

接着就可以看到你下载好的驱动软件了，软件的后缀是 *.run
（如果服务器下载很慢，或者无法下载，那么只有自己在本地下载之后传上服务器了）

**2.3 手动安装**

**如果之前安装过NVIDIA驱动，那么你最好先删除之前安装的驱动，并且重启一次，再进行下面的步骤：**
`sudo apt-get remove --purge nvidia-*(卸载驱动)`

接下来就是安装，安装前需要关闭系统原有的开源驱动`nouveau`：

`sudo vim /etc/modprobe.d/blacklist-nouveau.conf （编辑自带驱动的配置文件）`

在文件末尾添加：

```
blacklist nouveau
options nouveau modeset=0
```

然后更新一下用户环境：

`sudo update-initramfs -u`

重启：

`sudo reboot`

重启之后检查一下`nouveau`驱动是否已经关闭:

`lsmod | grep nouveau （没有任何输出就是已经关闭了）`

然后关闭图形界面（远程安装以保万一，可以也执行一遍,如果提示`Failed to stop lightdm.service`,那么说明你的远程电脑没有图形界面 ）：

`sudo service lightdm stop`

接下来就是激动人心的安装时刻，首先给run文件可执行权限：

`sudo chmod  a+x NVIDIA-Linux-x86_64-460.32.03.run`

然后就是安装：

`sudo ./NVIDIA-Linux-x86_64-460.32.03.run -no-x-check -no-nouveau-check -no-opengl-files`

下面是安装的时候的一些选项以及回答

`The distribution-provided pre-install script failed! Are you sure you want to continue? 选择 continue installation 继续`

`Would you like to register the kernel module souces with DKMS? This will allow DKMS to automatically build a new module, if you install a different kernel later? 选择 No 继续`

`Install NVIDIA's 32-bit compatibility libraries?选择 No 继续。`

`Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up. 选择 No 继续`

如果中间出现一些Warning信息，可以忽略。

接下来就是激动人心的时刻，输入：
`nvidia-smi`

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/nvidia-smi.jpg"/></div>

## 二、docker配置

首先，docker是什么，我就拿菜鸟教程里面的介绍来做引吧：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/docker.jpg"/></div>

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），我们用社区版就可以了。

[菜鸟教程原地址](https://www.runoob.com/docker/docker-tutorial.html)

简要的介绍就是：
docker就是一个非常适合linux系统的虚拟机，而这个docker虚拟机最适合安装linux系统！（就和绕口令一样）

### 1.安装docker：

**1.1 官方脚本自动安装**

没有curl的先安装curl：

`sudo apt-get install curl`

然后使用阿里云的镜像自动安装docker：

`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`

或者直接从daocloud安装：

`curl -sSL https://get.daocloud.io/docker | sh`

**1.2 手动安装**

因为我直接就自动安装上了docker所以就不作了（不再为了演示手动安装方法再卸载了来手动安装了）就直接介绍[菜鸟教程的安装方法](https://www.runoob.com/docker/ubuntu-docker-install.html)：

**卸载旧版本**
Docker 的旧版本被称为 `docker` , `docker.io` 或 `docker-engine` 。如果已安装，请卸载它们：

`sudo apt-get remove docker docker-engine docker.io containerd runc`

**使用 Docker 仓库进行安装**

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker ，更新 apt 包索引:

`sudo apt-get update`

安装 apt 依赖包，用于通过HTTPS来获取仓库:

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加 Docker 的官方 GPG 密钥：

没有安装curl的同样要先安装curl：`sudo apt-get install curl`

`curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`

9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 通过搜索指纹的后8个字符，验证您现在是否拥有带有指纹的密钥:

```
$ sudo apt-key fingerprint 0EBFCD88
   
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

使用以下指令设置稳定版仓库:

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"
```

**安装 Docker Engine-Community**

`sudo apt-get update`

安装最新版本的 Docker Engine-Community 和 containerd ，或者转到下一步安装特定版本:

`sudo apt-get install docker-ce docker-ce-cli containerd.io`

测试 Docker 是否安装成功，输入以下指令，打印出以下信息则安装成功:

`sudo docker run hello-world`

屏幕出现下面的结果就是安装成功！！！：

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

## 三、如何使用docker搭建多个CUDA的深度学习环境

首先修改一下docker容器的存储位置吧，多人使用的环境的话，最好还是自己设置一下，避免占用了别人的存储资源，其实我最好的建议还是大家拿一个公共存储地址作为docker的存储地址，这样大家都可以使用。

### 1. 修改docker存储位置（根据自身情况选择）
所以可以修改一下docker的存储位置，并且修改一下镜像拉取的地址：

`sudo vim /lib/systemd/system/docker.service`

`ExecStart=/usr/bin/dockerd`这一项后面添加：`--graph=你的新地址 --storage-driver=overlay --registry-mirror=https://jxus37ad.mirror.aliyuncs.com`

重新加载环境
`sudo systemctl daemon-reload`

重启docker
`sudo systemctl restart docker`

然后输入：

`sudo docker info`

你找到：` Docker Root Dir: （这里应该就是你的新地址）`

### 2. 把自己的用户加入docker用户组（让你的用户不用sudo命令也能正常使用）

如果没有docker用户组就创建一个：

`sudo groupadd docker`

把你的用户加入docker用户组：

`sudo gpasswd -a （你的用户名） docker `

更新用户组：

`sudo newgrp docker`

重启docker：

`sudo systemctl restart docker`

这时候你就会发现docker已经不需要使用sudo就可以运行了

### 3.拉取有CUDA的深度学习docker镜像（终于到了最后一步，也是最重要的一步）

**3.1 拉取镜像并运行**

[点击进入docker的官方镜像中心](https://hub.docker.com/search?q=&type=image)

进去之后可以自行搜索包含某些关键字的镜像，比如直接搜索cuda10.0：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/dockerhub.jpg"/></div>

你可以直接点击你想要拉取的那个镜像，然后在跳转出来的页面中，你可以直接点击下图的红色圆圈部分：

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/dockerimage.jpg"/></div>

然后把复制好的代码粘贴到终端，这样就可以直接拉取镜像了

这里我推荐几个我比较喜欢的镜像把，因为我比较懒，所以都是用的别人基本上全覆盖安装之后的版本，所以还是根据自身需要把，比如你要自己搭建一个深度学习的框架就可以直接使用上图这个比较纯净的docker镜像：

[cuda9.0-cudnn7.0-tf1.5-keras2.1-pytorch1.0](https://hub.docker.com/r/ianren/cuda9.0-cudnn7.0-tf1.5-keras2.1-pytorch1.0)

[cuda10.0_cudnn7_tfpytorch_py3_openpose_nvdoc](https://hub.docker.com/r/fangchen1988/cuda10_0_cudnn7_tfpytorch_py3_openpose_nvdoc)

[(cuda11.0)mlframeworks](https://hub.docker.com/r/andresfr/mlframeworks)


拉取完成之后输入：

`docker images`
就可以查看到刚刚你拉取的镜像，输入：

`docker run -it (镜像ID，ID可以只输入前面几位) /bin/bash`

这时候你就会发现，你已经进入了这个docker，输入`python3`,然后输入 `import torch` 成功了，是不是很开心，结果`import tensorflow`出现错误:

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/importtfw.jpg"/></div>

其实并不需要太担心，这是因为docker这个时候还调用不了nvidia的显卡，所以就无法调用GPU版本的tensorflow，因此只需要再安装一下nvidia-docker2就行。

**3.2 安装nvidia-docker2（这真的是最后一步了）**

[官方提供的安装方式](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

建立稳定的源仓库并且获得GPG key：

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

添加nvidia-container-runtime的源：

```
curl -s -L https://nvidia.github.io/nvidia-container-runtime/experimental/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
```

更新源：

`sudo apt-get update`

安装nvidia-docker2

`sudo apt-get install -y nvidia-docker2`

重启docker

`sudo systemctl restart docker`

重启之后使用一下代码就可以打开一个容器进入docker镜像了：

`docker run -it --gpus all （你的镜像ID） /bin/bash`

然后你发现：`nvidia-smi` 可用， `import tensorflow`可用！！！

<div align=center><img  src="https://cdn.jsdelivr.net/gh/unknown-kid/blog-images@main/2021-01-27/cgyx.jpg"/></div>


结语：所有的搭建过程也就结束了，码字教程不易啊，图文码字教程更不易，希望如果大家成功按照本方法搭建起环境，给个赞吧（如果要打赏我就再感谢不过了），谢谢~

**<font color='red'> 希望转载的朋友能够附加上来源，码字不易，盗版必追。 </font>**

最后送大家一些基本的docker操作吧：

`docker images`  查看本地的images镜像及其信息

`docker ps -a`  查看所有运行着和退出的容器及其信息

`docker rm (容器 ID)` 删除并退出指定的容器

`docker rmi (镜像 ID)` 删除指定的镜像

`docker run -it --gpus all 镜像ID /bin/bash` 打开一个新的容器，并进入镜像

`docker start (容器 ID)` 开始一个已经暂停的容器

`docker attach (容器 ID)` 进入一个正在运行的容器

`docker exec -it (容器 ID) /bin/bash` 容器多开
