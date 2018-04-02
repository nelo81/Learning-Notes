## hyper-v安装ubuntu server

1. 新建虚拟机，选择外部网络适配器（详情在"hyper-v网络设置中有讲"），选择ubuntu server iso安装
2. 语言选择英文
3. dhcp分配失败的话直接跳过
4. 安装完成后配置网络ip

- 检查是否启用 Hyper-V IC module
`lsmod|grep hv_vmbus`

- 如果启用会有以下显示
`hv_vmbus 65536 7 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc`

- 配置网卡IP地址，vi /etc/network/interfaces，添加以下代码

	- 静态地址方式（推荐）: 注意一下, 数据要按照你实际本机的网段进行设置
```
auto eth0  
iface eth0 inet static  
address 192.168.1.122
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255 
gateway 192.168.1.1
dns-nameservers 8.8.8.8
```

	- 动态地址方式:
```
auto eth0  
iface eth0 inet dhcp
```

- 设置DNS，vi /etc/resolv.conf，添加
`nameserver 8.8.8.8`

- 生效
```
sudo /etc/init.d/networking restart
reboot
```

- 至此已经可以连接外网

- 检测ssh服务
`sudo service ssh status` 

- 如果出现not found，则通过
```
sudo apt-get update
sudo apt-get install openssh-server
```

- 安装好就可以用
`ifconfig`
查看ip地址，然后就可以用 xshell 连接22端口登录了！