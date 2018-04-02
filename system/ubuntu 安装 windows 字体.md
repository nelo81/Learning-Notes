## ubuntu 安装 windows 字体

1. 从 windows 上传字体文件 (.ttc) 到 ubuntu 系统上，一般在 "C:\Windows\Fonts" 里面

2. ubuntu 创建文件夹 `sudo mkdir /usr/share/fonts/winfonts`，然后把字体文件复制进去

3. 修改权限
```
sudo chmod -R 644 /usr/share/fonts/winfonts/
sudo chmod 755 /usr/share/fonts/winfonts
```

4. 缓存字体 `sudo fc-cache -fv`

- 如果要删除字体，就把 'winfonts' 目录删掉，然后再执行 `sudo fc-cache -fv` 就好了