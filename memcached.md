<!--ts-->
* [memcached](#memcached)
   * [安装](#安装)
   * [开启SASL及安全设置](#开启sasl及安全设置)
   * [安装libmemcached](#安装libmemcached)
   * [安装php扩展](#安装php扩展)
   * [安装python扩展](#安装python扩展)
<!--te-->

# memcached

## 安装

```shell
yum -y install wget libevent-devel gcc gcc-c++ make cyrus-sasl-devel cyrus-sasl-plain
wget http://memcached.org/files/memcached-1.6.8.tar.gz
tar xf memcached-1.6.8.tar.gz && cd memcached-1.6.8
./configure --prefix=/usr/local/memcached --enable-sasl
make && make install
ln -sf /usr/local/memcached/bin/memcached /usr/bin/memcached
cp scripts/memcached.sysconfig /etc/sysconfig/memcached
cp scripts/memcached.service /etc/systemd/system/
# 默认用户是nobody
id nobody
systemctl daemon-reload
systemctl enable memcached
systemctl start memcached
systemctl status memcached
```
## 开启SASL及安全设置

SASL开启后，只有二进制模式可以使用，telnet无法使用

```shell
mkdir /etc/sasl2
cat > /etc/sasl2/memcached.conf << EOF
mech_list: plain
log_level: 5
sasldb_path: /etc/sasl2/memcached-sasldb2
EOF
saslpasswd2 -a memcached -c -f /etc/sasl2/memcached-sasldb2 cacheuser
chown nobody:nobody /etc/sasl2/memcached-sasldb2
# 仅监听本机回环IP，开启SASL
sed -i 's/OPTIONS=""/OPTIONS="-l 127.0.0.1 -S"/' /etc/sysconfig/memcached
# 修改PORT为随机端口，并iptables中开启
sed -i 's/PORT="11211"/PORT="32112"/' /etc/sysconfig/memcached
systemctl restart memcached
systemctl status memcached
#局域网使用需要iptables白名单限制
#iptables -A INPUT -p tcp --dport 11211 !-s 192.168.1.1 -j DROP
```
## 安装libmemcached

php和python等语言使用memcached需要安装这个库

可以yum安装，这里不使用yum安装

```shell
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
tar xf libmemcached-1.0.18.tar.gz && cd libmemcached-1.0.18
./configure --prefix=/usr/local/libmemcached --with-memcached
make && make install
```

## 安装php扩展

memcache扩展因为很多问题很少人使用，memcached扩展比较不错，而且支持SASL

php5需要使用2.2.0版本，php7以上可以使用3.x版本

```shell
wget http://pecl.php.net/get/memcached-3.1.5.tgz
tar xf memcached-3.1.5.tgz && cd memcached-3.1.5
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config --enable-memcached --with-libmemcached-dir=/usr/local/libmemcached
make && make install
cat >/usr/local/php/conf.d/005-memcached.ini<<EOF
extension = memcached.so
memcached.use_sasl =1
EOF
/etc/init.d/php-fpm restart
```

php使用：

```php
$m = new Memcached();
$m->addServer('127.0.0.1', 11211);
$m->setOption(Memcached::OPT_BINARY_PROTOCOL, true);
$m->setSaslAuthData('cacheuser', 'cacheuserpass');
$m->add('ip', '192.168.1.2', 86400);
var_dump($m->get('ip'));
```

另外，memcached还支持CAS乐观锁

## 安装python扩展

```shell
yum install python-setuptools
easy_install -i https://pypi.tuna.tsinghua.edu.cn/simple pip
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple python-binary-memcached
```

简单测试：

```shell
Python 2.7.5 (default, Apr  2 2020, 13:16:51) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import bmemcached
>>> bm = bmemcached.Client(('127.0.0.1:11211',),'cacheuser','cacheuser')
>>> bm.set('key1','value1')
True
>>> bm.get('key1')
'value1'
>>>
```

最后，虽然有获取所有键的方法，但各个客户端都没有支持，所以一定要记得key

参考：https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-memcached-on-centos-7