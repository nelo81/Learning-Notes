## docker 开启远程 api (ubuntu 16.04试验过，其他linux系统可能有些不同)

1. `sudo vim /lib/systemd/system/docker.service`修改以下字段
`ExecStart=/usr/bin/dockerd -H fd://`后面增加变成下面的
`ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375`
保存

2. 重启 daemon 和 docker 服务
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

3. 验证
`curl http://127.0.0.1:2375/version`有输出结果说明成功