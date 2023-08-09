## Docker简介

### 1. 特点

### 2. 基本组成

#### 2.1 容器（container）

#### 2.2 镜像（image）

一个只读的模板，可以用来创建多个容器。

#### 2.3 仓库（repository）

### 3. 安装

1. 卸载老版本

   ```shell
   sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

2. 安装gcc依赖

   ```shell
   yum -y install gcc
   yum -y install gcc-c++
   ```

3. 设置仓库下载源

   ```shell
   yum install -y yum-utils
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

4. 安装docker引擎

   ```shell
   yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

   如果出现 podman 、containerd 等包冲突，执行以下命令，再重新执行上述安装命令

   ```shell
   yum install https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.4.3-3.1.el8.x86_64.rpm --allowerasing
   ```

5. 启动docker

   ```shell
   systemctl start docker
   ```

6. 运行 hello world

   ```shell
   docker run hello-world
   ```

7. 停止docker

   ```shell
   systemctl stop docker
   ```

8. 卸载docker

   ```shell
   yum remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   
   rm -rf /var/lib/docker
   rm -rf /var/lib/containerd
   ```

### 4. 命令

#### 4.1 镜像命令

- docker images：列出所有镜像
- docker search xxx：查询某个镜像
- docker pull xxx：拉取某个镜像
- docker system df：查看镜像/容器/数据卷所占的空间
- docker rmi 镜像id：删除某个镜像 
- docker rmi -f 镜像id：强行删除某个镜像

#### 4.2 容器命令

##### 4.2.1 新建启动

`docker run [OPTIONS] image [COMMAND][ARG...]`

OPTIONS

- --name=”xx”：为容器指定一个名称
- -d：后台运行容器，守护式
- -it：提供伪终端等待交互
- -P：随机端口映射
- -p：指定端口映射

##### 4.2.2 正在运行的容器

`docker ps [OPTIONS]`

OPTIONS

- -a：列出当前所有正在运行的和历史上运行过的容器
- -l：列出最近创建的容器
- -n：列出最近n个创建的容器
- -q：静默模式，只显示容器编号

##### 4.2.3 退出容器

停止容器：`exit `

不停止容器：`ctrl + p + q`

##### 4.2.4 已停止运行的容器

启动容器id或容器名 `docker start xxx`

重启容器 `docker restart xxx`

停止容器 `docker stop xxx`

强制停止容器 `docker kill xxx`

删除已停止的容器 `docker rm <cid>`

##### 4.2.5 守护式启动

`docker run -d 容器`

Docker容器后台运行，必须有一个前台进程。除非一直挂起的命令，就会自动退出。

##### 4.2.6 查看容器

查看容器日志 `docker logs <cid>`

查看容器内运行的进程 `docker top <cid>`

查看容器内部细节 `docker inspect <cid>`

##### 4.2.7 进入容器

`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`

OPTIONS

- -d：后台执行
- -i：交互
- -t：命令行终端

`docker attach <cid>`

attch 直接进入容器启动命令的终端，不会启动新进程，用exit退出，会导致容器停止

exec在容器中打开新终端，并且可以启动新进程，用exit退出，不会导致容器停止

##### 4.2.8 复制容器文件到宿主机

`docker cp 容器id:容器路径 宿主机路径`

##### 4.2.9 导入导出容器

导出容器 `docker export <cid> > file.tar`

导入容器 `cat file.tar | docker import - 用户/镜像:版本号`