## ubuntu mysql 单机多实例部署

- 安装 mysql：`apt-get install mysql-server`

- 建立需要的文件夹
'mysql' 文件夹可以放在你自己想放的目录下，我放在 '/home/kongin/' 中，这个名字也可以随便换
```
mkdir /home/kongin/mysql
mkdir /home/kongin/mysql/3306
mkdir /home/kongin/mysql/3306/data
mkdir /home/kongin/mysql/3306/binlog
mkdir /home/kongin/mysql/3306/relay_log

mkdir /home/kongin/mysql/3307
mkdir /home/kongin/mysql/3307/data
mkdir /home/kongin/mysql/3307/binlog
mkdir /home/kongin/mysql/3307/relay_log
```
查看 my.cnf 的位置
`whereis mysql`
一般在 '/etc/mysql/' 下
`cp /etc/mysql/my.cnf /home/kongin/mysql/multi.cnf`

- 修改文件夹权限
`chmod -R 777 /home/kongin/mysql`
`chmod 711 /home/kongin/mysql/multi.cnf`

- 修改 multi.cnf `vim multi.cnf`
加入以下配置
```
[mysqld_multi]
mysqld = /usr/bin/mysqld_safe
mysqladmin = /usr/bin/mysqladmin
log = /home/kongin/mysql/mysqld_multi.log

[mysqld1]
socket = /home/kongin/mysql/3306/mysql.sock
port = 3306
pid-file = /home/kongin/mysql/3306/mysqld.pid
datadir = /home/kongin/mysql/3306/data
log_bin = /home/kongin/mysql/3306/binlog
server-id = 1
relay_log = /home/kongin/mysql/3306/relay_log/mysql-relay-bin

[mysqld2]
socket = /home/kongin/mysql/3307/mysql.sock
port = 3307
pid-file = /home/kongin/mysql/3307/mysqld.pid
datadir = /home/kongin/mysql/3307/data
log_bin = /home/kongin/mysql/3307/binlog
server-id = 2
relay_log = /home/kongin/mysql/3307/relay_log/mysql-relay-bin
```

- 初始化数据库(--user kongin是linux的用户名)
'--initialize-insecure' 代表初始用户 'root@localhost' 密码为空
```
sudo mysqld --no-defaults --initialize-insecure --user=kongin --basedir=/usr/share/mysql --datadir=/home/kongin/mysql/3306/data

sudo mysqld --no-defaults --initialize-insecure --user=kongin --basedir=/usr/share/mysql --datadir=/home/kongin/mysql/3307/data
```
如果出现文件权限问题是因为这里datadir用的目录是不能被mysql使用的
那么需要修改 '/etc/apparmor.d/usr.sbin.mysqld'
`sudo vim /etc/apparmor.d/usr.sbin.mysqld`
在大括号中的最后添加以下目录配置（这里是我的目录）
```
/home/kongin/mysql/ r,
/home/kongin/mysql/** rw,
```
执行下面命令载入配置
`sudo /etc/init.d/apparmor reload`
这样就可以初始化了

- 运行 mysql_multi(start 后面的数字和上面配置中[mysqld1]的1相同，即[mysqld#]中的#也可以写1-2)
`sudo mysqld_multi --defaults-extra-file=/home/kongin/mysql/multi.cnf start 1,2`

- 查看运行情况
`sudo mysqld_multi --defaults-extra-file=/home/kongin/mysql/multi.cnf report`
成功的话可以看到
```
Reporting MySQL servers
MySQL server from group: mysqld1 is running
MySQL server from group: mysqld2 is running
```

- 连接mysql
```
mysql -uroot -p -h127.0.0.1 -P3306
mysql -uroot -p -h127.0.0.1 -P3307
```
密码为空，不用输入

- 注意：如果使用了3306端口，记住把原来开机启动的mysql服务关掉，不然会出现端口冲突
`sudo update-rc.d mysql disable` 关掉mysql服务开机启动