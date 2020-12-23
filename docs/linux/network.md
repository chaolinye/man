# 网络

## 网络参数设置

- `ifconfig`：查询、设置网卡与 IP 地址等相关参数；

- `ifup`, `ifdown`：这两个命令是 script，透过更简单的方式来启动网卡；

- `route`：查询、设置路由表(route table)

- `ip`：复合式的指令，可以直接修改上述提到的功能；

### ifconfig：网卡设置

> 网卡的术语是 interface，ifconfig 是 interfaceconfig 的意思

查看、启停网卡: `ifconfig [interface] [up|down]`

```bash
# 查看所有网卡
ifconfig

# 查看 eth0网卡
ifconfig etho0

# 设置网卡 IP、mask、mtu
ifconfig eth0 192.168.100.100 netmask 255.255.255.128 mtu 8000

# 下线网卡
ifconfig eth0 down

# 上线网卡
ifconfig eth0 up

# 重启网络服务，将手动设置的参数全部取消
/etc/init.d/network restart
```

> ifconfig 修改的网卡参数只是暂时的，重启后就失效了

### ifup，ifdown

即时手动修改一些网卡参数，可以利用 ifconfig 来达成

如果是要永久修改，就需要在 `/etc/sysconfig/network-scripts` 里面的 `ifcfg-<interface>` 文件中修改

修改后，需要通过 `ifdown` 或 `ifup` 来重启生效

```bash
ifdown eth0

ifup eth0
```

### route： 路由修改

!> `-n`  是很多网络工具中的选项，表示显示 IP 地址，无需通过 DNS 获取和显示主机名，可以提高命令速度

```bash
# 查看路由
route -n
```

```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.111.253  0.0.0.0         UG    0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.31.96.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
```

- Destination, Genmask：组合定义一个网段，满足这个网段的，走这条路由规则

- Gateway：该网域是通过哪个gateway连接出去的？如果显示 `0.0.0.0` 表示目的机在同一网段，通过 MAC 直接传输

- Flags: 总共有多个选项，代表的意义如下：

    - U (route is up)：该路由是启动的；
    - H (target is a host)：目标是一部主机(IP) 而非网域；
    - G (use gateway)：需要透过外部的主机(gateway) 来转递封包；
    - R (reinstate route for dynamic routing)：使用动态路由时，恢复路由资讯的旗标；
    - D (dynamically installed by daemon or redirect)：已经由服务或转port 功能设定为动态路由
    - M (modified from routing daemon or redirect)：路由已经被修改了；
    - ! (reject route)：这个路由将不会被接受(用来抵挡不安全的网域！)

```bash
# 删除路由，需要指定 network、mask、device
route del -net 10.32.244.0 netmask 255.255.255.0 dev eth0

# 新增路由
route add -net 10.32.244.0 netmask 255.255.255.0 dev eth0 gw 0.0.0.0

# 新增默认路由
route add default gw 192.168.1.250 
```

### 网络参数设置综合命令：ip

> ip 命令具备 ifconfig 和 route 的功能

- 设置数据链路层：ip link

```bash
# 查询网卡参数
ip -s link show

# 设置网卡 mtu
ip link set eth0 mtu 1000

# 启动网卡
ip link set eth0 up

# 下线网卡
ip link set eth0 down

# 修改网卡名称，需要先下线才能设置
ip link set eth0 name vbird

# 设置网卡 MAC 地址，需要网卡支持
ip link set eth0 address aa:aa:aa:aa:aa:aa 
```

## 网络连通调试

### 连通性：ping

### 网络中间节点分析：traceroute

## 抓包

## 网页浏览和下载

## 通讯

## Referennces

- []