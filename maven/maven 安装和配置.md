## maven 入门级安装和配置

1. 下载 maven 压缩包并解压

2. 配置环境变量
```
M2_HOME = "maven 根目录"
PATH = %M2_HOME%/bin;PATH
```

3. 查看是否安装成功 `mvn -v`

4. 修改根目录下 "conf/settings.xml"， 主要增加以下几个部分 \
  - 你的仓库目录： 默认放在${user.home}/.m2/repository，以后mvn install 的包和你要用到的包都放在这里
  ```
  <settings>
    ....
      <localRepository>/path/to/repo</localRepository>
    ....
  </settings>
  ```
  - 公共仓库镜像地址，导入外部的依赖包时，用镜像仓库速度快不止一点，下面是阿里云的镜像示例
  ```
  <mirrors>
    .....
      <mirror>
          <id>alimaven</id>
          <mirrorOf>central</mirrorOf>
          <name>aliyun maven</name>
          <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
      </mirror>
  </mirrors>

  <mirror>
      <id>repo1</id>
      <mirrorOf>central</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://repo1.maven.org/maven2/</url>
  </mirror>

  <mirror>
      <id>repo2</id>
      <mirrorOf>central</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://repo2.maven.org/maven2/</url>
  </mirror>
  ```

5. 新建一个 maven 项目 (第一次运行可能会下载很多的启动包) \
`mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`
- 其中 -DgroupId,-DartifactId 代表这个maven项目资源的唯一身份 id，-DarchetypeArtifactId 是用什么新建项目，这里是普通的项目，-DinteractiveMode 表示是否用交互模式新建项目，就是会在命令行问你建项目的参数，包括上面的 groupId 等。

6. maven 项目结构
```
maven 根目录
│  
│  pom.xml
│
├─src
│  ├─main
│  │  ├─java
│  │  │  └─package
│  │  │     └─app.java
│  │  │
│  │  └─resources
│  │
│  └─test
│      └─java
│         └─package
│            └─app.java
│
└─target
```
- pom.xml: maven 项目配置文件，代表这是一个 maven 项目
- src/main/java: 放置源代码文件
- src/main/resources: 放置静态文件
- src/test/java: 放置测试代码
- target: 打包输出文件夹
- 这个文件结构的名字都不能改动，否则执行 mvn 命令时会扫描不到，特别是这个 "resources" 不要少个 's' 不然 maven 打包时不会扫描里面的静态文件