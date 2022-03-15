## Kubernetes 网络模型

![image-20220315211928481](/Users/zhangyukun/development/blog/image/image-20220315211928481.png)

- 虚拟网桥：创建一个虚拟网卡对（veth pair），一头在容器内，一头在宿主机的root namespace。容器内发出的数据包，可以通过网桥进入宿主机网络栈，而发往容器的网络数据包可以进过网桥进入容器
- 多路复用：使用一个中间设备，暴露多个虚拟网卡接口，容器网卡都可以接入中间设备，并通过mac地址/IP地址来区分packet发送给那个容器设备
- 硬件交换：为每个Pod分配一个虚拟网卡。这样Pod和Pod之间的通信就近乎物理及之间的通信

## Kubernetes CNI Flannel

github介绍地址：https://github.com/flannel-io/flannel

flannel是一种为Kubernetes设计的配置第三层网络结构的一种简单方式

### 工作原理

flannel运行客户端进程flanneld在每一个node上提供代理，负责从更大的预配置地址空间中为每台主机分配子网租约。Flannel使用Kubernetes API或Etcd直接存储网络配置、分配的子网和任何辅助数据（如主机的公共IP）。数据包使用多种后端机制转发，包括vxlan和不同的云集成

### 网络细节

Kubernetes等平台假设每个pod在集群内都有一个唯一的可路由IP。这种模式可以消除共享一个主机IP所带来的端口映射的复杂性。

Flannel负责在集群中的多个节点之间提供3层IPV4网络，Flannel不控制容器如何与主机联网，只控制流量在主机之间的传输方式，Flannel为Kubernetes提供了CNI插件，并提供了对Docker的集成。flannel专注于网络，对于网络策略则需要calico

## 网络通信

### 相同Pod内的不同容器

![image-20220315213412926](/Users/zhangyukun/development/blog/image/image-20220315213412926.png)

一个容器直接使用另一个已经存在容器的网络配置：IP信息和网络端口等所有网络相关的信息都是共享的。需要注意的是两个容器的计算和存储资源还是相互隔离的。Kubernetes的pod中container容器的网络就是通过共享同一个network namespace。container网络模式用于容器之间频繁交流的情况

### 同一个Nod的不同Pod

![image-20220315213941703](/Users/zhangyukun/development/blog/image/image-20220315213941703.png)

通常情况下，在flannel上解决同节点Pod之间的通信依赖的是Linux Bridge，不过在docker中采用的docker0，而在Kubernetes中使用的是Linux Bridge为cni0。可以通过brctl show查看对应Linux Bridge的信息

