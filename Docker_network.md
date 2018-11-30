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

