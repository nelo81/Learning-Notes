## maven 中 pom.xml 的基本使用

1. maven 项目的坐标 (项目建立时就有的)
```
<groupId>group</groupId>
<artifactId>group-artifact</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>
```
- groupId 和 artifactId 代表这个项目的id，以便被其他项目引入
- version：版本号
- packaging: 打包格式 (这里是 jar)

2. maven parent (作用在 3 有说)
```
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.5.8.RELEASE</version>
  <relativePath />
</parent>
```

3. maven 依赖
```
<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.3.5.RELEASE</version>
  </dependency>

  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.3.5.RELEASE</version>
  </dependency>
<dependencies>
```
- 每个 dependency 里面的 groupId，artifactId 就相当与其他 maven 项目中的 1：maven 的坐标
- 如果 2：parent 中的 dependencies 有声明和这个 pom.xml 一样的依赖，dependency 中可以不写版本号（只是版本号）

4. maven 构建配置
```
<build>
  <finalName>my-app</finalName>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    <plugin>
        <artifactId>maven-resources-plugin</artifactId>
        <executions>
            <execution>
                <id>copy-Dockerfile</id>
                <phase>process-sources</phase>
                <goals>
                    <goal>copy-resources</goal>
                </goals>
                <configuration>
                    <outputDirectory>${basedir}/target</outputDirectory>
                    <resources>
                        <resource>
                            <directory>${basedir}/src/main/docker</directory>
                        </resource>
                    </resources>
                </configuration>
            </execution>
        </executions>
    </plugin>
  </plugins>
</build>
```
- finalName: 最后打包出来的文件名
- plugins: 打包时的依赖，不同的 plugin 插件有不同的用法
- plugin: 也是 maven 项目，所以用 groupId 和 artifactId 引入
- executions: plugin 要执行的操作
- phase： plugin 在生命周期的哪个阶段执行这个 execution
- goals： plugin 要执行的指定命令，像 mvn 的 install，docker 的 build 命令那样
- configuration: 执行 goal 命令时的一些参数

5. maven 常量
```
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  <java.version>1.8</java.version>
  <aaa>123</aaa>
</properties>
```
- 这里设置了之后，可以在 3,4 配置中使用 `${java.version}` 和 `${aaa}` 调用它们的值
- 除了这里设置的常量以外，还有这个 xml 中的所有标签，因为 pom.xml 的根标签就是 'project'，所以: \
`${project.artifactId}` 就可以调用 `<artifactId>???</artifactId>` 的值 \
`${project.build.finalName}` 就可以调用 `<build><finalName>???</finalName></build>` 的值。

6. maven 命令和 pom.xml 的关系
- 基本上所有要经过生命周期的命令都需要在 pom.xml 下的文件夹才能执行。
- 执行需要打包的命令后，如 `mvn package` ，会检测本地仓库否有 dependencies 和 plugins 的包，如果没有就会从公共仓库下载，本地仓库目录和公共仓库源的设置在 "/maven/maven 安装和配置.md" 中有写。
- 执行命令时如果经过 plugin-executions-phase 中指定的周期，就会执行 plugin-goals 的命令。