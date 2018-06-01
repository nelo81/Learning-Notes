# linux service

1. 新建一个文件 shell 并添加执行内容
```
#!/bin/bash # 必须的

CMD="java"
SERVICE_NAME="water-api" # 服务名
JAR_OPT="-jar /home/ubuntu/www/water/jar/water-api.jar"
PIDFILE="/home/ubuntu/www/water/pid/api.pid" # 存放进程 id 文件
JAVA_OPTS="-Xms32m -Xmx128m -XX:MaxMetaspaceSize=64m -XX:CompressedClassSpaceSize=8m -Xss256k -Xmn8m -XX:InitialCodeCacheSize=4m -XX:ReservedCodeCacheSize=8m -XX:MaxDirectMemorySize=16m"
EXT_OPTS="--spring.profiles.active=prod" # 额外参数
LOGFILE="/home/ubuntu/www/water/logs/api.log" # 输出重定向文件

# 检查当前进程 id 是否存在
checkpid(){
    if [ -f $PIDFILE ]; then
        PID=$(cat $PIDFILE)
        if [ ! -x /proc/${PID} ]; then
            return 1
        else
            return 0
        fi
    else
        return 1
    fi
}

# 启动服务
start(){
    checkpid
    RETVAL=$?
    if [ $RETVAL -eq 0 ]; then
        echo "$PIDFILE exists, process is already running or crashed"  
        exit 1
    fi

    echo "Starting $SERVICE_NAME ..."
    $CMD $JAVA_OPTS $JAR_OPT $EXT_OPTS > $LOGFILE &
    RETVAL=$?
    if [ $RETVAL -eq 0 ]; then
        echo "$SERVICE_NAME is started"  
        echo $! > $PIDFILE
        exit 0
    else
        echo "Stopping $SERVICE_NAME"  
        rm -f $PIDFILE
        exit 1
    fi
}

# 停止服务
stop(){
    checkpid
    RETVAL=$?
    if [ $RETVAL -eq 0 ]; then
        echo "Shutting down $SERVICE_NAME"  
        kill `cat $PIDFILE`
        RETVAL=$?
        if [ $RETVAL -eq 0 ]; then
            rm -f $PIDFILE
        else
            echo "Failed to stopping $SERVICE_NAME"  
        fi
    fi
}

restart(){
    $0 stop
    $0 start
}

help(){
    echo "Usage: $0 {start|stop|restart|status}"
}

status(){
    checkpid
    RETVAL=$?
    if [ $RETVAL -eq 0 ]; then
        PID=$(cat $PIDFILE)
        echo "$SERVICE_NAME is running ($PID)"  
    else
        echo "$SERVICE_NAME is not running"  
    fi
}

# 接受参数
case "$1" in
    start)
        start
    ;;
    stop)
        stop
    ;;
    restart)
        restart
    ;;
    status)
        status
    ;;
    *)
        help
    ;;
esac
           
```

2. 为脚本添加执行权限：`chmod +x shell` \
其实这里就已经可以直接执行，如 `./shell start`，但是还是不属于 linux 的服务

3. 在 /lib/systemd/system/ 下添加你的服务脚本 shell.service，例子如下
```
[Unit]  
Description=your_shell  # 你的服务名
After=syslog.target network.target remote-fs.target nss-lookup.target  

[Service]  
Type=forking  
ExecStart=/path/to/shell start # 2 中说的执行命令
ExecStop=/path/to/shell stop  # 停止命令
PrivateTmp=true

[Install]  
WantedBy=multi-user.target
```
- 除了有注释的几个参数，其他一般可以照搬

4. 之后就可以使用 systemctl 操作服务了
    - 启动服务：`systemctl start shell.service`
    - 停止服务：`systemctl stop shell.service`
    - 设置开机启动：`systemctl enable shell.service`
    - 取消开机启动：`systemctl disable shell.service`