# Network 命名空间

> Network 命名空间用于隔离网络设备、IP 地址端口、路由表、防火墙规则等网络栈。Network 命名空间可以让每个容器拥有独立的虚拟的网络设备。

## 网络设备

- 每个新的 Network 命名空间默认有一个本地环回（lo）接口，但 lo 接口默认是关闭的，需要手动启动。
- 除了 lo 接口外，所有的其他网络设备（物理/虚拟网络接口，网桥等）只能属于一个 Network 命名空间。每个 socket 也只能属于一个 Network 命名空间。
- 标记为 `local devices` 的设备不能从一个 Network 命名空间移动到另一个 Network 命名空间，比如 `loopback`, `bridge`。可以通过 `ethtool -k` 命令来查看设备的 `netns-local` 属性。

```bash
$ ethtool -k lo | grep netns-local
netns-local: on [fixed]

# on [fixed] 表示该设备不能被移动到其他 Network 命名空间
```

## Go 语言实现 Network 命名空间隔离

```go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS | syscall.CLONE_NEWUSER | syscall.CLONE_NEWNET,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

## 实验过程

1. 当前宿主机网络设备

```bash
$  ifconfig 
eno1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 38:c9:86:51:14:f7  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 19  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 730962  bytes 417992895 (417.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 730962  bytes 417992895 (417.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# 可以看到，宿主机有多个网络设备
```

2. 执行 `go run network.go` 命令

```bash
$ go run network.go
$ ifconfig
$

# 什么网络设备都没有
```

## 拓展实验

### Network 命名空间之间通信

**1号终端**

1. 当前终端所属 Network 命名空间
```bash
$ echo $$
13178

$ readlink /proc/$$/ns/net
net:[4026532008]
```

2. 创建新的 Network 命名空间

```bash
$ unshare --net /bin/bash
$ readlink /proc/$$/ns/net
net:[4026532441]
```

3. 运行 `ifconfig`

```bash
$ ifconfig
$

# 无网络设备
```

4. 启动 lo 接口

```bash
$ ip link set lo up
$ ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

$ ping -c 2 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.043 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.073 ms
```

**2号终端**

1. 创建新的虚拟以太网设备

```bash
$ ip link add veth2 type veth peer name veth1

# 将 veth1 移动到1号终端的命名空间，13178是其 PID
$ ip link set veth1 netns 13178
```

2. 为 veth2 分配 IP 并启动 veth2

```bash
$ ip address add dev veth2 192.168.8.2/24
$ ip link set veth2 up
$ ifconfig veth2
veth2: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.8.2  netmask 255.255.255.0  broadcast 0.0.0.0
        ether 22:3e:0a:5e:4b:a4  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

**切换回1号终端**

1. 为 veth1 分配 IP 并启动 veth1

```bash
$ ip address add dev veth1 192.168.8.1/24
$ ip link set veth1 up
$ ifconfig veth1
veth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.8.1  netmask 255.255.255.0  broadcast 0.0.0.0
        inet6 fe80::2cb6:cfff:fedd:daf9  prefixlen 64  scopeid 0x20<link>
        ether 2e:b6:cf:dd:da:f9  txqueuelen 1000  (Ethernet)
        RX packets 20  bytes 2656 (2.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6  bytes 516 (516.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

2. 尝试让两个 Network 命名空间通信

```bash
$ ping -c 2 192.168.8.2
PING 192.168.8.2 (192.168.8.2) 56(84) bytes of data.
64 bytes from 192.168.8.2: icmp_seq=1 ttl=64 time=0.148 ms
64 bytes from 192.168.8.2: icmp_seq=2 ttl=64 time=0.097 ms

# 成功通信
```

### Network 命名空间访问外网

**切换回2号终端**

1. 确认 IP 转发是否已经开启

```bash
$ cat /proc/sys/net/ipv4/ip_forward
1

# 1表示已开启，0表示未开启，开启命令：
# sysctl -w net.ipv4.ip_forward=1
```

2. 添加 NAT 规则

```bash
# wlp3s0 是机器的物理网卡
$  iptables -t nat -A POSTROUTING -o wlp3s0 -j MASQUERADE
```

**切换回1号终端**

1. 添加默认网关

```bash
$ ip route add default via 192.168.8.2

# 移除命令：
# ip route del default via 192.168.8.2

$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.8.2       0.0.0.0             UG     0       0        0   veth1
192.168.8.0     0.0.0.0         255.255.255.0      U      0       0        0   veth1
```

2. 测试外网连通性

```bash
$ ping baidu.com
```

> **失败**：原因未知，关于 Linux 网络这块知识不曾了解，以后再说。

## ip netns 实现原理

> 在单独操作 Network 命名空间时，ip netns 是一个很方便的工具，它可以给 Network 命名空间取一个名字，然后根据名字来操作 Network 命名空间。这里演示一下其基本实现原理。

1. 获取当前终端所属 Network 命名空间

```bash
$ echo $$
16210

$ readlink /proc/$$/ns/net
net:[4026532008]
```

2. 创建一个用于绑定 Network 命名空间的文件

```bash
# ip netns 将文件都放到了目录 /var/run/netns 下
$ mkdir -p /var/run/netns
$ touch /var/run/netns/net1
```

3. 创建新的 Network 命名空间

```bash
$ unshare --net bash
$ readlink /proc/$$/ns/net
net:[4026532441]
```

4. 将当前终端的 Network 命名空间绑定到上面创建的文件上

```bash
$ mount --bind /proc/$$/ns/net /var/run/netns/net1

# 文件编号与 Network 命名空间编号相同，说明绑定成功
$ ls -i /var/run/netns/net1
4026532441 /var/run/netns/net1
```

5. 退出终端

```bash
$ exit
exit

# 文件仍存在，说明 Network 命名空间未被销毁
$ ls -i /var/run/netns/net1
4026532441 /var/run/netns/net1
$ 
```

这一系列操作等同于执行了命令：

```bash
$ ip netns add net1
```

6. 再次进入之前的 Network 命名空间

```bash
$ nsenter --net=/var/run/netns/net1 bash
$ readlink /proc/$$/ns/net
net:[4026532441]
```

这步操作等同于执行了命令：

```bash
$ ip netns exec net1 bash
```

从上面的实验可以看出，给 Network 命名空间取名字其实就是创建一个文件，然后通过 `mount --bind` 将新创建的 Network 命名空间文件和该文件绑定，就算该 Network 命名空间里的所有进程都退出了，内核还是会保留该 Network 命名空间，以后我们还可以通过这个绑定的文件来加入该 Network 命名空间。

通过这种办法，理论上我们也可以给其他类型的命名空间取名字。

### 章节导航

- 上一节：[User 命名空间](User%20命名空间.md)
- 下一节：[Cgroup 命名空间](Cgroup%20命名空间.md)
