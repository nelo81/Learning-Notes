## spring cloud 用 docker 部署

1. 首先说用 docker 部署 spring boot

- 先用 maven 把 docker 打包，然后在把打包出来的 app.jar 和下面编写的 dockerfile 放在同一目录下，最后执行 `docker build`，`docker run`

```
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD water-route.jar app.jar
ENTRYPOINT ["java", \
"-Djava.security.egd=file:/dev/./urandom", \
"-jar","/app.jar", \
"--spring.profiles.active=prod"]
EXPOSE 80
```
- 这个 frolvlad/alpine-oraclejdk8:slim 镜像肯定有 /tmp，其他目录就不知道了
- 最后的 `--spring.profiles.active=prod` 参数是 spring boot 的配置参数

2. docker 部署 spring cloud

- spring cloud 和 spring boot 部署的不同之处是，spring cloud 中不同的 spring boot 服务要互相调用，但是用 docker 部署时，注册中心中记录的主机名并不是 ip，而是一串 docker 容器的 id，这样会使不同服务之间无法相互调用，例如 feign。

- 因此，在上面步骤中使用 `docker build` 建立好全部的镜像之后，使用 docker-compose 统一运行便可解决这个问题

- 当然，如果你不是用 docker 部署的就没有这个问题了