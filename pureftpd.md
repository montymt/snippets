<!--ts-->
* [PureFTPd](#pureftpd)
   * [安装](#安装)
   * [添加登录用户](#添加登录用户)
   * [SSL登录配置](#ssl登录配置)
<!--te-->

# PureFTPd

## 安装
```shell
yum -y install make gcc gcc-c++ gcc-g77 openssl openssl-devel bzip2 wget
tar xf pure-ftpd-1.0.49.tar.bz2  && cd pure-ftpd-*
./configure --prefix=/usr/local/pureftpd CFLAGS=-O2 --with-puredb --with-quotas --with-cookie --with-virtualhosts --with-diraliases --with-sysquotas --with-ratios --with-altlog --with-paranoidmsg --with-shadow --with-welcomemsg --with-throttling --with-uploadscript --with-language=english --with-ftpwho --with-tls
make && make install

# 服务脚本
cat >/etc/systemd/system/pureftpd.service <<EOF
[Unit]
Description=Pure-FTPd is an FTP server
After=network.target

[Service]
Type=forking
PIDFile=/var/run/pure-ftpd.pid
ExecStart=/usr/local/pureftpd/sbin/pure-ftpd /usr/local/pureftpd/etc/pure-ftpd.conf
PrivateTmp=false

[Install]
WantedBy=multi-user.target
EOF

# 禁止匿名登录
sed -i "s/NoAnonymous.*/NoAnonymous yes/g" /usr/local/pureftpd/etc/pure-ftpd.conf
sed -i "s/AnonymousCantUpload.*/AnonymousCantUpload yes/g" /usr/local/pureftpd/etc/pure-ftpd.conf

# 用户数据库文件
sed -i "s@#\s*PureDB\s*/etc/pureftpd.pdb@PureDB /usr/local/pureftpd/etc/pureftpd.pdb@" /usr/local/pureftpd/etc/pure-ftpd.conf

# 端口设置
#PassivePortRange             20000 21000
# Bind 						0.0.0.0,15117
systemctl start pureftpd
systemctl enable pureftpd
systemctl status pureftpd
```
## 添加登录用户
```shell
groupadd ftpuser
useradd -s /sbin/nologin -g ftpuser ftpuser
touch /usr/local/pureftpd/etc/pureftpd.passwd
touch /usr/local/pureftpd/etc/pureftpd.pdb
mkdir /var/ftp && chown ftpuser.ftpuser /var/ftp
id ftpuser
/usr/local/pureftpd/bin/pure-pw useradd ftpuser -f /usr/local/pureftpd/etc/pureftpd.passwd -u 1000 -g 1000 -d /var/ftp -m
```

## SSL登录配置
```shell
mkdir -p /etc/ssl/private
openssl dhparam -out /etc/ssl/private/pure-ftpd-dhparams.pem 2048
openssl req -x509 -nodes -newkey rsa:2048 -sha256 -keyout \
  /etc/ssl/private/pure-ftpd.pem \
  -out /etc/ssl/private/pure-ftpd.pem
chmod 600 /etc/ssl/private/*.pem
# TLS改为2，强制所有登录使用SSL
sed -i "s@#\s*TLS\s*1@TLS 2@" /usr/local/pureftpd/etc/pure-ftpd.conf
systemctl restart pureftpd
# 防火墙配置，因为用了加密，关联数据端口已失效
iptables -A INPUT -p tcp -m multiport --dports 15117,20000-21000 -j ACCEPT 
```