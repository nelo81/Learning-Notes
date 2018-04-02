## kafka 启动

1. 下载并解压对应版本 tar.gz 包

2.  配置外网访问 ，修改 "config/server.properties" 添加以下配置
```
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://119.29.206.251:9092
```
- listeners 是代表 kafka 真正绑定的地址，设置为 0.0.0.0 即所有主机都可以访问
- advertised.listeners 代表让客户端访问的地址，即当前服务器的外网地址
- PLAINTEXT 是协议，还能选择 SSL 等

3. 后台启动: `nohup bin/kafka-server-start.sh config/server.properties &`

- 备注: 假如启动时出现以下错误
```
Java HotSpot(TM) Server VM warning: INFO: os::commit_memory(0x67e00000, 1073741824, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /opt/kafka_2.11-0.9.0.1/hs_err_pid2249.log
```
则需要修改 "bin/kafka-server-start.sh" 下的 `export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"` 改成适合自己服务器内存的大小，例如 `export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"`