一、Docker网络管理

  1、关于命名空间
  
  关于UTS，USER，MOUNT,IPC，PID，NET
       
       No.1 	MNT Namespace 	提供磁盘挂载点和文件系统的隔离能力
       No.2 	IPC Namespace 	提供进程间通信的隔离能力
       No.3 	Net Namespace 	提供网络隔离能力
       No.4 	UTS Namespace 	提供主机名隔离能力
       No.5 	PID Namespace 	提供进程隔离能力
       No.6 	User Namespace 	提供用户隔离能力
       
  容器节点之间通讯
    
      A、虚拟网络交换机（SDN、OpenVswitch）
  
      B、容器节点跨节点访问，可以使用nat技术实现。
      
      c
