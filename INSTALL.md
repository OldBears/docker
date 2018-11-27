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
    
