## ubuntu 16.04 server hive安装

注：这里的 hive 安装在 hadoop master 机上

1. 下载 hive-2.3.0.tar.gz 并解压

2. 修改配置

- 修改环境变量
`sudo vim /etc/profile`添加
```
export HIVE_HOME=/chenjie/apache-hive-2.3.0-bin
export PATH=$PATH:$HIVE_HOME/bin
```
`source /etc/profile`生效

- 复制默认配置文件模版
```
cp hive-default.xml.template hive-site.xml
cp hive-env.sh.template hive-env.sh
```

- hive-site.xml: 修改以下配置（对应自己的远程数据库，用户名和密码）
```
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value>
    <description>password to use against metastore database</description>
  </property>
</configuration>
```
并且将全部 ${system:java.io.tmpdir} 和 ${system:user.name} 替换成自己的目录

- hive-env.sh: 添加以下配置(对应自己实际的)
`HADOOP_HOME=/usr/hadoop`

- mysql-connector-java
下载 mysql-connector-java-5.1.44.tar.gz (版本不限)并解压，并将其中的 mysql-connector-java.jar 复制到 hive 文件夹下 /lib 中

3. 启动 hive

- 先启动 hadoop （看 /hadoop/ubuntu 16.04 server hadoop安装和配置.md）

- `schematool -initSchema -dbType mysql` 初始化 hive 数据库

- 输入 `hive` 运行 hive，出现 `>hive` 即成功