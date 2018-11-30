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


      
    
   
