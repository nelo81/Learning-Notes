## maven 生命周期介绍

一、生命周期分为三大部分

1. Clean Lifecycle 在进行真正的构建之前进行一些清理工作
  - pre-clean  执行一些需要在clean之前完成的工作
  - clean  移除所有上一次构建生成的文件
  - post-clean  执行一些需要在clean之后立刻完成的工作

2. Default Lifecycle 构建的核心部分，编译，测试，打包，部署等等
  - validate     检测代码合法性
  - generate-sources
  - process-sources
  - generate-resources
  - process-resources     复制并处理资源文件，至目标目录，准备打包。
  - compile     编译项目的源代码。
  - process-classes
  - generate-test-sources 
  - process-test-sources 
  - generate-test-resources
  - process-test-resources     复制并处理资源文件，至目标测试目录。
  - test-compile     编译测试源代码。
  - process-test-classes
  - test     使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
  - prepare-package
  - package     接受编译好的代码，打包成可发布的格式，如 JAR 。
  - pre-integration-test
  - integration-test
  - post-integration-test
  - verify
  - install     将包安装至本地仓库，以让其它项目依赖。
  - deploy     将最终的包复制到远程的仓库，以让其它开发人员与项目共享。


3. Site Lifecycle 生成项目报告，站点，发布站点
  - pre-site     执行一些需要在生成站点文档之前完成的工作
  - site    生成项目的站点文档
  - post-site     执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
  - site-deploy     将生成的站点文档部署到特定的服务器上

二、生命周期和 maven 命令的关系

1. 直接执行 `mvn clean` 即可完成 Clean Lifecycle

2. 直接执行 `mvn install` 即可完成 Default Lifecycle
  - 一般不安装到仓库的话执行 `mvn package`
  - 可以在后面添加参数 `-Dmaven.test.skip=true` 跳过单元测试和 `-Dmaven.javadoc.skip=true` 跳过 javadoc 生成

3. 直接执行 `mvn site` 即可完成 Site Lifecycle

4. 注意：以上执行的命令都要在 maven 根目录下执行，即存在 "pom.xml" 的目录下