一、学习docker基础知识

1、docker build \docker pull \docker run  

2、docker registry

3、容器仓库，使用dockerhub管理镜像，公共存储领域，使用镜像先默认下载到本地，使用https协议

4、docker HOST与 registry使用https协议，client与docker HOST之间也是使用https、http协议（you can run your private registry）
  docker registry、docker HOST、docker client

5、通常一个docker只放一个应用程序，通过应用程序名就是这个docker的镜像名。用仓库名和标签（tag）用来标识一个镜像，eg:nginx:stable稳定版
、nginx:latest最新版

6、Docker Images

    镜像是静态的，不会运行，而容器是动态的，有生命周期，
    images,containers,network,plugins,volumes，对其进行增删改查
    镜像是只读的
    
二、安装docker基础

1、配置yum
   
    wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    yum list docker-ce.x86_64  --showduplicates |sort -r
    yum install -y --setopt=obsoletes=0 docker-ce-18.06.1.ce-3.el7
    
2、启动docker
  
    systemctl start docker
    systemctl enable docker
    
3、配置镜像加速
  
    vim /etc/docker/daemon.json 
    {
       "registry-mirrors":["https://registry.docker-cn.com"]
    }
    systemctl restart docker
    
4、docker命令解析
  
    docker
    Management Commands:
      config      Manage Docker configs
      container   Manage containers
      image       Manage images
      network     Manage networks
      node        Manage Swarm nodes
      plugin      Manage plugins
      secret      Manage Docker secrets
      service     Manage services
      stack       Manage Docker stacks
      swarm       Manage Swarm
      system      Manage Docker
      trust       Manage trust on Docker images
      volume      Manage volumes
    docker info
    
5、docker常见操作
  
    docker search:  Search the Docker Hub for images 
    docker pull:  Pull an image or a repository from a registry
    docker images:  List images
    docker create:  Create a new container
    docker start: Start one or more stopped containers
    docker run: Run a command in a new container
    docker attach:  Attach to a running container
    docker ps:  List containers
    docker logs:  Fetch the logs of a container
    docker restart: Restart a container
    docker stop:  Stop one or more running containers
    docker kill:  Kill one or more running containers
    docker rm:  Remove one or more containers
    
6、docker images操作

    docker images  --quiet --no-trunc
    [root@localhost docker]# docker images  --quiet --no-trunc
    sha256:a02eab9e24344139c38beb02d6428cc5981015d8ca6d8cdf707d98c4c2b5b184
    sha256:08bef618c30adc2eebe5c590150c7f16f20a0dcec1152171e78a3a028df92545
    sha256:2d51edfd5961044e58367a1985e7c09f31648459b72e5564bc0395adfa42c5d3
    sha256:e1ddd7948a1c31709a23cc5b7dfe96e55fc364f90e1cebcde0773a1b5a30dcda

    docker images --all
    [root@localhost docker]# docker images --all
    REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
    mysql/mysql-server        latest              a02eab9e2434        4 weeks ago         276MB
    redis                     4-alpine            08bef618c30a        3 months ago        28.6MB
    portainer/agent           latest              2d51edfd5961        3 months ago        10.1MB
    busybox                   latest              e1ddd7948a1c        3 months ago        1.16MB
    portainer/portainer       <none>              6827bc26a94d        4 months ago        58.5MB

7、docker containers操作

    docker run 
    Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    说明：
    -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

    -d: 后台运行容器，并返回容器ID；

    -i: 以交互模式运行容器，通常与 -t 同时使用；

    -p: 端口映射，格式为：主机(宿主)端口:容器端口

    -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

    --name="nginx-lb": 为容器指定一个名称；

    --dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

    --dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

    -h "mars": 指定容器的hostname；

    -e username="ritchie": 设置环境变量；

    --env-file=[]: 从指定文件读入环境变量；

    --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

    -m :设置容器使用内存最大值；

    --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

    --link=[]: 添加链接到另一个容器；

    --expose=[]: 开放一个端口或一组端口； 

    实例

    使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。

    docker run --name mynginx -d nginx:latest

    使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。

    docker run -P -d nginx:latest

    使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

    docker run -p 80:80 -v /data:/data -d nginx:latest

    绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。

    $ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash

    使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

    runoob@runoob:~$ docker run -it nginx:latest /bin/bash
    root@b8573233d675:/# 
