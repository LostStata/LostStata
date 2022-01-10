### 容器化技术Docker

![](https://i.loli.net/2019/06/09/5cfd1e94f1fb169708.jpg)

 **Docker介绍**
官网：
 docker.io
  docker.com
    公司名称：原名dotCloud  14年改名为docker
    容器产品：docker 16年已经被更名为**Moby**

​    docker-hub
​        docker.io

#### docker容器历史

和虚拟机一样，容器技术也是一种资源隔离的虚拟化技术。我们追溯它的历史，会发现它的技术雏形早已有之。

**容器简史**

容器概念始于 1979 年提出的 UNIX chroot，它是一个 UNIX 操作系统的系统调用，将一个进程及其子进程的根目录改变到文件系统中的一个新位置，让这些进程只能访问到这个新的位置，从而达到了进程隔离的目的。

2000 年的时候 FreeBSD 开发了一个类似于 chroot 的容器技术 Jails，这是最早期，也是功能最多的容器技术。Jails 英译过来是监狱的意思，这个“监狱”（用沙盒更为准确）包含了文件系统、用户、网络、进程等的隔离。

2001 Linux 也发布自己的容器技术 Linux VServer，2004 Solaris 也发布了 Solaris Containers，两者都将资源进行划分，形成一个个 zones，又叫做虚拟服务器。

2005 年推出 OpenVZ，它通过对 Linux 内核进行补丁来提供虚拟化的支持，每个 OpenVZ 容器完整支持了文件系统、用户及用户组、进程、网络、设备和 IPC 对象的隔离。

2007 年 Google 实现了 Control Groups( cgroups )，并加入到 Linux 内核中，这是划时代的，为后期容器的资源配额提供了技术保障。

2008 年基于 cgroups 和 linux namespace 推出了第一个最为完善的 Linux 容器 LXC。

2013 年推出到现在为止最为流行和使用最广泛的容器 Docker，相比其他早期的容器技术，Docker 引入了一整套容器管理的生态系统，包括分层的镜像模型，容器注册库，友好的 Rest API。

2014 年 CoreOS 也推出了一个类似于 Docker 的容器 Rocket，CoreOS 一个更加轻量级的 Linux 操作系统，在安全性上比 Docker 更严格。

2016 年微软也在 Windows 上提供了容器的支持，Docker 可以以原生方式运行在 Windows 上，而不是需要使用 Linux 虚拟机。

基本上到这个时间节点，容器技术就已经很成熟了，再往后就是容器云的发展，由此也衍生出多种容器云的平台管理技术，其中以 kubernetes 最为出众，有了这样一些细粒度的容器集群管理技术，也为微服务的发展奠定了基石。因此，对于未来来说，应用的微服务化是一个较大的趋势。

**为什么需要容器**

其一，这是技术演进的一种创新结果，其二，这是人们追求高效生产活动的一种工具。

随着软件开发的发展，相比于早期的集中式应用部署方式，现在的应用基本都是采用分布式的部署方式，一个应用可能包含多种服务或多个模块，因此多种服务可能部署在多种环境中，如虚拟服务器、公有云、私有云等，由于多种服务之间存在一些依赖关系，所以可能存在应用在运行过程中的动态迁移问题，那这时如何保证不同服务在不同环境中都能平滑的适配，不需要根据环境的不同而去进行相应的定制，就显得尤为重要。

就像货物的运输问题一样，如何将不同的货物放在不同的运输机器上，减少因货物的不同而频繁进行货物的装载和卸载，浪费大量的人力物力。

为此人们发明了集装箱，将货物根据尺寸形状等的不同，用不同规格的集装箱装载，然后再放到运输机上运输，由于集装箱密封，只有货物到达目的地才需拆封，在运输过程能够再不同运输机上平滑过渡，所以避免了资源的浪费。

因此集装箱被誉为是运输业与世界贸易最重要的发明。

Docker 容器的思想就是采用集装箱思想，为应用提供了一个基于容器的标准化运输系统。Docker 可以将任何应用及其依赖打包成一个轻量级、可移植、自包含的容器。容器可以运行在几乎所有的操作系统上。这样容器就可以跑在任何环境中，因此才有了那句话：

Build Once, Run Anywhere

这种集装箱的思想我们也能从 Docker 的 Logo 中看出来，这不就是一堆集装箱吗？

#### Docker对paas的降维打击(了解)

```
IaaS infrastructure as a service 基础设施及服务
PaaS platform as a service
SaaS software as a service
dSaaS data storage as a service 
CaaS  container as a service

PaaS 项目成功的主要原因
     是它提供了一种名叫"应用托管"的能力。 
     paas之前主流用户的普遍用法是租一批 AWS 或者 OpenStack 的虚拟机，然后像以前管理物理服务器那样，用脚本或者手工的方式在这些机器上部署应用。

     这个部署过程会碰到云端虚拟机和本地环境不一致的问题，所以当时的云计算服务，比的就是谁能更好地模拟本地服务器环境，能带来更好的"上云"体验。而 PaaS 开源项目的出现，就是当时解决这个问题的一个最佳方案。

PaaS 如何部署应用
    虚拟机创建好之后，运维人员只需要在这些机器上部署一个 Cloud Foundry 项目，然后开发者只要执行一条命令就能把本地的应用部署到云上，这条命令就是：
    # cf push " 应用 "

PaaS 项目的核心组件
    像 Cloud Foundry 这样的 PaaS 项目，最核心的组件就是一套应用的打包和分发机制。 Cloud Foundry 为每种主流编程语言都定义了一种打包格式，而"cf push"的作用，基本上等同于用户把应用的可执行文件和启动脚本打进一个压缩包内，上传到云上 Cloud Foundry 的存储中。接着，Cloud Foundry 会通过调度器选择一个可以运行这个应用的虚拟机，然后通知这个机器上的 Agent 把应用压缩包下载下来启动。

    由于需要在一个虚拟机上启动很多个来自不同用户的应用，Cloud Foundry 会调用操作系统的 Cgroups 和 Namespace 机制为每一个应用单独创建一个称作"沙盒"的隔离环境，然后在"沙盒"中启动这些应用进程。这就实现了把多个用户的应用互不干涉地在虚拟机里批量自动地运行起来的目的。
    这正是 PaaS 项目最核心的能力。 而这些 Cloud Foundry 用来运行应用的隔离环境，或者说"沙盒"，就是所谓的"容器"。

注：
    Cloud Foundry是当时非常主流非常火的一个PaaS项目
    
    
    
 Docker 镜像
    Docker 项目确实与 Cloud Foundry 的容器在大部分功能和实现原理上都是一样的，可偏偏就是这剩下的一小部分不一样的功能，成了 Docker 项目接下来"呼风唤雨"的不二法宝。这个功能，就是 Docker 镜像。

    恐怕连 Docker 项目的作者 Solomon Hykes 自己当时都没想到，这个小小的创新，在短短几年内就如此迅速地改变了整个云计算领域的发展历程。

PaaS的问题：
    PaaS 之所以能够帮助用户大规模部署应用到集群里，是因为它提供了一套应用打包的功能。可就是这个打包功能，却成了 PaaS 日后不断遭到用户诟病的一个"软肋"。

    根本原因：
        一旦用上了 PaaS，用户就必须为每种语言、每种框架，甚至每个版本的应用维护一个打好的包。这个打包过程，没有任何章法可循，更麻烦的是，明明在本地运行得好好的应用，却需要做很多修改和配置工作才能在 PaaS 里运行起来。而这些修改和配置，并没有什么经验可以借鉴，基本上得靠不断试错，直到你摸清楚了本地应用和远端 PaaS 匹配的"脾气"才能够搞定。

    最后结局是，"cf push"确实是能一键部署了，但是为了实现这个一键部署，用户为每个应用打包的工作可谓一波三折，费尽心机。

    而Docker 镜像解决的，恰恰就是打包这个根本性的问题。 

Docker 镜像的精髓
    所谓 Docker 镜像，其实就是一个压缩包。但是这个压缩包里的内容，比 PaaS 的应用可执行文件 + 启停脚本的组合就要丰富多了。实际上，大多数 Docker 镜像是直接由一个完整操作系统的所有文件和目录构成的，所以这个压缩包里的内容跟你本地开发和测试环境用的操作系统是完全一样的。

这就有意思了：假设你的应用在本地运行时，能看见的环境是 CentOS 7.2 操作系统的所有文件和目录，那么只要用 CentOS 7.2 的 ISO 做一个压缩包，再把你的应用可执行文件也压缩进去，那么无论在哪里解压这个压缩包，都可以得到与你本地测试时一样的环境。当然，你的应用也在里面！

这就是 Docker 镜像最厉害的地方：只要有这个压缩包在手，你就可以使用某种技术创建一个"沙盒"，在"沙盒"中解压这个压缩包，然后就可以运行你的程序了。

更重要的是，这个压缩包包含了完整的操作系统文件和目录，也就是包含了这个应用运行所需要的所有依赖，所以你可以先用这个压缩包在本地进行开发和测试，完成之后，再把这个压缩包上传到云端运行。

在这个过程中，你完全不需要进行任何配置或者修改，因为这个压缩包赋予了你一种极其宝贵的能力：本地环境和云端环境的高度一致！

这，正是 Docker 镜像的精髓。

那么，有了 Docker 镜像这个利器，PaaS 里最核心的打包系统一下子就没了用武之地，最让用户抓狂的打包过程也随之消失了。相比之下，在当今的互联网里，Docker 镜像需要的操作系统文件和目录，可谓唾手可得。

所以，你只需要提供一个下载好的操作系统文件与目录，然后使用它制作一个压缩包即可，这个命令就是：
# docker build " 镜像 "

镜像制作完成，用户就可以让 Docker 创建一个"沙盒"来解压这个镜像，然后在"沙盒"中运行自己的应用，这个命令就是：
# docker run " 镜像 "

Docker 项目给 PaaS 世界带来的"降维打击"
    其实是提供了一种非常便利的打包机制。这种机制直接打包了应用运行所需要的整个操作系统，从而保证了本地环境和云端环境的高度一致，避免了用户通过"试错"来匹配两种不同运行环境之间差异的痛苦过程。
```

#### 容器和虚拟机的区别

```
容器和 VM 的主要区别：
容器提供了基于进程的隔离，而虚拟机提供了资源的完全隔离。虚拟机可能需要一分钟来启动，而容器只需要一秒钟或更短。容器使用宿主操作系统的内核，而虚拟机使用独立的内核。Docker 的局限性之一是，它只能用在 64 位的操作系统上。

Docker对服务器端开发/部署带来的变化：
实现更轻量级的虚拟化，方便快速部署
对于部署来说可以极大的减少部署的时间成本和人力成本
Docker支持将应用打包进一个可以移植的容器中，重新定义了应用开发，测试，部署上线的过程，核心理念就
是 Build once, Run anywhere
1）标准化应用发布，docker容器包含了运行环境和可执行程序，可以跨平台和主机使用；
2）节约时间，快速部署和启动，VM启动一般是分钟级，docker容器启动是秒级；
3）方便构建基于SOA架构或微服务架构的系统，通过服务编排，更好的松耦合；
4）节约成本，以前一个虚拟机至少需要几个G的磁盘空间，docker容器可以减少到MB级；
5）方便持续集成，通过与代码进行关联使持续集成非常方便；
6）可以作为集群系统的轻量主机或节点，在IaaS平台上，已经出现了CaaS，通过容器替代原来的主机。

Docker 优势：
1、交付物标准化
Docker是软件工程领域的"标准化"交付组件，最恰到好处的类比是"集装箱"。
集装箱将零散、不易搬运的大量物品封装成一个整体，集装箱更重要的意义在于它提供了一种通用的封装货物的
标准，卡车、火车、货轮、桥吊等运输或搬运工具采用此标准，隧道、桥梁等也采用此标准。以集装箱为中心的
标准化设计大大提高了物流体系的运行效率。
传统的软件交付物包括：应用程序、依赖软件安装包、配置说明文档、安装文档、上线文档等非标准化组件。
Docker的标准化交付物称为"镜像"，它包含了应用程序及其所依赖的运行环境，大大简化了应用交付的模式。

2、一次构建，多次交付
类似于集装箱的"一次装箱，多次运输"，Docker镜像可以做到"一次构建，多次交付"。当涉及到应用程序多副本
部署或者应用程序迁移时，更能体现Docker的价值。

3、应用隔离
集装箱可以有效做到货物之间的隔离，使化学物品和食品可以堆砌在一起运输。Docker可以隔离不同应用程序
之间的相互影响，但是比虚拟机开销更小。
总之，容器技术部署速度快，开发、测试更敏捷；提高系统利用率，降低资源成本。
﻿
Docker的度量：
Docker是利用容器来实现的一种轻量级的虚拟技术，从而在保证隔离性的同时达到节省资源的目的。Docker的
可移植性可以让它一次建立，到处运行。Docker的度量可以从以下四个方面进行：
1）隔离性
 Docker采用libcontainer作为默认容器，代替了以前的LXC。libcontainer的隔离性主要是通过内核的命名空
 间来实现 的，有pid、net、ipc、mnt、uts命令空间，将容器的进程、网络、消息、文件系统和主机名进行隔
 离。
2）可度量性
 Docker主要通过cgroups控制组来控制资源的度量和分配。
3）移植性
 Docker利用AUFS来实现对容器的快速更新。
 AUFS是一种支持将不同目录挂载到同一个虚拟文件系统下的文件系统，支持对每个目录的读写权限管理。AUFS具有层
 的概念，每一次修改都是在已有的只写层进行增量修改，修改的内容将形成新的文件层，不影响原有的层。
4）安全性
 安全性可以分为容器内部之间的安全性；容器与托管主机之间的安全性。
 容器内部之间的安全性主要是通过命名空间和cgroups来保证的。
 容器与托管主机之间的安全性主要是通过内核能力机制的控制，可以防止Docker非法入侵托管主机。

Docker容器使用AUFS作为文件系统，有如下优势：
1）节省存储空间
 多个容器可以共享同一个基础镜像存储。
2）快速部署
 如果部署多个来自同一个基础镜像的容器时，可以避免多次复制操作。
3）升级方便
 升级一个基础镜像即可影响到所有基于它的容器。
4）增量修改
 可以在不改变基础镜像的同时修改其目录的文件，所有的更高都发生在最上层的写操作层，增加了基础镜像的可共
 享内 容。

```



#### 容器应用场景

```
### 1. 简化配置

这是Docker公司宣传的Docker的主要使用场景。虚拟机的最大好处是能在你的硬件设施上运行各种配置不一样的平台（软件、系统），Docker在降低额外开销的情况下提供了同样的功能。它能让你将运行环境和配置放在代码中然后部署，同一个Docker的配置可以在不同的环境中使用，这样就降低了硬件要求和应用环境之间耦合度。

### 2. 代码流水线（Code Pipeline）管理

前一个场景对于管理代码的流水线起到了很大的帮助。代码从开发者的机器到最终在生产环境上的部署，需要经过很多的中间环境。而每一个中间环境都有自己微小的差别，Docker给应用提供了一个从开发到上线均一致的环境，让代码的流水线变得简单不少。

### 3. 提高开发效率

这就带来了一些额外的好处：Docker能提升开发者的开发效率。如果你想看一个详细一点的例子，可以参考Aater在

DevOpsDays Austin 2014 

大会或者是DockerCon上的演讲。

不同的开发环境中，我们都想把两件事做好。一是我们想让开发环境尽量贴近生产环境，二是我们想快速搭建开发环境。

理想状态中，要达到第一个目标，我们需要将每一个服务都跑在独立的虚拟机中以便监控生产环境中服务的运行状态。然而，我们却不想每次都需要网络连接，每次重新编译的时候远程连接上去特别麻烦。这就是Docker做的特别好的地方，开发环境的机器通常内存比较小，之前使用虚拟的时候，我们经常需要为开发环境的机器加内存，而现在Docker可以轻易的让几十个服务在Docker中跑起来。

### 4. 隔离应用

有很多种原因会让你选择在一个机器上运行不同的应用，比如之前提到的提高开发效率的场景等。

我们经常需要考虑两点，一是因为要降低成本而进行服务器整合，二是将一个整体式的应用拆分成松耦合的单个服务（译者注：微服务架构）。如果你想了解为什么松耦合的应用这么重要，请参考Steve Yege的这篇论文，文中将Google和亚马逊做了比较。

### 5. 整合服务器

正如通过虚拟机来整合多个应用，Docker隔离应用的能力使得Docker可以整合多个服务器以降低成本。由于没有多个操作系统的内存占用，以及能在多个实例之间共享没有使用的内存，Docker可以比虚拟机提供更好的服务器整合解决方案。

### 6. 调试能力

Docker提供了很多的工具，这些工具不一定只是针对容器，但是却适用于容器。它们提供了很多的功能，包括可以为容器设置检查点、设置版本和查看两个容器之间的差别，这些特性可以帮助调试Bug。你可以在《Docker拯救世界》的文章中找到这一点的例证。

### 7. 多租户环境

另外一个Docker有意思的使用场景是在多租户的应用中，它可以避免关键应用的重写。我们一个特别的关于这个场景的例子是为IoT（译者注：物联网）的应用开发一个快速、易用的多租户环境。这种多租户的基本代码非常复杂，很难处理，重新规划这样一个应用不但消耗时间，也浪费金钱。

使用Docker，可以为每一个租户的应用层的多个实例创建隔离的环境，这不仅简单而且成本低廉，当然这一切得益于Docker环境的启动速度和其高效的命令。

### 8. 快速部署

在虚拟机之前，引入新的硬件资源需要消耗几天的时间。虚拟化技术（Virtualization）将这个时间缩短到了分钟级别。而Docker通过为进程仅仅创建一个容器而无需启动一个操作系统，再次将这个过程缩短到了秒级。这正是Google和Facebook都看重的特性。
```

#### Docker容器基本概念

```
Docker系统
Docker系统有两个程序：docker服务端和docker客户端
    docker服务端：
        是一个服务进程，管理着所有的容器。

    docker客户端：
        扮演着docker服务端的远程控制器，可以用来控制docker的服务端进程。

Docker三大核心组件：
    Docker 镜像 - Docker  images 
    Docker 仓库 - Docker  registeries
    Docker 容器 - Docker  containers 

docker 仓库：
    用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。
公有的 Docker  仓库名字是 Docker Hub。Docker Hub  提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。 

Docker 镜像 
    Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成。Docker 使用  UnionFS 来将这些层联合到单独的镜像中。UnionFS  允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker  是如此的轻量。当你改变了一个 Docker  镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。 

    在 Docker 的术语里，一个只读层被称为镜像，一个镜像是永久不会变的。
    由于 Docker 使用一个统一文件系统，Docker 进程认为整个文件系统是以读写方式挂载的。 但是所有的变更都发生顶层的可写层，而下层的原始的只读镜像文件并未变化。由于镜像不可写，所以镜像是无状态的。
每一个镜像都可能依赖于由一个或多个下层的组成的另一个镜像。下层那个镜像是上层镜像的父镜像。

镜像名字：
    registry/repo:tag
    daocloud.io/library/centos:7

基础镜像：
一个没有任何父镜像的镜像，谓之基础镜像。

镜像ID：
所有镜像都是通过一个 64 位十六进制字符串 （内部是一个 256 bit 的值）来标识的。 为简化使用，前 12 个字符可以组成一个短ID，可以在命令行中使用。短ID还是有一定的碰撞机率，所以服务器总是返回长ID。
    
Docker 容器
    Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker  镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是  Docker 的运行部分。 
```



#### docker安装

##### 自带源安装

```
CentOS 7 中 Docker 的安装:
Docker 软件包已经包括在默认的 CentOS-Extras 软件源(联网使用centos7u2自带网络Yum源)里。因此想要安装 docker，只需要运行下面的 yum 命令：       
# yum install docker
    
启动 Docker 服务:
    # service docker start
    # chkconfig docker on

    CentOS 7    
    # systemctl start docker.service
    # systemctl enable docker.service

确定docker服务在运行：
结果会显示服务端和客户端的版本，如果只显示客户端版本说明服务没有启动
    [root@docker1 yum]# docker version
Client: Docker Engine - Community
 Version:           19.03.4
 API version:       1.40
 Go version:        go1.12.10
 Git commit:        9013bf583a
 Built:             Fri Oct 18 15:52:22 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.4
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.10
  Git commit:       9013bf583a
  Built:            Fri Oct 18 15:50:54 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

﻿查看docker基本信息：
    #docker info

校验Docker的安装
[root@master ~]# docker run -it ubuntu bash
Unable to find image 'ubuntu:latest' locally
Trying to pull repository daocloud.io/ubuntu ... 
latest: Pulling from daocloud.io/ubuntu
22ecafbbcc4a: Pull complete 
580435e0a086: Pull complete 
Digest: sha256:80c2902178d79f439b13c5a244f3b1ef67ca890dbbe58d19caa13301ca56a505

如果自动进入下面的容器环境，说明﻿ubuntu镜像运行成功，Docker的安装也没有问题：可以操作容器了
root@50a0449d7729:/# pwd     
/
root@50a0449d7729:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```

##### docker版本和官方源安装

```
moby、docker-ce与docker-ee
最早时docker是一个开源项目，主要由docker公司维护。
2017年3月1日起，docker公司将原先的docker项目改名为moby，并创建了docker-ce和docker-ee。

三者关系：
    moby是继承了原先的docker的项目，是社区维护的的开源项目，谁都可以在moby的基础打造自己的容器产品
    docker-ce是docker公司维护的开源项目，是一个基于moby项目的免费的容器产品
    docker-ee是docker公司维护的闭源产品，是docker公司的商业产品。

    moby project由社区维护，docker-ce project是docker公司维护，docker-ee是闭源的。

    要使用免费的docker，从https://github.com/docker/docker-ce上获取。
    要使用收费的docker，从https://www.docker.com/products/docker-enterprise上获取。

docker-ce的发布计划
    v1.13.1之后，发布计划更改为:
    Edge:   月版本，每月发布一次，命名格式为YY.MM，维护到下个月的版本发布
    Stable: 季度版本，每季度发布一次，命名格式为YY.MM，维护4个月

安装：
    docker-ce的release计划跟随moby的release计划，可以使用下面的命令直接安装最新的docker-ce:
    # curl -fsSL https://get.docker.com/ | sh

CentOS
   如果是centos，上面的安装命令会在系统上添加yum源:/etc/yum.repos.d/docker-ce.repo 
   # wget https://download.docker.com/linux/centos/docker-ce.repo
   # mv docker-ce.repo /etc/yum.repos.d
   # yum install -y docker-ce

    或者直接下载rpm安装:

    # wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.09.0.ce-1.el7.centos.x86_64.rpm
    # yum localinstall docker-ce-17.09.0.ce-1.el7.centos.x86_64.rpm
    
注意：
    在说docker的时候尽量说Linux docker
    因为Docker on Mac，以及 Windows Docker（Hyper-V 实现），实际上是基于虚拟化技术实现的，跟我们介绍使用的 Linux 容器完全不同。
```

##### 国内源安装新版docker

```
使用aliyun docker yum源安装新版docker
删除已安装的Docker

    # yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

配置阿里云Docker Yum源
    # yum install -y yum-utils device-mapper-persistent-data lvm2 git
    # yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

安装指定版本
    查看Docker版本：
    # yum list docker-ce --showduplicates

    安装较旧版本（比如Docker 17.03.2) ：
        需要指定完整的rpm包的包名，并且加上--setopt=obsoletes=0 参数：
        # yum install -y --setopt=obsoletes=0 \
        docker-ce-17.03.2.ce-1.el7.centos.x86_64 \
        docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch

    安装Docker新版本（比如Docker 18.03.0)：
        加上rpm包名的版本号部分或不加都可以：
        # yum install docker-ce-18.03.0.ce  -y
        或者
        # yum install docker-ce -y

启动Docker服务：
    #systemctl enable docker
    #systemctl start docker

查看docker版本状态： 
    # docker -v
    Docker version 1.13.1, build 8633870/1.13.1  
    
    # docker version
    Client:
     Version:           18.09.0
     API version:       1.39
     Go version:        go1.10.4
     Git commit:        4d60db4
     Built:             Wed Nov  7 00:48:22 2018
     OS/Arch:           linux/amd64
     Experimental:      false

    Server: Docker Engine - Community
     Engine:
      Version:          18.09.0
      API version:      1.39 (minimum version 1.12)
      Go version:       go1.10.4
      Git commit:       4d60db4
      Built:            Wed Nov  7 00:19:08 2018
      OS/Arch:          linux/amd64
      Experimental:     false
      
查看docker运行状态：
    # docker info
    Containers: 0
     Running: 0
     Paused: 0
     Stopped: 0
    Images: 0
    Server Version: 18.09.0
    Storage Driver: overlay2
     Backing Filesystem: xfs
     Supports d_type: true
     Native Overlay Diff: true
    Logging Driver: json-file
    Cgroup Driver: cgroupfs
    Plugins:
     Volume: local
     Network: bridge host macvlan null overlay
     Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
    Swarm: inactive
    Runtimes: runc
    Default Runtime: runc
    Init Binary: docker-init
    containerd version: c4446665cb9c30056f4998ed953e6d4ff22c7c39
    runc version: 4fc53a81fb7c994640722ac585fa9ca548971871
    init version: fec3683
    Security Options:
     seccomp
      Profile: default
    Kernel Version: 3.10.0-957.el7.x86_64
    Operating System: CentOS Linux 7 (Core)
    OSType: linux
    Architecture: x86_64
    CPUs: 4
    Total Memory: 1.934GiB
    Name: docker
    ID: MF5S:ZX25:SWJ3:XEIG:FFHP:5VXF:F5AL:KQFF:KKXP:XZGY:YGTE:EBQF
    Docker Root Dir: /var/lib/docker
    Debug Mode (client): false
    Debug Mode (server): false
    Registry: https://index.docker.io/v1/
    Labels:
    Experimental: false
    Insecure Registries:
     127.0.0.0/8
    Live Restore Enabled: false
    Product License: Community Engine

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

bonding

报错1：
    docker info的时候报如下错误
    bridge-nf-call-iptables is disabled

解决1：
    追加如下配置,然后重启系统
    # vim /etc/sysctl.conf   
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.bridge.bridge-nf-call-arptables = 1

问题2：
    虚拟机ping百度也能ping通，但是需要等好几秒才出结果，关键是下载镜像一直报错如下
    # docker pull daocloud.io/library/nginx
    Using default tag: latest
    Error response from daemon: Get https://daocloud.io/v2/: dial tcp: lookup daocloud.io on 192.168.1.2:53: read udp   192.168.1.189:41335->192.168.1.2:53: i/o timeout

解决2：
    我的虚拟机用的网关和dns都是虚拟机自己的.1或者.2，把DNS改成8.8.8.8问题就解决了，ping百度也秒出结果
    # vim /etc/resolv.conf
    nameserver 8.8.8.8

```

##### 登入登出docker hub

```
login     Register or log in to a Docker registry
登录到自己的Docker register，需有Docker Hub的注册账号
      # docker login
      Username: test
      Password: 
      Email: xxxx@foxmail.com
      WARNING: login credentials saved in /root/.docker/config.json
      Login Succeeded

logout    Log out from a Docker registry
退出登录
      # docker logout
      Remove login credentials for https://index.docker.io/v1/

注：推送镜像库到私有源（可注册 docker 官方账户，推送到官方自有账户）
```

##### 国内镜像源

```
去查看如何使用aliyun的docker镜像库
去查看如何使用网易蜂巢的docker镜像库

Docker 加速器  
使用 Docker 的时候，需要经常从官方获取镜像，但是由于显而易见的网络原因，拉取镜像的过程非常耗时，严重影响使用 Docker 的体验。因此 DaoCloud 推出了加速器工具解决这个难题，通过智能路由和缓存机制，极大提升了国内网络访问 Docker Hub 的速度，目前已经拥有了广泛的用户群体，并得到了 Docker 官方的大力推荐。
如果您是在国内的网络环境使用 Docker，那么 Docker 加速器一定能帮助到您。    

Docker 加速器对 Docker 的版本有要求吗？    
需要 Docker 1.8 或更高版本才能使用，如果您没有安装 Docker 或者版本较旧，请安装或升级。    

Docker 加速器支持什么系统？    
Linux, MacOS 以及 Windows 平台。    

Docker 加速器是否收费？    
DaoCloud 为了降低国内用户使用 Docker 的门槛，提供永久免费的加速器服务，请放心使用。  

国内比较好的镜像源：网易蜂巢、aliyun和daocloud,下面是daocloud配置方式：
﻿Docker Hub并没有在国内部署服务器或者使用国内的CDN服务，因此在国内特殊的网络环境下，镜像下载十分耗时。
为了克服跨洋网络延迟，能够快速高效地下载Docker镜像，可以采用DaoCloud提供的服务Docker Hub Mirror，速度
快很多
1.注册网站账号
2.然后进入你自己的""制台"，选择"加速器"，点"立即开始"，接入你自有的主机，就看到如下的内容了
   curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361XXX.m.daocloud.io
该脚本可以将 --registry-mirror 加入到你的 Docker 配置文件 /etc/docker/daemon.json 中。适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7、Fedora、Arch Linux、openSUSE Leap 42.1，其他版本可能有细微不同。更多详情请访问文档。
   
    
3.配置完成后从Docker Hub Mirror下载镜像，命令：
    #dao pull ubuntu

注1：第一次使用daocloud是配置了加速器的，可以直接使用dao pull centos拉取经过加速之后的镜像，但是后来发现，不使用加速器也可以直接在daocloud官网上找到想要拉取的镜像地址进行拉取，﻿比如：#docker pull 
daocloud.io/library/tomcat:6.0-jre7
注2：上面配置加速器的方法，官网会更新，最新方法你应该根据官网提示去操作。
===========以下为tigerfive亲测================
使用国内镜像：
进入网站：https://hub.daocloud.io/
注册帐号：tigerfive
进入镜像市场：填写搜索的镜像名称


选择第一个

点击右边快速部署：


写入名称，选择我的主机，按提示继续在主机上进行所有操作

# mkdir /docker
# cd /docker
# curl -L -o /tmp/daomonit.x86_64.rpm https://get.daocloud.io/daomonit/daomonit.x86_64.rpm
# rpm -Uvh /tmp/daomonit.x86_64.rpm
# daomonit -token=36e3dedaa2e6b352f47b26a3fa9b67ffd54f5077 save-config
# service daomonit start

出现如下界面：说明自有主机接入完成（注意：这里我用的主机是我自己笔记本上的一台虚拟机）


接下来我们在镜像市场找到一个centos的镜像:点击右面的拉取按钮,会出现拉取命令如下：
我们按命令执行：
    # docker pull daocloud.io/library/centos:latest
    出现如下提示：说明拉取成功
    Trying to pull repository daocloud.io/library/centos ... 
    latest: Pulling from daocloud.io/library/centos

    08d48e6f1cff: Pull complete 
    Digest: sha256:934ff980b04db1b7484595bac0c8e6f838e1917ad3a38f904ece64f70bbca040
    Status: Downloaded newer image for daocloud.io/library/centos:latest

查看一下本地镜像：      
    [root@docker1 docker]# docker images
    REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
    daocloud.io/library/centos   latest              0584b3d2cf6d        3 weeks ago         196.5 MB

在拉取回来的本地镜像执行命令：
    万年不变的"你好世界"：    
    # docker run daocloud.io/library/centos:7 /bin/echo "hello world"
        hello world
    
    使用容器中的shell：
        [root@docker1 docker]# docker run -i -t centos /bin/bash  
    Unable to find image 'centos:latest' locally
    Trying to pull repository docker.io/library/centos ...   
    注意上面这样是不行的，因为默认使用的是docker官方默认镜像库的位置，需要按如下命令执行：
    [root@docker1 docker]# docker run -i -t daocloud.io/library/centos /bin/bash
    -i   捕获标准输入输出
    -t   分配一个终端或控制台
    进去之后可以在里面执行其他命令
    [root@336412c1b562 /]# ls
    anaconda-post.log  bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
    [root@336412c1b562 /]# df -h   #可以看到这里的磁盘分区是镜像本身的磁盘分区
    Filesystem                                                                                           Size  Used Avail Use% Mounted on
    /dev/mapper/docker-253:0-134984824-bc712573ec743c160ea903f6196ff4814056230d6a232eb3d39761d182bc7d1c   10G  240M  9.8G   3% /
    tmpfs                                                                                                497M     0  497M   0% /dev
    tmpfs                                                                                                497M     0  497M   0% /sys/fs/cgroup
    /dev/mapper/centos-root                                                                               38G  2.3G   36G   7% /etc/hosts
    shm                                                                                                   64M     0   64M   0% /dev/shm
    [root@336412c1b562 /]# exit
    exit
    [root@docker1 docker]# df -h   #这是我本地主机系统的磁盘分区
    文件系统                 容量  已用  可用 已用% 挂载点
    /dev/mapper/centos-root   38G  2.3G   36G    7% /
    devtmpfs                 487M     0  487M    0% /dev
    tmpfs                    497M     0  497M    0% /dev/shm
    tmpfs                    497M   20M  478M    4% /run
    tmpfs                    497M     0  497M    0% /sys/fs/cgroup
    /dev/vda1                497M  107M  391M   22% /boot
    tmpfs                    100M     0  100M    0% /run/user/0
    /dev/sr0                 7.3G  7.3G     0  100% /mnt/centos7u2

重新进入容器：执行其他命令试一下，可以看到我们的容器可以做我们熟悉的所有的事情：
1.可以上网
    [root@9990e6c99bbd /]# ping www.baidu.com -c 2
    PING www.a.shifen.com (180.97.33.107) 56(84) bytes of data.
    64 bytes from 180.97.33.107: icmp_seq=1 ttl=52 time=19.8 ms
    64 bytes from 180.97.33.107: icmp_seq=2 ttl=52 time=19.1 ms
2.网络yum源已经配置好
    [root@9990e6c99bbd /]# yum repolist
    Loaded plugins: fastestmirror, ovl
    base                                                                                                                           | 3.6 kB  00:00:00     
    extras                                                                                                                         | 3.4 kB  00:00:00     
    updates                                                                                                                        | 3.4 kB  00:00:00     
    (1/4): base/7/x86_64/group_gz                                                                                                  | 155 kB  00:00:01     
    (2/4): extras/7/x86_64/primary_db                                                                                              | 166 kB  00:00:04     
    (3/4): updates/7/x86_64/primary_db                                                                                             | 9.1 MB  00:00:07     
    (4/4): base/7/x86_64/primary_db                                                                                                | 5.3 MB  00:00:22     
    Determining fastest mirrors
     * base: mirrors.tuna.tsinghua.edu.cn
     * extras: mirrors.tuna.tsinghua.edu.cn
     * updates: mirrors.163.com
    repo id                                                                repo name                                                                status
    base/7/x86_64                                                          CentOS-7 - Base                                                          9007
    extras/7/x86_64                                                        CentOS-7 - Extras                                                         393
    updates/7/x86_64                                                       CentOS-7 - Updates                                                       2560
    repolist: 11960

3.可以安装软件：
    [root@9990e6c99bbd /]# lsof -i:80
    bash: lsof: command not found
    [root@9990e6c99bbd /]# yum install lsof httpd -y
    Loaded plugins: fastestmirror, ovl
    Loading mirror speeds from cached hostfile
     * base: mirrors.tuna.tsinghua.edu.cn
     * extras: mirrors.tuna.tsinghua.edu.cn
     * updates: mirrors.163.com
    Resolving Dependencies
    --> Running transaction  check
    ---> Package lsof.x86_64 0:4.87-4.el7 will be installed
    --> Finished Dependency Resolution

    Dependencies Resolved

    ======================================================================================================================================================
     Package                          Arch                               Version                                   Repository                        Size
    ======================================================================================================================================================
    Installing:
     lsof                             x86_64                             4.87-4.el7                                base                             331 k
```

#### 国内源介绍及使用

```
Docker 加速器  
使用 Docker 的时候，需要经常从官方获取镜像，但是由于显而易见的网络原因，拉取镜像的过程非常耗时，严重影响使用 Docker 的体验。因此 DaoCloud 推出了加速器工具解决这个难题，通过智能路由和缓存机制，极大提升了国内网络访问 Docker Hub 的速度，目前已经拥有了广泛的用户群体，并得到了 Docker 官方的大力推荐。
如果您是在国内的网络环境使用 Docker，那么 Docker 加速器一定能帮助到您。    

Docker 加速器对 Docker 的版本有要求吗？    
需要 Docker 1.8 或更高版本才能使用，如果您没有安装 Docker 或者版本较旧，请安装或升级。    

Docker 加速器支持什么系统？    
Linux, MacOS 以及 Windows 平台。    

Docker 加速器是否收费？    
DaoCloud 为了降低国内用户使用 Docker 的门槛，提供永久免费的加速器服务，请放心使用。  

国内比较好的镜像源：网易蜂巢、aliyun和daocloud,下面是daocloud配置方式：
Docker Hub并没有在国内部署服务器或者使用国内的CDN服务，因此在国内特殊的网络环境下，镜像下载十分耗时。
为了克服跨洋网络延迟，能够快速高效地下载Docker镜像，可以采用DaoCloud提供的服务Docker Hub Mirror，速度
快很多
1.注册网站账号
2.然后进入你自己的""制台"，选择"加速器"，点"立即开始"，接入你自有的主机，就看到如下的内容了
    • 下载并安装相关软件
     #curl -L -o /tmp/daomonit_amd64.deb https://get.daocloud.io/daomonit/daomonit_amd64.debsudo 
     #dpkg -4i /tmp/daomonit_amd64.deb
    • 配置
     #sudo daomonit -token=e16ed16b2972865e19b143695cf08cad850d5570 save-config
    • 启动服务
     #service daomonit start
    
3.配置完成后从Docker Hub Mirror下载镜像，命令：
    #dao pull ubuntu
```

#### 镜像管理

```
搜索镜像：
    这种方法只能用于官方镜像库
    搜索基于 centos 操作系统的镜像
    # docker search centos

    按星级搜索镜像：        
    查找 star 数至少为 100 的镜像，默认不加 s 选项找出所有相关 ubuntu 镜像：         
    # docker search ubuntu -f stars=100    
      
拉取镜像：
    # docker pull centos

查看本地镜像：  
    # docker image list
    
查看镜像详情：
    # docker image inspect 镜像id
    
删除镜像：
    删除一个或多个，多个之间用空格隔开，可以使用镜像名称或id
    # docker rmi daocloud.io/library/mysql

    强制删除：--force
    如果镜像正在被使用中可以使用--force强制删除    
    # docker rmi docker.io/ubuntu:latest --force

    删除所有镜像：
    # docker rmi $(docker images -q)

只查看所有镜像的id:
    # docker images -q 
  
查看镜像制作的过程：
    相当于dockfile
    # docker history daocloud.io/ubuntu
    
    dive docker_image_id/docker_image_name
```

#### 容器管理

```
创建新容器但不启动：
# docker create -it daocloud.io/library/centos:5 /bin/bash

创建并运行一个新Docker 容器：
    同一个镜像可以启动多个容器,每次执行run子命令都会运行一个全新的容器
    # docker run -it --restart=always centos /bin/bash
﻿    如果执行成功，说明CentOS 容器已经被启动，并且应该已经得到了 bash 提示符。
    -i   
        捕获标准输入输出
    -t   
        分配一个终端或控制台
    --restart=always   
        容器随docker engine自启动，因为在重启docker的时候默认容器都会被关闭   
        也适用于create选项
         
    --rm
        默认情况下，每个容器在退出时，它的文件系统也会保存下来，这样一方面调试会方便些，因为你可以通过查看日志等方式来确定最终状态。另一方面，也可以保存容器所产生的数据。
        但是当你仅仅需要短暂的运行一个容器，并且这些数据不需要保存，你可能就希望Docker能在容器结束时自动清理其所产生的数据。这个时候就需要--rm参数了。注意：--rm 和 -d不能共用

若要断开与容器的连接，并且关闭容器：
   容器内部执行如下命令
    [root@d33c4e8c51f8 /]#exit

如果只想断开和容器的连接而不关闭容器：
    快捷键：ctrl+p+q

查看容器：
    只查看运行状态的容器：
    #docker ps

    #docker ps -a
    -a  查看所有容器
    
    只查看所有容器id:
    # docker ps -a -q
 
    列出最近一次启动的容器
    # docker ps -l   

查看容器详细信息：
inspect   Return low-level information on a container or image
用于查看容器的配置信息，包含容器名、环境变量、运行命令、主机配置、网络配置和数据卷配置等。

目标：
查找某一个运行中容器的id，然后使用docker inspect命令查看容器的信息。

提示：
可以使用镜像id的前面部分，不需要完整的id。
[root@master ~]# docker inspect d95    //d95是我机器上运行的一个容器ID的前3个字符
[
    {
        "Id": "d95a220a498e352cbfbc098c949fc528dbf5a5c911710b108ea3a9b4aa3a4761",
        "Created": "2017-07-08T03:59:16.18225183Z",
        "Path": "bash",
        "Args": [],
        "State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
容器信息很多，这里只粘贴了一部分

比如：容器里在安装ip或ifconfig命令之前，查看网卡IP显示容器IP地址和端口号，如果输出是空的说明没有配置IP地址（不同的Docker容器可以通过此IP地址互相访问）
# docker inspect --format='{{.NetworkSettings.IPAddress}}'  容器id
列出所有绑定的端口:
# docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> 
{{(index $conf 0).HostPort}} {{end}}' $INSTANCE_ID

# docker inspect --format='{{range $p, 
$conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' 
b220fabf815a
22/tcp -> 20020

找出特殊的端口映射:
比如找出容器里22端口所映射的docker本机的端口：
# docker inspect --format='{{(index (index .NetworkSettings.Ports "22/tcp") 
0).HostPort}}' $INSTANCE_ID
[root@localhost ~]# docker inspect --format='{{(index (index .NetworkSettings.Ports "22/
tcp") 0).HostPort}}' b220fabf815a
20020

http://note.youdao.com/noteshare?id=4ee7884b3d25c16850d78cfdec276add

启动容器：
# docker start  name

关闭容器：
# docker stop  name
# docker kill    name      --强制终止容器

杀死所有running状态的容器
# docker kill $(docker ps  -q)  

stop和kill的区别：
    docker stop命令给容器中的进程发送SIGTERM信号，默认行为是会导致容器退出，当然，容器内程序可以捕获该信号并自行处理，例如可以选择忽略。而docker kill则是给容器的进程发送SIGKILL信号，该信号将会使容器必然退出。 

删除容器：
    # docker rm 容器id或名称
    要删除一个运行中的容器，添加 -f 参数
    
    根据格式删除所有容器：
    # docker rm $(docker ps -qf status=exited)
        
重启容器：
#docker restart name

暂停容器：
pause   --暂停容器内的所有进程，
    通过docker stats可以观察到此时的资源使用情况是固定不变的，通过docker logs -f也观察不到日志的进一步输出。

恢复容器：
unpause  --恢复容器内暂停的进程，与pause参数相对应

# docker start infallible_ramanujan  //这里的名字是状态里面NAMES列列出的名字，这种方式同样会让容器运行在后台

让容器运行在后台：
﻿如果在docker run后面追加-d=true或者-d，那么容器将会运行在后台模式。此时所有I/O数据只能通过网络资源或者共享卷组来进行交互。因为容器不再监听你执行docker run的这个终端命令行窗口。但你可以通过执行
docker attach来重新附着到该容器的回话中。注意，容器运行在后台模式下，是不能使用--rm选项的

#docker run -d IMAGE[:TAG] 命令
#docker logs container_id   #打印该容器的输出

# docker run -it -d --name mytest docker.io/centos:7 /bin/sh -c "while true; do echo hello world; sleep 2; done"
37738fe3d6f9ef26152cb25018df9528a89e7a07355493020e72f147a291cd17

[root@localhost ~]# docker logs mytest
hello world
hello world

docker attach container_id #附加该容器的标准输出到当前命令行
[root@localhost ~]# docker attach mytest
hello world
hello world
.......
此时，ctrl+d等同于exit命令，按ctrl+p+q可以退出到宿主机，而保持container仍然在运行

rename 
    Rename a container
    
stats     
    Display a live stream of container(s) resource usage statistics    
    --动态显示容器的资源消耗情况，包括：CPU、内存、网络I/O   
    
port    
    List port mappings or a specific mapping for the CONTAINER
     --输出容器端口与宿主机端口的映射情况
    # docker port blog
        80/tcp -> 0.0.0.0:80
     容器blog的内部端口80映射到宿主机的80端口，这样可通过宿主机的80端口查看容器blog提供的服务
 
连接容器：    
方法1.attach
# docker attach 容器id   //前提是容器创建时必须指定了交互shell

方法2.exec      
    通过exec命令可以创建两种任务：后台型任务和交互型任务
    交互型任务：
        # docker exec -it  容器id  /bin/bash
        root@68656158eb8e:/# ls     
    
    后台型任务：
        # docker exec 容器id touch /testfile

监控容器的运行：
可以使用logs、top、events、wait这些子命令
    logs:
        使用logs命令查看守护式容器
        可以通过使用docker logs命令来查看容器的运行日志，其中--tail选项可以指定查看最后几条日志，而-t选项则可以对日志条目附加时间戳。使用-f选项可以跟踪日志的输出，直到手动停止。
        # docker logs    App_Container   //不同终端操作
        # docker logs -f App_Container

    top:
        显示一个运行的容器里面的进程信息
        # docker top birdben/ubuntu:v1

    events    
        Get real time events from the server
        实时输出Docker服务器端的事件，包括容器的创建，启动，关闭等。

        # docker start loving_meninsky
        loving_meninsky

        # docker events  //不同终端操作
        2017-07-08T16:39:23.177664994+08:00 network connect  
        df15746d60ffaad2d15db0854a696d6e49fdfcedc7cfd8504a8aac51a43de6d4 
        (container=50a0449d7729f94046baf0fe5a1ce2119742261bb3ce8c3c98f35c80458e3e7a, 
        name=bridge, type=bridge)
        2017-07-08T16:39:23.356162529+08:00 container start 
        50a0449d7729f94046baf0fe5a1ce2119742261bb3ce8c3c98f35c80458e3e7a (image=ubuntu, 
        name=loving_meninsky)

    wait      
        Block until a container stops, then print its exit code   
        --捕捉容器停止时的退出码
        执行此命令后，该命令会"hang"在当前终端，直到容器停止，此时，会打印出容器的退出码
        # docker wait 01d8aa  //不同终端操作
           137

    diff
        查看容器内发生改变的文件，以elated_lovelace容器为例
        root@68656158eb8e:/# touch c.txt

        用diff查看：
        包括文件的创建、删除和文件内容的改变都能看到 
        [root@master ~]# docker diff  容器名称
        A /c.txt

        C对应的文件内容的改变，A对应的均是文件或者目录的创建删除
        [root@docker ~]# docker diff 7287
        A /a.txt
        C /etc
        C /etc/passwd
        A /run
        A /run/secrets  
        
宿主机和容器之间相互COPY文件
    cp的用法如下：
    Usage:    docker cp [OPTIONS] CONTAINER:PATH LOCALPATH
                   docker cp [OPTIONS] LOCALPATH CONTAINER:PATH

    如：容器mysql中/usr/local/bin/存在docker-entrypoint.sh文件，可如下方式copy到宿主机
    #  docker cp mysql:/usr/local/bin/docker-entrypoint.sh   /root

    修改完毕后，将该文件重新copy回容器
    # docker cp /root/docker-entrypoint.sh mysql:/usr/local/bin/      
```

#### 容器打包

```
将容器的文件系统打包成tar文件,也就是把正在运行的容器直接导出为tar包的镜像文件

export    
    Export a container's filesystem as a tar archive

有两种方式（elated_lovelace为容器名）：
第一种：
    [root@master ~]# docker export -o elated_lovelace.tar elated_lovelace

第二种：
    [root@master ~]# docker export 容器名称 > 镜像.tar

导入镜像归档文件到其他宿主机：
import    
    Import the contents from a tarball to create a filesystem image
    # docker import elated_lovelace.tar  elated_lovelace:v1    
```

#### 镜像迁移

```
保存一台宿主机上的镜像为tar文件，然后可以导入到其他的宿主机上：
save      
    Save an image(s) to a tar archive
    将镜像打包，与下面的load命令相对应
    #docker save -o nginx.tar nginx

load      
    Load an image from a tar archive or STDIN
    与上面的save命令相对应，将上面sava命令打包的镜像通过load命令导入
    #docker load < nginx.tar
    
容器的迁移  
docker export b25f3cb2bf1f  | gzip > mynginx123.tar
zcat mynginx123.tar | docker import  - mynginx123
```

#### Dockerfile构建镜像

```
通过Dockerfile创建镜像
虽然可以自己制作 rootfs(见'容器文件系统那些事儿')，但Docker 提供了一种更便捷的方式，叫作 Dockerfile

docker build命令用于根据给定的Dockerfile和上下文以构建Docker镜像。

docker build语法：
# docker build [OPTIONS] <PATH | URL | ->

1. 常用选项说明
--build-arg，设置构建时的变量
--no-cache，默认false。设置该选项，将不使用Build Cache构建镜像
--pull，默认false。设置该选项，总是尝试pull镜像的最新版本
--compress，默认false。设置该选项，将使用gzip压缩构建的上下文
--disable-content-trust，默认true。设置该选项，将对镜像进行验证
--file, -f，Dockerfile的完整路径，默认值为‘PATH/Dockerfile’
--isolation，默认--isolation="default"，即Linux命名空间；其他还有process或hyperv
--label，为生成的镜像设置metadata
--squash，默认false。设置该选项，将新构建出的多个层压缩为一个新层，但是将无法在多个镜像之间共享新层；设置该选项，实际上是创建了新image，同时保留原有image。
--tag, -t，镜像的名字及tag，通常name:tag或者name格式；可以在一次构建中为一个镜像设置多个tag
--network，默认default。设置该选项，Set the networking mode for the RUN instructions during build
--quiet, -q ，默认false。设置该选项，Suppress the build output and print image ID on success
--force-rm，默认false。设置该选项，总是删除掉中间环节的容器
--rm，默认--rm=true，即整个构建过程成功后删除中间环节的容器

2. PATH | URL | -说明：
给出命令执行的上下文。
上下文可以是构建执行所在的本地路径，也可以是远程URL，如Git库、tarball或文本文件等。
如果是Git库，如https://github.com/docker/rootfs.git#container:docker，则隐含先执行git clone --depth 1 --recursive，到本地临时目录；然后再将该临时目录发送给构建进程。
构建镜像的进程中，可以通过ADD命令将上下文中的任何文件（注意文件必须在上下文中）加入到镜像中。
-表示通过STDIN给出Dockerfile或上下文。
示例：

    docker build - < Dockerfile

说明：该构建过程只有Dockerfile，没有上下文

    docker build - < context.tar.gz

说明：其中Dockerfile位于context.tar.gz的根路径

    docker build -t champagne/bbauto:latest -t champagne/bbauto:v2.1 .
    docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .


2.1、 创建镜像所在的文件夹和Dockerfile文件 
          命令： 
         1、mkdir sinatra 
         2、cd sinatra 
         3、touch Dockerfile 

2.2、 在Dockerfile文件中写入指令，每一条指令都会更新镜像的信息例如： 
         # This is a comment 
         FROM ubuntu:14.04 
         MAINTAINER tiger tiger@localhost.localdomain
         RUN apt-get update && apt-get install -y ruby ruby-dev 
         RUN gem install sinatra 
         
          格式说明： 
          每行命令都是以  INSTRUCTION statement 形式，就是命令+ 清单的模式。命令要大写，"#"是注解。 
         FROM 命令是告诉docker 我们的镜像什么。 
         MAINTAINER 是描述 镜像的创建人。 
         RUN 命令是在镜像内部执行。就是说他后面的命令应该是针对镜像可以运行的命令。 

 2.3、创建镜像 
          命令：docker build -t tiger/sinatra:v2 . 
         docker build  是docker创建镜像的命令 
         -t 是标识新建的镜像属于 ouruser的  
         sinatra是仓库的名称  
        ：v2 是tag 
          "."是用来指明 我们的使用的Dockerfile文件当前目录的 

         详细执行过程：
        [root@master sinatra]# docker build -t tiger/sinatra:v2 . 
        Sending build context to Docker daemon 2.048 kB
        Step 1 : FROM daocloud.io/ubuntu:14.04
        Trying to pull repository daocloud.io/ubuntu ... 
        14.04: Pulling from daocloud.io/ubuntu
        f3ead5e8856b: Pull complete 
        Digest: sha256:ea2b82924b078d9c8b5d3f0db585297a5cd5b9c2f7b60258cdbf9d3b9181d828
         ---> 2ff3b426bbaa
        Step 2 : MAINTAINER tiger tiger@localhost.localdomain
         ---> Running in 948396c9edaa
         ---> 227da301bad8
        Removing intermediate container 948396c9edaa
        Step 3 : RUN apt-get update && apt-get install -y ruby ruby-dev
         ...
        Step 4 : RUN gem install sinatra
        ---> Running in 89234cb493d9

 2.4、创建完成后，从镜像创建容器 
         #docker run -t -i tiger/sinatra:v2 /bin/bash
```



Dockerfile分为四个部分: 基础镜像信息、维护者信息、镜像操作指令和容器启动指令。 即FROM、MAINTAINER、RUN、CMD四个部分

##### 指令说明

```
FROM         指定所创建镜像的基础镜像
MAINTAINER   制定维护者信息
RUN          运行命令
CMD          容器启动是默认执行的命令
LABEL        指定生成镜像的元数据标签信息
EXPOSE       声明镜像内服务所监听的端口
ENV          指定环境变量
ADD          复制指定src路径的内容到容器的dest路径下，如果src为tar文件，则自动解压到dest路径下
copy         复制指定src路径的内容到镜像的dest路径下
ENTERPOINT   指定镜像的默认入口
VOLUME       创建数据卷挂载点
USER         指定运行容器是的用户名或UID
WORKDIR      配置工作目录
ARG          指定镜像内使用的参数
ONBUILD      配置当所创建的镜像作为其他镜像的基础镜像时，所执行创建操作指令
STOPSIGAL    容器退出信号值
HEALTHCHECK  如何进行健康检查
SHELL        指定使用shell的默认shell类型
```

##### nginx-dockerfile示例

vim Dockerfile

```
FROM centos:7.2.1511

ENV TZ=Asia/Shanghai

RUN yum -y install epel* \
	yum -y install gcc openssl openssl-devel  pcre-devel zlib-devel

ADD nginx-1.14.0.tar.gz /opt/

WORKDIR /opt/nginx-1.14.0

RUN ./configure --prefix=/opt/nginx  --http-log-path=/opt/nginx/logs/access.log --error-log-path=/opt/nginx/logs/error.log --http-client-body-temp-path=/opt/nginx/client/  --http-proxy-temp-path=/opt/nginx/proxy/  --with-http_stub_status_module --with-file-aio --with-http_flv_module --with-http_gzip_static_module --with-stream --with-threads --user=www --group=www
RUN make && make install
RUN groupadd www && useradd -g www www
WORKDIR /opt/nginx
RUN rm -rf /opt/nginx-1.14.0

ENV NGINX_HOME=/opt/nginx
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/nginx/sbin

EXPOSE 80 443
  
CMD ["nginx", "-g", "daemon off;"]
```

需要先下载nginx-1.14.0.tar.gz在Dockerfile同级目录下，然后执行如下命令 docker build -t nginx_image ./Dockerfile

```
#mkdir -p /data/nginx/{conf,conf.d,html,logs}
```

```
配置nginx启动时的挂在目录、开放端口号
#docker run --name nginx_emp -d -p 80:80  -p  443:443 --restart=always  -v /www/html/attachment:/www/html/attachment -v /data/nginx/html:/usr/share/nginx/html -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf  -v /data/nginx/logs:/var/log/nginx -v /data/nginx/conf.d:/etc/nginx/conf.d nginx:1.14.0
```



```
为解决nginx容器启动时 nginx无法自启动,
方案一:
在CMD上添加 RUN nginx,启动命令为 docker  run -itd  --name container_name image_name:tag ,即可解决该问题
方案二:
替换最后一条
CMD /bin/sh -c 'nginx -g "daemon off;"'
启动命令为: docker  run -itd  --name container_name image_name:tag
```



##### tomcat-dockerfile示例

```
FROM centos:7.2.1511

ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-7.0.88.tar.gz /usr/local/

WORKDIR /usr/local/

RUN mv jdk1.8.0_171 jdk && mv apache-tomcat-7.0.88 tomcat

ENV JAVA_HOME=/usr/local/jdk
ENV CLASS_PATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
ENV PATH=$JAVA_HOME/bin:$PATH
ENV CATALINA_HOME /usr/local/tomcat

EXPOSE 8080

CMD /usr/local/tomcat/bin/catalina.sh run
```

需要先下载jdk和tomcat在dockerfile的同级目录下，然后执行如下命令 docker build -t tomcat_image ./Dockerfile

如果无法自启动,修改CMD为ENTRYPOINT 

ENTRYPOINT ["/usr/local/apache-tomcat-8.5.47/bin/catalina.sh","run"]



##### 使用.dockerignore文件

可以通过.dockerigore文件(每一行添加一条匹配模式)来让Docker忽略匹配模式路径的目录和文件。例如

```
# commit
  */tmep*
  */*/tmp*
  tmp?
  ~*
```

```
怎么减小镜像的大小?
1. 减少镜像层级
2. 及时清理缓存
3. 不添加多余文件
4. 尽可能选用较小的基础镜像

```



#### 容器和进程

```
何为进程：
假如要写一个计算加法的小程序，这个程序需要的输入来自于一个文件，计算完成后的结果则输出到另一个文件中。

由于计算机只认 0 和 1，无论用哪种语言编写这段代码，最后都要通过某种方式翻译成二进制文件，才能在计算机操作系统中运行起来。

而为了能够让这些代码正常运行，往往还要给它提供数据，比如这个加法程序所需的输入文件。这些数据加上代码本身的二进制文件，放在磁盘上，就是我们平常所说的一个"程序"，也叫代码的可执行镜像（executable image）。

然后，就可以在计算机上运行这个"程序"了。

首先，操作系统从"程序"中发现输入数据保存在一个文件中，所以这些数据就被会加载到内存中待命。同时，操作系统又读取到了计算加法的指令，这时，它就需要指示 CPU 完成加法操作。而 CPU 与内存协作进行加法计算，又会使用寄存器存放数值、内存堆栈保存执行的命令和变量。同时，计算机里还有被打开的文件，以及各种各样的 I/O 设备在不断地调用中修改自己的状态。

就这样，一旦"程序"被执行起来，它就从磁盘上的二进制文件，变成了计算机内存中的数据、寄存器里的值、堆栈中的指令、被打开的文件，以及各种设备的状态信息的一个集合。像这样一个程序运起来后的计算机执行环境的总和，就是：进程。

所以，对于进程来说，它的静态表现就是程序，平常都安安静静地待在磁盘上；而一旦运行起来，它就变成了计算机里的数据和状态的总和，这就是它的动态表现。

容器与进程：
而容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个"边界"。
```



#### 认识Namespace

```
Linux 容器中用来实现"隔离"的技术手段：Namespace。Namespace 技术实际上修改了应用进程看待整个计算机"视图"，即它的"视线"被操作系统做了限制，只能"看到"某些指定的内容。但对于宿主机来说，这些被"隔离"了的进程跟其他进程并没有太大区别。
 
假设已经有一个 Linux 操作系统上的 Docker 项目在运行，环境为：Ubuntu 16.04 和 Docker CE 18.05
    
1.创建一个容器：
# docker run -it busybox /bin/sh
/ #

这条指令翻译成人类的语言就是：请帮我启动一个容器，在容器里执行 /bin/sh，并且给我分配一个命令行终端跟这个容器交互。

这样，这台Ubuntu 16.04 机器就变成了一个宿主机，而一个运行着 /bin/sh 的容器，就跑在了这个宿主机里面。

2. 在容器里执行 ps 指令，会发现一些有趣的事情：/bin/sh，就是这个容器内部的第 1 号进程（PID=1），而这个容器里一共只有两个进程在运行。这就意味着，前面执行的 /bin/sh，以及我们刚刚执行的 ps，已经被 Docker 隔离在了一个跟宿主机完全不同的世界当中。

/ # ps
PID  USER   TIME COMMAND
  1   root   0:00 /bin/sh
  10 root   0:00 ps

本来，每当在宿主机上运行一个 /bin/sh 程序，操作系统都会给它分配一个进程编号，比如 PID=100。这个编号是进程的唯一标识，就像员工的工牌一样。所以 PID=100，可以粗略地理解为这个 /bin/sh 是我们公司里的第 100 号员工，而第 1 号员工就自然是比尔 · 盖茨这样统领全局的人物。

而现在，要通过 Docker 把/bin/sh 运行在一个容器当中。这时，Docker 就会在这个第 100 号员工入职时给他施一个"障眼法"让他永远看不到前面的其他 99 个员工，更看不到比尔 · 盖茨。这样，他就会错误地以为自己就是公司里的第 1 号员工。

这种机制，其实就是对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，比如 PID=1。可实际上，他们在宿主机的操作系统里，还是原来的第 100 号进程。

这种技术，就是 Linux 里面的 Namespace 机制。
```



#### 深入理解Namespace机制

```
Namespace 的使用方式：
其实只是 Linux 创建新进程的一个可选参数。

在 Linux 系统中创建线程的系统调用是 clone()，比如：
    int pid = clone(main_function, stack_size, SIGCHLD, NULL); 
    这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。

当用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数，比如：
    int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
    这时，新创建的这个进程将会"看到"一个全新的进程空间，在这空间里，它的 PID 是 1。之所以说"看到"，是因为这只是一个"障眼法"，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

多次执行上面的 clone() 调用，就会创建多个 PID Namespace，而每个 Namespace 里的应用进程，都会认为自己是当前容器里的第 1 号进程，它们既看不到宿主机里真正的进程空间，也看不到其他 PID Namespace 里的具体情况。

而除了刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行"障眼法"操作。

比如，Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置。

这，就是 Linux 容器最基本的实现原理了。

所以，Docker 容器这个听起来玄而又玄的概念，实际上是在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能"看"到当前 Namespace 所限定的资源、文件、设备、状态，或者配置。而对于宿主机以及其他不相关的程序，它就完全看不到了。

所以说，容器，其实是一种特殊的进程而已。
```



#### 深入理解Cgroup机制

```
Linux 中，Cgroups 给用户暴露出来的操作接口是文件系统，即它以文件和目录的方式组织在操作系统的 /sys/fs/cgroup 路径下。

用 mount 指令把它们展示出来：
# mount -t cgroup 
cpuset on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cpu on /sys/fs/cgroup/cpu type cgroup (rw,nosuid,nodev,noexec,relatime,cpu)
cpuacct on /sys/fs/cgroup/cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct)
blkio on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
memory on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
...
输出结果，是一系列文件系统目录。在 /sys/fs/cgroup 下面有很多诸如 cpuset、cpu、 memory 这样的子目录，也叫子系统。这些都是这台机器当前可以被 Cgroups 进行限制的资源种类。而在子系统对应的资源种类下，你就可以看到该类资源具体可以被限制的方法。

比如，对 CPU 子系统来说，可以看到如下几个配置文件：
# ls /sys/fs/cgroup/cpu
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks

比如：cfs_period 和 cfs_quota 这两个参数需要组合使用，可以用来限制进程在长度为 cfs_period 的一段时间内，只能被分配到总量为 cfs_quota 的 CPU 时间。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

实例验证:
1. 需要在对应的子系统下面创建一个目录，这个目录称为一个"控制组",操作系统会在你新创建的目录下，自动生成该子系统对应的资源限制文件。
# cd /sys/fs/cgroup/cpu
# mkdir container    
# ls container/
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks

2. 在后台执行一条脚本：
# while : ; do : ; done &
[1] 226

它执行了一个死循环，可以把计算机的 CPU 吃到 100%，根据它的输出，可以看到这个脚本在后台运行的进程号（PID）是 226。

3. 用 top 指令确认一下 CPU 有没有被打满：用 1 分开查看cpu
# top
%Cpu0 :100.0 us, 0.0 sy, 0.0 ni, 0.0 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
在输出里可以看到，CPU 的使用率已经 100% 了（%Cpu0 :100.0 us）。

此时，通过查看 container 目录下的文件，看到 container 控制组里的 CPU quota 还没有任何限制（即：-1），CPU period 则是默认的 100 ms（100000 us）：

$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us 
-1
$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_period_us 
100000

4. 通过修改这些文件的内容来设置限制。
    比如，向 container 组里的 cfs_quota 文件写入 20 ms（20000 us）：
    # echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us

    意味着在每 100 ms 的时间里，被该控制组限制的进程只能使用 20 ms 的 CPU 时间，也就是说这个进程只能使用到 20% 的 CPU 带宽。

5. 把被限制的进程的 PID 写入 container 组里的 tasks 文件，上面的设置就会对该进程生效：
    # echo 226 > /sys/fs/cgroup/cpu/container/tasks 

6. 再次用 top 查看，验证效果：
    # top
    %Cpu0 : 20.3 us, 0.0 sy, 0.0 ni, 79.7 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st

    可以看到，计算机的 CPU 使用率立刻降到了 20%（%Cpu0 : 20.3 us）。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
除 CPU 子系统外，Cgroups 的每一项子系统都有其独有的资源限制能力：
    blkio：
        为​​​块​​​设​​​备​​​设​​​定​​​I/O 限​​​制，一般用于磁盘等设备；
    cpuset：
        为进程分配单独的 CPU 核和对应的内存节点；
    memory：
        为进程设定内存使用的限制。

Linux Cgroups 的设计，简单粗暴地理解，就是一个子系统目录加上一组资源限制文件的组合。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了。

至于在这些控制组下面的资源文件里填上什么值，就靠用户执行 docker run 时的参数指定了，比如这样一条命令：
# docker run -it --cpu-period=100000 --cpu-quota=20000 daocloud.io/centos /bin/bash

在启动这个容器后，通过查看 Cgroups 文件系统下，CPU 子系统中，"docker"这个控制组里的资源限制文件的内容来确认：
# cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_period_us 
100000
# cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_quota_us 
20000

在centos7里面是下面这个目录：不是上面的docker目录
# cat /sys/fs/cgroup/cpu,cpuacct/system.slice/docker-92f76c52e9c0c34e0f8e5bac01f75919ce59838951a86501e7114d218f8aaf3e.scope/cpu.cfs_quota_us
20000
这就意味着这个 Docker 容器，只能使用到 20% 的 CPU 带宽。

由于一个容器的本质就是一个进程，用户的应用进程实际上就是容器里 PID=1 的进程，也是其他后续创建的所有进程的父进程。这就意味着，在一个容器中，你没办法同时运行两个不同的应用，除非你能事先找到一个公共的 PID=1 的程序来充当两个不同应用的父进程，这也是为什么很多人都会用 systemd 或者 supervisord 这样的软件来代替应用本身作为容器的启动进程。

这是因为容器本身的设计，就是希望容器和应用能够同生命周期，这个概念对后续的容器编排非常重要。否则，一旦出现类似于"容器是正常运行的，但是里面的应用早已经挂了"的情况，编排系统处理起来就非常麻烦了。
```



#### 理解容器文件系统

Overlayfs

```
 Overlayfs是一种类似aufs的一种堆叠文件系统，于2014年正式合入Linux-3.18主线内核，目前其功能已经基本稳定（虽然还存在一些特性尚未实现）且被逐渐推广，特别在容器技术中更是势头难挡。
它依赖并建立在其它的文件系统之上（例如ext4fs和xfs等等），并不直接参与磁盘空间结构的划分，仅仅将原来底层文件系统中不同的目录进行"合并"，然后向用户呈现。因此对于用户来说，它所见到的overlay文件系统根目录下的内容就来自挂载时所指定的不同目录的"合集"。
```

![](https://i.loli.net/2019/06/10/5cfd3feebf32011320.jpg)

overlayfs最基本的特性，简单的总结为以下3点：
（1）上下层同名目录合并；
（2）上下层同名文件覆盖；
（3）lower dir文件写时拷贝。
  这三点对用户都是不感知的。

lower dirA / lower dirB目录和upper dir目录

1. 他们都是来自底层文件系统的不同目录，用户可以自行指定，内部包含了用户想要合并的文件和目录，merge dir目录为挂载点。
2. 当文件系统挂载后，在merge目录下将会同时看到来自各lower和upper目录下的内容，并且用户也无法（无需）感知这些文件分别哪些来自lower dir，哪些来自upper dir，用户看见的只是一个普通的文件系统根目录而已（lower dir可以有多个也可以只有一个）。

upper dir和各lower dir这几个不同的目录并不完全等价，存在层次关系。

1. 当upper dir和lower dir两个目录存在同名文件时，lower dir的文件将会被隐藏，用户只能看见来自upper dir的文件
2. lower dir也存在相同的层次关系，较上层屏蔽较下层的同名文件。
3. 如果存在同名的目录，那就继续合并（lower dir和upper dir合并到挂载点目录其实就是合并一个典型的例子）。

读写数据：

1. 各层目录中的upper dir是可读写的目录，当用户通过merge dir向其中一个来自upper dir的文件写入数据时，那数据将直接写入upper dir下原来的文件中，删除文件也是同理；
2. 而各lower dir则是只读的，在overlayfs挂载后无论如何操作merge目录中对应来自lower dir的文件或目录，lower dir中的内容均不会发生任何的改变。
3. 当用户想要往来自lower层的文件添加或修改内容时，overlayfs首先会的拷贝一份lower dir中的文件副本到upper dir中，后续的写入和修改操作将会在upper dir下的copy-up的副本文件中进行，lower dir原文件被隐藏。

overlayfs特性带来的好处和应用场景
实际的使用中，会存在以下的多用户复用共享文件和目录的场景。

见图

![](https://i.loli.net/2019/06/10/5cfd401cc414590609.jpg)

复用共享目录文件

在同一个设备上，用户A和用户B有一些共同使用的共享文件（例如运行程序所依赖的动态链接库等），一般是只读的；同时也有自己的私有文件（例如系统配置文件等），往往是需要能够写入修改的；最后即使用户A修改了被共享的文件也不会影响到用户B。

对于以上的需求场景，我们并不希望每个用户都有一份完全一样的文件副本，因为这样不仅带来空间的浪费也会影响性能，因此overlayfs是一个较为完美的解决方案。我们将这些共享的文件和目录所在的目录设定为lower dir (1~n)，将用户私有的文件和目录所在的目录设定为upper dir，然后挂载到用户指定的挂载点，这样即能够保证前面列出的3点需求，同时也能够保证用户A和B独有的目录树结构。最后最为关键的是用户A和用户B在各自挂载目录下看见的共享文件其实是同一个文件，这样磁盘空间的节省自是不必说了，还有就是共享同一份cache而减少内存的使用和提高访问性能，因为只要cache不被回收，只需某个用户首次访问时创建cache，后续其他所有用户都可以通过访问cache来提高IO性能。

上面说的这种使用场景在容器技术中应用最为广泛

Overlay和Overlay2
以docker容器为例来介绍overlay的两种应用方式：Overlay和Overlay2.

1. Docker容器将镜像层（image layer）作为lower dir
2. 将容器层（container layer）作为upper dir
3. 最后挂载到容器merge挂载点，即容器的根目录下。

遗憾的是，早期内核中的overlayfs并不支持多lower layer，在Linux-4.0以后的内核版本中才陆续支持完善。而容器中可能存在多层镜像，所以出现了两种overlayfs的挂载方式，早期的overlay不使用多lower layer的方式挂载而overlay2则使用该方式挂载。

1. Overlay Driver

Overlay挂载方式如下
--该图引用自Miklos Szeredi的《overlayfs and containers》2017 linux内核大会演讲材料

![](https://i.loli.net/2019/06/10/5cfd403cbd91496002.jpg)

Overlay Driver

镜像层和容器层的组织方式
黄色框中的部分是镜像层和容器层的组织方式

1. 各个镜像层中，每下一层中的文件以硬链接的方式出现在它的上一层中，以此类推，最终挂载overlayfs的lower dir为最上层镜像层目录imager layer N。
2. 与此同时，容器的writable dir作为upper dir，挂载成为容器的rootfs。

本图中虽然只描述了一个容器的挂载方式，但是其他容器也类似，镜像层lower dir N共享，只是各个容器的upper dir不同而已。

2. Overlay2 Driver
   Overlay2挂载方式如下。
   --该图引用自Miklos Szeredi的《overlayfs and containers》2017 linux内核大会演讲材料

![](https://i.loli.net/2019/06/10/5cfd40717de7868533.jpg)

Overlay2 Driver

Overlay2的挂载方式比Overlay的要简单许多，它基于内核overlayfs的Multiple lower layers特性实现，不再需要硬链接，直接将镜像层的各个目录设置为overlayfs的各个lower layer即可（Overlayfs最多支持500层lower dir），对比Overlay Driver将减少inode的使用。

分别查看镜像的详细信息和运行成容器之后的容器详细信息

lower1:lower2:lower3 :
    表示不同的lower层目录，不同的目录使用":"分隔，层次关系依次为lower1 > lower2 > lower3 
    注：多lower层功能支持在Linux-4.0合入，Linux-3.18版本只能指定一个lower dir

upper和目录:
    表示upper层目录
    

work目录：    

​    和文件系统挂载后用于存放临时和间接文件的工作基目录（work base dir）

merged目录:
    就是最终的挂载点目录

#### 理解chroot

```
理解chroot
在 Linux 操作系统里，有一个名为 chroot 的命令可以帮助你在 shell 中方便地完成这个工作。顾名思义，它的作用就是帮你"change root file system"，即改变进程的根目录到你指定的位置。

假设，现在有一个 /home/tiger/test 目录，想要把它作为一个 /bin/bash 进程的根目录。

首先，创建一个 test 目录和几个 lib 文件夹：
# mkdir -p /home/tiger/test
# mkdir -p /home/tiger/test/{bin,lib64,lib}

然后，把 bash 命令拷贝到 test 目录对应的 bin 路径下：
# cp -v /bin/{bash,ls}  /home/tiger/test/bin

接下来，把 ls和bash命令需要的所有 so 文件，也拷贝到 test 目录对应的 lib 路径下。找到 so 文件可以用 ldd 命令：

# list="$(ldd /bin/ls | egrep -o '/lib.*\.[0-9]')"
# for i in $list; do cp -v "$i" "/home/tiger/test/${i}"; done

# list="$(ldd /bin/bash | egrep -o '/lib.*\.[0-9]')"
# for i in $list; do cp -v "$i" "/home/tiger/test/${i}"; done

最后，执行 chroot 命令，告诉操作系统，将使用 /home/tiger/test 目录作为 /bin/bash 进程的根目录：
# chroot /home/tiger/test /bin/bash

这时，执行 "ls /"，就会看到，它返回的都是 /home/tiger/test 目录下面的内容，而不是宿主机的内容。

更重要的是，对于被 chroot 的进程来说，它并不会感受到自己的根目录已经被"修改"成 /home/tiger/test 了。

这种视图被修改的原理，是不是跟之前介绍的 Linux Namespace 很类似呢？
```

#### 容器卷操作

在Docker中，要想实现数据的持久化（所谓Docker的数据持久化即**数据不随着** **Container** **的结束而结束**），需要将数据从宿主机挂载到容器中。

目前Docker提供了三种不同的方式将数据从宿主机挂载到容器中：


(1)volumes：Docker管理宿主机文件系统的一部分，默认位于 /var/lib/docker/volumes 目录中；(**最常用的方式**)

(2) bind mounts：意为着可以存储在宿主机系统的任意位置；（**比较常用的方式**）


(3) tmpfs：挂载存储在宿主机系统的内存中，而不会写入宿主机的文件系统；(**一般都不会用的方式**)



```
新卷只能在容器创建过程当中挂载
# docker run -it --name="voltest" -v /tmp:/test  daocloud.io/library/centos:5 /bin/bash

共享其他容器的卷：
# docker run -it --volumes-from bc4181  daocloud.io/library/centos:5  /bin/bash

实际应用中可以利用多个-v选项把宿主机上的多个目录同时共享给新建容器：
比如：
# docker run -it -v /abc:/abc -v /def:/def 1ae9
```

#### 访问容器应用-端口转发

```
使用端口转发解决容器端口访问问题

-p:
创建应用容器的时候，一般会做端口映射，这样是为了让外部能够访问这些容器里的应用。可以用多个-p指定多个端口映射关系。

mysql应用端口转发：
查看本地地址：
#ip a
    ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:0a:5b:8b brd ff:ff:ff:ff:ff:ff
    inet 192.168.245.134/24 brd 192.168.245.255 scope global dynamic ens33
       valid_lft 1444sec preferred_lft 1444sec

运行容器：使用-p作端口转发，把本地3307转发到容器的3306，其他参数需要查看发布容器的页面提示
# docker run --name mysql1 -p 3307:3306  -e MYSQL_ROOT_PASSWORD=123 daocloud.io/library/mysql:5.7

查看Ip地址：
#docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器id
[root@master /]# docker inspect  mysql1 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",

通过本地IP：192.168.245.134的3307端口访问容器mysql1内的数据库，出现如下提示恭喜你
[root@master /]# mysql -u root -p123 -h 192.168.245.134 -P3307
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> 


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
-P:
当使用-P标记时，Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口。如下：
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/redis     latest              e4a35914679d        2 weeks ago         182.9 MB

[root@localhost ~]# docker run --name myredis -P -d docker.io/redis
805d0e21e531885aad61d3e82395210b50621f1991ec4b7f9a0e25c815cc0272

[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
805d0e21e531        docker.io/redis     "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        0.0.0.0:32768->6379/tcp   myredis

从上面的结果中可以看出，本地主机的32768端口被映射到了redis容器的6379端口上，也就是说访问本机的32768
端口即可访问容器内redis端口。

测试看下，登陆redis容器，随意写个数据
[root@localhost ~]# docker run --rm -it --name myredis2 --link myredis:redisdb docker.io/redis /bin/bash
root@be44d955d6f4:/data# redis-cli -h redisdb -p 6379
redisdb:6379> set tiger 123
OK
redisdb:6379>

在别的机器上通过上面映射的端口32768连接这个容器的redis
# redis-cli -h 192.168.245.134 -p 32768
192.168.1.23:32768> get tiger
"123"
```

#### 部署Docker web UI应用

```
下载并运行容器：
#docker pull uifd/ui-for-docker 
#docker run -it -d --name docker-web -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock docker.io/uifd/ui-for-docker 

浏览器访问测试：
    ip:9000
    

```

![Docker web UI](https://i.loli.net/2019/06/10/5cfe18e3b533e35937.jpg)



#### 部署私有仓库应用

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1580913035689&di=e4ea75aafc8dac8fe5a849f67f254152&imgtype=0&src=http%3A%2F%2Fimg2.ctoutiao.com%2Fuploads%2F2016%2F12%2F07%2F1481099954500237.png)

```
仓库镜像
Docker hub官方已提供容器镜像registry,用于搭建私有仓库

拉取镜像：
# docker pull daocloud.io/library/registry:latest

运行容器：                                   
# docker run --restart=always -d -p 5000:5000 daocloud.io/library/registry 

注：如果创建容器不成功，报错防火墙，解决方案如下
    #systemctl stop firewalld
    #yum install iptables*
    #systemctl start iptables
    #iptables -F
    #systemctl restart docker
    
# docker ps
CONTAINER ID  IMAGE  COMMAND   CREATED  STATUS    PORTS    NAMES
1f444285bed8        daocloud.io/library/registry   "/entrypoint.sh /etc/"   23 seconds ago      Up 21 seconds       0.0.0.0:5000->5000/tcp   elegant_rosalind

连接容器查看端口状态：
# docker exec -it  1f444285bed8  /bin/sh      //这里是sh 不是bash
/ # netstat -lnp                                              //查看5000端口是否开启
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 :::5000                 :::*                    LISTEN      1/registry
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node PID/Program name    Path

在本机查看能否访问该私有仓库, 看看状态码是不是200
[root@master registry]# curl  -I  127.0.0.1:5000     //参数是大写的i
HTTP/1.1 200 OK        

为了方便，下载1个比较小的镜像,buysbox
# docker pull busybox

上传前必须给镜像打tag  注明ip和端口：
# docker tag busybox  本机IP：端口/busybox

这是直接从官方拉的镜像，很慢：
# docker tag busybox 192.168.245.136:5000/busybox

下面这个Mysql是我测试的第二个镜像，从daocloud拉取的：
# docker tag daocloud.io/library/mysql 192.168.245.136:5000/daocloud.io/library/mysql
注：tag后面可以使用镜像名称也可以使用id,我这里使用的镜像名称，如果使用官方的镜像，不需要加前缀，但是daocloud.io的
得加前缀

修改请求方式为http:
    默认为https，不改会报以下错误:
        Get https://master.up.com:5000/v1/_ping: http: server gave HTTP response to HTTPS client
    
    # vim /etc/docker/daemon.json
    { "insecure-registries":["192.168.245.136:5000"] }
    
重启docker:
# systemctl restart docker

上传镜像到私有仓库：
# docker push 192.168.245.136:5000/busybox

# docker push 192.168.245.136:5000/daocloud.io/library/mysql

查看私有仓库里的所有镜像：
    注意我这里是用的是ubuntu的例子
    # curl 192.168.245.130:5000/v2/_catalog
        {"repositories":["daocloud.io/ubuntu"]}

    或者：
    
    # curl  http://192.168.245.130:5000/v2/daocloud.io/ubuntu/tags/list
        {"name":"daocloud.io/ubuntu","tags":["v2"]}

```



#### 部署Centos7容器应用

```
镜像下载：
# docker pull daocloud.io/library/centos:latest

包管理:
默认情况下，为了减小镜像的尺寸，在构建 CentOS 镜像时用了yum的nodocs选项。 如果您安装一个包后发现文件缺失，请在/etc/yum.conf中注释掉tsflogs=nodocs并重新安装您的包。 

systemd 整合:
因为 systemd 要求 CAPSYSADMIN 权限，从而得到了读取到宿主机 cgroup 的能力，CentOS7 中已经用 fakesystemd 代替了 systemd 来解决依赖问题。 如果仍然希望使用 systemd，可用参考下面的 Dockerfile：

# vim Dockerfile
FROM daocloud.io/library/centos:latest
MAINTAINER "tiger"  tiger@aliyun.com
ENV container docker
RUN yum -y swap -- remove fakesystemd -- install systemd systemd-libs
RUN yum -y update; yum clean all; \
(cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]


这个Dockerfile删除fakesystemd 并安装了 systemd。然后再构建基础镜像:
# docker build --rm -t local/c7-systemd .

一个包含 systemd 的应用容器示例

为了使用像上面那样包含 systemd 的容器，需要创建一个类似下面的Dockerfile：
# vim Dockerfile
FROM local/c7-systemd
RUN yum -y install httpd; yum clean all; systemctl enable httpd.service
EXPOSE 80
CMD ["/usr/sbin/init"]

构建镜像:
# docker build --rm -t local/c7-systemd-httpd .

运行包含 systemd 的应用容器:
为了运行一个包含 systemd 的容器，需要使用--privileged选项， 并且挂载主机的 cgroups 文件夹。 下面是运行包含 systemd 的 httpd 容器的示例命令：

# docker run --privileged -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 local/c7-systemd-httpd
注意：上条命令不能添加/bin/bash，添加了会导致服务不可用，而且有些服务可能会发现之前提到的权限不够的问题，但是如果不加会运行在前台(没有用-d)，可以用ctrl+p+q放到后台去


测试可用：
# elinks --dump http://docker       //下面为apache默认页面
                                            Testing 123..
   This page is used to test the proper operation of the [1]Apache HTTP
   server after it has been installed. If you can read this page it means
   that this site is working properly. This server is powered by [2]CentOS.
```

#### 容器网络

![](https://i.loli.net/2019/06/10/5cfe19e2a618834176.jpg)

```
小规模docker环境大部分运行在单台主机上，如果公司大规模采用docker，那么多个宿主机上的docker如何互联

Docker默认的内部ip为172.17.42.0网段，所以必须要修改其中一台的默认网段以免ip冲突。
#vim /etc/sysconfig/docker-network   (docker18)
DOCKER_NETWORK_OPTIONS= --bip=172.18.42.1/16

#cat /etc/docker/daemon.json  (docker19)
{ "bip": "172.18.0.1/16" }
#reboot


docker 130上：
#route add -net 172.18.0.0/16 gw 192.168.18.128

docker 128上：
#route add -net 172.17.0.0/16 gw 192.168.18.130

现在两台宿主机里的容器就可以通信了。
```



#### 容器固定IP

```
docker安装后，默认会创建三种网络类型，bridge、host和none
显示当前网络：
# docker network list
NETWORK ID            NAME                DRIVER              SCOPE
90b22f633d2f          bridge              bridge              local
e0b365da7fd2          host                host                local
da7b7a090837         none                null                local

bridge:网络桥接
默认情况下启动、创建容器都是用该模式，所以每次docker容器重启时会按照顺序获取对应ip地址，这就导致容器每次重启，ip都发生变化

none：无指定网络
启动容器时，可以通过–network=none,docker容器不会分配局域网ip
host：主机网络
docker容器的网络会附属在主机上，两者是互通的。

创建固定ip容器
1、创建自定义网络类型，并且指定网段
#docker network create --subnet=192.168.0.0/16 staticnet

通过docker network ls可以查看到网络类型中多了一个staticnet

2、使用新的网络类型创建并启动容器
#docker run -it --name userserver --net staticnet --ip 192.168.0.2 centos:6 /bin/bash

通过docker inspect可以查看容器ip为192.168.0.2，关闭容器并重启，发现容器ip并未发生改变
```

#### BUG整理

```
基于centos7的docker容器出现的一个bug
centos7下部署的docker容器中启动服务，报错如下：
[root@a3c8baf6961e .ssh]# systemctl restart sshd.service
Failed to get D-Bus connection: Operation not permitted
 
这是centos7容器里面出现的一个BUG！
即centos7镜像创建的容器里面安装服务后，不能用systemctl/service启动服务，centos6的容器里没有这个坑！
可以通过使用其他的方式启动或者换用centos6的镜像来避免这个错误。
 
解决方案如下：
原因是dbus-daemon没能启动。其实systemctl并不是不可以使用，可以将你的CMD设置为/usr/sbin/init即可。
这样就会自动将dbus等服务启动起来。即采用 /usr/sbin/init自动启动dbus daemon
 
即把之前的容器关闭并删除（docker stop container-id），然后重新启动容器，注意：
启动时一定要加上参数--privileged和/sbin/init，如下：
 
[root@localhost ~]#  docker run --privileged -it centos7:7.3.1611 /sbin/init     
上面的容器启动后，会一直在卡着的状态中，先不用管，打开另一个终端窗口，查看容器这里注意，宿主机可能会出现注销当前登陆账户的情况
 
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS            
PORTS                     NAMES
af40bd07fa0f        centos7:7.3.1611    "/sbin/init"        28 seconds ago      Up 28 
seconds                                 nauseous_shirley
  
然后按照容器的ID进去，这个时候再根据/bin/bash进入容器（前面加exec -it参数），接着重启ssh服务就ok了
[root@localhost ~]# docker exec -it af40bd07fa0f /bin/bash
[root@af40bd07fa0f /]# systemctl restart sshd.service
```





```
docker: Error response from daemon: Cannot start container b0a845a3dedeac7b46002d1c8514077309d88dcc0667b7080bc1ab67d70eb167: [9] System error: SELinux policy denies access..
如上出现上面的报错，这是由于selinux造成的！需要关闭selinux，如下：
[root@localhost ~]# setenforce 0
[root@localhost ~]# getenforce
Permissive
```



```
docker: Error response from daemon: failed to create endpoint mosredis on network bridge: iptables failed: iptables --wait -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.17.0.6  -j ACCEPT: iptables: No chain/target/match by that name.
 (exit status 1).
  
解决办法：
一般来说，重启docker服务，即可解决这个问题。
[root@localhost ~]# systemctl restart docker
[root@localhost ~]#
－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
如果重启docker服务解决不了，那么如下操作：
[root@localhost ~]# pkill docker
[root@localhost ~]# iptables -t nat -F
[root@localhost ~]# ifconfig docker0 down
[root@localhost ~]# brctl delbr docker0
```



```
docker中有两个status为dead的容器，删除时报错如下：
Error  response from daemon: Driver devicemapper failed to remove root  filesystem  33ddd2513fc3cb732fa02e912be1927929d8d028abefa945d8a3984d700a4d74: Device  is Busy
解决办法：
1）看容器进程是否已经杀掉。没有的话，可以手动杀死。
2）mount -l看是不是该容器的路径还在挂载状态。是的话，umount掉。
3）然后再次尝试docker rm container_id
尽量不要手动去移除dm和docker里面container的相关文件，以免造成垃圾数据。
4）尝试docker rm -f <container-id>，强制删除
这样可以删除掉docker daemon中的container信息（但是已经创建的dm还是ACTIVE的，所以还要再去把dm给删除了）
[root@linux-node2 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
c7bd050b0a23        centos              "/bin/bash"         About an hour ago   Up About an hour                             small_almeida
33ddd2513fc3        centos              "/bin/bash"         7 weeks ago         Dead                                         App_Container
eaf66f1e43ab        centos              "/sbin/init"        7 weeks ago         Up 7 weeks          0.0.0.0:8888->8080/tcp   hungry_khorana
[root@linux-node2 ~]# docker rm -f 33ddd2513fc3
Error response from daemon: Driver devicemapper failed to remove root filesystem 33ddd2513fc3cb732fa02e912be1927929d8d028abefa945d8a3984d700a4d74: Device is Busy
  
加上-f参数后，虽然删除时还是报错，然是container信息已经删除了
[root@linux-node2 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
c7bd050b0a23        centos              "/bin/bash"         About an hour ago   Up About an hour                             small_almeida
eaf66f1e43ab        centos              "/sbin/init"        7 weeks ago         Up 7 weeks          0.0.0.0:8888->8080/tcp   hungry_khorana
```



```
问题：因为修改了配置文件错误导致docker启动失败，但是还原配置之后也启不来了
# systemctl  start docker
Job for docker.service failed because start of the service was attempted too often. See "systemctl status docker.service" and "journalctl -xe" for details.
To force a start use "systemctl reset-failed docker.service" followed by "systemctl start docker.service" again.

解决：如题，等一会儿再启动就好了
```

