# Golang

## 下载安装

```shell
curl -O https://dl.google.com/go/go1.17.8.linux-amd64.tar.gz

tar -C /usr/local -xzf go1.17.8.linux-amd64.tar.gz && rm -f go1.17.8.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin' > /etc/profile.d/go.sh

source /etc/profile.d/go.sh

go env -w GOPROXY=https://goproxy.cn,direct
```