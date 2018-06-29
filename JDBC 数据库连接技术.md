###### JDBC 数据库连接技术

```java
1.介绍
2.好处
3.JDBC 使用步骤 😀
4.JDBC 相关 API 😀
5.PreparedStatement 和 Statement 的区别 😀
6.封装 JDBCUtils 类
7.结果集元数据
8.大数据 Blob（图片）存取
9.批处理
10.事务
11.DBUtils 开源框架
12.数据库连接池 😀
13.封装 DAO 😀
```

###### JDBC

```java
理解：
	JDBC（Java Database Connectivity）java 和 数据库连接技术
	sun 公司推出一套用于 java 应用程序访问数据库的连接技术或规范
		规范：	
			接口或抽象类

作用：
	1.不用记多套 API,减轻了开发人员压力
	2.提高代码维护性和扩展性
	
使用步骤:😀
 	准备操作：
 		① 将驱动包复制到项目目录下
 		② build path
 		
 	开始步骤：	
		1.加载驱动：[将 myql 实现类加载到内存中]
			//方式1：不推荐
				DriverManager.registerDriver(new Driver());
					缺点：
						1.导致 Driver new 了两遍，效率较低
						2.属于静态加载，依赖性太强

			//方式2：推荐
				Class.forName("com.mysql.jdbc.Driver");
			
 		2.获取连接：[将 mysql 和应用程序搭上桥梁]
 			Connection connection = DriverManager.getConnection(
            					  "jdbc:mysql://localhost:3306/girls","root","root");
				
			url:jdbc:mysql://主机名:端口号/库名 😀
					服务器IP：127.0.0.1 / localhost， 端口号：3306 ，简化为：jdbc:mysql:///库名
			
			user：用户名
				
			password：密码

 		3.读写操作：[访问]
			//①获取执行sql语句命令对象
				Statement statement = connection.createStatement();
				或：
				PreparedStatement preparedStatement = connection.prepareStatement(sql);
		
			//②执行sql语句
			//方式1：Statement
				//增加：
					String sql = "insert into admin value(3,'donkin',1234)";
					int executeUpdate = statement.executeUpdate(sql);
			    // 删除：
					String sql = "delete from admin where id = 1";
					int executeUpdate = statement.executeUpdate(sql);
				//修改：
					String sql = "update admin set id = 1 where id = 3";
					int executeUpdate = statement.executeUpdate(sql);
				//查询：
					String sql  = "select * from admin;";
					ResultSet resultSet = statement.executeQuery(sql);

			//方式2：PreparedStatement
				//增加、删除、修改：
					String sql = "insert into 表名 value(?,?,?)";
					String sql = "delete from 表名 where id = ?";
					String sql = "update 表名 value set name = ? where id = ?";
					PreparedStatement preparedStatement = connection.prepareStatement(sql);
					//设置 sql 占位符的值：
						prepareStatement.setInt(parameterIndex, x);
						prepareStatement.setString(parameterIndex, x);
						prepareStatement.setObject(parameterIndex, x);
						
						//setXX(parameterIndex, x) ：没有char
						int update = preparedStatement.executeUpdate();

				//查询：
					resultSet = prepareStatement.executeQuery();
					
					while(resultSet.next()) {
						int id = resultSet.getInt(columnIndex);
						String name = resultSet.getString(columnIndex);
						String pwd = resultSet.getObject(columnIndex);
					}
					  //getXX(parameterIndex, x) ：没有char

 		4.关闭连接
 			resultSet.close();
			statement.close() / preparedStatement.close();
			connection.close()
            
		5.Properties 😀
			driver=com.mysql.jdbc.Driver
			url=jdbc:mysql://127.0.0.1:3306/girls
			username=root
			password=1234
```

###### API 😀

```java
1.DriverManager 类
	registerDriver():注册驱动
		public static void registerDriver(Driver driver) throws SQLException
			Registers the given driver with the DriverManager. 
    
	getConnection():获取连接
		public static Connection getConnection(String url,String user,String password) throws SQLException
			Attempts to establish a connection to the given database URL. 

2.Connection 接口
	createStatement()：获取命令对象，返回 Statement
		Statement createStatement() throws SQLException
			Creates a Statement object for sending SQL statements to the database.
                
	prepareStatement(sql)：获取预编译命令对象，返回 PreparedStatement
		PreparedStatement prepareStatement(String sql) throws SQLException
			Creates a PreparedStatement object for sending parameterized SQL statements to the database. 

	close()：关闭资源
		void close() throws SQLException
			Releases this Connection object's database and JDBC resources immediately instead of waiting for them to be automatically released. 
                
3.Statement 接口
	executeQuery(sql)：执行查询语句，返回 ResultSet 结果集对象
		ResultSet executeQuery(String sql) throws SQLException
			Executes the given SQL statement, which returns a single ResultSet object. 
                
	executeUpdate(sql)：执行增删改语句，返回 int 类型受影响行数
		int executeUpdate(String sql) throws SQLException
			Executes the given SQL statement, which may be an INSERT, UPDATE, or DELETE statement or an SQL statement that returns nothing, such as an SQL DDL statement.
                
	execute(sql) 执行任何语句，返回 boolean
		boolean execute(String sql) throws SQLException
			Executes the given SQL statement, which may return multiple results. 
   
	close()：关闭资源：
		void close() throws SQLException
			Releases this Statement object's database and JDBC resources immediately instead of waiting for this to happen when it is automatically closed.

4.PreparedStatement 接口
	executeQuery()：执行查询语句，返回 ResultSet 结果集对象
		ResultSet executeQuery() throws SQLException
			Executes the SQL query in this PreparedStatement object and returns the ResultSet object generated by the query.
                
	executeUpdate()：执行增删改语句，返回 int 类型受影响行数
		int executeUpdate() throws SQLException
			Executes the SQL statement in this PreparedStatement object, which must be an SQL Data Manipulation Language (DML) statement, such as INSERT, UPDATE or DELETE; or an SQL statement that returns nothing, such as a DDL statement.
                
	execute()：执行任何语句，返回 boolean
		boolean execute() throws SQLException
			Executes the SQL statement in this PreparedStatement object, which may be any kind of SQL statement. 
                
	setXX(占位符索引，参数值):设置索引处占位符的值，类型为 XX 类型
		void setInt(int parameterIndex,int x) throws SQLException
			Sets the designated parameter to the given Java int value. 
		...
                
	setObject(占位符索引，参数值):设置该索引处占位符的值，类型为 Object 类型
		void setObject(int parameterIndex,Object x) throws SQLException
			Sets the value of the designated parameter using the given object.
                
	close()：关闭资源：
		void close() throws SQLException
			Releases this Statement object's database and JDBC resources immediately instead of waiting for this to happen when it is automatically closed.
                
5.ResultSet 接口
	next():下移一行，返回下一行是否有值
		boolean next() throws SQLException
			Moves the cursor forward one row from its current position.
                
	getXX(int/String)：根据列索引或列名或别名获取该列值,返回类型 XX
		int getInt(int columnIndex) throws SQLException
			Retrieves the value of the designated column in the current row of this ResultSet object as an int in the Java programming language.
		...
                
	getObject(int/String)：根据列索引或列名或别名获取该列值,返回类型 Object
		Object getObject(int columnIndex) throws SQLException
			Gets the value of the designated column in the current row of this ResultSet object as an Object in the Java programming language. 

	close():关闭ResultSet资源
		void close() throws SQLException
			Releases this ResultSet object's database and JDBC resources immediately instead of waiting for this to happen when it is automatically closed.
                
PreparedStatement 与 Statement 对比：😀
	PreparedStatemen：
		1.不再使用 + 拼接 sql 语句，减少语法错误
		2.提高代码分离性和维护性
		3.避免了 sql 注入问题
		4.编译一遍，执行10遍，效率高
		
	Statement:
		1.无法避免 sql 注入问题
		2.编译十遍，执行10遍，效率低
```

###### JDBC 基础 😀

```java
	package com.atguigu.jdbc;
    
    import java.io.FileInputStream;
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.sql.Statement;
    import java.util.Properties;
    
    import com.mysql.jdbc.Driver;
    
    public class JDBCUtils {
    	public static void main(String[] args) throws Exception {
    		//初始化参数:
    		Properties pro = new Properties();
    		pro.load(new FileInputStream("src\\jdbc.properties"));
    		
    		String driver = pro.getProperty("driver");
    		String url = pro.getProperty("url");
    		String user = pro.getProperty("user");
    		String password = pro.getProperty("password");
    		
    		//1.加载驱动：
    			//方式 1：
    				DriverManager.registerDriver(new Driver());	
    			//方式2：
    				Class.forName(driver);
    		
    		//2.创建链接：
    				Connection connection = DriverManager.getConnection(url, user, password);
    		
    		//3.获取对象：
            		//方式1：Statement:
    				Statement statement = connection.createStatement();
            		//方式2：PreparedStatement：可有效防止sql注入
            		😀PreparedStatement prepareStatement = connection.prepareStatement(sql);
    		
    		//4.执行语句：
    			//4.1 查询语句：
    				String sql  = "select * from admin;";
    				ResultSet resultSet = statement.executeQuery(sql);
    			
    				while(resultSet.next()) {
    					System.out.println(resultSet.getInt(1));
    				}
    				
    			//4.2更新语句：
    				//增加：
    				String sql = "insert into admin value(3,'donkin',1234)";
    				int executeUpdate = statement.executeUpdate(sql);
    			    // 删除：
    				String sql = "delete from admin where id = 1";
    				int executeUpdate = statement.executeUpdate(sql);
    				//修改：
    				String sql = "update admin set id = 1 where id = 3";
    				int executeUpdate = statement.executeUpdate(sql);
    		
    		//关闭连接：
    				CloseUtils.close(null, statement, connection);
    	}
    }

CloseUtils

    package com.atguigu.jdbc;
    
    import java.sql.Connection;
    import java.sql.SQLException;
    import java.sql.Statement;
    
    public class CloseUtils {
    	//关闭连接方法：
    	public static void close(ResulSet resultSet,Statement statement,Connection connection) {
    		//关闭resultSet:
            if(resulSet != null){
                try {
                    resultSet.close();
                } catch {
                    e.printStackTrace();
                }
            }
            
    		//关闭statement：
    		if(statement != null) {
    			try {
    				statement.close();
    			} catch (SQLException e) {
    				e.printStackTrace();
    			}
    		}
    		
    		//关闭连接：
    		if(connection != null) {
    			try {
    				connection.close();
    			} catch (SQLException e) {
    				e.printStackTrace();
    			}
    		}
    	}
    }

jdbc.properties

    driver=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306/girls
    user=root
    password=1234

```

###### 结果集元数据 😀

```java
package com.atguigu.dbutils;

import java.lang.reflect.Field;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;

import com.atguigu.java.beans.Account;

public class ResultSetMetaDate {
	public static void main(String[] args) throws Exception {
		😀String sql = "select id,name account,money salary from account where id =  ?";
		
		Account account = getObject(Account.class, sql, 1);
		System.out.println(account);
	}
	
	public static <T> T getObject(Class<T> clazz,String sql,Object...objects) throws Exception {
				//获取数据库连接：
				Connection connection = JDBCUtils.getConnectionByC3p0();
				
				//获取执行命令对象：
				PreparedStatement preparedStatement = connection.prepareStatement(sql);
				preparedStatement.setInt(1, 1);
				
				//获取结果集：
				ResultSet resultSet = preparedStatement.executeQuery();
				
				//获取结果集元数据：
				ResultSetMetaData resultSetMetaData = resultSet.getMetaData();
				
				//获取结果集列数：
				int count = resultSetMetaData.getColumnCount();
				
				//创建对象：
				T t = clazz.newInstance();
				
				//循环遍历每一行：
				while(resultSet.next()) {
					//循环获取每一行：
					for (int i = 1; i <= count; i++) {
						//获取列名：
						String name = resultSetMetaData.getColumnLabel(i);

						//获取类属性名：
						Field field = clazz.getDeclaredField(name);

						//将属性爆破：
						field.setAccessible(true);

						//设置属性值：
						field.set(t, resultSet.getObject(i));
					}
				}
				
				//关闭连接：
				JDBCUtils.close(resultSet, preparedStatement, connection);
		return t;
	}
}	
```

###### 大数据 Blob (图片) 存储 😀

```java
package com.atguigu.blob;

import java.io.File;
import java.io.FileInputStream;
import java.sql.Connection;
import java.sql.PreparedStatement;

public class Blob {
	public static void main(String[] args) {
		Connection connection = null;
		PreparedStatement preparedStatement =null;
		
		String sql = "update customers set photo = ? where id = ?";
		
		try {
			//获取链接：
			connection = JDBCUtils.getConnection();
			//获取命令对象：
			preparedStatement = connection.prepareStatement(sql);
			
			//设置占位符值：
			preparedStatement.setBlob(1, new FileInputStream(new File("D:\\DonkinFiles\\Images\\java.png")));
			preparedStatement.setInt(2, 1);
			
			//执行命令：
			int executeUpdate = preparedStatement.executeUpdate();
			System.out.println(executeUpdate > 0 ? "执行成功！" : "执行失败！");
			
		} catch (Exception e) {
			throw new RuntimeException(e);
		} finally {
			JDBCUtils.close(null, preparedStatement, connection);
		}
	}
}
```

###### 大数据 Blob (图片) 读取 😀

```java
package com.atguigu.blob;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class BlobReadPhoto {
	public static void main(String[] args) {
		Connection connection = null;
		PreparedStatement preparedStatement =null;
		
		String sql = "select photo from customers where id = ?";
		
		try {
			//获取链接：
			connection = JDBCUtils.getConnection();
			//获取命令对象：
			preparedStatement = connection.prepareStatement(sql);
			
			//设置占位符值：
			preparedStatement.setInt(1, 16);
			
			//执行命令：
			ResultSet resultSet = preparedStatement.executeQuery();
			
			//获取结果集：
			if(resultSet.next()) {
				InputStream inputStream = resultSet.getBinaryStream(1);
				OutputStream outputStream = new FileOutputStream(new File("D:\\DonkinFiles\\Images\\girls.png"));
				
				byte[] buffer = new byte[1024];
				int len;
				while((len = inputStream.read(buffer)) != -1) {
					outputStream.write(buffer, 0, len);
				}
				
				System.out.println("读取完成！");
				outputStream.close();
				inputStream.close();
			}
			
		} catch (Exception e) {
			throw new RuntimeException(e);
		} finally {
			JDBCUtils.close(null, preparedStatement, connection);
		}
	}
}
```

###### 批处理 😀

```java
package com.atguigu.batch;

import java.sql.Connection;
import java.sql.PreparedStatement;

public class Batching {
	public static void main(String[] args) throws Exception {
		//获取链接：
		Connection connection = JDBCUtils.getConnection();
		
		String sql = "insert into admin values(null,?,?)";
		//获取执行命令对象：
		PreparedStatement preparedStatement = connection.prepareStatement(sql);
		
		for (int i = 1; i<= 10000; i++) {
			//设置占位符值：
			preparedStatement.setString(1, "donkin" + i);
			preparedStatement.setString(2, "0000" + i );
			
			//添加sql语句到缓存包 ：
			preparedStatement.addBatch();
			
			if(i % 1000 == 0) {
				//执行缓存包语句：
				preparedStatement.executeBatch();
				//清空缓存：
				preparedStatement.clearBatch();
			}
		}
		
		//关闭连接：
		JDBCUtils.close(null, preparedStatement, connection);
	}
}
```

###### 事务处理 😀

```java
package com.atguigu.transaction;

import java.sql.Connection;
import java.sql.SQLException;

public class Transaction {
	public static void main(String[] args) {
		Connection connection = null;
		String sql = "update account set money = ? where name  = ?";
		
		try {
				//获取连接：
				connection = JDBCUtils.getConnection();
				
				//开启事务：
				connection.setAutoCommit(false);
				
				//更新数据：
				DBUtils.update(connection, sql, 1500,"donkin");
				
				//运行异常：
				int i = 10 / 0;
				
				//更新数据：
				DBUtils.update(connection, sql, 2500,"pxy");
				
				//提交事物：
				connection.commit();
				
				System.out.println("执行成功！");
            
		} catch (Exception e) {
			
			//如果有异常回滚：
			try {
				connection.rollback();
			} catch (SQLException e1) {
				e1.printStackTrace();
			}
            
		} finally {
			//关闭连接：
			JDBCUtils.close(null, null, connection);
		}
	}
}
```

###### C3P0 数据库连接池 😀

```java
	package com.atguigu.blob;
    
    import java.sql.Connection;
    
    import com.mchange.v2.c3p0.ComboPooledDataSource;
    
    public class C3P0 {
    	public static void main(String[] args) throws Exception {
    		//创建数据库连接池对象：😀
    		ComboPooledDataSource c3p0 = new ComboPooledDataSource("jdbc");
    		
    		//获取数据库连接：
    		Connection connection = c3p0.getConnection();
    		
    		//执行语句:
    		System.out.println(connection);
    		
    		//释放资源到数据库连接池：
    		
    		connection.close();
    	}
    }

src\\c3p0-config.xml

    <c3p0-config>
      <named-config name="jdbc"> 
      
      <!-- 驱动类 -->
      <property name="driverClass">com.mysql.jdbc.Driver</property>
      
      <!-- url-->
      	<property name="jdbcUrl">jdbc:mysql://127.0.0.1:3306/girls</property>

      <!-- 用户名 -->
      <property name="user">root</property>
      
      <!-- 密码 -->
      <property name="password">1234</property>
      
      <!-- 每次增长的连接数-->
      <property name="acquireIncrement">5</property>
      
      <!-- 初始的连接数 -->
      <property name="initialPoolSize">10</property>
        
      <!-- 最小连接数 -->
      <property name="minPoolSize">5</property>
        
      <!-- 最大连接数 -->
      <property name="maxPoolSize">10</property>
    
	  <!-- 可连接的最多的命令对象数 -->
	  <property name="maxStatements">5</property> 
        
	  <!-- 每个连接对象可连接的最多的命令对象数 -->
	  <property name="maxStatementsPerConnection">2</property>
      </named-config>
    </c3p0-config>

jar 包

    c3p0-0.9.1.2.jar
```

###### DBCP 数据库连接池 😀

```java
	package com.atguigu.blob;
    
    import java.io.FileInputStream;
    import java.sql.Connection;
    import java.util.Properties;
    
    import javax.sql.DataSource;
    
    import org.apache.commons.dbcp.BasicDataSourceFactory;
    
    public class DBCP {
    	public static void main(String[] args) throws Exception {
    		Properties pro = new Properties();
    		pro.load(new FileInputStream("src\\dbcp.properties"));
    		
    		//创建数据库连接池：😀
    		DataSource dbcp = BasicDataSourceFactory.createDataSource(pro);
    		
    		//获取连接：
    		Connection connection = dbcp.getConnection();
    		
    		//执行语句：
    		System.out.println(connection);
    		
    		//释放资源：
    		connection.close();
    	}
    }
    

dbcp.properties：😀

    driverClassName=com.mysql.jdbc.Driver
    url=jdbc:mysql://127.0.0.1:3306/girls
    username=root
    password=1234
    initialSize=10
    maxActive=10

jar 包
    1.commons-dbcp-1.4.jar
    2.commons-pool-1.5.5.jar
```

###### JDBCUtils 😀

```java
1.加入JAR包：😀
	① dbcp数据库连接池 jar 包：
		1.commons-dbcp-1.4.jar
		2.commons-pool-1.5.5.jar
		
	② c3p0数据库连接池 jar 包：
		c3p0-0.9.1.2.jar

2.Build Path 😀

3.添加配置文件：😀
	dbcp.properties配置文件：
		/*
		
		driverClassName=com.mysql.jdbc.Driver
		url=jdbc:mysql://127.0.0.1:3306/girls
		username=root
		password=1234
		initialSize=10
		maxActive=10
		
		*/
	
	c3p0-config.xml配置文件：😀
		/*
		
			<c3p0-config>
  			<named-config name="jdbc"> 
			<!-- 驱动类 -->
  			<property name="driverClass">com.mysql.jdbc.Driver</property>
  			<!-- url-->
  			<property name="jdbcUrl">jdbc:mysql://127.0.0.1:3306/girls</property>
  			<!-- 用户名 -->
  			<property name="user">root</property>
  			<!-- 密码 -->
  			<property name="password">1234</property>
  			<!-- 每次增长的连接数-->
    		<property name="acquireIncrement">5</property>
    		<!-- 初始的连接数 -->
  
    		<property name="initialPoolSize">10</property>
    
   		 	<!-- 最小连接数 -->
    		<property name="minPoolSize">5</property>
    
   			<!-- 最大连接数 -->
    		<property name="maxPoolSize">10</property>

			<!-- 可连接的最多的命令对象数 -->
    		<property name="maxStatements">5</property> 
    
    		<!-- 每个连接对象可连接的最多的命令对象数 -->
   			<property name="maxStatementsPerConnection">2</property>
 		 	</named-config>
			</c3p0-config>
			
		*/
		
4.创建数据库连接池：😀
	① 获取数据库连接：
	② 关闭数据库连接：

package com.atguigu.dbutils;

import java.io.File;
import java.io.FileInputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

import javax.sql.DataSource;

import org.apache.commons.dbcp.BasicDataSourceFactory;

import com.mchange.v2.c3p0.ComboPooledDataSource;

/**
 * 功能：获取数据库连接和释放连接：
 * @author Administrator
 *
 */
public class JDBCUtils {
	
	static ComboPooledDataSource c3p0 = null;
	static DataSource dbcp = null;
	
	//初始化时获取数据库连接池对象：
	static {
		//创建c3p0数据库连接池：
		c3p0 = new ComboPooledDataSource("jdbc");
		
		//创建dbcp数据库连接池：
		Properties pro = new Properties();
		try {
			pro.load(new FileInputStream(new File("src\\dbcp.properties")));
			dbcp = BasicDataSourceFactory.createDataSource(pro);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	/**
	 * 功能：c3p0方式：获取数据库连接池中的连接：
	 * @return 返回数据库连接：
	 * @throws Exception
	 */
	public static Connection getConnectionByC3p0() throws Exception {
		return c3p0.getConnection();
	}
	
	/**
	 * 功能：dbcp方式：获取数据库连接：
	 * @return
	 * @throws Exception
	 */
	public static Connection getConnectionByDbcp() throws Exception {
		return dbcp.getConnection();
	}
	
	/**
	 * 功能：关闭数据库连接
	 * @param resultSet 待关闭结果集
	 * @param statement 待关闭执行命令对象
	 * @param connection 带关闭数据库连接
	 */
	public static void close(ResultSet resultSet,Statement statement,Connection connection) {
		//关闭结果集资源：
		if(resultSet != null) {
			try {
				resultSet.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		
		//关闭执行命令对象：
		if(statement != null) {
			try {
				statement.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		
		//关闭数据库连接：
		if(connection != null) {
			try {
				connection.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
}
```

###### DAO😀

```java
开源框架 DButils 中的类和方法实现通用增删改查：
	QueryRunner：查询器
		1.update(connection,sql,objs):
			通用的增删改
		2.query(connection,sql,ResultSetHandler,objs):
			通用的查询，根据结果集处理器返回不同类型对象

	ResultSetHandler：结果集处理器
		1.BeanHandler：
			将结果集第一行转换成Bean对象并返回
			
		2.BeanListHandler:
			将结果集所有行转换成 List<Bean> 集合并返回
			
		3.ScalarHandler:
			将结果集第一行第一列的值以 Object 返回
			
		4.MapListHandler：
			将结果集所有行转换成 List<Map> 并返回

案例：😀
package com.atguigu.dbutils;

import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.sql.Connection;
import java.util.List;

import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.apache.commons.dbutils.handlers.ScalarHandler;
/**
 * 功能：通用的数据库增删改查类
 * @author Administrator
 */
@SuppressWarnings("unchecked")
public class DAO <T>{
	//获取执行命令对象：
	QueryRunner qr =  new QueryRunner();

	//获取泛型类型：
	Class<T> clazz =null;
	
	//为clazz赋值:
	{
		//通过反射获取父类泛型
			//获取父类类型+T
				Type type = this.getClass().getGenericSuperclass();
			//向下转型
				ParameterizedType pt = (ParameterizedType) type;
			//获取泛型：
				clazz = (Class<T>) pt.getActualTypeArguments()[0];
	}
	
	/**
	 * 功能：增删改功能，返回受影响行数
	 * @param sql 增删改语句
	 * @param objects 可变参数
	 * @return 
	 */
	public  int update(String sql,Object...objects) {
		Connection connection = null;
		
		try {
			//获取数据库连接：
			connection = JDBCUtils.getConnectionByC3p0();
			
			int update = qr.update(connection, sql, objects);
			
			return update;
		} catch (Exception e) {
			//将编译异常转换为运行异常：
			throw new RuntimeException(e);
		} finally {
			//释放数据库连接：
			JDBCUtils.close(null, null, connection);
		}
	}
	
	/**
	 * 功能：放回结果第一行第一列
	 * @param sql 查询语句
	 * @param objects 可变参数
	 * @return 
	 */
	public Object scalar(String sql,Object...objects){
		Connection connection = null;
        
		try {
			//获取数据库连接：
			connection = JDBCUtils.getConnectionByC3p0();
			
			Object object = qr.query(connection, sql,new ScalarHandler(),objects);
			
			return object;
		} catch (Exception e) {
			//将编译异常转换为运行异常：
			throw new RuntimeException(e);
		} finally {
			//关闭连接：
			JDBCUtils.close(null, null, connection);
		}
	}
	
	/**
	 * 功能：返回结果集多行多列数据
	 * @param clazz 
	 * @param sql 查询语句
	 * @param objects 可变参数
	 * @return
	 */
	public List<T> queryMultiple(String sql,Object...objects) {
		Connection connection = null;
		try {
			//获取数据库连接：
			connection = JDBCUtils.getConnectionByC3p0();
			
			List<T> list = qr.query(connection, sql, new BeanListHandler<T>(clazz), objects);
			
			return list;
		} catch (Exception e) {
			//将编译异常转换为运行异常：
			throw new RuntimeException(e);
		} finally {
			JDBCUtils.close(null, null, connection);
		}
	}
	
	/**
	 * 功能：返回 结果集第一行数据
	 * @param clazz
	 * @param sql 查询语句
	 * @param objects 可变参数
	 * @return
	 */
	public T querySingle(String sql,Object ...objects) {

		Connection connection = null;
		
		try {
			//获取数据库连接：
			connection = JDBCUtils.getConnectionByC3p0();
			
			T t = qr.query(connection, sql, new BeanHandler<T>(clazz), objects);
			
			return t;
		} catch (Exception e) {
			//转换编译异常为运行异常：
			throw new RuntimeException(e);
			
		} finally {
			//释放数据库连接资源：
			JDBCUtils.close(null, null, connection);
		}
	}
}
```





​		