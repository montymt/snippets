<!--ts-->
* [Linux](#linux)
   * [初始化](#初始化)
   * [Tuning](#tuning)
   * [参数优化实例](#参数优化实例)
   * [磁盘管理](#磁盘管理)
   * [命令行显示二维码](#命令行显示二维码)
<!--te-->

# Linux

## 初始化

```shell
hostnamectl set-hostname node-60-5

nmcli con mod ens33 ipv4.method manual ipv4.address 192.168.60.5/24 ipv4.gateway 192.168.60.2 ipv4.dns 114.114.114.114 \
connection.autoconnect yes
nmcli con down ens33 && nmcli con up ens33

#systemctl stop firewalld && systemctl disable firewalld
setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux

echo "* - nofile 65535" >> /etc/security/limits.conf && ulimit -n 65535
echo "* - nproc 65535" >> /etc/security/limits.conf && ulimit -u 65535

timedatectl set-timezone PRC
yum -y install chrony
systemctl start chronyd
systemctl enable chronyd
```

## Tuning

**ulimit**

`fs.file-max`: 系统打开的文件数，一个socket连接需要使用一个文件描述符

`fs.file-nr`: 系统已打开的文件数（第一个值）

nofile: 用户打开的文件数, `ulimit -n`

nproc: 用户打开的进程数, `ulimit -u`

**tcp/ip参数**

端口范围：`net.ipv4.ip_local_port_range = 1024   65000`

syn重传次数: `net.ipv4.tcp_syn_retries`, 默认5次，等待秒数是上次的2倍：1，2，4，8，16，32

synack重传次数: `net.ipv4.tcp_synack_retries`, 默认5次，等待秒数是上次的2倍：1，2，4，8，16，32

网卡溢出队列：`net.core.netdev_max_backlog`，网卡收包速率溢出时的队列长度

半连接队列：`net.ipv4.tcp_max_syn_backlog`，检查：`netstat -s|grep "SYNs to LISTEN"`

全连接队列：`net.core.somaxconn`，某些程序需设置应用层backlog才生效，如nginx，常用16384，取值为：`min(somaxconn, backlog)`

使用`netstat -s|grep overflowed`查看队列溢出情况

全连接队列溢出：`net.ipv4.tcp_abort_on_overflow`，默认为0，丢弃客户端的ack，1表示发送rst给客户端

cookie: `net.ipv4.tcp_syncookies`，默认为1，当半连接队列满时，使用cookie并忽略半连接队列，可防范syn flood

Fast Open：`net.ipv4.tcp_fastopen`，利用cookie可减少三次握手的时间，0关闭，1作为客户端开启，2作为服务端开启，3均开启

TCP连接保持时间：`net.ipv4.tcp_keepalive_time`，默认为7200，2小时

TCP连接保持间隔：`net.ipv4.tcp_keepalive_intvl`，默认为75秒

TCP连接保持探测次数：`net.ipv4.tcp_keepalive_probes`，默认为9次

FIN重试次数：`net.ipv4.tcp_orphan_retries`，默认为0，重发8次。

主动方收到fin报文的ack就会从FIN_WAIT1变成FIN_WAIT2，收不到ack就会重发fin报文

孤儿连接最大值：`net.ipv4.tcp_max_orphans`，无法正常关闭的连接（调用close后）为孤儿连接，FIN_WAIT1过多时可以调整此值，默认为16384

FIN_WAIT2持续时间：`net.ipv4.tcp_fin_timeout`，默认为60，对方可发送数据的时间

TIME_WAIT最大值：`net.ipv4.tcp_max_tw_buckets`，高并发可适当调大以免数据错乱，默认为16384

复用TIME_WAIT：`net.ipv4.tcp_tw_reuse`，对客户端有用，需开启时间戳

时间戳：`net.ipv4.tcp_timestamps`，默认为开启状态

回收TIME_WAIT：`tcp_tw_recycle`：需要时间戳递增，NAT，LVS不适用

滑动窗口扩大因子：`net.ipv4.tcp_window_scaling`，滑动窗口最大值，从64k扩大到1GB

接受缓冲区最大值：`net.core.rmem_max`

发送缓冲区最大值：`net.core.wmem_max`

发送缓冲区：`net.ipv4.tcp_wmem`，第一个为最小值，一般为4096，第二个为默认值，第三个为最大值，可设置为16M，即16777216

接受缓冲区：`net.ipv4.tcp_rmem`，第一个为最小值，一般为4096，第二个为默认值，第三个为最大值，可设置为16M

自动调节接受缓冲区：`net.ipv4.tcp_moderate_rcvbuf`

TCP内存范围：`net.ipv4.tcp_mem`，单位为页，一页一般为4k，第一个是最大无需动态调节内存，第二为开始动态调节内存，第三个为TCP最大内存

可减少默认缓冲区大小，以节约内存。最大缓冲区大小应与飞行报文匹配，大约为带宽时延积，带宽乘以网络时延。

查看socket状态：`cat /proc/net/sockstat`

防火墙最大跟踪连接：`net.netfilter.nf_conntrack_max`，默认为65536

## 参数优化实例

```
# 100万
fs.file-max = 1048576
net.ipv4.ip_local_port_range = 1024   65000
net.core.netdev_max_backlog = 32768
net.ipv4.tcp_max_syn_backlog = 16384
net.core.somaxconn = 16384
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_tw_buckets = 16384
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 87380 16777216
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.netfilter.nf_conntrack_max = 1048576
```

## 磁盘管理

mbr磁盘扩容(online):

```shell
# 备份分区表
dd if=/dev/vda of=bk.mbr count=1 bs=512
# 删除最后一个分区，并重新创建一个，大小保持默认
fdisk /dev/xvda
# 更新系统分区信息
partx -u /dev/xvda
# 扩容分区（ext3，ext4，xfs）
resize2fs /dev/xvda2
```

gpt磁盘扩容：

```shell
# 扩容分区，resize
parted /dev/vdb

# 查看分区
lsblk

# 扩容pv
pvresize /dev/vdb1

# 扩容lv
lvresize /dev/mapper/vg1-lv1 /dev/vdb1

# 扩容分区
resize2fs /dev/mapper/vg1-lv1
```

## 命令行显示二维码

`yum -y install qrencode`

`qrencode -o - -t utf8 https://github.com`

