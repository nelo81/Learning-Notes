## ubuntu 16.04 server hadoop安装和配置

### 在master机的操作

1. 修改hostname
`sudo vim /etc/hostname`

2. 修改hosts文件
`sudo vim /etc/hosts`,如添加如下
```
192.168.1.141 master
192.168.1.142 slave1
192.168.1.137 slave2
```
(左边ip对应每台虚拟机的ip,右面对应主机名)

3. 安装JDK
使用 `sudo apt-cache search jdk` 查找jdk
使用 `sudo apt-get install (java包名)` 安装jdk
设置环境变量
`sudo vim /etc/profile`,然后添加对应自己目录的变量
```
export JAVA_HOME=/usr/java/jdk1.7.0_25/
export JRE_HOME=/usr/java/jdk1.7.0_25/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```
source /etc/profile 生效

4. 安装hadoop
下载hadoop tar.gz包并解压，放在 /usr/下，并授权给你要用的hadoop用户
`chown -R hadoop:hadoop(用户名:用户名) hadoop2.7.4 (文件名)`

- 设置环境变量
`sudo vim /etc/profile`,然后添加对应自己目录的变量
```
export HADOOP_HOME="/opt/hadoop-2.7.3"
export PATH="$HADOOP_HOME/bin:$PATH"
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

- 然后开始修改配置(所有配置文件在hadoop文件夹下 /etc/hadoop/ 下)
注：下面/home/xiaolei是你linux用户的文件夹，hadoop1为master主机名

- hadoop-env.sh: 将$JAVA_HOME改成你真正的java目录
```
export JAVA_HOME=/opt/jdk1.8.0_111
```

- slaves: 只有master需要，指定从机(主机名，在第2步设置的可以对应从机ip)
```
slave1
slave2
```

- core-site.xml
```
<configuration>
        <!-- 指定hdfs的nameservice为ns1 -->
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://Hadoop1:9000</value>
        </property>
        <!-- Size of read/write buffer used in SequenceFiles. -->
        <property>
         <name>io.file.buffer.size</name>
         <value>131072</value>
       </property>
        <!-- 指定hadoop临时目录,自行创建 -->
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/xiaolei/hadoop/tmp</value>
        </property>
</configuration>
```

- hdfs-site.xml
```
<configuration>
    <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop1:50090</value>
    </property>
    <property>
      <name>dfs.replication</name>
      <value>2</value>
    </property>
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>file:/home/xiaolei/hadoop/hdfs/name</value>
    </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>file:/home/xiaolei/hadoop/hdfs/data</value>
    </property>
</configuration>
```

- yarn-site.xml
```
<configuration>
<!-- Site specific YARN configuration properties -->
<!-- Configurations for ResourceManager -->
     <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
     </property>
     <property>
           <name>yarn.resourcemanager.address</name>
           <value>hadoop1:8032</value>
     </property>
     <property>
          <name>yarn.resourcemanager.scheduler.address</name>
          <value>hadoop1:8030</value>
      </property>
     <property>
         <name>yarn.resourcemanager.resource-tracker.address</name>
         <value>hadoop1:8031</value>
     </property>
     <property>
         <name>yarn.resourcemanager.admin.address</name>
         <value>hadoop1:8033</value>
     </property>
     <property>
         <name>yarn.resourcemanager.webapp.address</name>
         <value>hadoop1:8088</value>
     </property>
     <property>
         <name>yarn.nodemanager.resource.memory-mb</name>
         <value>1024</value><!-- 内存大小，单位是mb，默认是8192(8g) -->
     </property>
     <property>
         <name>yarn.nodemanager.resource.cpu-vcores</name>
         <value>1</value><!-- 使用cpu核数，默认是8 -->
     </property>
</configuration>
```

- mapred-site.xml
```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop1:10020</value>
  </property>
  <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop1:19888</value>
  </property>
</configuration>
```

### 复制配置到slave机

1. 复制master主机到slave1,slave2(具体操作看'/hyper-v/hyper-v虚拟机复制.md')

2. 配置Master无密码登录所有Salve('/protocol/SSH无密码登录.md')

### 启动hadoop

1. 格式化结点
`hdfs namenode -format`

2. 集群启动
```
cd $HADOOP_HOME/sbin
./start-all.sh
```

3. 分别在master和slave机输入 jps

- master:
```
ResourceManager
NameNode
SecondaryName
Jps
```
如果你在slaves文件中有写localhost,还有可能有DataNode和NodeManager

- slave:
```
Jps
DataNode
NodeManager
```

如果上述服务显示成功，说明启动成功

### 注意，如果在 windows 上启动可能会出现以下问题

1. Failed to locate the winutils binary in the hadoop binary path Java.io.IOException: Could not locate executablenull\bin\winutils.exe in the Hadoop binaries.

2. org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z

解决方法：从 [github][1] 下载对应版本文件夹，把其中的 "hadoop.dll" 以及 "winutils.exe" 都复制到 "hadoop/bin" 目录下, 重新运行就可以同时解决上面两个问题。

[1]:https://github.com/steveloughran/winutils