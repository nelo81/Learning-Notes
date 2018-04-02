## Google Chrome 插件安装

方法一：从扩展应用商店中下载

方法二：将 .crx 文件拉到浏览器扩展应用页面，在弹出框中按安装即可

##### 将扩展应用商店下载的应用打包成本地的 .crx 文件的方法

1. 浏览器访问`chrome://version`，获取"个人资料路径"找到本地对应目录(chrome_path)，进入"chrome_path/Extensions"目录

2. 浏览器访问`chrome://extensions`，勾上"开发者模式"，复制"chrome_path/Extensions"中名称和应用ID相同的文件夹(extendir)出来

3. 按"打包扩展程序..."，"扩展程序根目录"输入 extendir 目录下的其中一个版本的文件夹的目录，"私有密钥文件"可以不填，直接打包即可，之后会在 extendir 目录下出现一个新的 .crx 文件和 .pem 公钥文件，如果之前没有输入私钥，则可以删除 .pem 文件