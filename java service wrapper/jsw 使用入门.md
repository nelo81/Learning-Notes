## jsw 使用入门

- 下载对应版本的 jsw 包，解压（假设文件夹名称为jsw）

- 创建一个新文件夹mywrapper作为你要运行的项目的根目录，并把必要的文件复制进去

	- jsw文件结构如下：
```
jsw/
├── bin
│   ├── demoapp
│   ├── testwrapper
│   └── wrapper
├── conf
│   ├── demoapp.conf
│   ├── wrapper.conf
│   └── wrapper-license.conf
├── doc
│   ├── index.html
│   ├── revisions.txt
│   ├── wrapper-development-license-1.3.txt
│   ├── wrapper-server-license-1.3.txt
│   └── wrapper-tsims-addendum-1.3.txt
├── lang
│   ├── wrapper_de.mo
│   ├── wrapper_ja.mo
│   ├── wrapperjni_de.mo
│   ├── wrapperjni_ja.mo
│   ├── wrapperTestApp_de.mo
│   └── wrapperTestApp_ja.mo
├── lib
│   ├── libwrapper.so
│   ├── wrapperdemo.jar
│   ├── wrapper.jar
│   └── wrappertest.jar
├── logs
│   └── wrapper.log
├── README_de.txt
├── README_en.txt
├── README_es.txt
├── README_ja.txt
└── src
    ├── bin
    │   └── sh.script.in
    └── conf
        ├── wrapper.conf.in
        ├── wrapper.conf.in_ja
        └── wrapper-license-time.conf
```

	- 新建的项目文件如下：
```
mywrapper/
├── bin
│   ├── myapp # 复制自 jsw/src/bin/sh.script.in 并改名
│   └── wrapper # 复制自 jsw/bin/wrapper
├── conf
│   ├── myapp.conf # 复制自 jsw/src/conf/wrapper.conf.in 并改名
│   └── wrapper-license.conf # 复制自 jsw/conf/wrapper-license.conf (community 版不需要)
├── lib
│   ├── depencies.jar # 你自己项目需要的依赖包，不止一个
│   ├── libwrapper.so # 复制自 jsw/lib/libwrapper.so
│   └── wrapper.jar # 复制自 jsw/lib/wrapper.jar
└── logs
    └── wrapper.log # 不创建也会根据配置自动生成
```

		- 觉得麻烦的可以使用以下脚本建立文件结构
```
sudo mkdir jenkins/bin
sudo mkdir jenkins/logs
sudo mkdir jenkins/lib
sudo mkdir jenkins/conf
sudo cp wrapper-linux-x86-64-3.5.34/bin/wrapper jenkins/bin/
sudo cp wrapper-linux-x86-64-3.5.34/src/bin/sh.script.in jenkins/bin/
sudo cp wrapper-linux-x86-64-3.5.34/src/conf/wrapper.conf.in jenkins/conf/
sudo cp wrapper-linux-x86-64-3.5.34/lib/wrapper.jar jenkins/lib/
sudo cp wrapper-linux-x86-64-3.5.34/lib/libwrapper.so jenkins/lib/
sudo mv jenkins/bin/sh.script.in jenkins/bin/jenkins
sudo mv jenkins/conf/wrapper.conf.in jenkins/conf/jenkins.conf
```

- 修改配置

	- "/conf/myapp.conf"
```
# 依赖包
wrapper.java.classpath.1=../lib/wrapper.jar
wrapper.java.classpath.2=...(your denpencies package)
wrapper.java.classpath.3=...
wrapper.java.classpath.4=...
...

wrapper.java.additional.1=  # 附加参数（java命令可选参数）
wrapper.java.additional.2=
...

#该类需要实现WrapperListener 接口并保证WrapperManager 得到初始化(调用WrapperManager.start(WrapperListener listener, String[] args) 方法)。默认使用 org.tanukisoftware.wrapper.WrapperSimpleApp
wrapper.java.mainclass=com.helloworld.hello.HelloWorld

wrapper.app.parameter.1=  # Main函数参数
wrapper.app.parameter.2= 

# Initial Java Heap Size (in MB)
wrapper.java.initmemory=64 # java堆初始内存，相当于Xms64m

# Maximum Java Heap Size (in MB)
wrapper.java.maxmemory=128 # java堆最大内存，相当于Xmx128m
...
```

	- "/bin/myapp"
```
APP_NAME="Jenkins" # 应用名称，以后便于在 ps -ef|grep 命令中找到
APP_LONG_NAME="Jenkins Continuous build server" # 应用全名称

WRAPPER_CMD="./wrapper"  # 调用wrapper命令，这个一般不变
WRAPPER_CONF="../conf/myapp.conf" # 读取配置文件位置

RUN_AS_USER=user # 使用这个用户的权限操作文件
```

	- 启动应用
```
cd mywrapper/bin
./myapp start
```
如果出现运行不了myapp，使用`chmod 755 myapp`修改权限即可

	- 使用root权限启动："/bin/myapp"中设置`RUN_AS_USER=root`，然后执行`sudo ./myapp start`

- 设置开机启动，两种方法

	- 手动：在"/etc/init.d/"创建文件myapp，然后连接`ln -s mywrapper/bin/myapp /etc/init.d/myapp`，使得开机通过快捷方式"/etc/init.d/myapp"自动运行"mywrapper/bin/myapp"

	- 自动：`./myapp install`

- 关闭或者重启应用
```
./myapp stop
./myapp restart
```