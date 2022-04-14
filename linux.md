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