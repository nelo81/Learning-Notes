## spring cloud 服务配置

- 这里只介绍比较主要的，其他的使用默认值应该也没什么问题，如心跳时间间隔等

- 服务注册中心(eureka server):
1. eureka.server.enable-self-preservation (default:true) \
注册中心的保护机制，Eureka 会统计15分钟之内心跳失败的比例低于85%将会触发保护机制，如果这个参数为 true ，那么在出现保护机制之后，之后任何服务与服务中心心跳断开连接时，服务中心也不会将其剔除。

2. security.basic.enable (default:false) \
启用安全验证，即开启后，其他服务注册到这个注册中心需要密码，具体操作在下面"服务注册类"的 1 有说到

3. security.user.name \
安全校验的用户名

4. security.user.password \
用户的密码

- 服务实例:
1. eureka.instance.prefer-ip-address (default:false) \
不使用主机名来定义注册中心的地址，而使用IP地址的形式，如果设置了 eureka.instance.ip-address 属性，则使用该属性配置的IP，否则自动获取除环路IP外的第一个IP地址。

2. eureka.instance.ip-address \
eureka.instance.prefer-ip-address 为 true 时使用的 ip 地址

```
注意：1,2 这两个配置很重要，表示其他服务访问这个服务时使用的 socket/url 地址，之前在腾讯云的服务器上，服务器内部只有内网网卡，它是通过两个网卡映射的，所以在腾讯云服务器外网时可以映射到内网 ip， 但是如果不设置 eureka.instance.ip-address，那么其他服务在访问这个服务时就会访问到这个服务器的内网 ip ，这是不可被访问到的，所以要把 eureka.instance.ip-address 设置成外网 ip 才能访问，另外如果 eureka.instance.prefer-ip-address 为 false，那么其他服务访问就要通过主机名，如果你的主机名不能解析成 ip 地址，那么更加不能访问了。
```

3. eureka.instance.hostname \
当前服务的主机名

4. eureka.instance.appname (default:spring.application.name) \
服务名称

- 服务注册类:
1. eureka.client.serviceUrl \
指定服务注册中心地址，类型为 HashMap，并设置有一组默认值，默认的Key为 defaultZone；默认的Value为 http://localhost:8761/eureka ，如果服务注册中心为高可用集群时，多个注册中心地址以逗号分隔。 \
如果服务注册中心加入了安全验证，这里配置的地址格式为： http://\<username\>:\<password\>@localhost:8761/eureka 其中 \<username\> 为安全校验的用户名；\<password\> 为该用户的密码 \
自定义了服务中心地址或者是远程地址的话，把 http://localhost:8761 改成自己的即可

2. eureka.client.fetchRegistry (default:true) \
检索服务

3. eureka.client.registerWithEureka (default:true) \
注册到服务中心

```
在注册中心要把 2,3 这两个配置都设成 false
```