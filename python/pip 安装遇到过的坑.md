## pip 安装遇到过的坑

1. 缺少 requirement
>	Could not find a version that satisfies the requirement **${package}** (from service-identity->Scrapy) (from versions: )
>	No matching distribution found for pyasn1-modules (from service-identity->Scrapy)
	
	解决方法：安装需要的包 `pip install ${package}`

2. 缺少 whl 驱动 
>	building **${package}** extension
>	error: Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools": http://landinghub.visualstudio.com/visual-cpp-build-tools

	解决方法：在网上下载 ${package} 对应你Python版本的 abc.whl 文件，然后安装 `pip install abc.whl`，然后再重新安装之前的那个