## maven 集成 docker

- 使用 docker-maven-plugin 插件

- "pom.xml"示例(<\build><\plugins><\/plugins><\/build>中的)，有两种写法

1. 用execution绑定docker命令到maven生命周期中
```
<build>
	<plugins>
		<plugin>
			<groupId>com.spotify</groupId>
			<artifactId>docker-maven-plugin</artifactId>//声明插件
			<version>0.4.13</version>
			<configuration>
				//指定远程docker，使用远程docker命令，不写默认是 localhost:2375
				<dockerHost>http://192.168.0.123:2375</dockerHost>
				<imageName>${project.artifactId}:${project.version}</imageName> //生成的镜像名
				<dockerDirectory>src/main/docker</dockerDirectory> //Dockerfile 位置
				<resources>
					<resource> //把生成的 jar 包引进来，基本可以照搬
						<targetPath>/</targetPath>
						<directory>${project.build.directory}</directory>
						<include>${project.build.finalName}.jar</include>
					</resource>
				</resources>
			</configuration>
			<executions>
				<execution>
					<id>build-image</id>
					<phase>package</phase> //在 mvn package 阶段
					<goals>
						<goal>build</goal> //执行 docker build，使用Dockerfile生成镜像到指定ip的docker
					</goals>
				</execution>
				<execution>
					<id>tag-image</id>
					<phase>package</phase> //在 mvn package 阶段
					<goals>
						<goal>tag</goal> //执行 docker tag 复制镜像并更改镜像名
					</goals>
					<configuration>
						//镜像原名
						<image>${project.artifactId}</image>
						//镜像新的名字，${docker.registry} 是自己定义的，指定仓库地址
						<newName>${docker.registry}/${project.artifactId}:latest</newName>
					</configuration>
				</execution>
				<execution>
					<id>push-image</id>
					<phase>package</phase> //还是在 mvn package 阶段
					<goals>
						<goal>push</goal> //执行 docker push 把镜像推送到仓库
					</goals>
					<configuration>
						//要发送的镜像名，一般和tag中的newName一致
						<imageName>${docker.registry}/${project.artifactId}:latest</imageName>
					</configuration>
				</execution>
			</executions>
		</plugin>
		.......... //其他 plugin
	</plugins>
</build>
```
配置好之后执行`mvn clean package`
即可生成并且发送镜像到仓库
其中executions中的步骤可以自行调整，需要哪些操作来加减

2. 直接使用 docker 插件命令
```
<build>
	<plugins>
		<plugin>
			<groupId>com.spotify</groupId>
			<artifactId>docker-maven-plugin</artifactId>//声明插件
			<version>0.4.13</version>
			<configuration>
				//指定远程docker，使用远程docker命令，不写默认是 localhost:2375
				<dockerHost>http://192.168.0.123:2375</dockerHost>
				<imageName>${project.artifactId}:${project.version}</imageName> //生成的镜像名
				<dockerDirectory>src/main/docker</dockerDirectory> //Dockerfile 位置
				<resources>
					<resource> //把生成的 jar 包引进来，基本可以照搬
						<targetPath>/</targetPath>
						<directory>${project.build.directory}</directory>
						<include>${project.build.finalName}.jar</include>
					</resource>
				</resources>
			</configuration>
		</plugin>
		.......... //其他 plugin
	</plugins>
</build>
```
配置完之后执行`mvn clean package docker:build`
如果要发送到仓库的话，在imageName前面加上仓库 "ip:port/" 地址
然后执行`mvn clean package docker:build -DpushImage`就会在build之后自动push