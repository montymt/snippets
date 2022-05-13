<!--ts-->
* [supervisor](#supervisor)
   * [install](#install)
   * [manage](#manage)
   * [manage process](#manage-process)
<!--te-->
# supervisor

## install

`pip3 install supervisor`

`mkdir -p /etc/supervisor/conf.d`

`echo_supervisord_conf > /etc/supervisor/supervisord.conf`

`sed -i '$a [include] \
files = /etc/supervisor/conf.d/*.ini' /etc/supervisor/supervisord.conf`

## manage

因为使用的scl的python 3.8，supervisord的路径需要更改

```shell
cat > /usr/lib/systemd/system/supervisord.service <<EOF
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service nss-user-lookup.target

[Service]
Type=forking
ExecStart=/opt/rh/rh-python38/root/usr/local/bin/supervisord -c /etc/supervisor/supervisord.conf

[Install]
WantedBy=multi-user.target
EOF
```

启动：`systemctl start supervisord`

自启：`systemctl enable supervisord`

## manage process

```ini
[program:test]
command=/usr/local/my-test/venv/bin/python my-test.py
process_name=%(program_name)s
numprocs=1
autostart=true
autorestart=unexpected
directory=/usr/local/my-test
user=my-test
redirect_stderr=true
stdout_logfile=/usr/local/my-test/logs/my-test.log
```

查看帮助：`supervisorctl help`

更新配置：`supervisorctl reread`

获取pid：`supervisorctl pid xxx`