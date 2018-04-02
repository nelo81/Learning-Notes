## 配置Master无密码登录所有Salve

- 在master机上
`ssh-keygen -t rsa -P ''`
生成无密码的ssh密钥

- 接着在Master节点上做如下配置，把id_rsa.pub追加到授权的key里面去
`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

- 用root用户登录修改SSH配置文件"/etc/ssh/sshd_config"的下列内容
检查下面几行前面”#”注释是否取消掉：没有就删掉 '#'
RSAAuthentication yes # 启用 RSA 认证
PubkeyAuthentication yes # 启用公钥私钥配对认证方式
AuthorizedKeysFile  %h/.ssh/authorized_keys # 公钥文件路径 

- 重启ssh服务
`sudo service ssh restart`

- 使用ssh-copy-id命令将公钥传送到远程主机上(hadoop@slave1指slave1主机的hadoop用户)
```
ssh-copy-id hadoop@slave1
ssh-copy-id hadoop@slave2
```
注意这条命令是把自己的公钥添加到目标主机用户的 ~/.ssh/authorized_keys，而不是替换

- 验证无密码登录，在master主机输入 ssh slave1, ssh slave2 看是否可以无密码登录