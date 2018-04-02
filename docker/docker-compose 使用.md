## docker-compose 使用

- docker-compose 是方便我们同时启动和管理多个容器的工具，可通过[github][1]下载最新版本，使用前需先安装 docker

- docker-compose 执行命令必须指定某个yml/yaml文件（默认为"./docker-compose.yml"），表示管理的容器范围，两个不同的yml/yaml文件在 docker-compose 看来是不同环境的

- docker-compose command(基本命令):
	- -f: 指定基于yml/yaml文件（必须写的，不写默认为"./docker-compose.yml"）
	- up: 以镜像建立并启动容器，如果容器已经在之前建立了，优先启动之前建立的容器
	- ps: 查看容器情况，和 docker ps 一样，不过是基于某个yml/yaml文件
	- rm: 删除容器，也是基于yml/yaml文件
	- stop: 停止范围内容器
	- start: 启动范围内容器
  - restart: 重启范围内容器

- 示例 docker-compose.yml
```
version: '2' # 根据 docker-compose 的版本
services:
  service-1: # 服务名
    image: kongin/water-eureka-server # 镜像名
    ports: # 端口映射 同 docker run 中的 -p
      - 8761:8761
    environment: # 设置容器中的环境变量，同 docker run 中的 -e
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    volumes: # 文件系统映射（宿主机文件系统目录:容器中文件系统目录）同 docker run 中的 -v
      - usr/data:/var/lib/registry
      - usr/certs:/certs

  service-2: 第二个服务
    image: kongin/water-dao
    ports:
      - 8762:8080
    volumes:
      - /opt/files:/tmp/files

  ......(其他服务)
```

- 启动服务: `docker-compose -f docker-compose.yml up -d`

- 查看服务运行情况: `docker-compose -f docker-compose.yml ps` 或者 `docker ps`

[1]:https://github.com/docker/compose/releases