## win10 设置右键菜单在当前目录启动程序

1. `win+r` 输入：`regedit` 打开注册表编辑器

2. 进入 HKEY_CLASSES_ROOT\Directory\Background\shell 目录

3. 新建项如图
![](./imgs/right-menu-1.png)
这里的项名称可以自定义

4. 新建子项如图
![](./imgs/right-menu-2.png)
注意：子项名称一定要是 "command"