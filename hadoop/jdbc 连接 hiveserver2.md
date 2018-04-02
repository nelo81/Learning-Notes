## jdbc 连接 hiveserver2

1. 安装 hive 2.0 以上版本

2. 运行（最后加上'&'说明在后台运行）
```
	hive --service metastore &
	hive --service hiveserver2 &
```

3. `tail -f /tmp/{user}/hive.log'` 观察 hiveserver 有没有正常运行
	也可以通过下述命令查看hiveserver2是否已经开启
	`netstat -nl |grep 10000`
	若已经开启，结果为：
	`tcp        0      0 （你的IP）:10000        0.0.0.0:*                   LISTEN`

4. 修改hadoop 配置文件 etc/hadoop/core-site.xml,加入如下配置项
```
# root代表服务器用户
<property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
</property>
```
否则之后连接可能会出现一下异常：
```
Failed to open new session: java.lang.RuntimeException: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.authorize.AuthorizationException): User:xiaosiis not allowed to impersonate anonymous
```

5. 使用 jdbc 连接 hiveserver2

- pom.xml
```
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.3.0</version>
</dependency>

<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.4</version>
</dependency>
```

- HiveClient.java
```
package hive.hive_client;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.log4j.Logger;

public class HiveClient {
	private static Logger log = Logger.getLogger(HiveClient.class);
	private static String driver = "org.apache.hive.jdbc.HiveDriver";
	private static String url = "jdbc:hive2://192.168.0.122:10000/default";//ip换成自己服务器ip
	private static String username = "kongin";//换成上一步hadoop上设置的用户名
	private static String password = "";
	
	private static Connection defaultConnection = null;
	
	public static Connection getDefaultConnection() {
		if (defaultConnection == null) defaultConnection = getConnection();
		return defaultConnection;
	}
	
	public static Connection getConnection() {
		Connection con = null;
		try {
			Class.forName(driver);
			con = DriverManager.getConnection(url,username,password);
			log.debug("connected success");
		}
		catch(Exception e) {
			e.printStackTrace();
		}
		return con;
	}
	
	public static void release(Connection conn) {
		if(conn != null) {
			try {
				conn.close();
			}
			catch(SQLException e) {
				e.printStackTrace();
			}
			finally {
				conn = null;
			}
		}
		log.debug("connection released successfully");
	}
	
	public static void closeStatement(Statement stmt, ResultSet rs) {
		if(stmt != null) {
			try {
				stmt.close();
			}
			catch(SQLException e) {
				e.printStackTrace();
			}
			finally {
				stmt = null;
			}
		}
		if(rs != null) {
			try {
				rs.close();
			}
			catch(SQLException e) {
				e.printStackTrace();
			}
			finally {
				rs = null;
			}
		}
		log.debug("statement closed successfully");
	}
	
	public static List<Map<String, Object>> query(Connection conn, String sql) {
		Statement stmt = null;
		ResultSet rs = null;
		List<Map<String, Object>> list = new ArrayList<Map<String,Object>>();
		try {
			stmt = conn.createStatement();
			rs = stmt.executeQuery(sql);
			ResultSetMetaData md = rs.getMetaData();
			int col = md.getColumnCount();
			while(rs.next()) {
				Map<String,Object> rowData = new HashMap<String,Object>();  
	            for (int i = 1; i <= col; i++) {  
	                rowData.put(md.getColumnName(i), rs.getObject(i));  
	            }  
	            list.add(rowData);
			}
		}
		catch(SQLException e) {
			e.printStackTrace();
		}
		finally {
			closeStatement(stmt, rs);
		}
		return list;
	}
	
	public static int update(Connection conn, String sql) {
		Statement stmt = null;
		int rs = 0;
		try {
			stmt = conn.createStatement();
			rs = stmt.executeUpdate(sql);
		}
		catch(SQLException e) {
			e.printStackTrace();
		}
		finally {
			closeStatement(stmt, null);
		}
		return rs;
	}

	public static void main(String[] args) throws Exception {
        Connection conn = HiveClient.getDefaultConnection();
        conn.close();
    }
}
```