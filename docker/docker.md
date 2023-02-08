# 基础概念

- 镜像
- 容器
- 宿主机
- 服务
- Dockerfile
- docker-compose
- 数据卷
  容器在运行过程中产生的数据随着容器销毁就会消失，为了持久化存储数据，可以使用数据卷
- 网络
  - 局域网，容器之间通信
  - 端口与宿主机映射，让容器与外部环境通信

# docker命令

## 镜像相关

**拉取仓库**

```shell
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

# 简单例子
docker pull alpine
```

**列出镜像**

```shell
docker images
# 输出
REPOSITORY       TAG             IMAGE ID         CREATED             SIZE
alpine           latest        965ea09ff2eb      7 weeks ago         5.55MB
```

**操作镜像**

```shell
# 相当于 docker images
docker image ls 

# 查看镜像的摘要
docker image ls --digests

# 通过摘要删除镜像
docker image rm [镜像摘要]

# 删除虚悬镜像，也就是镜像名称、标签都为 none 的镜像
docker image prune

# 查看镜像的历史记录
docker image history [镜像名]
```

**构建镜像**

```shell
# build 后面还有一个 . 这是因为 build 的工作原理需要一个上下文以方便一些如 COPY 指令的执行，这个 . 表示将当前工作区间作为 build 的上下文
docker build .
```

## 容器相关

**run**

基于镜像启动容器，每次run都会手动启动一个容器，及时它停止运行，依旧存在，可以通过--rm参数来停止即销毁

```shell
# 简单例子，-it 将容器的输入输出重定向到终端， --rm，终端退出容器即销毁
docker run -it --rm alpine sh

# 参数帮助，启动容器时，可以进行各种配置，如端口映射，网络设置，数据卷设置等等
docker run --help
```

**container**

对容器的各种操作，查看，停止，暂停，启动，重启，枚举，删除等

```shell
# 查看运行中的容器
docker container ls

# 查看所有容器
docker container ls -a

# 启动停止状态的容器
docker container start -ai [容器id] sh

# 进入容器
docker container exec -it [容器id] sh

# 查看容器日志
docker container logs [容器id]

# 删除容器
docker container rm [容器id]

# 删除处于终止状态的容器
docker container prune

# 查看进程，容器就是进程
docker container ps

# 在宿主机和容器之间拷贝文件
docker container cp [源文件地址][目标地址]
docker container cp ./test [容器id:/root/] #将本地文件拷贝到容器内

# 查看容器详细信息
docker container inspect [容器id]
```

**exec**： 连接上正在运行中的容器，通常可携带 -it 参数以及 sh 命令来登录容器，以便操作容器

**logs**： 查看后台运行的容器所产生的日志，因为后台运行，日志不会输出到终端

**stop**： 停止指定容器，通常是为了删除，先停后删

**prune**：删除所有处于终止状态的容器，删通常是为了重建，不新建容器，一些配置的修改不会生效

**inspect**： 查看容器的详细信息，比如端口映射、网络配置、数据卷配置等

## 数据卷相关

**volume**

```shell
# 查看所有数据卷
docker volume ls

# 查看指定数据卷的详情信息
docker volume inspect [数据卷名]

# 创建数据卷
docker volume create [数据卷名]

# 删除数据卷
docker volume rm [数据卷名]

# 删除无主数据卷
docker volume prune
	 
```

## 网络相关

**network**

```shell
# 创建网络, -d 指定类型，有bridge,overlay, -p 参数指定端口映射，--network 参数指定连接的网络
docker network create -d bridge my-net	

# 查看网络列表
docker network ls

# 删除
docker network rm/prune
```

## **其他**

**system df**

```shell
# 查看镜像，容器，卷轴占据的空间
docker system df
# 输出
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              17                  5                   1.861GB             925MB (49%)
Containers          6                   6                   2.124MB             0B (0%)
Local Volumes       4                   2                   652.3MB             334.6MB (51%)
Build Cache         0                   0                   0B                  0B
```

## docker-compose

```shell
# 一键创建启动容器
docker-compose up -d

# 重新创建指定的服务容器
docker-compose up -d --force-recreate [服务名]

# 启动处于终止状态的容器
docker-compose up -d --no-recreate

# 查看配置是否正确
docker-compose config

# 构建指定服务依赖的镜像
docker-compose build [服务名]

# 停止up命令启动的所有容器
docker-compose down

# 查看服务的输出
docker-compose logs [服务名]

# 列出compose管理的容器进程
docker-compose ps

# 查看各个容器内的进程
docker-compose top

# 停止运行中的容器
docker-compose stop [服务名]

# 删除停止状态的容器
docker-compose rm [服务名]
```

