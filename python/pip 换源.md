## pip 换源

- linux 下：修改 "~/.pip/pip.conf"
```
[global]  
index-url = https://pypi.doubanio.com/simple/  
[install]  
trusted-host=pypi.doubanio.com  
disable-pip-version-check = true  
timeout = 6000  
```

- windows 下：修改 "c:\user\${username}\pip\pip.ini" (原来可能会没有这个目录，要自己创建)
```
[global]  
index-url=http://mirrors.aliyun.com/pypi/simple
[install]  
trusted-host=mirrors.aliyun.com  
disable-pip-version-check = true  
timeout = 6000
```

- index-url 是下载源，可以自由更换
- trusted-host 是指信任的域名，不写这个的话，不能访问 http (非 https) 的源