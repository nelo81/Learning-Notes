## ubuntu 安装 docker

- 方法1：
1. 下载 ubuntu 对应版本的 docker\*\*.deb 包以及依赖包 libltdl7\*\*.deb
2. 运行`sudo dpkg -i **.deb`先安装依赖包 libltdl7, 在安装 docker
3. 测试启动
```
sudo docker pull hello-world
sudo docker run hello-world
```

- 方法2：
增加apt-get插件的源，然后直接`apt-get install docker-ce/docker-ee(收费)`，具体查看官网文档[doc.docker.com][1]

- 安装后更换国内源，防止pull速度过慢
修改 /etc/docker/daemon.json（没有就创建），添加如下变量
```
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```
然后重启docker服务即可

[1]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce