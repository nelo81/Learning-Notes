## docker 私有仓库搭建

- 运行以下命令：
```
docker pull registry
docker run -d -p 5000:5000 registry
或者 docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
# 把容器中的镜像仓库映射到本机中，防止容器删除后失去当中的镜像
```

- 可以把本地docker镜像发送到私有仓库中
```
docker tag image_name ip:port/image_name # ip:port 指仓库的地址
docker push ip:port/image_name
```
如果出现异常`Get https://hub.docker.jiankunking.io:5000/v1/_ping: http: server gave HTTP response to HTTPS client`，则需要修改 /etc/docker/daemon.json（没有就创建），添加私有仓库地址，格式如下
```
{
    "insecure-registries": [
        "registry-ip1:port1", 
        "registry-ip2:port2"
    ]
}
```
然后重启 docker 服务即可重新 push 成功

- 从仓库中拉取镜像
`docker pull 10.10.105.71:5000/image_name`(镜像名对应你push上去的)

- 查询私有仓库镜像和 tag
镜像：`curl  http://10.10.105.71:5000/v2/_catalog`
tag：`curl  http://10.10.105.71:5000/v2/tonybai/busybox/tags/list`
**注意我这里使用的是2版本,1版本是`curl  http://10.10.105.71:5000/v1/search`**

*注意：push和pull默认以docker hub仓库，除外你使用 tag命令附加了ip:port，就可以push或者pull到指定仓库，也就是上面的步骤为什么有tag的步骤指定私有仓库的ip:port*


### 限制用户访问权限
上面我们使用了"insecure-registries"，使得我们可以用http访问registry，但是这样存在安全问题，所以我们接下来介绍使用https访问并且需要登录授权的方法

- 获取数字证书 \*\*crt.pem 文件和私钥 \*\*key.pem 文件（如果需要免费证书具体可以参考"/protocol/申请 Let's Encrypt 数字证书.md"或者使用openssl自己签发证书）

- 创建放置证书的目录并复制证书文件进去（这里以及下面新建的文件夹目录可以自己决定）
```
sudo mkdir certs
sudo cp **crt.pem certs/yourdomain.crt # yourdomain改成你的域名
sudo cp **key.pem certs/yourdomain.key
```

- 创建放置用户密码文件的目录，利用htpasswd写入授权的用户和密码
```
mkdir auth
docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
```
testuser 改成你的用户名，testpassword 改成你的密码

- 启动registry容器
```
docker run -d -p 5000:5000 --name registry \
  -v yourpath/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v yourpath/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```
或者使用docker-compose
```
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - yourpath/data:/var/lib/registry
    - yourpath/certs:/certs
    - yourpath/auth:/auth
```
把 yourpath 改成你自己创建的目录

- 查看证书是否正常
`curl -i -k -v https://yourdomain:5000`
如果正常会输出
```
* Rebuilt URL to: https://yourdomain:5000/
*   Trying xx.xx.xx.xx （服务器 IP ）...
* Connected to dockie.mydomain.com (xx.xx.xx.xx) port 5000 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* SSL connection using TLS_RSA_WITH_AES_128_CBC_SHA
* Server certificate:
* 	subject: CN=dockie.mydomain.com
* 	start date: May 05 00:48:00 2016 GMT
* 	expire date: Aug 03 00:48:00 2016 GMT
* 	common name: dockie.mydomain.com
* 	issuer: CN=Let's Encrypt Authority X3,O=Let's Encrypt,C=US
> GET / HTTP/1.1
> User-Agent: curl/7.40.0
> Host: dockie.mydomain.com:5000
> Accept: */*
> 
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
< Cache-Control: no-cache
Cache-Control: no-cache
< Date: Sat, 07 May 2016 06:28:17 GMT
Date: Sat, 07 May 2016 06:28:17 GMT
< Content-Length: 0
Content-Length: 0
< Content-Type: text/plain; charset=utf-8
Content-Type: text/plain; charset=utf-8
```
如果失败考虑同步一下时区

- docker客户端登录
```
docker login yourdomain:5000
```

- 之后就有权限执行 docker pull, docker push 等操作

- 注意事项
1. 这里注意push和pull时，镜像名必须改成 yourdomain:5000/image，不能用ip要写域名，否则无法使用证书验证 
2. 如果遇到X509错误，先查看该网站证书的路径（可以从浏览器访问该域名查看证书），然后从网上下载并更新对应需要的上级证书
```
sudo mkdir /usr/local/share/ca-certificates/yourdomain
sudo cat lets-encrypt-x3-cross-signed.pem >> /usr/local/share/ca-certificates/dockie.mydomain.com/ca.crt
sudo cat isrgrootx1.pem >> /usr/local/share/ca-certificates/dockie.mydomain.com/ca.crt
sudo update-ca-certificates
```
以上是ubuntu上的例子，其他系统的目录有不同，lets-encrypt-x3-cross-signed.pem 和 isrgrootx1.pem 是从 [Let's Encrypt Certificates][1] 上复制或者下载下来的

[1]:https://letsencrypt.org/certificates/