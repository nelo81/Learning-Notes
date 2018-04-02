## docker 私有仓库 ui

使用镜像 docker-registry-frontend，也有很多其他的，但是我试过只有这个是同时支持https和v2的所以就用这个

- 下载镜像 `docker pull konradkleine/docker-registry-frontend:v2`，注意要下载v2版本而不是latest

- 启动 docker registry (/docker/docker 私有仓库搭建.md)

- 启动 docker-registry-frontend
```
sudo docker run -d -p 5050:443 --name registry-web \
  -e ENV_MODE_BROWSE_ONLY=true \ # 开启预览模式不能删除镜像，改成false就可以删除（不过我自己还是不能删除，Bug）
  -e ENV_DOCKER_REGISTRY_HOST=www.domain.com \ # 私有仓库host
  -e ENV_DOCKER_REGISTRY_PORT=5000 \ # 私有仓库port
  -e ENV_DOCKER_REGISTRY_USE_SSL=1 \  # 私有仓库如果要使用https访问就要设置这项
  -e ENV_USE_SSL=yes \  # 要求本ui网页使用https访问
  -v $PWD/server.crt:/etc/apache2/server.crt:ro \ # ssl证书文件（设置了ENV_USE_SSL=yes才需要）
  -v $PWD/server.key:/etc/apache2/server.key:ro \ # ssl私钥（设置了ENV_USE_SSL=yes才需要）
  konradkleine/docker-registry-frontend:v2
```

- 注意事项
1. 注意这个容器会监听2个端口，如果你使用https即设置了`-e ENV_USE_SSL=yes`，那么请使用443端口，否则使用http的使用80端口；
2. 访问对应协议和网页对应端口就可以看到你私有仓库的容器了，如果你在私有仓库设置了登录访问权限，那么浏览器会弹框要求你输入用户名和密码，才能查看私有仓库中的镜像。