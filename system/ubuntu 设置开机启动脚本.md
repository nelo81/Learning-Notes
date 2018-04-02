## ubuntu 设置开机启动脚本

1. 把脚本 "my_shell" 复制到 "/etc/init.d"

2. 修改脚本权限 `sudo chmod 775 my_shell`

3. 在 "/etc/init.d" 目录下执行 `sudo update-rc.d my_shell defaults 90`，defaults 后面的数是启动顺序

4. 如果出现 `missing LSB tags and overrides` 错误，则要在脚本开头添加 lsb 信息
```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          my_shell
# Required-start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the my_shell daemon
# Description:       starts my_shell using start-stop-daemon
### END INIT INFO
```
- Provides 此脚本文件的名称
- Short-Description 和 Description 是描述
- 其他选项基本一样

4. 取消开机启动脚本 `sudo update-rc.d my_shell remove`