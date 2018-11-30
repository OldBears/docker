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
