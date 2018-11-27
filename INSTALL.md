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
    
5、
