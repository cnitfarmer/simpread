> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/kuangdaoyizhimei/p/14175143.html)

本篇文章会介绍 win10 中 wsl2 的安装和使用以及遇到的常见问题比如如何固定 wsl2 地址等问题的总结。

一、wsl2 简介
---------

wsl 是适用于 Linux 的 Windows 子系统，安装指南：[适用于 Linux 的 Windows 子系统安装指南 (Windows 10)](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)

简单来说，以前想在 windows 中使用 linux，需要安装 vmware 虚拟机，现在则不比这么麻烦了，直接安装 linux 子系统，秒开。

二、使用 wsl2
---------

按照官方文档安装好 wsl2 之后，再顺便安装下 [Windows 终端](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#install-windows-terminal-optional)，一起使用，效果更佳。

安装好 wsl2 后，直接在菜单中找到对应的 wsl 终端直接打开即可，第一次用的时候让你初始化一个用户名和密码，根据提示几秒钟即可初始化完成。

安装好之后就可以愉快的玩耍了，貌似一切都 ok。。。慢着，用久了，你会发现一些问题：

### 1. 安装软件太慢了

比如我使用的 ubuntu20，安装和更新软件都特别慢，因为毕竟国内，这时候就要使用国内镜像进行加速 [[0]](https://www.cnblogs.com/zqifa/p/12910989.html)。

第一步：备份源文件

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

第二步：编辑 / etc/apt/sources.list 文件

在文件最前面添加以下条目，之后保存即可生效 (以阿里云镜像为例，操作前请做好相应备份)：

```
vi /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

### 2. wsl2 地址每次重新开机之后都会发生变化

一般来说这不是什么大问题，但是别忘了我们要 wsl 是干啥的，我们总是希望能够在 windows 中访问 wsl 中的一些服务，比如安装的 mysql、redis 等，如果 wsl 的 ip 地址总是变化，岂不是每次开机都要在 windows 中手动设置一次 ip 地址 [[1]](https://blog.csdn.net/manbu_cy/article/details/108476859)？固定 ip 地址的方法比较简单，直接运行以下脚本即可，我这里安装了 docker，有些小伙伴没安装 docker 则需要修改下脚本才行。

```
@echo off
setlocal enabledelayedexpansion

wsl -u root service docker start | findstr "Starting Docker" > nul
if !errorlevel! equ 0 (
    echo docker start success
    :: set wsl2 ip
    wsl -u root ip addr | findstr "192.168.169.2" > nul
    if !errorlevel! equ 0 (
        echo wsl ip has set
    ) else (
        wsl -u root ip addr add 192.168.169.2/28 broadcast 192.168.169.15 dev eth0 label eth0:1
        echo set wsl ip success: 192.168.169.2
    )

    :: set windows ip
    ipconfig | findstr "192.168.169.1" > nul
    if !errorlevel! equ 0 (
        echo windows ip has set
    ) else (
        netsh interface ip add address "vEthernet (WSL)" 192.168.169.1 255.255.255.240
        echo set windows ip success: 192.168.169.1
    )
)
pause
```

将它保存到文件，比如`划分虚拟局域网&启动 docker.bat`，然后将其放到 windows 启动目录下 [[2]](https://www.cnblogs.com/eliteboy/p/7838091.html)：

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```

之后，每次电脑启动之后就会自动执行该脚本了

### 3.windows 本地 ssh 登陆 wsl2

首先，wsl 中已经安装的 ssh 是不完整的或者有问题的，所以无论你怎么改都不会生效，甚至 ssh 服务都无法正常启动，正确的做法是先卸载 ssh 再重新安装。

```
sudo apt-get purge openssh-server  # purge 是卸载并删除配置文件
sudo apt-get install openssh-server
```

然后修改配置文件 / etc/ssh/sshd_config，新增如下配置

```
PermitRootLogin yes
```

修改 `PasswordAuthentication` 配置项为 yes，修改 `Port` 端口号为 `2222`（22 端口比较特殊，windows 可能会使用到）

使用命令 `service sshd restart` 重启 ssh 服务

**做完以上修改之后，解决了第一个问题，即如何在 windows 使用工具 ssh 连接 wsl2，** 接下来要做的事情是如何在局域网中远程登陆 wsl2。

### 4. 局域网远程登陆 wsl2

> 首先分析下为啥局域网其他机器无法连接 wsl2
> 
> *   第一个原因，windows 防火墙没关闭或者没有设置入站规则
> *   第二个原因，也是最本质的原因, wsl2 的地址是虚拟地址，并非是局域网中的物理地址

那怎么解决呢？

1.  关闭防火墙或者设置入站规则  
    其中设置入站规则是推荐的方式，网上教程较多，不赘述。
2.  设置端口转发，让 windows 转发来自特定端口的请求到 wsl2  
    设置端口转发的方法如下：

```
interface portproxy add v4tov4 listenport=【宿主机windows平台监听端口】 listenaddress=0.0.0.0 connectport=【wsl2平台监听端口】 connectaddress=【wsl2平台ip】
```

比如，我这里使用如下命令配置了 win10 IpV4 协议端口号 2222 转发到地址为 192.168.169.2 的 wsl 端口号 2222

```
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=2222 connectaddress=192.168.169.2
```

3.  查看端口转发

```
netsh interface portproxy show all
```

4.  删除端口转发

```
netsh interface portproxy delete v4tov4 listenport=9696 listenaddress=0.0.0.0
```

完成以上四步设置，即可在局域网使用 securityCRT 工具或者 putty 远程连接 wsl 了。

三、在 wsl 中使用 docker
------------------

### 1. 安装 docker

正常来说，应当上 docker 官网按照安装文档来安装，但是你会发现及时你更新了源，安装速度仍然特别慢，高速打开方式 [[3]](https://blog.csdn.net/manbu_cy/article/details/108476859) 为

```
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -


sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"


    sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### 2. 安装 docker-compose

同理，正常来说，docker-compose 的安装方式应该遵循官方网站的指导 [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)，但是会非常慢，还是要另辟蹊径 [[4]](https://www.jianshu.com/p/1fd03bf4998d)

```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

最后，欢迎关注我的私有博客~ [https://blog.kdyzm.cn/](https://blog.kdyzm.cn/)