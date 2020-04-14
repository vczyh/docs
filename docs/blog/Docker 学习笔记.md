---
title: Docker学习笔记
date: 2020-2-2 14:38:44
updated: 2020-2-3 14:51:40
tags:
- Docker
- Ubuntu
---
## 环境

Ubuntu16.04

## 安装Docker CE

若之前安装过老版本的`docker`，先卸载

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

1.更新`apt`包索引

```bash
sudo apt-get update
```

2.安装软件包支持`apt`使用基于Https的仓库

```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

3.添加Docker官方的GPG key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

验证key

```bash
sudo apt-key fingerprint 0EBFCD88
```

4.添加Docker仓库

```bash
# 官方
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

5.更新`apt`包索引

```bash
sudo apt-get update
```

6.安装最新的Docker CE

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 查看版本信息

查看Docker版本

``` bash
docker --version

Docker version 18.09.3, build 774a1f4
```

查看更多Docker信息，docker version 或者 docker info，这里使用`sudo `，否则会出现权限错误

```bash
$ sudo docker info
```

一般使用docker命令得加上`sudo`，为了避免出现权限错误 ，设置把用户加入到docker组中 [Read more](https://docs.docker.com/engine/installation/linux/linux-postinstall/).

创建docker用户组，可能会提示已经存在

```bash
sudo groupadd docker
```

把用户添加到docker用户组

```bash
sudo usermod -aG docker $USER
```

登出账号 

```bash
logout
```

然后重新登录，现在不加`sudo`也可以使用

```bash
docker info
```

## 测试

通过运行一个简单的`image`来测试Docker

```bash
$ docker run hello-world
```
> Unable to find image 'hello-world:latest' locally
> docker: Error response from daemon: Get https://registry-1.docker.io/v2/library/hello-world/manifests/latest: Get https://auth.docker.io/token?scope=repository%3Alibrary%2Fhello-world%3Apull&service=registry.docker.io: net/http: request canceled (Client.Timeout exceeded while awaiting headers).
> See 'docker run --help'.

**错误解决：**

**打开`/etc/docker/daemon.json`，如果不存在创建文件，添加内容**

```json
{
  "registry-mirrors": [
    "http://141e5461.m.daocloud.io"
  ]
}
```
**重启Docker服务**

```bash
sudo service docker restart
```
再次运行
```bash
$ docker run hello-world
```

查看下载的`image`

```bash
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        2 months ago        1.84kB

```

列出所有的容器，不加`--all`默认列出仍在运行的容器，`hello-world`输出语句后就结束了，因此不加`--all`就看不到`hello-world`容器

```bash
docker container ls --all
```

## Containers

简单来说，`Container`就是包含运行环境，代码，以及构建运行命令的容器，他和传统运行程序的区别就是环境不随宿主机的环境变化，自己拥有自己的一套运行环境，环境变量，可以通过端口与宿主机交流信息。下面示例构建了一个容器：

在新建文件夹下，添加三个文件

**Dockerfile**

 ```dockerfile
# 程序运行需要的环境，也是一个镜像
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# 需要运行的命令，这里使用pip安装需要的模块
# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 暴露80端口
# Make port 80 available to the world outside this container
EXPOSE 80

# 定义环境变量
# Define environment variable
ENV NAME World

# 当容器运行的时候执行的命令
# Run app.py when the container launches
CMD ["python", "app.py"]
 ```

**requirements.txt**

```
Flask
Redis
```

**app.py**

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

### 构建应用

当前的目录结构

```bash
$ ls
Dockerfile		app.py			requirements.txt
```

`--tag`设置镜像的名字，可以使用`-t`代替

```bash
docker build --tag=friendlyhello
```

查看镜像，这里显示了所有本地镜像，包括刚构建的以及从远程仓库下载的

```bash
$ docker image ls
```

### 运行应用

```bash
$ docker run -p 4000:80 friendlyhello
```

`-p 4000:80`将宿主机的4000端口映射到容器的80端口

访问`http://宿主机IP:4000`

![mark](http://p.vczyh.com/blog/20190303/lwaBhpRnXtEM.png?imageslim)

在终端按`CTRL+C`停止容器

**在后台运行app**

```bash
$ docker run -d -p 4000:80 friendlyhello

d5d06d6b7efc6c9e8e2e730ee0139c78200951743df80afd0ce2bc604da917a1
```

`d5d06d6b7efc6c9e8e2e730ee0139c78200951743df80afd0ce2bc604da917a1`是容器ID，每个运行的容器实例都有一个ID

**查看运行的容器**

```bash
$ youthful_mirzakhani
```

**停止运行app**

```bash
 docker container stop d5d06d6b7efc
```

**或者**

```bash
docker container stop youthful_mirzakhani
```

`youthful_mirzakhani`是实例的`NAMES`属性

### 分享镜像

**使用官方Registry非常慢，这里只是学习一下，建议使用其他Registry** [阿里云容器镜像服务的基本使用](https://blog.csdn.net/TKDK_bot/article/details/88097069)

Docker支持将镜像上传到一个类似`Github`和`Maven`的公共仓库，需要在 [hub.docker.com](https://hub.docker.com/) 注册账号

**登录Docker账号**

```bash
$ docker login
```

**为镜像打标签**

格式

```bash
$ docker tag image username/repository:tag
```

例如

```bash
$ docker tag friendlyhello vczyh/get-started:part2
```

```bash
$ docker image ls
```

**发布镜像**

```bash
$ docker push username/repository:tag
```

**拉取镜像并运行**

```bash
$ docker run -p 4000:80 username/repository:tag
```

## Services

在分布式应用中，不同的部分被称为服务。比如，你想做一个视频分享网站，网站可能包括存储服务，用户上传视频后的转码服务，前台服务等。

服务实际上只是生产中的容器，一个服务仅运行一个镜像，但它通过编码决定了镜像的运行方式——使用的端口，运行副本的数量，便于容器拥有它所需要的容量等。拓展服务改变了运行软件的容器实例数量，从而为进程中的服务分配更多的资源。

幸运的是，使用Docker平台定义、运行和扩展服务非常容易——只需编写一个`docker-compose.yml`文件。

`docker-compose.yml`文件是一个YAML文件，它定义Docker容器在生产环境中的行为。

 **docker-compose.yml**

``` yaml
version: "3"
services:
  # 定义一个叫web的服务
  web:
  	# 镜像，本地或者远程
    # replace username/repo:tag with your name and image details
    # image: username/repo:tag
    image: registry.cn-hangzhou.aliyuncs.com/zhangyuheng/hello 
    deploy:
      # 包含5个副本
      replicas: 5
      resources:
        limits:
          # 每个副本最多使用10%的CPU
          cpus: "0.1"
          # 每个副本最多使用50M内存空间
          memory: 50M
      restart_policy:
        # 如果有任何一个失败，则立即重启
        condition: on-failure
    ports:
      # 将主机4000端口映射到web服务的80端口
      - "4000:80"
    networks:
      # 指示web容器通过称为webnet的负载均衡网络共享80端口(在内部，容器本身在一个临时端口上发布到web的80端口。)
      - webnet
networks:
  # 使用默认设置定义webnet网络(一个负载均衡的覆盖网络)。
  webnet:
```

### 运行你的负载均衡应用

首先先运行：

```bash
$ docker swarm init
```

`getstartedlab`是app的名字：

```bash
$ docker stack deploy -c docker-compose.yml getstartedlab

Creating network getstartedlab_webnet
Creating service getstartedlab_web
```

现在单个服务堆栈在一台主机上运行了5个已部署镜像的实例

查看服务：

```bash
$ docker service ls
```

也可以这样查看服务：

```bash
$ docker stack services getstartedlab 
```

服务中运行的单个容器称为任务。任务被赋予惟一的id，这些id在数字上递增，直到您在`docker-comp.yml`中定义的副本的数量。

列出你的服务任务：

```bash
$ docker service ps getstartedlab_web 
```

也可以通过查看容器来查看任务：`-q`仅仅列出容器ID

```bash
$ docker container ls
```

浏览器多次访问：`http://localhost:4000`

或者多次输入：

```bash
$ curl -4 http://localhost:4000
```

可以看到显示的**Hostname**(**容器ID**)每次都不一样，也验证了轮询方式的负载均衡

显示一个**stack**的所有任务：

```bash
$ docker stack ps getstartedlab
```

### 拓展应用

可以通过改变`replicas` 的值来拓展应用：

**docker-compose.yml**

```yaml
 deploy:
      # 包含5个副本
      replicas: 7
```

```bash
$ docker stack deploy -c docker-compose.yml getstartedlab
```

Docker更新服务不需要关掉服务或者关掉任何容器

查看目前的容器和任务：

```bash
$ docker stack ps getstartedlab
```

### 移除应用

```bash
$ docker stack rm getstartedlab
```

### 移除集群

```bash
$ docker swarm leave --force
```

### 常用命令

```bash
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```

## Swarms

### 安装docker-machine

```bash
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

这个岂止是慢，根本就没速度，所以采用手动安装，先去`https://github.com/docker/machine/releases/`下载，这里下载也得翻墙==

下载好安装：

```bash
$ sudo install docker-machine-Linux-x86_64 /usr/local/bin/docker-machine
```

验证：

```bash
$ docker-machine version

docker-machine version 0.16.0, build 702c267f
```

### 创建虚拟机集群

安装virtualbox驱动：

```bash
$ sudo apt install virtualbox
```

创建名为`myvm1`的虚拟机：

```bash
$ docker-machine create --driver virtualbox myvm1

Running pre-create checks...
Error with pre-create check: "This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory"
```

出现错误，这里使用的是VMware，解决：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031301111025.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RLREtfYm90,size_16,color_FFFFFF,t_70)

```bash
$ docker-machine create --driver virtualbox myvm1
```

从github下载`iso`文件非常慢，[采用手动下载](https://github.com/boot2docker/boot2docker/releases)，[参考](https://segmentfault.com/a/1190000017001848)：

把`iso`文件放到默认位置：

```bash
$ cp boot2docker.iso ~/.docker/machine/cache/
```

```bash
$ docker-machine create --driver virtualbox myvm1

Running pre-create checks...
Creating machine...
(myvm1) Copying /home/vczyh/.docker/machine/cache/boot2docker.iso to /home/vczyh/.docker/machine/machines/myvm1/boot2docker.iso...
(myvm1) Creating VirtualBox VM...
(myvm1) Creating SSH key...
(myvm1) Starting the VM...
(myvm1) Check network to re-create if needed...
(myvm1) Found a new host-only adapter: "vboxnet0"
(myvm1) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env myvm1

```

同样，再创建一个名为`myvm2`的虚拟机：

```bash
$ docker-machine create --driver virtualbox myvm2
```

查看虚拟机：

```bash
$ docker-machine ls  
```

### 连接虚拟机

```bash
$ docker-machine --native-ssh ssh myvm1
docker@myvm1:~$ 
```

### 初始化swarm以及添加节点

连接`myvm1`后：指定`myvm1`为`swarm manager`，`192.168.99.100`是`myvm1`的IP

```bash
docker@myvm1:~$ docker swarm init --advertise-addr 192.168.99.100
```

`CTRL+D`退出连接，连接`myvm2`后：设置`myvm2`为`swarm worker`

```bash
docker@myvm2:~$ docker swarm join --token SWMTKN-1
```

现在一个swarm已经创建完成

在`swarm manager(myvm1)`中查看所有节点：

```bash
docker@myvm1:~$ docker node ls
```

### 另外一种与虚拟机对话方式

```bash
$ docker-machine env myvm1

export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```

运行：

```bash
eval $(docker-machine env myvm1)
```

现在实现了和ssh连接`myvm1`相同的结果:

```bash
docker node ls
```

好处之一就是虚拟机可以直接使用宿主机的文件

**断开对话：**

```bash
eval $(docker-machine env -u)
```

### 部署应用到swarm

使用`eval $(docker-machine env myvm1)`打开与`myvm1`的对话

```bash
$ docker stack deploy -c docker-compose.yml getstartedlab
```

如果`docker-compose.yml`里使用的镜像不是`Docker Hub`的：

```bash
docker login registry.example.com

docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab
```

```bash
$ docker stack ps getstartedlab 
```

现在5个实例(容器)已经分布在俩个虚拟机中

查看`myvm1`的容器：其他的3个实例在`myvm2`中

```bash
$ docker container ls
```

### 测试

宿主机运行：`192.168.99.101`是任意一个虚拟机的IP

```bash
curl 192.168.99.101:4000
```

每次访问的`Hostname`不同，证实了集群的负载均衡

### 拓展应用

第一种：修改`docker-compose.yml`

第二种：使用`docker swarm join`为集群添加机器

这俩种操作之后，重新执行`docker stack deploy -c docker-compose.yml getstartedlab`

### 关闭应用

关闭`stack`

```bash
docker stack rm getstartedlab
```

关闭` swarm`:

```bash
docker swarm leave  // worker
docker swarm leave --force // manager
```

### 常用命令

```bash
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```



## Docker其他配置

### 镜像配置

`/etc/docker/daemon.json`

```json
{
  "registry-mirrors": [
    "https://d1zbnocg.mirror.aliyuncs.com",
    "http://141e5461.m.daocloud.io"
  ]
}
```

```bash
sudo service docker restart
```

### DNS配置

Docker实例里的DNS可能出问题，所以配置Docker使用的DNS

`/etc/docker/daemon.json`

```json
{
  "dns": ["your_dns_address", "8.8.8.8"]
}
```

```bash
sudo service docker restart
```

