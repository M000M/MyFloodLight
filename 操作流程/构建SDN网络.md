## 构建SDN网络

1. ### 包含两个OVS和两个主机，每个主机链接到一个OVS上，OVS之间互联

```
创建两个OVS交换机
ovs-vsctl add-br s1
ovs-vsctl add-br s2

查看
ovs-vsctl show

s1添加端口p1, p2
ovs-vsctl add-port s1 p1
ovs-vsctl set Interface p1 ofport_request=10
ovs-vsctl set Interface p1 type=internal
ethtool -i p1

ovs-vsctl add-port s1 p2
ovs-vsctl set Interface p2 ofport_request=11
ovs-vsctl set Interface p2 type=internal
ethtool -i p2

s2添加端口p3, p4
ovs-vsctl add-port s2 p3
ovs-vsctl set Interface p3 ofport_request=1
ovs-vsctl set Interface p3 type=internal
ethtool -i p3

ovs-vsctl add-port s2 p4
ovs-vsctl set Interface p4 ofport_request=2
ovs-vsctl set Interface p4 type=internal
ethtool -i p4


创建虚拟主机h1, h2
ip netns add h1
ip link set p1 netns h1
ip netns exec h1 ip addr add 192.168.10.10/24 dev p1
ip netns exec h1 ifconfig p1 promisc up

ip netns add h2
ip link set p4 netns h2
ip netns exec h2 ip addr add 192.168.10.11/24 dev p4
ip netns exec h2 ifconfig p4 promisc up

建立交换机之间的链路
对应端口设置为patch
ovs-vsctl set interface p2 type=patch
ovs-vsctl set interface p3 type=patch

创建p2和p3之间的链路
ovs-vsctl set interface p2 options:peer=p3
ovs-vsctl set interface p3 options:peer=p2

添加流表项
ovs-ofctl add-flow s1 "in_port=10, actions=output:11"
ovs-ofctl add-flow s1 "in_port=11, actions=output:10"

ovs-ofctl add-flow s2 "in_port=1, actions=output:2"
ovs-ofctl add-flow s2 "in_port=2, actions=output:1"

测试
ip netns exec h1 ping 192.168.10.11
ip netns exec h2 ping 192.168.10.10

s1, s2连接到控制器
ovs-vsctl set-controller s1 tcp:127.0.0.1:6653
ovs-vsctl set-controller s2 tcp:127.0.0.1:6653
```

![image-20200809164118601](/home/zen/.config/Typora/typora-user-images/image-20200809164118601.png)

拓扑图

![image-20200811131312733](/home/zen/.config/Typora/typora-user-images/image-20200811131312733.png)

控制器

![image-20200811131343519](/home/zen/.config/Typora/typora-user-images/image-20200811131343519.png)



2. 包含3个OVS，3个主机，每个主机和一个OVS相连，3个OVS之间互联

```
创建三个OVS交换机
ovs-vsctl add-br s1
ovs-vsctl add-br s2
ovs-vsctl add-br s3

查看
ovs-vsctl show

s1添加端口p1, p2
ovs-vsctl add-port s1 p1
ovs-vsctl set Interface p1 ofport_request=10
ovs-vsctl set Interface p1 type=internal
ethtool -i p1

ovs-vsctl add-port s1 p2
ovs-vsctl set Interface p2 ofport_request=11
ovs-vsctl set Interface p2 type=internal
ethtool -i p2

s2添加端口p3, p4
ovs-vsctl add-port s2 p3
ovs-vsctl set Interface p3 ofport_request=1
ovs-vsctl set Interface p3 type=internal
ethtool -i p3

ovs-vsctl add-port s2 p4
ovs-vsctl set Interface p4 ofport_request=2
ovs-vsctl set Interface p4 type=internal
ethtool -i p4

s3添加端口p5, p6
ovs-vsctl add-port s3 p5
ovs-vsctl set Interface p5 ofport_request=10
ovs-vsctl set Interface p5 type=internal
ethtool -i p5

ovs-vsctl add-port s3 p6
ovs-vsctl set Interface p6 ofport_request=11
ovs-vsctl set Interface p6 type=internal
ethtool -i p6


创建虚拟主机h1, h2, h3
ip netns add h1
ip link set p1 netns h1
ip netns exec h1 ip addr add 192.168.10.10/24 dev p1
ip netns exec h1 ifconfig p1 promisc up

ip netns add h2
ip link set p4 netns h2
ip netns exec h2 ip addr add 192.168.10.11/24 dev p4
ip netns exec h2 ifconfig p4 promisc up

ip netns add h3
ip link set p5 netns h3
ip netns exec h3 ip addr add 192.168.10.12/24 dev p5
ip netns exec h3 ifconfig p5 promisc up

建立交换机之间的链路
对应端口设置为patch
ovs-vsctl set interface p2 type=patch
ovs-vsctl set interface p3 type=patch
ovs-vsctl set interface p6 type=patch

创建p2和p3之间的链路
ovs-vsctl set interface p2 options:peer=p3
ovs-vsctl set interface p3 options:peer=p2

创建p2和p6之间的链路
ovs-vsctl set interface p2 options:peer=p6
ovs-vsctl set interface p6 options:peer=p2

创建p3和p6之间的链路
ovs-vsctl set interface p3 options:peer=p6
ovs-vsctl set interface p6 options:peer=p3



添加流表项
s1的流表项
ovs-ofctl add-flow s1 "in_port=10, actions=output:11"
ovs-ofctl add-flow s1 "in_port=11, actions=output:10"

s2的流表项
ovs-ofctl add-flow s2 "in_port=1, actions=output:2"
ovs-ofctl add-flow s2 "in_port=2, actions=output:1"

s3的流表项
ovs-ofctl add-flow s3 "in_port=10, actions=output:11"
ovs-ofctl add-flow s3 "in_port=11, actions=output:10"

测试
ip netns exec h1 ping 192.168.10.11
ip netns exec h2 ping 192.168.10.10
```



![image-20200811131454434](/home/zen/.config/Typora/typora-user-images/image-20200811131454434.png)



![image-20200811131706415](/home/zen/.config/Typora/typora-user-images/image-20200811131706415.png)



代码下发流表

![image-20200811132314755](/home/zen/.config/Typora/typora-user-images/image-20200811132314755.png)



![image-20200811132352821](/home/zen/.config/Typora/typora-user-images/image-20200811132352821.png)

