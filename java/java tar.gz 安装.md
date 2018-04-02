## java tar.gz 在 linux 上安装

1. 下载对应版本 tar.gz 包并解压，得到java目录

2. 执行 `sudo vim /etc/profile`，新增以下代码并保存
```
export JAVA_HOME=/usr/java/jdk1.8.0_65 # according to your java path
export JRE_HOME=$JAVA_HOME/jre 
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib 
export PATH=$JAVA_HOME/bin:$PATH
```

3. 执行`source /etc/profile`

4. 执行 `java`，`javac` 查看输出是否正确

- 备注：如果最后出现 `cannot execute binary file: Exec format error` 的错误，可能是安装错了32位或者64位