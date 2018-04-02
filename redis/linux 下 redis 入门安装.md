## linux 下 redis 入门安装

1. 下载 redis 的tar包，输入以下命令
```
tar -xzf redis.tar.gz
cd redis
make

cd src
./redis-server (redis.conf配置文件，可以不写用默认配置)
```
即默认启动到 localhost:6379

2. 使用 redis-cli 验证
`./redis-cli`

3. redis.conf 配置
	- 把`bind address 127.0.0.1`改成`bind address 0.0.0.0`，以便可以用外网访问
	- `port`指端口
	- `requirepass 000`表示设置密码为000，不设置默认没有密码
	- 把 daemonize 设置为 `daemonize yes` 开启后台pid启动redis
	- 配置文件写好之后，用`./redis-server redis.conf(你修改过的配置文件)`启动即可