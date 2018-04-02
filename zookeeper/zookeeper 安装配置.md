## zookeeper 安装配置

1. 下载并解压对应版本 tar.gz 包

2. 复制一份配置，`cp conf/zoo_sample.cfg conf/zoo.cfg` (默认使用 "zoo.cfg" 配置)

3. 修改 "zoo.cfg" 实例
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/app/soft/zookeeper-3.4.6/data （换成真实输出目录）
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890
```

4. "zoo.cfg" 配置说明
> **tickTime：** 这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳，以毫秒为单位。

> **initLimit：** LF初始通信时限，集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）

> **syncLimit：** 集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。

> **dataDir：** 顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。

> **clientPort：** 这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。　　　
> **dataLogDir：** 日志文件目录，Zookeeper保存日志文件的目录

> **服务器名称与地址：** 集群信息（服务器编号，服务器地址，LF通信端口，选举端口），规则如：**server.N=yyy:A:B** ，其中N表示服务器编号，YYY表示服务器的IP地址，A为LF通信端口，表示该服务器与集群中的leader交换的信息的端口。B为选举端口，表示选举新leader时服务器间相互通信的端口（当leader挂掉时，其余服务器会相互通信，选择出新的leader）。一般来说，集群中每个服务器的A端口都是一样，每个服务器的B端口也是一样。但是当所采用的为伪集群时，IP地址都一样，只能时A端口和B端口不一样。

5. 启动: `bin/zkServer.sh start`

6. 查看是否成功: `bin/zkServer.sh status`