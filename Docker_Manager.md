一、Docker镜像管理

  1、docker images manager
  
    docker 采用分层构建机制，最底层为bootfs，其之为rootfs
    bootfs 用于系统引导的文件系统，包括bootloader和kernel，容器启动完成后会被卸载以节省内存资源
    rootfs 位于bootfs之上，表现为docker的根文件系统，传统模式中，系统重启之时，内核挂载rootfs时会首先将其挂载“只读”模式，完整性校验之后，再重新挂载为
    读写模式。
    docker中，rootfs为只读模式，而后通过“联合挂载”技术额外挂载一个“可写”层。aufs竞争产品是overlayfs
    docker还支持分层镜像，btrfs、devicemapper和vfs
  
  2、Docker registry
  
    docker registry 
    docker client ---->docker daemon(local driver)----->docker registry(Docker Hub \public Docker registry\private Docker registry)
    用户可自建registry，也可以使用官方的docker hub
    Docker registry分类：
        Sponsor registry 第三方的registry，供客户和docker社区使用
        Mirror registry 第三的的registry，只让客户使用
        Vendor registry 由发布的Docker镜像的供应商提供的registry
        Private Registry 私有实体提供的registry
        
  3、cloud native
  
    云原生，面向云环境中运行，调动云环境中的某项原生应用功能。
    云原生开发的程序，
   
  4、Docker hub
    
      Docker hub provides the following major features
      1,Image Repositories（https://quay.io）
          私有仓库
      2,Automated Builds
          自动构建，会通过dockerfile创建docker
      3,Webhooks
      4,Organizations
      5,Github And Bitbucket Intergration
  5、docker pull
  
    docker pull --help
    Usage:	docker pull [OPTIONS] NAME[:TAG|@DIGEST]
    Pull an image or a repository from a registry
    
  6、镜像相关的操作
  
   基于容器做镜像
    
      docker commit
        A、docker run --name b1 -it busybox /bin/bash
        B、修改镜像，如mkdir /data/html/&&&echo "this is httpd server on busybox " >> /data/html/index.html
        C、不要退出A操作，在另一个窗口执行，docker tag 4f673d276b04 mybusyos/bbos_http:v0.1
          [root@docker01 ~]# docker image ls
          REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
          <none>                    <none>              4f673d276b04        59 seconds ago      1.16MB
        D、使用docker tag修改标签
          docker tag 4f673d276b04 mybusyos/bbos_http:v0.1
         [root@docker01 ~]# docker image ls
         REPOSITORY                TAG                 IMAGE ID            CREATED              SIZE
         mybusyos/bbos_http        v0.1                4f673d276b04        About a minute ago   1.16MB
    
   基于容器做一个默认httpd服务器的镜像
    
      #docker commit -a "thissgavin@github.com" -c 'CMD ["/bin/httpd","-f","-h","/data/html"]'-p b1 mybusyos/bbos_http:v0.2
      #docker run --name t2 -it mybusyos/bbos_http:v0.2
      [root@docker01 ~]# docker image ls
       REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
       mybusyos/bbos_http        v0.2                8db64268ec1f        5 minutes ago       1.16MB
      #docker inspect mybusyos/bbos_http:v0.2   #查询镜像信息
      #docker ps      #查询正在运行的容器进程信息
      #docker inspect t2   #查询t2容器的运行信息，包括IP、网关、掩码、运行的程序参数，目录位置、container ID

  7、阿里云镜像仓库
      
      1、阿里云镜像仓库
          httpd://dev.aliyun.com
      2、Docker Hub
          https://hub.docker.com 
      3、docker login 
      4、docker push

  8、备份还原
      
      1、备份
      docker save -o myimage.tar mybusyos/bbos_http:v0.2 mybusyos/bbos_http:v0.1
      
      2、删除备份
      docker rm CONTAINERS
      docker rmi IMAGES
      
      3、恢复备份
      docker load -i myimage.tar
