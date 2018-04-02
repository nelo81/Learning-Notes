## spring boot 使用 ssl 加密

1. 先去申请一个证书，例如 Let's Encrypt ("/protocol/申请 Let's Encrypt 数字证书.md")

2. 把得到的证书转换成 PKCS12 格式

- 例如，你拥有证书 "cert.pem" 和私钥 "key.pem"
- 那么可以执行 `openssl pkcs12 -export -in cert.pem -inkey key.pem -out keystore.p12` (当中可能需要你输入密码，下面的 key-store-password 就是指这个)，就可以得到 keystore.p12 的证书文件
- 可以通过 `keytool -list -keystore keystore.p12 -storetype pkcs12 -v -storepass 000` 查看证书别名

3. 最后在 spring boot 中的配置文件 "application.yml" 添加如下配置项
```
server:
  port: 8443
  ssl:
    key-store: ${file.ssl-dir}/keystore.p12 # 证书路径
    key-store-password: "000" # 证书密码
    keyStoreType: PKCS12 # 证书格式
    keyAlias: 1 # 证书别名
```

4. 启动 spring boot 程序后就可以在浏览器中访问 https://yourdomain.com:8443 啦