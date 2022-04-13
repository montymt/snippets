<!--ts-->
* [frp](#frp)
   * [服务端](#服务端)
   * [暴露内网特定服务](#暴露内网特定服务)
   * [通过自定义域名访问内网的 Web 服务](#通过自定义域名访问内网的-web-服务)
   * [静态文件服务](#静态文件服务)
   * [https转http](#https转http)
   * [带密钥的服务代理](#带密钥的服务代理)
   * [udp穿透](#udp穿透)
<!--te-->

# frp

简单、高效的内网穿透工具

仓库：https://github.com/fatedier/frp

文档：https://gofrp.org/docs/

## 服务端

```
[common]
bind_port = 7000
```

监听于tcp的7000端口

## 暴露内网特定服务

SSH服务：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

DNS服务：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[dns]
type = udp
local_ip = 8.8.8.8
local_port = 53
remote_port = 6000
```

socket服务：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[unix_domain_socket]
type = tcp
remote_port = 6000
plugin = unix_domain_socket
plugin_unix_path = /var/run/docker.sock
```

## 通过自定义域名访问内网的 Web 服务

服务端：`vhost_http_port = 8080`

通过域名访问该端口，则可请求到内网的web服务

```
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.yourdomain.com
```

## 静态文件服务

```
[common]
server_addr = x.x.x.x
server_port = 7000

[test_static_file]
type = tcp
remote_port = 6000
plugin = static_file
# 要对外暴露的文件目录
plugin_local_path = /tmp/file
# 用户访问 URL 中会被去除的前缀，保留的内容即为要访问的文件路径
plugin_strip_prefix = static
plugin_http_user = abc
plugin_http_passwd = abc
```

## https转http

```
[common]
server_addr = x.x.x.x
server_port = 7000

[test_htts2http]
type = https
custom_domains = test.yourdomain.com

plugin = https2http
plugin_local_addr = 127.0.0.1:80

# HTTPS 证书相关的配置
plugin_crt_path = ./server.crt
plugin_key_path = ./server.key
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp
```

## 带密钥的服务代理

服务提供方：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[secret_ssh]
type = stcp
# 只有 sk 一致的用户才能访问到此服务
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22
```

访问方：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[secret_ssh_visitor]
type = stcp
# stcp 的访问者
role = visitor
# 要访问的 stcp 代理的名字
server_name = secret_ssh
sk = abcdefg
# 绑定本地端口用于访问 SSH 服务
bind_addr = 127.0.0.1
bind_port = 6000
```

## udp穿透

服务端：`bind_udp_port = 7000`

服务提供方：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[p2p_ssh]
type = xtcp
# 只有 sk 一致的用户才能访问到此服务
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22
```

访问方：

```
[common]
server_addr = x.x.x.x
server_port = 7000

[p2p_ssh_visitor]
type = xtcp
# xtcp 的访问者
role = visitor
# 要访问的 xtcp 代理的名字
server_name = p2p_ssh
sk = abcdefg
# 绑定本地端口用于访问 ssh 服务
bind_addr = 127.0.0.1
bind_port = 6000
```
