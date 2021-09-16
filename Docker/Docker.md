# Docker

环境：树莓派 Ubuntu 20.04 64位

## 安装

使用官方安装脚本自动安装，安装命令如下（亲测）：

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```shell
curl -sSL https://get.daocloud.io/docker | sh
```

### 镜像加速

阿里云镜像获取地址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://THIS_IS_MY_ID.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 常用命令

### 帮助命令

#### `docker version`

查看 docker 版本信息

#### `docker info`

查看 docker 的详细信息

#### `docker --help`

查看 docker 帮助手册

### 镜像命令

#### `docker images`

列出本地的镜像

##### `-a` all

列出本地所有的镜像（含中间镜像层）

##### `-q` quiet

静默模式，只显示容器编号

##### `--digests`

显示镜像的摘要信息

##### `--no-trunc`

显示完整的镜像信息（完整的 IMAGE ID）

#### `docker search xxx `

在[官网](https://hub.docker.com)搜索镜像

##### `-f=stars=*` filter

列出收藏数不小于指定值的镜像

##### `--no-trunc`

显示完整的镜像描述

##### `--automated`

只列出 automated build 类型的镜像

#### `docker pull xxx[:TAG]`

下载镜像，省略 TAG 等价于 `:latest`

#### `docker rmi xxx ` remove image

删除镜像

##### `-f` force

强制删除

##### ``docker rmi -f $(docker images -qa)``

全部删除

### 容器命令

#### docker run

docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

新建并启动容器

##### `-i` interactive

以**交互**模式运行容器，通常与 `-t` 同时使用

##### `-t` tty

为容器重新分配一个伪输入**终端**，通常与 `-i` 同时使用

##### `--name`

为容器指定名称

##### `-d` detach

以后台模式启动容器

==Docker 容器后台运行，就必须有一个前台进程==

##### `-p 宿主机ip:Docker内ip` publish

指定端口映射

4 种格式：

1. ip:hostPart:containerPort
2. ip::containerPort
3. hostPort:containerPort
4. containerPort

##### `-P` publish-all

随机端口映射

#### `docker ps`

列出当前所有**正在运行**的容器

##### `-a` all

列出当前所有**正在运行**和**历史上运行过**的容器

##### `-l` latest

列出最近创建的容器

##### `-n`

显示最近 n 个容器

##### `-q` quiet

静默模式，只显示容器编号

##### `--no-trunc`

不截断输出

#### 容器内：`exit`

退出并停止容器（==出门关灯==）

#### 容器内：`Ctrl + P + Q`

退出但不停止容器（==出门没关灯==）

#### `docker restart 容器id/name`

重启容器

#### `docker stop 容器id/name`

停止容器

#### `docker kill 容器id/name`

强制停止容器

#### `docker rm 容器id`

删除已停止的容器

删除全部容器：

1.   `docker rm -f $(docker ps -aq)`
2.   `docker ps -aq | xargs docker rm`

#### `docker logs 容器id`

查看容器日志

##### `-t` timestamps

加入时间戳

##### `-f` follow

跟随最新的日志打印

##### `--tail n`

显示最后 n 条

#### `docker top 容器id`

查看容器内运行的进程

#### `docker inspect 容器id`（inspect 检查）

查看容器内部细节

#### `docker attach 容器id`

连接到正在运行中的容器（进入容器启动命令的终端，不会启动新的进程）

#### `docker exec 命令参数 容器id 执行命令` execute

在运行的容器中执行命令（不需要进到容器内，==隔山打牛==）

##### `-d` detach

以后台模式执行

##### `-i` -interactive

创建标准 IO 接口，如果只有 i，虽然也可以输入命令得到输出，但是结果很不友好

##### `-t` tty

伪造 tty 终端，如果只有 t，输入什么也没有反应，因为没有开放对应的输入接口

#### `docker exec -it 容器id bashShell`

在正在运行中的容器中打开一个新的终端并进入（一个新的进程）

####  `docker cp 容器id: 容器内文件 拷贝到宿主机的位置`

从容器内拷贝文件到主机上

#### 总结

![Docker 命令图表](C:/Users/V-arc/Desktop/Docker/Docker.assets/Docker%20%E5%91%BD%E4%BB%A4%E5%9B%BE%E8%A1%A8.png)

官方文档：[Docker run reference | Docker Documentation](https://docs.docker.com/engine/reference/run/)

## 镜像

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到**镜像的顶部**，这一层通常被称为**容器层**，容器层之下都叫**镜像层**

### Commit

`docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]`

（message author）

将容器封装为镜像

## 容器数据卷 volumes

类似 Redis 里面的 rdb 和 aof 文件（个人理解：类似 VMware 的共享文件夹）

### 特点

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止

### 作用

1. 容器的持久化

2. 容器间继承、共享数据

### 添加

#### 直接命令添加

`docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名`

限制容器内对数据卷的只读权限：`docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名` read only

#### Dockerfile 添加

此处不详细介绍 Dockerfile，详见下一节

`VOLUME ["/dataVolumeContainer1", "dataVolumeContainer2"]`

> Docker 挂载主机目录 Docker 访问出现 cannot open directory . Permission denied
> 解决办法：在挂载目录后多加一个 `--privileged=true` 参数即可

### 数据卷容器

命名的容器挂载数据卷，其它容器通过挂载这个“父容器”实现数据共享，挂载数据卷的容器，称之为**数据卷容器**。

>活动硬盘上面挂活动硬盘，实现数据的传递依赖。——周阳
>
>插排上面插插排。——弹幕
>
>**个人认为可以类比 Linux 中的硬连接。**

#### `--volumes -from 父容器`

在运行容器的时候添加该命令可以使当前运行的容器继承父容器的数据卷~~，父容器即为“数据卷容器”~~。



## Dockerfile

定义：Dockerfile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本。

### 构建过程

$Dockerfile \stackrel{build}\longrightarrow Image \stackrel{run}\longrightarrow Container$

从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段：

- Dockerfile 是软件的原材料（==Java 代码==）
- Docker 镜像是软件的交付品（==exe 文件==）
- Docker 容器则可以认为是软件的运行态（==加载到内存后的应用==）

Dockerfile 面向开发，Docker 镜像称为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石。

### 体系结构（保留字指令）

#### `FROM`

指定**基础镜像**

格式：`FROM 敬香茗`

#### `MAINTAINER`

标注镜像维护者的**姓名**与**邮箱地址**

格式：`MAINTAINER 名称<邮箱地址>`

#### `RUN`

添加容器**构建**时需要在容器内**运行**的**命令**

两种格式：

1. shell 格式：`RUN 命令`
2. exec 格式：`RUN ["可执行文件", "参数1", "参数2" ...]`

#### `EXPOSE`

指明当前容器对宿主机**暴露**出的**端口**

格式：`EXPOSE 端口号`

#### `WORKDIR`

指定在创建容器后，终端默认登陆进来的**工作目录**，默认为根目录

格式：`WORKDIR 目录`

#### `ENV`

在构建镜像过程中设置**环境变量**

格式：`ENV 环境变量名 路径`

#### `ADD`

将宿主机目录下的文件**拷贝**进镜像，并自动处理 URL 和**解压** tar 压缩包

格式：`ADD 宿主机文件 解压目录`

#### `COPY`

将宿主机目录下的文件**拷贝**进镜像

两种格式：

1. `COPY 宿主机文件 容器内位置`
2. `COPY ["宿主机文件", "容器内位置"]`

#### `VOLUME`

添加**容器数据卷**，用于数据保存和持久化工作

格式：`VOLUME ["容器内数据卷目录1", "容器内数据目录2"]`

#### `CMD`

指定一个容器**启动**时要**运行**的**命令**。只生效一条命令，意味着：

1. Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效
2. CMD 会被 docker run 之后的参数替换

两种格式：

1. shell 格式：`CMD <命令>`
2. exec 格式：`CMD ["可执行文件", "参数1", "参数2" ...]`

#### `ENTRYPOINT`

指定一个容器**启动**时要**运行**的**命令**，docker run 之后的参数会**追加**到命令中，形成新的命令组合

两种格式：

1. shell 格式：`CMD <命令>`
2. exec 格式：`CMD ["可执行文件", "参数1", "参数2" ...]`

#### `ONBUILD`

当构建一个被继承的 Dockerfile 时运行命令，父镜像在被子继承后父镜像的 onbuild 被触发，类似一个**触发器**

格式：`ONBUILD 其他指令`，如 `ONBUILD RUN echo "hello"`

### 其他相关命令

#### `docker build`

创建镜像

##### `-f` file

指明 Dockfile 文件的位置，不指明默认为当前目录的“Dockerfile”（规范名称）

##### `-t` tag

为当前构建的镜像命名并声明标签

##### `docker build -f Dockerfile路径 -t 新镜像名字:TAG . `

使用指定的 Dockerfile 再当前路径下构建镜像

#### `docker history 镜像id`

查看指定镜像的创建历史

### 总结

![Dockerfile 命令图表](C:/Users/V-arc/Desktop/Docker/Docker.assets/Dockerfile%20%E5%91%BD%E4%BB%A4%E5%9B%BE%E8%A1%A8.png)

## 常用安装

### 整体步骤

1. **搜**索镜像
2. **拉**取镜像
3. 查**看**镜像
4. **启动**镜像
5. **停止**容器
6. **移除**容器

## 本地镜像发布到阿里云

### 流程

![阿里云 ECS Docker 生态](Docker.assets/%E9%98%BF%E9%87%8C%E4%BA%91%20ECS%20Docker%20%E7%94%9F%E6%80%81.png)

1. 在阿里云创建镜像仓库

    - [命名空间的基本操作 (aliyun.com)](https://help.aliyun.com/document_detail/60765.html)
    - [构建仓库与镜像 (aliyun.com)](https://help.aliyun.com/document_detail/60997.html)

    设置代码源选择本地

2. 将镜像推送到镜像仓库

>## 1. 登录阿里云Docker Registry
>
>```
>$ docker login --username=我的阿里云账号全名 registry.cn-hangzhou此处的域名与仓库所在地域有关.aliyuncs.com
>```
>
>用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。
>
>您可以在访问凭证页面修改凭证密码。
>
>## 2. 从Registry中拉取镜像
>
>```
>$ docker pull registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名称:[镜像版本号]
>```
>
>## 3. 将镜像推送到Registry
>
>```
>$ docker login --username=我的阿里云账号全名 registry.cn-hangzhou.aliyuncs.com$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名称:[镜像版本号]$ docker push registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名称:[镜像版本号]
>```
>
>请根据实际镜像信息替换示例中的[ImageId]和[镜像版本号]参数。
>
>## 4. 选择合适的镜像仓库地址
>
>从ECS推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。
>
>如果您使用的机器位于VPC网络，请使用 registry-vpc.cn-hangzhou.aliyuncs.com 作为Registry的域名登录。
>
>## 5. 示例
>
>使用"docker tag"命令重命名镜像，并将它通过专有网络地址推送至Registry。
>
>```
>$ docker images
>REPOSITORY						TAG			IMAGE ID		CREATED 		VIRTUAL	SIZE
>registry.aliyuncs.com/acs/agent	0.7-dfb6816	37bb9c63c8b2	7 days ago		37.89 MB
>$ docker tag 37bb9c63c8b2 registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816
>```
>
>使用 "docker push" 命令将该镜像推送至远程。
>
>```
>$ docker push registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816
>```

