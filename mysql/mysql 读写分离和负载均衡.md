## mysql 读写分离和负载均衡

### 读写分离

1. 在两台或以上主机上安装同版本的mysql，里面的数据一样，又或者在同一台主机安装两个以上mysql实例（可以看'/mysql/mysql 单机多实例部署.md'），这里演示的只有两台。

2. 实现主从复制

	- 主机，或者mysql主实例配置
	```
	mysql> GRANT REPLICATION SLAVE ON *.* to 'rep1'@'你的从机ip' identified by 'password';
	mysql> show master status;
	+------------------+----------+--------------+------------------+
	| File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	+------------------+----------+--------------+------------------+
	| mysql-bin.000005 | 261 | | |
	+------------------+----------+--------------+------------------+
	```

	- 从机，或者mysql从实例配置
	```
	mysql> change master to
	master_host='192.168.10.130', #主机IP
	master_user='rep1', #主机刚新建的用户
	master_password='password', #主机刚新建的用户密码
	master_log_file='mysql-bin.000005', #这里是上面主机的数据
	master_log_pos=261; #这个也是上面主机的数据
	```

	- 启动从机同步
	`mysql> start slave;`

	- 检查是否在同步
	`mysql> show slave status\G`
	如果出现
	```
	Slave_IO_Running: YES
	Slave_SQL_Running: YES
	```
	则已经开始了同步

	- 接下来我们在主机做库级，表级，行级的操作都可以自动同步到从机上

3. java 实现读写分离

	- 写操作在主库，读操作在从库