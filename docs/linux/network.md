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

- 设置网络层: ip address

    ```bash
    # 查看 ip 地址信息
    ip address show
    ```

- 设置路由: ip route

    ```bash
    # 查看路由规则
    ip route show

    # 添加路由
    ip route add 192.168.10.0/24 via 192.168.5.100 dev eth0

    # 删除路由
    ip rouet del 192.168.10.0/24
    ```

## 网络连通调试

### 连通性：ping, telnet

> ping 主要透过 ICMP 封包来进行整个网络的状况报告

语法格式: `ping [options] IP`

```
选项与参数：
-c 数值：后面接的是执行ping 的次数，例如-c 5 ；
-n ：在输出资料时不进行IP 与主机名称的反查，直接使用IP 输出(速度较快)；
-s 数值：发送出去的ICMP 封包大小，预设为56bytes，不过你可以放大此一数值；
-t 数值：TTL 的数值，预设是255，每经过一个节点就会少一；
-W 数值：等待回应对方主机的秒数。
-M [do|dont] ：主要在侦测网路的MTU 数值大小，两个常见的项目是：
   do ：代表传送一个DF (Don't Fragment) 旗标，让封包不能重新拆包与打包；
   dont：代表不要传送DF 旗标，表示封包可以在其他主机上拆包与打包
```

测试网络最大 MTU

> 由于 IP 包表头（不含 options）已经占用了 20 bytes，再加上 ICMP 的表头有 8 bytes，因此如果要是使用 MTU 为 1500 时，就得要下达 `ping -s 1472 -M do xx.yy.zz.ip` 才行啊！

```bash
# 把本机 mtu 扩大（可选）
ip link set eth0 mtu 2000

ping -c 2 -s 1472 -M do 192.168.1.254
```

telnet 调试端口连通性

```bash
telnet <ip> <port>
```

### 网络中间节点分析：traceroute

```bash
# 侦测本机到 baidu 去的各节点连线状态
traceroute -n www.baidu.com
```

有些节点会有防护措施，对 UDP，ICMP 包不回复，遇到这种情况可以使用 TCP 来侦测，并设置回复超时时间

```bash
# 使用 TCP 侦测，回复超时时间 1s
traceroute -T -n -w 1 www.baidu.com
```

### 查看本机网络连线

语法: `netstat <optionns>`

```
与路由(route) 有关的参数说明：
    -r ：列出路由表(route table)，功能如同route 这个指令；
    -n ：不使用主机名称与服务名称，使用IP 与port number ，如同route -n

与网路介面有关的参数：
    -a ：列出所有的连线状态，包括tcp/udp/unix socket 等；
    -t ：仅列出TCP 封包的连线；
    -u ：仅列出UDP 封包的连线；
    -l ：仅列出有在Listen (监听) 的服务之网路状态；
    -p ：列出PID 与Program 的档名；
    -c ：可以设定几秒钟后自动更新一次，例如-c 5 每五秒更新一次网路状态的显示；
```

常用命令

```bash
# 查看路由，等同于 route -n
netstat -rn

# 查看 TCP 监听端口的连线，并显示对应的服务
netstat -tlnp

# 查看本机启动的网络服务
netstat -tulnp
```

### IP 和域名的互相查询

- host

    ```bash
    # 查询域名的 IP
    host www.baidu.com
    # 指定 DNS 服务器查询域名 IP
    host www.baidu.com 168.95.1.1
    ```

- nslookup

    ```bash
    # 查询域名的 IP
    nslookup www.google.com

    # 查询 IP 的主机名
    nslookup 174.36.228.136
    ```

- dig

    > TODO


## 网页浏览和下载

```bash
# 文字版浏览器
elinks https://www.baidu.com

# 下载百度首页
wget https://www.baidu.com
```

## 抓包

语法： `tcpdump [-AennqX] [-i介面] [-w储存档名] [-c次数] [-r档案] [所欲撷取的封包资料格式]`

```
选项与参数：
-A ：封包的内容以ASCII 显示，通常用来捉取WWW 的网页封包资料。
-e ：使用资料连接层(OSI 第二层) 的MAC 封包资料来显示；
-nn：直接以IP 及port number 显示，而非主机名与服务名称
-q ：仅列出较为简短的封包资讯，每一行的内容比较精简
-X ：可以列出十六进位(hex) 以及ASCII 的封包内容，对于监听封包内容很有用
-i ：后面接要『监听』的网路介面，例如eth0, lo, ppp0 等等的介面；
-w ：如果你要将监听所得的封包资料储存下来，用这个参数就对了！后面接档名
-r ：从后面接的档案将封包资料读出来。那个『档案』是已经存在的档案，
     并且这个『档案』是由-w 所制作出来的。
-c ：监听的封包数，如果没有这个参数， tcpdump 会持续不断的监听，
     直到使用者输入[ctrl]-c 为止。
所欲撷取的封包资料格式：我们可以专门针对某些通讯协定或者是IP 来源进行封包撷取，
     那就可以简化输出的结果，并取得最有用的资讯。常见的表示方法有：
     'host foo', 'host 127.0.0.1' ：针对单部主机来进行封包撷取
     'net 192.168' ：针对某个网域来进行封包的撷取；
     'src host 127.0.0.1' 'dst net 192.168'：同时加上来源(src)或目标(dst)限制
     'tcp port 21'：还可以针对通讯协定侦测，如tcp, udp, arp, ether 等
     还可以利用and 与or 来进行封包资料的整合显示呢！
```

```bash
# 抓 HTTP 包查看， -w - 表示输出到控制台
tcpdump -i any -nn -w - src net 10.0.75
```

tcpdump 的包可以通过图形化工具 wireshark 进行分析

1. 在服务器使用 tcpdump 导出文件

```bash
tcpdump -i any -nn -w result.pcap src net 10.0.75
```

2. 通过 scp 传输到本地

3. 使用 wireshark 打开分析

## 连接或者启动 TCP/UDP 端口

语法：

- 连接端口: `nc [-u] [IP|host] [port]`

    > -u: 不使用 TCP 而是使用 UDP 作为联机的封包状态

- 启动监听端口: `nc -l [IP|host] [port]`

    ```bash
    # 连接端口
    nc 10.0.75.1 9090

    # 监听端口，并打印连接方发送的数据
    nc -l localhost 9090
    ```

## FAQ

### 占用端口的进程

```bash
# 通过 port 查询 pid
netstat -tlnp | grep <port>

# 通过 pid 查询 进程
ps aux | grep <pid>
```

## Referennces

- [鸟叔 Linux 私房菜 -- Linux 常用网路指令](http://linux.vbird.org/linux_server/0140networkcommand.php)