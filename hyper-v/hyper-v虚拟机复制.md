## hyper-v虚拟机复制

1. 把你要复制的虚拟机对应的硬盘复制一份
2. 新建虚拟机，在选硬盘时选择复制出来的硬盘
3. 启动并用第一台虚拟机的用户登陆
4. `sudo vim /etc/hostname` 修改hostname
5. 如果之前的ip地址是静态的并且以外部网络连接的，需要修改ip地址
	具体看'/hyper/hyper-v安装ubuntu server.md'中静态ip的配置方法