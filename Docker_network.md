一、Docker网络管理

  1、关于命名空间
  
  关于UTS，USER，MOUNT,IPC，PID，NET
       
       No.1 	MNT Namespace 	提供磁盘挂载点和文件系统的隔离能力
       No.2 	IPC Namespace 	提供进程间通信的隔离能力
       No.3 	Net Namespace 	提供网络隔离能力
       No.4 	UTS Namespace 	提供主机名隔离能力
       No.5 	PID Namespace 	提供进程隔离能力
       No.6 	User Namespace 	提供用户隔离能力
       
  2、容器节点之间通讯
    
      A、虚拟网络交换机（SDN、OpenVswitch）
  
      B、容器节点跨节点访问，可以使用nat技术实现。
      
      c、同一节点之间，同一交换机的容器网络访问
      
      D、同一节点，不同交换机的容器网络访问，可以使用SDN,OPENswitch进行软件虚拟交换机访问
      
      E、不同物理节点之间的容器访问
          可以使用nat技术实现
          使用桥接方式进行实现，容易出现单点效率过低。
          使用Overlay network叠加网络，物理网卡只发报文，做隧道转发，使容器节点直接通讯。
       
  3、Docker网络
  
   A、运行一个容器
   
      #docker run --name t4 -it mybusyos/bbos_http:v0.2 /bin/sh
      #docker run --name t3 -d mybusyos/bbos_http:v0.2
      
   B、查询容器网络情况 
   
    #docker exec -it t3 /bin/sh
      
   C、查询iptables网络情况
      
      #iptables -t nat -nvL
      Chain PREROUTING (policy ACCEPT 312 packets, 49846 bytes)
      pkts bytes target     prot opt in     out     source               destination         
      31  1940 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

      Chain INPUT (policy ACCEPT 312 packets, 49846 bytes)
      pkts bytes target     prot opt in     out     source               destination         

      Chain OUTPUT (policy ACCEPT 125 packets, 9315 bytes)
      pkts bytes target     prot opt in     out     source               destination         
      0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

      Chain POSTROUTING (policy ACCEPT 125 packets, 9315 bytes)
      pkts bytes target     prot opt in     out     source               destination         
      0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
      0     0 MASQUERADE  all  --  *      !docker_gwbridge  172.18.0.0/16        0.0.0.0/0           

      Chain DOCKER (2 references)
      pkts bytes target     prot opt in     out     source               destination         
       0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
       0     0 RETURN     all  --  docker_gwbridge *       0.0.0.0/0            0.0.0.0/0           

二、网络命名空间管理
    
   1、创建网络命名空间netns
      
      #ip netns add w1
      #ip netns add w2
      
   2、创建虚拟网卡
   
      #ip link add name v1.0 type veth peer name v1.1
      #ip link show
      
   3、把创建的虚拟网卡赋予网络命名空间使用
   
      #ip link set v1.1 netns w1
      #ip netns exec w1 ifconfig -a
   
   4、改名
      
      #ip netns exec w1 ip link set dev v1.1 name eth0
      #ip netns exec w1 ifconfig -a
      
   5、设置IP，测试通讯
   
      #ipconfig v1.0 10.0.0.1/24 up
      #ip netns exec w1 ifconfig eth0 10.0.0.2/24 up
      #ping 10.0.0.2    #测试能够通讯
      
三、docker网络管理
    
  背景:docker的5种网络模式：
    
    docker run创建Docker容器时，可以用--net选项指定容器的网络模式，Docker有以下5种网络模式：
    bridge模式：使用–net =bridge指定，默认设置；
    host模式：使用–net =host指定；
    none模式：使用–net =none指定；
    container模式：使用–net =container:指定容器名；
    overlay模式：使用--net=overlay

   1、创建loopback Interface无对外网络的容器
      
      #docker run --name w1 -it --network none --rm busybox:latest /bin/sh
      #执行ifconfig -a发现无网卡
      
   2、创建bridge Interface桥接网卡，Bridge网卡和LO网卡
   
      #docker run --name w1 -it --network bridge --rm busybox:latest /bin/sh
      #执行ifconfig -a能够发现eth0和LO网卡。
      
   3、创建一个带有主机名的容器，默认采用的是container ID作为主机名
      
      #docker run --name w1 -it --hostname w1 --network bridge --rm busybox:latest /bin/sh
      #执行hostname可以查看主机名
      并且已经自动修改/etc/hosts，/etc/resolve.conf
      
   4、创建一个带有主机名为W1、网络桥接方式为bridge，退出自动删除，DNS服务器为202.106.0.20的容器。
   
      #docker run --name w1 -it --rm --network bridge --dns 202.106.0.20 busybox:latest /bin/sh
      #cat /etc/resolve.conf
        202.106.0.20
      #nslookup www.baidu.com
        Server:		202.106.0.20
        Address:	202.106.0.20:53

        Non-authoritative answer:
        www.baidu.com	canonical name = www.a.shifen.com
        Name:	www.a.shifen.com
        Address: 119.75.217.109
        Name:	www.a.shifen.com
        Address: 119.75.217.26
        
   5、创建一个带有主机解析的容器
   
      #docker run --name w1 -it --rm --network bridge --dns 202.106.0.20 --add-host docker01:172.17.0.1 --hostname w1 busybox:latest /bin/sh
      #cat /etc/hosts
        127.0.0.1	localhost
        ::1	localhost ip6-localhost ip6-loopback
        fe00::0	ip6-localnet
        ff00::0	ip6-mcastprefix
        ff02::1	ip6-allnodes
        ff02::2	ip6-allrouters
        172.17.0.1	docker01
        172.17.0.2	w1

  6、使用-P参数进行容器端口暴露
      
   A、将指定的容器端口映射至主机所有IP地址的一个动态端口（-p <contaainerPort>）
     
      #docker run --name myweb --rm -p 80  mybusyos/bbos_http:v0.2
      #使用docker port myweb，#查询端口暴露情况
      #iptables -t nat -nvL   #可以看出dpt:32771 to:172.17.0.2:80
      [root@docker01 ~]# iptables -t nat -nvL
        Chain PREROUTING (policy ACCEPT 10 packets, 823 bytes)
        pkts bytes target     prot opt in     out     source               destination         
          180 10424 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

        Chain INPUT (policy ACCEPT 10 packets, 823 bytes)
         pkts bytes target     prot opt in     out     source               destination         

        Chain OUTPUT (policy ACCEPT 1 packets, 76 bytes)
        pkts bytes target     prot opt in     out     source               destination         
          0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

        Chain POSTROUTING (policy ACCEPT 3 packets, 180 bytes)
        pkts bytes target     prot opt in     out     source               destination         
          8   572 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
          0     0 MASQUERADE  all  --  *      !docker_gwbridge  172.18.0.0/16        0.0.0.0/0           
          0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80

        Chain DOCKER (2 references)
        pkts bytes target     prot opt in     out     source               destination         
          2   168 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
          0     0 RETURN     all  --  docker_gwbridge *       0.0.0.0/0            0.0.0.0/0           
          2   104 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:32771 to:172.17.0.2:80

      
   B、将容器端口映射到指定的主机端口(-p <hostPort>:<containerPort>)
   
      指定myweb容器80端口映射到32800端口
      #docker run --name myweb -p 32800:80 mybusyos/bbos_http:v0.2
      #docker port myweb
      [root@docker01 ~]# docker port myweb
      80/tcp -> 0.0.0.0:32800
      
   C、将容器80端口映射到指定主机192.168.60.216的任意端口
   
      #docker run --name myweb1 --rm -p 192.168.60.216::80 mybusyos/bbos_http:v0.2
      
   D、将容器80端口映射到指定主机192.168.60.216的指定端口
   
      #docker run --name myweb1 --rm -p 192.168.60.216:32768:80 mybusyos/bbos_http:v0.2
    
 7、docker network 共享网络命名空间的方法
 
   A、容器之间共享网络
      
      #docker run --name web1 -it --rm mybusyos/bbos_http:v0.2 /bin/sh
      #docker run --name web2 -it --rm --network container:web1 mybusyos/bbos_http:v0.2 /bin/sh
      #ifconfig 查询发现web1,web2共享 IP
    
   B、容器与HOST之间共享网络
   
      #docker run --name web1 -it --rm --network host mybusyos/bbos_http:v0.2 /bin/sh
      # ifconfig
          docker0   Link encap:Ethernet  HWaddr 02:42:D3:27:8D:5C  
                    inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
                    inet6 addr: fe80::42:d3ff:fe27:8d5c/64 Scope:Link
                    UP BROADCAST MULTICAST  MTU:1500  Metric:1
                    RX packets:116 errors:0 dropped:0 overruns:0 frame:0
                    TX packets:118 errors:0 dropped:0 overruns:0 carrier:0
                    collisions:0 txqueuelen:0 
                    RX bytes:6984 (6.8 KiB)  TX bytes:11198 (10.9 KiB)

          docker_gwbridge Link encap:Ethernet  HWaddr 02:42:90:BE:62:89  
                    inet addr:172.18.0.1  Bcast:172.18.255.255  Mask:255.255.0.0
                    UP BROADCAST MULTICAST  MTU:1500  Metric:1
                    RX packets:0 errors:0 dropped:0 overruns:0 frame:0
                    TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
                    collisions:0 txqueuelen:0 
                    RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

          enp0s3    Link encap:Ethernet  HWaddr 08:00:27:67:C2:ED  
                    inet addr:192.168.60.216  Bcast:192.168.60.255  Mask:255.255.255.0
                    inet6 addr: fe80::1589:844:76ee:3f33/64 Scope:Link
                    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                    RX packets:56583 errors:0 dropped:0 overruns:0 frame:0
                    TX packets:9567 errors:0 dropped:0 overruns:0 carrier:0
                    collisions:0 txqueuelen:1000 
                    RX bytes:4281795 (4.0 MiB)  TX bytes:1522808 (1.4 MiB)

          lo        Link encap:Local Loopback  
                    inet addr:127.0.0.1  Mask:255.0.0.0
                    inet6 addr: ::1/128 Scope:Host
                    UP LOOPBACK RUNNING  MTU:65536  Metric:1
                    RX packets:32 errors:0 dropped:0 overruns:0 frame:0
                    TX packets:32 errors:0 dropped:0 overruns:0 carrier:0
                    collisions:0 txqueuelen:1000 
                    RX bytes:2592 (2.5 KiB)  TX bytes:2592 (2.5 KiB)

          v1.0      Link encap:Ethernet  HWaddr DA:29:DD:A4:F7:C4  
                    inet addr:10.0.0.1  Bcast:10.0.0.255  Mask:255.255.255.0
                    inet6 addr: fe80::d829:ddff:fea4:f7c4/64 Scope:Link
                    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                    RX packets:13 errors:0 dropped:0 overruns:0 frame:0
                    TX packets:13 errors:0 dropped:0 overruns:0 carrier:0
                    collisions:0 txqueuelen:1000 
                    RX bytes:1026 (1.0 KiB)  TX bytes:1026 (1.0 KiB)

  8、修改docker默认网络之一
   
   A、停止docker服务
      
      #systemctl stop docker
   
   B、修改/etc/docker/daemon.json
        
        "bip":"172.16.0.1/16",
        "default-gateway":"172.16.0.1",
        "dns":["202.106.0.20","202.106.46.151"]
        
   C、启动docker服务
      
      #systemctl start docker
      
   D、测试
      
      #docker start t4
      #docker exec -it t4 /bin/sh
   
  9、修改docker网络之二
  
   A、docker -H

    Options:
      --config string      Location of client config files (default "/root/.docker")
      -D, --debug              Enable debug mode
      -H, --host list          Daemon socket(s) to connect to

   B、修改/etc/docker/daemon.json
      
      "hosts":  ["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"]
      
   C、重启
   
    #systemctl restart docker
   
 10、创建自已的桥接网卡
 
    #docker network create -d bridge --subnet "172.20.0.0/16" --gateway "172.20.0.1" mybr0

    #docker network ls
    [root@docker01 ~]# docker network ls 
      NETWORK ID          NAME                DRIVER              SCOPE
      858a24cc6898        bridge              bridge              local
      d206bf11a0fe        docker_gwbridge     bridge              local
      b9865128c757        host                host                local
      a30138c6c98f        mybr0               bridge              local
      8beaf8ce72e6        none                null                local
  改名
    
    #ifconfig br-a30138c6c98f down
    #ip link set br-a30138c6c98f name docker0001
    #ifconfig docker0001 up
    
 11、使用自已的桥接网卡启动容器
 
    #docker run --name myprihost -it --rm --network mybr0 mybr0 mybusyos/bbos_http:v0.2 /bin/sh
 
 12、两个不同网段的容器
 
    #[root@docker01 ~]# cat /proc/sys/net/ipv4/ip_forward
     1

      
