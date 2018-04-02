## spark 安装启动服务

1. 下载并解压对应版本 tar.gz 包 (我这里下载的是2.2.1版本，不需要在下载 scala，java 还是要的)

2. 复制一份配置，`cp conf/spark-env.sh.template conf/spark-env.sh` (默认使用 "spark-env.sh" 配置)

3. 修改 "spark-env.sh"，增加以下示例
```
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64
export SCALA_HOME=/home/bl/app/scala-2.10.4
(在旧版本中可能需要添加以上两个配置，因为没有自带 scala)
export SPARK_MASTER_HOST=0.0.0.0
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8087
export SPARK_WORKER_PORT=7078
export SPARK_WORKER_WEBUI_PORT=8088
export SPARK_WORKER_CORES=1
export SPARK_WORKER_MEMORY=256m
```

4. "spark-env.sh" 配置说明
> **SPARK\_MASTER\_HOST：** master 监听主机 host/ip 的范围，默认 localhost/127.0.0.1

> **SPARK\_MASTER\_PORT：** master 监听端口

> **SPARK\_MASTER\_WEBUI_PORT：** 网页 UI 服务的端口

> **SPARK\_WORKER\_PORT：** worker 监听端口

> **SPARK\_WORKER\_CORES：** worker 可占用 cpu 核数　　　

> **SPARK\_WORKER\_MEMORY：** worker 可最大分配内存 (e.g. 128m, 1g)

5. 用和 1,2,3,4 相同的方法配置集群中的其他 slave 服务器，此时 SPARK\_MASTER 参数可以无视

6. master 主机修改 "conf/slaves" \
默认只有一个 slave (localhost), 在下面添加你集群中配置好的其他服务器

7. 配置Master无密码登录所有Salve (/protocol/SSH无密码登录.md)

8. 启动集群服务 \
执行 `./sbin/start-all.sh` 会同时启动 master 和所有的 slave 主机的 spark 服务

9. 停止集群服务 \
执行 `./sbin/stop-all.sh`