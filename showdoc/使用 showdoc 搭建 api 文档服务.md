## 使用 showdoc 搭建 api 文档服务

- 介绍：用于方便搭建 api doc 文档的 web 服务端，提供模版，轻松编写文档，并在浏览器可访问，也可以导出到各种格式文件到本地

- 安装：
	- 在[github][1]下载zip包，解压到安装了docker的服务器上
	- 进入showdoc跟目录下，执行`docker build -t showdoc ./`生成镜像
	- 用docker启动容器`docker run -d -p 4999:4999 --name showdoc showdoc`

- 打开网页初始化：在浏览器访问 `http://www.domain:4999/install` 进行初始化

- 开始使用
![](./imgs/showdoc.png)
右上角"项目"可以导出到html,markdown等格式文件
左边栏可新建目录和页面

[1]:https://github.com/star7th/showdoc