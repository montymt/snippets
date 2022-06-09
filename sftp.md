# sftp

chroot模式的sftp搭建

## 创建系统用户

```shell
useradd sftpuser -d /nonexistent -s /sbin/nologin
passwd sftpuser
mkdir /home/sftproot
```

## 配置sshd

复制配置文件：`cp -a /etc/ssh/sshd_config /etc/ssh/sshd_sftp`

修改：

```ini
Port 2221
PermitRootLogin no
PubkeyAuthentication no
GSSAPIAuthentication no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTTY no
UseDNS no
PidFile /var/run/sftp.pid
ChrootDirectory /home/sftproot/
Subsystem	sftp	internal-sftp
AllowUsers sftpuser
```
复制service文件：`cp -a /lib/systemd/system/sshd.service /lib/systemd/system/sftp.service`

sshd后加上`-f /etc/ssh/sshd_sftp`，如果有pidfile，改成配置文件一样的

`systemctl daemon-reload`

`systemctl start sftp`

## 配置上传目录

```shell
mkdir /home/sftproot/upload

chown sftpuser.sftpuser /home/sftproot/upload
```
