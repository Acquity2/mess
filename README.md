# 使用ZeroTier One建立虚拟局域网优化艾尔登法环、黑暗之魂等p2p联机游戏的联机体验

## 零基础按步骤来也能完成的ZeroTier One局域网与moon节点的组建说明

### 介绍

> 可以联系我获取已搭建好的局域网的用户端进行游戏联机测试，让我帮忙搭建也行。

试想情景：圈内组织了一场比赛，参赛选手众多并且互相之间的网络状况都无法保证，比赛中途因为网络问题浪费了大量的时间。又或是一个交流群内的群友希望互相联机游玩，但是互相拉符时总是出现拉不动、平移、透明人等情况，导致群友间不能称心地进行切磋。

* Zerotier One是一款开源的用于构建虚拟局域网的工具，他的工作效果和蒲公英类似。

    一句话总结效果：让无法互相联机的人可以互相联机，同时最大程度降低玩家联机时的延迟。

    > 蒲公英效果参考：【艾尔登法环】联机失败？优化网络简单工具！https://www.bilibili.com/video/BV1tY4y1V7Xk。

* Zerotier One相对于蒲公英的优点：
    - 一个虚拟局域网内最大100台设备的容量。蒲公英免费账户只有3台。
    - 开源


* 蒲公英免费账户三台设备的限制让人非常头疼。对于比赛的情景来说，蒲公英免费版三人的限制带来的问题是每打一把都需要进入网页后台，踢出前一局选手，再让后续的选手加入虚拟局域网。
而群友联机的情况就更糟了，蒲公英按接入设备收费的规则让它几乎不可能实现。

* Zerotier One的问题就在于搭建困难，无法和蒲公英一样做到开箱即用。

* 本文的目标就是让零基础的朋友也能按步骤完成搭建。
    
    （或者直接来找我帮忙搭建）

    （我觉得大部分人应该不用看下去了）



### 原理

众所周知，魂系游戏p2p联机的质量实在是差的不行，这是由于国内复杂的网络结构，以及地理意义上遥远的服务器导致的高延迟等一系列因素所产生的。因此我们可以通过Zerotier One、蒲公英等工具先建立虚拟局域网，之后游戏就会自动利用这个虚拟局域网进行联机。

* Zerotier One在建立虚拟局域网时，会尝试让连接到网络内的设备进行p2p连接组网。对于无法进行p2p连接的设备，若存在用户自建的MOON节点，就会优先让设备通过MOON节点的中转再与其他设备通信。
    
    蒲公英的原理也类似。

    > 从游戏的角度来看，就是让本来无法连通的两个用户，通过国内BGP机房内的服务器中转之后相连了，又因为服务器的优质线路，双方的延迟也会降低到接近最佳的程度。
    
    > 使用uu加速器之类软件无法避免的情况就是游戏流量先需要到加速器的出口节点（一般是香港出口）绕一圈，从而增加了无谓的延迟和丢包。

### 一些说明

* 名称解释
    - Zerotier One定义了几个名词:

    - PLANET 行星服务器，zerotier各地的根服务器，有日本、新加坡等地。
    
    - MOON 卫星服务器，用户自建的私有根服务器，起到中转加速的作用。
    
    - LEAF 枝叶，就是每台连接到该网络的设备。

* 后文分为三部分，第一部分是ZT控制端以及MOON节点的搭建，第二部分是ZeroTier One用户端的配置，第三部分提供了一个用户端简单配置批处理文件。

### 第一部分

* 首先需要准备一台服务器，个人推荐腾讯云国内的轻量应用服务器，淘宝上面一台2核4g8m带宽的服务器价格大约是1年50到100元人民币不等。

* 准备好服务器之后，将服务器系统重新安装成Debian11，本文所有操作都是基于Debian11系统的。

    图

* 重装完系统后，进入防火墙界面将所有tcp和udp端口开放

    图

* 使用finalshell等ssh终端设备连接至服务器，按顺序运行以下命令安装必须程序

    ```
        $ apt update
        $ apt install curl wget nano -y
    ```

* 安装docker
    ```
        $ curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
        $ systemctl enable docker 
        $ systemctl start docker
    ```

* 安装ZTCUI控制端

    ```
    $ docker run --restart=on-failure:3 -d --name ztncui -e HTTP_PORT=4000 -e HTTP_ALL_INTERFACES=yes -e ZTNCUI_PASSWD=mrdoc.fun -p 4000:4000 keynetworks/ztncui
    ```

* 安装Zerotier One

    ```
    $ curl -s https://install.zerotier.com | sudo bash
    ```

* 配置ZTCUI

    进入ZTCUI后台

    > 浏览器打开 http://[你服务器的ip]:4000/
    
    使用 用户名:admin 密码:mrdoc.fun 登陆后台

    > 首次登陆后台会自动让你修改登陆密码，务必将其保存好

    点进Add network选项卡，随便起个名字

    图

    点击easy setup

    图

    点击Generate network address自动生成网段配置，再点击submit提交

    > 可选，点击private可以配置网络公开或私有，默认为私有。私有网络，成员加入时需要管理员进入network界面手动批准，公开网络则无需批准自动入网。

* 制作moon节点

    - 运行命令

        ```
        $ zerotier-cli join [你的网络id]
        ```
    - 进入ZeroTier的默认安装目录，生成Moon配置文件：
        ```
        $ cd /var/lib/zerotier-one
        $ zerotier-idtool initmoon identity.public > moon.json
        $ nano moon.json
        ```
    - 生成的文件样式如下：
        ```
         "id": "1c110b9ac2",
         "objtype": "world",
         "roots": [
          {
           "identity": "9c960b9ac2:0:daca38dfc5f3",
          "stableEndpoints": []
          }
         ],
         "signingKey": "676f0c29eb8d6f2f00ce22ee2082b3ec",
         "signingKey_SECRET": "39de9f7ab16d0adb035276b7281f73344",
         "updatesMustBeSignedBy": "676f0c29eb8d6f2f00ce22ee",
         "worldType": "moon"
         }
        ```
    - 这里我们需要根据自己服务器的公网静态IP，修改stableEndpoints
    
        那一行格式如下，其中11.22.33.44为你的公网IP，9993是默认的端口号：

        ```
        "stableEndpoints": [ "11.22.33.44/9993" ]
        ```
    - 生成.moon签名文件
        ```
        $ zerotier-idtool genmoon moon.json
        ```
      > 执行该命令以后会在软件目录下生成一个类似000000xxxxxxxxx.moon的文件。
    - 移动.moon签名文件到moons.d目录下并且重启服务
        ```
        $ cd /var/lib/zerotier-one
        $ mkdir moons.d
        $ mv 000000*.moon moons.d
        $ service zerotier-one restart
        ```

### 第二部分

- 下载安装windows Zerotier One客户端

    zerotier官网 https://www.zerotier.com/download/

    点击windows图标下载并安装客户端

- 管理员模式运行CMD，运行以下命令
    ```
    zerotier-cli join [你的网络id]
    zerotier-cli orbit [MOON的id] [MOON的id]
    ```
- 打开开始菜单输入'服务'，打开服务，找到Zerotier One服务，点击重启动
    
    图

- 管理员模式运行CMD，运行以下命令

    ```
    zerotier-cli peers
    ```

    > 如果输出中出现一条最后为MOON的记录，说明已经成功连接Moon服务器

        zerotier-cli listpeers
        200 listpeers <ztaddr> <path> <latency> <version> <role>
        200 listpeers id myip/9993;6012;1706 -1 1.8.4 MOON
***完成***

### 第三部分

- 用户端简单配置批处理文件
    > 用于替代第二部分，一键运行，一部分需要修改

        Rd "%WinDir%\system32\test_permissions" >NUL 2>NUL
        Md "%WinDir%\System32\test_permissions" 2>NUL||(Echo 请使用管理员运行&&Pause >nul&&Exit)
        Rd "%WinDir%\System32\test_permissions" 2>NUL

        SC STOP vkdpi > NUL 2>&1
        SC STOP uuwfp > NUL 2>&1
        SC STOP uupacket > NUL 2>&1
        SC STOP LonlifeFD > NUL 2>&1
        SC STOP lgdcatcher > NUL 2>&1
        SC STOP netfilter2 > NUL 2>&1
        SC STOP qmproxyacc > NUL 2>&1
        SC STOP qiyoudaemon > NUL 2>&1
        SC STOP qeeyoupacket > NUL 2>&1
        SC STOP heyboxfilter > NUL 2>&1

        SC DELETE vkdpi > NUL 2>&1
        SC DELETE uuwfp > NUL 2>&1
        SC DELETE uupacket > NUL 2>&1
        SC DELETE LonlifeFD > NUL 2>&1
        SC DELETE lgdcatcher > NUL 2>&1
        SC DELETE netfilter2 > NUL 2>&1
        SC DELETE qmproxyacc > NUL 2>&1
        SC DELETE qiyoudaemon > NUL 2>&1
        SC DELETE qeeyoupacket > NUL 2>&1
        SC DELETE heyboxfilter > NUL 2>&1

        ipconfig /flushdns > NUL 2>&1
        netsh firewall set icmpsetting 8 > NUL 2>&1
        netsh winsock reset > NUL 2>&1
        netsh winsock reset catalog > NUL 2>&1
        netsh winsock set autotuning on > NUL 2>&1
        :: netsh advfirewall firewall delete rule all > NUL 2>&1
        :: netsh advfirewall set allprofiles state off > NUL 2>&1
        netsh interface tcp set global ecn=enabled > NUL 2>&1
        netsh interface tcp set global timestamps=enabled > NUL 2>&1
        netsh interface tcp set global ecncapability=enabled > NUL 2>&1
        netsh interface ipv4 set dynamicport tcp start=10000 num=55535 > NUL 2>&1
        netsh interface ipv4 set dynamicport udp start=10000 num=55535 > NUL 2>&1
        netsh interface ipv6 set dynamicport tcp start=10000 num=55535 > NUL 2>&1
        netsh interface ipv6 set dynamicport udp start=10000 num=55535 > NUL 2>&1

        cd "C:\Program Files (x86)\ZeroTier\One"

        call zerotier-cli.bat deorbit [MOON的id]
        call zerotier-cli.bat leave [你的网络id]

        call zerotier-cli.bat orbit [MOON的id] [MOON的id]
        call zerotier-cli.bat orbit [MOON的id] [MOON的id]
        call zerotier-cli.bat orbit [MOON的id] [MOON的id]
        call zerotier-cli.bat orbit [MOON的id] [MOON的id]
        call zerotier-cli.bat join [你的网络id]

        sc stop ZeroTierOneService
        sc start ZeroTierOneService

        call zerotier-cli.bat peers

        @echo Done!
