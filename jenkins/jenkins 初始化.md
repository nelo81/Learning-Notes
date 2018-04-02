## jenkins 初始化

- 下载和启动jenkins，有3种方法
1. 使用docker
```
docker pull jenkins
docker run -p 8080:8080 -p 50000:50000 \
-v /your/home:/var/jenkins_home \ # 放置jenkins数据的主目录
--name jenkins jenkins
```
2. 直接下载war包并运行（不推荐）
```
java -jar jenkins.war \
--httpPort=$HTTP_PORT \ # 用来设置jenkins运行时的web端口，默认8080。
--httpsPort=$HTTP_PORT \ # 表示使用https协议。
--httpListenAddress=$HTTP_HOST \ # 用来指定jenkins监听的ip范围，默认为所有的ip都可以访问此jenkins server。
```
3. 使用java service wrapper	启动2中下载的 jenkins.war（推荐，可简单移植或者设置开机自启动）
(jsw的使用详见"/java service wrapper/jsw 使用入门.md")
其中 "/conf/myapp.conf" 修改以下变量
```
wrapper.java.classpath.1=../lib/jenkins.war
wrapper.java.classpath.2=../lib/wrapper.jar

wrapper.java.additional.1=-DJENKINS_HOME=../data # 放置jenkins基本文件的目录，默认是~/.jenkins，用相对路径方便移植
wrapper.java.additional.2=-Djava.io.tmpdir=../tmp # java 临时文件目录
wrapper.java.additional.3=-server

wrapper.app.parameter.1=Main
wrapper.app.parameter.2=--httpPort=8080
```
"/bin/myapp" 修改以下变量
```
APP_NAME="Jenkins"
APP_LONG_NAME="Jenkins Continuous build server"

WRAPPER_CONF="../conf/myapp.conf"
``` 
然后执行`./bin/myapp start`就行了

- jenkins 初始化
1. 一开始访问网页会让你输入密码，不过它会提示你密码的目录位置的，或者也可以在日志里看到
2. 选择自己需要的插件安装，或者安装推荐的
3. 创建第一个管理员，输入用户名密码等信息即可

**初始化完成！**

- 移植到其他linux机器上
把整个文件夹复制过去，然后执行`./bin/myapp start`就行了