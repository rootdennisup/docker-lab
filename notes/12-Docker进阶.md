:::info
<font style="color:rgba(0, 0, 0, 0.85) !important;">大处立向，小处拆解；难题细分，精进自来。</font>

:::

## 1 使用镜像
### 1.1 获取镜像
使用 `docker pull` 命令从 Docker 镜像仓库获取镜像，格式为：

```latex
docker pull [选项] [Registry 地址[:端口号]/]仓库名[:标签]
```

示例：

```latex
# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a0bcbecc962e: Pull complete
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

+ docker pull nginx ：没带 tag，则使用默认 tag；
+ 镜像是分层储存的，所以下载也是一层层下载；下载过程中给出了<font style="color:#601BDE;">每一层的 ID 的前 12 位</font>。
+ 下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。

**运行nginx容器**日志如下：

```plain
root@iZwz9fcf5vfrvbtzhk81111:/data/docker-registry# docker run -it --rm nginx:latest bash
root@8856f0a79797:/# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

+ 从主服务器（root@iZwz9fcf5vfrvbtzhk81111），切换到容器（root@8856f0a79797） 说明执行成功了；
+ `docker run -it --rm nginx:latest bash`:
    - `-it` ：-i ，交互式操作，-t，终端，以 bash 命令方式交互；
    - `--rm` ：容器退出后随之删除，默认情况，退出容器不会立即删除，除非执行 `docker rm`
    - `nginx:latest` ：以 nginx:latest 镜像为基础启动容器
+ `cat /etc/os-release`：查看容器基本信息
+ 退出容器：exit，如果没退出则连续执行两次 exit

### 1.2 镜像操作
##### 列出镜像
+ `docker image ls`：列出已经下载的镜像，只显示顶层镜像；
+ `docker image ls -a`：列出包括中间层的所有镜像；
+ `docker image ls nginx` ：列出 nginx 镜像；
+ `docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"` ：格式化查看镜像；
+ `docker system df`：查看镜像、容器、数据卷所占用的空间。

##### 删除镜像
可使用 `docker image rm` 命令删除本地镜像，格式形如：`docker image rm [选项] <镜像1> [<镜像2> ...]`

+ <镜像> 可以是 <font style="color:#601BDE;">镜像短 ID、镜像长 ID、镜像名 或 镜像摘要</font>。
+ 查看镜像摘要：`docker image ls --digests`



**删除镜像的理解：**

+ 删除行为分为：<font style="color:rgb(80, 80, 80);background-color:rgba(127, 127, 127, 0.12);">Untagged</font> 、<font style="color:rgb(80, 80, 80);background-color:rgba(127, 127, 127, 0.12);">Deleted</font> ，因为一个镜像可能会有多个标签，只有所有标签都删除后，镜像才会删除

```plain
# 拉取不同标签的相同镜像
docker pull nginx:latest
docker pull nginx:1.21

# 删除 nginx:1.21
docker image rm nginx:1.21

# 输出，镜像层还被 nginx:latest 引用
-->Untagged: nginx:1.21

# 删除最后一个标签
docker image rm nginx:latest

# 输出 Untagged 、Deleted
-->
Untagged: nginx:latest
Deleted: sha256:2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f
Deleted: sha256:e379e8aedd4d72bb4c529a4ca07a4e4d230b5a1d3f7a61bc80179e8f02421ad8
...
```

+ 镜像是多层存储结构，删除镜像是从上层向基础层方向依次进行判断删除。如果当前层没有任何层依赖，则可删除；如果有镜像依赖镜像的当前层，则不会触发删除该层的行为。
+ 若某镜像启动的容器存在（即便容器没运行），镜像也不可删除。

### 1.3 使用 docker commit 理解镜像构成
镜像是多层存储，每一层是在前一层的基础上进行修改；容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

Docker 提供了 `docker commit` 命令，可以将容器的存储层保存下来成为镜像（<font style="color:#601BDE;">在原有镜像的基础上，叠加上容器的存储层，构成新的镜像</font>）。

语法：`docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]`

_注：使用 _`_docker commit_`_ 生成的镜像会添加大量的无关内容，使镜像变得臃肿；并且很难清除知道制作镜像过程中，执行过什么命令、怎么生成的镜，被称为__**黑箱镜像**__。_

_不使用 docker commit 定制镜像，定制镜像应该使用 Dockerfile 来完成。_

### 1.4 使用 Dockerfile 定制镜像
镜像的定制实际上是<font style="color:#601BDE;">定制每一层所添加的配置、文件</font>。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么上一节提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

在一个空白目录中，建立一个文本文件，并命名为 Dockerfile：

```plain
mkdir mynginx
cd mynginx
vim Dockerfile
```

Dockerfile 文件类容：

```plain
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

两个命令：

+ `FROM`：指定基础镜像
+ `RUN`：用来执行命令行命令，两种格式：
    - shell 格式：`RUN <命令>`
    - exec 格式：`RUN ["可执行文件", "参数1", "参数2"]`

##### 构建镜像
使用 `docker build` 命令进行镜像构建，格式为：

```latex
docker build [选项] <上下文路径/URL/->
```

当构建镜像时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

`docker build` 的工作原理：Docker 在运行时分为 Docker 引擎（服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 [Docker Remote API](https://docs.docker.com/develop/sdk/)，而如 `docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。

### 1.5 实现原理
每个镜像都由很多层次构成，Docker 使用 **Union FS** 将这些不同的层结合到一个镜像中去。

通常 Union FS 有两个用途：可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录下；将一个只读的分支和一个可写的分支联合在一起，Live CD 正是基于此方法可以在镜像不变的基础上允许用户在其上进行一些写操作。

Docker 在 OverlayFS 上构建的容器也是利用了类似的原理。

## 2 Dockerfile 指令详解
Dockerfile 指令除了上述介绍的 `FROM、RUN`，还有如下指令：

+ **COPY 复制文件**

```tcl
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```

+ **ADD 更高级的复制文件**  
和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。在 Docker 官方的 Dockerfile 最佳实践文档中要求，尽可能的使用 COPY，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。
+ **CMD 容器启动命令**

```plain
shell 格式：CMD <命令>
exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
```

参数列表格式：CMD ["参数1", "参数2"...]。在指定了 `ENTRYPOINT` 指令后，用 CMD 指定具体的参数。容器是进程，在容器启动时，需指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令，如：`docker run -it ubuntu cat /etc/os-release`。

+ **ENTRYPOINT 入口点**  
ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 `--entrypoin`t 来指定。
+ **ENV 设置环境变量**

```latex
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

## 3 操作容器
容器是独立运行的一个或一组应用，以及它们的运行态环境。

### 3.1 启动
启动容器有两种方式：

+ 基于镜像新建一个容器并启动，使用 `docker run`

```plain
docker run ubuntu:18.04 /bin/echo 'Hello world'
```

+ 将在终止状态（stopped）的容器重新启动，使用 `docker container start`

**守护态运行：**

可以添加 -d 参数，让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。如：

```plain
docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

**进入容器：**

当容器使用 -d 参数进入后台后，可以通过 `docker exec` 命令进入容器。

```plain
[root@izwz98jvb8bcz3rf5ydvzpz mynginx]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
51fdf30cc6c4        nginx:v2            "/docker-entrypoint.…"   12 seconds ago      Up 12 seconds              80/tcp              suspicious_morse
5d3b8ccf0bb4        hello-world         "/hello"                 4 hours ago         Exited (0) 6 minutes ago                       reverent_jepsen
[root@izwz98jvb8bcz3rf5ydvzpz mynginx]# docker exec -it  51fdf30cc6c4 bash
```

### 3.2 终止
使用 `docker container stop` 来终止一个运行中的容器。也可使用 `exit` 或 `ctrl+d` 来退出终端时，所创建的容器立刻终止。

```latex
D:\docker\mynginx> docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
b3a59b5dd631        nginx:v2            "/docker-entrypoint.…"   2 hours ago         Up 5 minutes        0.0.0.0:81->80/tcp   web2
D:\docker\mynginx> docker container stop b3a59b5dd631
```

### 3.3 导出和导入
使用 `docker export、docker import` 导出和导入容器快照。

### 3.4 删除
使用 `docker container rm` 来删除一个处于终止状态的容器。使用 `docker container prune` 清理所有处于终止状态的容器。

## 4 访问仓库
仓库（`Repository`）是集中存放镜像的地方。区分于注册服务器（`Registry`），注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。例如对于仓库地址 `docker.io/ubuntu` 来说，`docker.io` 是注册服务器地址，`ubuntu` 是仓库名。

### 4.1 Docker Hub
Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)，大部分镜像可以直接在 Docker Hub 中下载。

##### 登录
可以在 [Docker Hub](https://hub.docker.com/) 上注册一个 docker 账号，然后执行 `docker login` 命令交互式的输入用户名及密码来完成在命令行界面登录，使用`docker logout` 退出登录。

##### 拉取镜像
使用 `docker search + 关键词`，查找官方仓库中的镜像，并使用 `docker pull` 命令下载到本地。

##### 推送镜像
使用 docker push 将自定义镜像推送到 Docker Hub。

### 4.2 私有仓库


## 5 数据管理
### 5.1 数据卷（Volumes）
数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

+ 数据卷可以在容器之间共享和重用
+ 对数据卷的修改会立马生效
+ 对数据卷的更新，不会影响镜像
+ 数据卷默认会一直存在，即使容器被删除

**常用命令：**

```plain
// 创建数据卷
docker volume create myvol
// 查看数据卷列表
docker volume ls
// 查看指定 数据卷 的信息
docker volume inspect myvol
// 删除数据卷
docker volume rm myvol
// 清理无主数据卷
docker volume prune
```

**启动一个挂载数据卷的容器：**

在用 docker run 命令的时候，使用 --mount 标记来将 数据卷 挂载到容器里。在一次 docker run 中可以挂载多个 数据卷。

```plain
 docker run -d -P \
    --name web \
    # -v myvol:/usr/share/nginx/html \
    --mount source=myvol,target=/usr/share/nginx/html \
    nginx:alpine
```

### 5.2 挂载主机目录 (Bind mounts)
使用 --mount 标记可以指定挂载一个本地主机的目录到容器中去：

```plain
docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

在主机里使用以下命令可以查看 web 容器的信息:

```plain
docker inspect web
```

--mount 标记也可以从主机挂载单个文件到容器中：

```plain
docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
   bash
```

# 参考资料
+ [Docker 官方文档](https://docs.docker.com/)
+ [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)

