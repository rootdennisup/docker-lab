# 1 Docker 基础 

## 1 Docker 概述
### 1.1 虚拟机 vs 容器

**1、什么是虚拟机？**

+ 虚拟机（Virtual Machine）指通过<font style="color:#601BDE;">软件模拟的具有完整硬件系统功能的</font>、<font style="color:#601BDE;">运行在一个完全隔离环境中的完整计算机系统</font>。在实体计算机中能够完成的工作，在虚拟机中都能够实现。
+ 在计算机中创建虚拟机时，需要将实体机的部分硬盘和内存容量作为虚拟机的硬盘和内存容量。每个虚拟机都有独立的 CMOS、硬盘和操作系统，可以像使用实体机一样对虚拟机进行操作。虚拟机的代表产品：VMWare 和 OpenStack。

> **什么是 CMOS？**
>
> + **物理机中**，一块真实的、由电池供电的物理芯片，存储BIOS设置；
> + **虚拟机中**，由虚拟化软件模拟出来的一个软件组件，功能与物理CMOS相同。

**2、什么是容器？**

容器在操作系统层虚拟化，是一个标准的软件单元。容器是完全使用沙箱机制，相互之间不会有任何接口，并且容器性能开销极低，容器具备如下特性：

+ **随处运行**：容器可以将代码与配置文件和相关依赖库进行打包，从而确保在任何环境下的运行都是一致的。
+ **高资源利用率**：容器提供进程级的隔离，因此可以更加精细地设置 CPU 和内存的使用率，进而更好地利用服务器的计算资源。
+ **快速扩展**：每个容器都可作为单独的进程予以运行，并且可以共享底层操作系统的系统资源，这样一来可以加快容器的启动和停止效率。

**3、虚拟机和容器的区别与联系**

+ 虚拟机虽然可以隔离出很多「子电脑」，但占用空间更大，启动更慢；虚拟机软件可能需要付费，如VMWare。
+ 容器技术不需要虚拟出整个操作系统，只需要虚拟一个小规模的环境，类似「沙箱」。
+ 运行空间，虚拟机一般要几 GB 到 几十 GB 的空间，而容器只需要 MB 级甚至 KB 级。

| **特性** | **虚拟机** | **容器** |
| --- | --- | --- |
| 隔离级别 | 操作系统级 | 进程 |
| 隔离策略 | Hypervisor（虚拟机监控器） | Cgroups（控制组群） |
| 系统资源 | 5 ～ 15% | 0 ～ 5% |
| 启动时间 | 分钟级 | 秒级 |
| 镜像存储 | GB - TB | KB - MB |
| 集群规模 | 上百 | 上万 |
| 高可用策略 | 备份、容灾、迁移 | 弹性、负载、动态 |


### 1.2 Docker 简介
**1、什么是 Docker？**

+ Docker 是 Google 开源、采用 Go 语言实现的；基于 Linux 内核的 `cgroup`、`namespace` ，以及 OverlayFS类的 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术；
+ Docker 在容器基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等，提供简单易用的容器使用接口；由于隔离的进程独立于宿主和其它的隔离进程，因此也称其为容器。
+ Docker 将应用程序及其依赖，打包在一个镜像文件里。运行这个镜像文件，就会生成一个虚拟容器；程序在这个虚拟容器里运行，就像在真实的物理机上运行一样。

**2、为什么要用 Docker？**

+ 由于容器是进程级别的，和传统的虚拟机相比，具有许多优势如：启动快、资源占用少、体积小 、高效部署和交付等。

**3、Docker 主要用途？**

+ 提供一次性的环境。如：本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
+ 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
+ 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## 2 Docker 架构和核心概念
### 2.1 Docker 架构图
![](11-01)![](..\images\11-01.png)

简介如下：

+ `Client`：Docker 客户端，使用 Docker Api 与 Docker 的守护进程进行通信；
+ `docker machine`：一个简化 Docker 安装的命令行工具，如：VirtualBox、Microsoft Azure；
+ `Host`：Docker 主机，一个物理或者虚拟的机器，用于执行 Docker 守护进程和容器；
+ `Daemon`：Docker 守护进程；
+ `Container`：Docker 容器，独立运行的一个或一组应用；
+ `Images`：Docker 镜像，用于创建 Docker 容器的模板；
+ `Registry`：Docker 仓库，用来保存镜像。

### 2.2 镜像（Image）
操作系统分为内核和用户空间，于 Linux 而言，内核启动后，会挂 root 文件系统为其提供用户空间支持。<font style="color:#601BDE;"> 镜像（Image）是一个特殊的文件系统，就相当于一个 root 文件系统</font>。

Docker 镜像包含：运行时所需的<font style="color:#601BDE;">程序、库、资源、配置、参数</font>等<font style="color:#601BDE;">静态</font>文件，不包含任何动态数据，其内容在构建之后不会被改变。

**镜像分层存储：**

+ Docker 镜像不是像.iso 那样的打包文件，镜像是一个<font style="color:#601BDE;">逻辑概念</font>，<font style="color:#601BDE;">由多层文件系统联合组成</font>，是采用 `UnionFS` 技术设计的分层存储的架构。
+ Docker 镜像特点：
    - 构建镜像时，一层一层构建，前一层是后一层的基础；
    - 每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

### 2.3 容器（Container）
镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，<font style="color:#601BDE;">镜像是静态的定义，容器是镜像运行时的实体</font>。容器可以被创建、启动、停止、删除、暂停等。

**容器的特点：**

+ 容器的本质是进程，容器进程运行在属于自己的独立的命名空间（区别于宿主中执行的进程）；
+ 因此容器可以拥有自己的 root 文件系统、网络配置、进程空间，甚至用户 ID 空间；
+ 容器内的进程是运行在一个隔离的环境里，使用起来像是在独立于宿主的系统下操作。

**容器存储层：**

+ 每一个容器运行时，以镜像为基础层，在其上创建一个为容器运行时读写而准备的<font style="color:#601BDE;">容器存储层</font>；
+ 容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡，并且缓存在容器存储层的数据也随之清空。
+ 按照 docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。
+ 所有文件写入操作，都应该使用数据卷（Volume）或绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高（不会丢失数据）。

### 2.4 仓库（Repository）
`Docker Registry`是<font style="color:#601BDE;">用于集中存储、分发镜像的服务</font>。

+ 一个 `Registry` 中可以包含多个`Repository`，每个仓库可以包含多个标签（`Tag`），每个标签对应一个镜像；
+ 通常，一个仓库包含同一软件不同版本的镜像，而标签常用于对应该软件的各个版本；
+ 可以通过格式形如：<font style="color:#601BDE;"><仓库名>:<标签></font> 来指定具体是哪个软件哪个版本的镜像。若没有标签，将以 `latest`作为默认标签，如：`ubuntu:16.04、ubuntu:latest`。

**Docker Registry 公开服务：**

Docker Registry 公开服务是开放给用户使用、允许用户管理镜像的	Registry 服务。一般公开服务允许用户免费上传、下载公开的镜像。常见的公开服务如：官方的 [Docker Hub](https://hub.docker.com/) 、[网易云镜像服务](https://c.163.com/hub#/m/library/) 、[阿里云镜像库](https://cr.console.aliyun.com/) 等。

**私有 Docker Registry**

用户还可以在本地搭建私有 Docker Registry。Docker 官方提供了 Docker Registry 镜像，可以直接使用做为私有 Registry 服务。

### 2.5 镜像、容器、仓库的关系
关系如下图：

![](11-02)![](..\images\11-02.png)

## 3 Docker 安装
我的目标是在阿里云 ECS 服务器上进行云原生环境整体练习，拆分事项有：

+ Docker
+ 博客搭建
+ Kubernetes
+ 微服务部署实战
+ CICD 流水线
+ 微服务部署进阶
+ 服务网格

所以先规划 ECS 整体目录。

### 3.1 ECS 整体目录
```latex
/
├── home/
│   └── your-username/           # 用户主目录
│       ├── projects/            # 项目代码
│       │   ├── blog/            # 博客源码
│       │   ├── cicd-scripts/    # CI/CD脚本
│       │   └── k8s-manifests/   # K8s配置清单
│       └── tools/               # 工具脚本
│
├── opt/                         # 主安装目录
│   ├── docker/                  # Docker相关
│   │   ├── data/                # Docker数据目录
│   │   └── config/              # Docker配置
│   ├── kubernetes/              # K8s相关
│   │   ├── bin/                 # K8s二进制文件
│   │   ├── config/              # K8s配置文件
│   │   └── addons/              # K8s插件
│   ├── gitlab/                  # GitLab（可选）
│   ├── jenkins/                 # Jenkins
│   │   ├── home/                # Jenkins主目录
│   │   └── plugins/             # Jenkins插件
│   └── monitoring/              # 监控组件
│
├── var/                         # 可变数据
│   ├── lib/
│   │   ├── docker/              # Docker默认数据目录（保持默认）
│   │   └── kubelet/             # Kubelet数据
│   ├── log/
│   │   ├── docker/              # Docker日志
│   │   ├── kubernetes/          # K8s组件日志
│   │   ├── jenkins/             # Jenkins日志
│   │   └── nginx/               # Nginx日志
│   └── backup/                  # 备份目录
│
└── data/                        # 应用数据（推荐挂载数据盘到这里）
    ├── docker-registry/         # 私有镜像仓库数据
    ├── gitlab-data/             # GitLab数据
    ├── mysql-data/              # 数据库数据
    ├── blog-data/               # 博客数据
    └── backup/                  # 数据备份
```

### 3.2 Ubuntu 上 Docker 安装与配置
```plain
# 1、创建目录
sudo mkdir -p /opt/docker/{data,config}
sudo mkdir -p /var/log/docker

# 2、安装 Docker（使用官方脚本）
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

## 3、查看版本
docker version;

# 4、配置 Docker 数据目录
sudo tee /etc/docker/daemon.json << EOF
{
  "data-root": "/opt/docker/data",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  "registry-mirrors": [	
    "https://naljryvl.mirror.aliyuncs.com",      
    "https://docker.nju.edu.cn"
  ]
}
EOF

# 5、重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker

## 6、测试docker是否安装成功
docker pull hello-world;
docker run hello-world;
```

说明：

+ 创建目录，参考目录整体规划，其他目录可需要时再新增；
+ `tee`：作用类似于 T 型管道--从标准输入读取数据，输出到标准输出，并保存文件
    - <<EOF：here document，将两个 EOF 之间所有内容作为标准输入
    - tee 接收这个输入，同时：写入到 /etc/docker/daemon.json，并输出到屏幕（方便查看执行结果）
+ /etc 目录的作用：
    - 配置文件专用目录，系统和服务的主要配置文件都放这里
    - Docker 默认会到 /etc/docker/daemon.json 寻找配置
+ registry-mirrors：配置国内镜像加速器，
  - _注：https://naljryvl.mirror.aliyuncs.com 是阿里云服务的加速器，对阿里云服务有效_
  - _若 docker pull hello-world 无法拉取镜像，记得检查 DNS 解析_
    ```text
    > cat /etc/resolv.conf

    nameserver 8.8.8.8
    nameserver 114.114.114.114
    nameserver 223.5.5.5
    nameserver 127.0.0.53
    ```

### 3.3 docker 常用命令
使用 `docker --help` 查看 docker 命令，可分为如下几类操作：

+ docker 操作：
    - 版本/信息：`docker [info | version]`
+ 容器操作 ：
    - 容器生命周期：`docker [ run | start | stop | restart | kill | rm | pause | unpause ]`
    - 容器运维：`docker [ ps | inspect | exec | logs | export | import | port ]`
    - 容器 rootfs ：`docker [ commit | cp | diff ]`
+ 镜像操作：
    - 镜像管理：`docker [ images | rmi | tag | build | history | save | import ]`
+ 仓库操作
    - 镜像仓库：`docker [ login | pull | push | search ]`

## 参考资料
+ [Docker 官方文档](https://docs.docker.com/)
+ [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)


