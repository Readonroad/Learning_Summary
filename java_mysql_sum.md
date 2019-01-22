MySQL
win7  64bit + MySQL 5.7
### MySQL服务安装
* 配置初始化文件mysql.ini
```
#注：windows下需要用\\s，否则转义失败
#安装路径
basedir = D:\\Program Files\mysql\mysql-5.7.23-winx64\mysql-5.7.23-winx64
#生成data文件下
datadir = D:\\Program Files\mysql\mysql-5.7.23-winx64\mysql-5.7.23-winx64\data
#服务端和客户端编码,通过查看数据库编码命令可以查询数据库编码
[mysqld]
character-set-server = utf8
[client]
default-character-set = utf8
```
* 下载mysql服务
* 进入mysql服务所在文件夹的bin文件夹
```
初始化：mysqld --initialize (初始化数据库，结果保存在data文件夹下，在.err文件中有生成的初始随机密码)
或 mysqld --initialize --console(初始化数据库，结果显示在界面上)
安装： mysqld -install
启动： net start mysql
关闭： net stop mysql
删除: mysqld -remove
```
注：配置文件中，[mysql]和[client]标识客户端相关配置，[mysqld]表示服务端相关配置。
* 修改mysql密码
```
登录：mysql -u root -p [初始密码]
修改密码：alter user 'root'@'localhost' identified by '新密码'；
```
* 查看数据库编码
```
show variables like 'char%';  //%表示匹配
```
### java连接mysql数据库
通过IntelliJ Idea连接mysql数据库。
#### IntelliJ Idea导入jar包
Java项目中需要连接mysql数据库,首先要导入mysql-connector的jar包，方法为：File-->Project Structure --> Modules --> Dependencies,点击+号，选择JARS or directions, 添加mysql-connector-java.jar, 选中，点击ok。

如果没有导入jar程序驱动包，或者其他与数据库建联的方式，则出现在无法找到驱动类：java.lang.ClassNotFoundException: com.mysql.jdbc.Driver
注：后面如果需要导入其他jar包，方法类似。

#### java.sql.SQLException:The server time zone value问题
异常：Caused by: com.mysql.cj.exceptions.InvalidConnectionAttributeException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.

分析：新版数据库使用的时区与本地时区有区别，标准时区使用的是Unix元年的时间为起始点到当前时间中间所做的动作。国际标准实际与本地相差8个小时。

解决方法：

* 修改数据库变量实现
```
#查看数据库本地时区
show variables like '%time_zone%';

#修改本地时区，与标准时区保持一致
set global time_zone='+8:00';
```
* 连接mysql服务时，直接指定时区为UTC
```
#更改连接，在连接mysql服务时，直接指定时区为UTC
String url = "jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC";
注：UTC需要大写，否则无效。
```
#### java连接mysql小测试
1.首先本地开启mysql服务，登录服务，创建数据库my_db,并在该数据库中创建数据表 test_t

2.java测试程序sqlTest.class连接mysql,从test_t中读取数据
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class sqlTest
{
    public static void main(String[] args)
    {
        Connection con;
        //jdbc驱动程序名称，告诉JVM使用的是哪一个数据库驱动
        String driver = "com.mysql.cj.jdbc.Driver";
        //URL指向访问的数据库名my_db
        String url = "jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC";
        //登录mysql时的用户名和密码
        String user = "root";
        String password = "422504";

        try
        {
            //1. 解析获取数据库驱动
            Class.forName(driver);
            //2.使用Jdbc中的类，获取与mysql的连接
            con = DriverManager.getConnection(url, user, password);
            if(!con.isClosed())
            {
                System.out.println("Succeeded connecting to the database.");
            }
            //3.创建一个statement对象，将sql语句发送到SQL
            Statement statement = con.createStatement();
            String sql = "select * from test_t";
            //4. 执行sql语句，ResultSet类，用来存放获取的结果集
            ResultSet rs = statement.executeQuery(sql);

            String name =null;
            int id = 0;
            //5.遍历结果集，打印数据
            while(rs.next())
            {
                name = rs.getString("name");
                id = rs.getInt("id");
                System.out.println(id +"\t" + name);
            }
            rs.close();
            con.close(); //关闭连接

        }catch (ClassNotFoundException e)
        {
            System.out.println("Cannot find the Driver");
        }
        catch (SQLException e)
        {
            e.printStackTrace();
           // System.out.println("Database closed");
        }
        finally
        {
            System.out.println("finished");
        }
    }
}
```
3.java与数据库MySQL相连总结

JDBC是java实现数据库访问的应用程序编程接口，主要功能是管理存放在数据库中的数据。通过接口对象，应用程序可以实现与数据库的连接，执行SQL语句，从数据库中获取结果，获取状态以及错误信息，终止事务与连接等。
```
第一步：导入对应数据库jdbc的jar包，加入工程项目中
第二步：编写代码
* 装载类对应的数据库的驱动器类：Class.forName()
* 获取到数据库对象：DriverManager.getConnection(String url,String user, String password)
DriverManager 类，管理一组JDBC驱动程序基本服务
url:路径地址，格式为：网络协议://IP地址:数据库端口/要查询的数据库名
* 包装sql语句
* 执行sql语句，获得结果
* 打印数据
```
### SQL注入攻击
发生原因：DAO(DataAccessObeject)数据操作对象中执行的SQL语句是拼接出来的，其中一部分内容是由用户从客户端传入，当用户传入的数据中包含sql关键字时，就有可能通过这些关键字改变sql语句的语义，从而执行一些特殊的操作，这种攻击方式就叫做sql注入攻击。（通过操作输入来修改后台SQL语句达到代码执行进行攻击目的的技术）
#### sql注入威胁
1.猜解后台数据库，盗取网站敏感信息（通过多次查询尝试，猜测数据库信息，获取数据库用户，版本，密码等敏感信息）
```sql
#查询语句：
SELECT first_name, last_name FROM users WHERE user_id = '$id'; //id为输入参数
#准确输入参数，可以得到正确的sql语句
SELECT first_name, last_name FROM users WHERE user_id = '1';
#恶意构造参数：输入id = 1’ order by 1#
SELECT first_name, last_name FROM users WHERE user_id = '1' order by 1#';
# 说明(sql语法，#后面的会被注释掉，这种构造参数方法可以屏蔽掉后面的单引号，避免语法错误)
```
2.绕过认证，登录网站后台
```sql
#登录数据库
select * from users where username='123' and password='123';
#恶意构造数据，绕过认证，输入：123' or 1=1# 和 123' or 1=1#
select * from users where username='123' or 1=1 #' and password='123' or 1=1 #'
#实际执行语句为
select * from users where username='123' or 1=1 //添加恒为true,执行成功
```
java程序示例--SQL注入风险
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class sqlTest
{
    public static void main(String[] args)
    {
        Connection con;
        //jdbc驱动程序名称，告诉JVM使用的是哪一个数据库驱动
        String driver = "com.mysql.cj.jdbc.Driver";
        //URL指向访问的数据库名my_db
        String url = "jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC";
        //登录mysql时的用户名和密码
        String user = "root";
        String password = "422504";

        try
        {
            //1. 解析获取数据库驱动
            Class.forName(driver);
            //2.使用Jdbc中的类，获取与mysql的连接
            con = DriverManager.getConnection(url, user, password);
            if(!con.isClosed())
            {
                System.out.println("Succeeded connecting to the database.");
            }
            //3.创建一个statement对象，将sql语句发送到SQL
            Statement statement = con.createStatement();
            //从键盘输入字符串
            Scanner sc = new Scanner();
            int id = sc.nextLine();
            String name = sc.nextLine();
            //字符串拼接的方式构造sql语句
            String sql = "select * from test_t where id = "+id+" and name = '"+name+"'";
            //4. 执行sql语句，ResultSet类，用来存放获取的结果集
            ResultSet rs = statement.executeQuery(sql);

            //5.遍历结果集，打印数据
            while(rs.next())
            {
                System.out.println(rs.getInt("id") +"\t" + rs.getString("name"));
            }
            rs.close();
            con.close(); //关闭连接

        }catch (ClassNotFoundException e)
        {
            System.out.println("Cannot find the Driver");
        }
        catch (SQLException e)
        {
            e.printStackTrace();
           // System.out.println("Database closed");
        }
        finally
        {
            System.out.println("finished");
        }
    }
}
```
#### 判断sql注入点
通常情况下，sql注入漏洞的url如下形式：
http://xxx.xxx.xxx/abcd.php?id=XX

判断sql注入点，主要有两个方面：1. 判断该参数的url是否存在sql注入;2. 若存在，是什么类型的sql注入。

##### 是否存在sql注入
判断方法，在参数后面加单引号：http://xxx/abc.php?id=1'。原理是：无论什么类型参数都会因为单引号个数不匹配而报错。

若返回错误，存在sql注入；若不返回错误，可能页面对单引号做了过滤，也有可能不存在sql注入，待进一步确认。

##### 判断sql注入类型
* 数字型
```
方法： 经典的 and 1=1 和 and 1=2
1. url中输入 http://xxx/abc.php?id= x and 1=1 ，正常运行，下一步
2. url中输入 http://xxx/abc.php?id= x and 1=2， 正常运行
存在数字型sql注入。
若为字符型注入，则对应的语句如下：
select * from <表名> where id = 'x and 1=1' 
select * from <表名> where id = 'x and 1=2' 
不会出现以上结果，故为数字型注入。
```
* 字符型
```
方法：  and '1'='1 和 and '1'='2来判断
1. url中输入 http://xxx/abc.php?id= x' and '1'='1 ，正常运行，下一步
2. url中输入 http://xxx/abc.php?id= x' and '1'='2， 运行失败
存在数字型sql注入。
若为字符型注入，则对应的语句如下：
select * from <表名> where id = 'x' and '1'='1' 
select * from <表名> where id = 'x' and '1'='2' 
故为字符型注入。
```
### java中防御sql注入
Java中提供了PreparedStatement类来防止SQL注入攻击。利用预编译机制，PreparedStatement实例中包含了以编译的SQL语句，该语句可具有一个或多个输入参数，这些参数在SQL语句创建时并未指定，而是为每个输入参数用“？”作为一个占位符。即，通过预编译机制将sql语句中的主干和参数分别传输给数据库服务器，从而避免出现参数携带sql关键字时，修改sql语句导致的sql注入风险。

PreparedStatement主要优点：
1. 防止sql注入攻击
2. 使用了预编译机制，执行效率高于Statement，如果存在相同的预编译语句就不需要再次编译，只需要传入参数将编译过的语句执行即可，从而提高性能
3. sql语句用 ？代替参数，再用方法设置？的值，比起拼接字符串，更简洁，便于维护。
```java
//PreparedStatement语句使用
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class sqlTest
{
    public static void main(String[] args)
    {
        Connection con;
        //jdbc驱动程序名称，告诉JVM使用的是哪一个数据库驱动
        String driver = "com.mysql.cj.jdbc.Driver";
        //URL指向访问的数据库名my_db
        String url = "jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC";
        //登录mysql时的用户名和密码
        String user = "root";
        String password = "422504";

        try
        {
            //1. 解析获取数据库驱动
            Class.forName(driver);
            //2.使用Jdbc中的类，获取与mysql的连接
            con = DriverManager.getConnection(url, user, password);
            if(!con.isClosed())
            {
                System.out.println("Succeeded connecting to the database.");
            }
              //从键盘输入字符串
            Scanner sc = new Scanner();
            int id = sc.nextLine();
            String name = sc.nextLine();

            //构造sql语句，用？代替参数
            String sql = "select * from test_t where id = ? and name = ?";
            //3.创建一个preparedStatement对象，将sql语句发送到SQL,sql语句中的参数全部采用问号占位符
            PreparedStatement pst = con.prepareStatement(sql);
            System.out.printIn(pst);
            
            //设置问号占位符上的参数
            pst.setObject(1, id);
            pst.setObject(2,name);
            //4. 执行sql语句，ResultSet类，用来存放获取的结果集
            ResultSet rs = pst.executeQuery();

            //5.遍历结果集，打印数据
            while(rs.next())
            {
                System.out.println(rs.getInt("id") +"\t" + rs.getString("name"));
            }
            rs.close();
            con.close(); //关闭连接

        }catch (ClassNotFoundException e)
        {
            System.out.println("Cannot find the Driver");
        }
        catch (SQLException e)
        {
            e.printStackTrace();
           // System.out.println("Database closed");
        }
        finally
        {
            System.out.println("finished");
        }
    }
}
```
#### Statement和PreparedStatement使用对比
Statement使用
```sql
# 定义sql语句
String sql = "";
#创建一个statement对象
Statement statement = con.createStatement();
#传入SQL，并执行sql语句
 ResultSet rs = statement.executeQuery(sql);
```
PreparedStatement使用
```sql
# 定义sql语句
String sql = "";
#PreparedStatement对象，预编译，分离sql语句的主干和参数
PreparedStatement pst = con.prepareStatement(sql);
#设置问号占位符上的参数
pst.setObject(1, para1);
pst.setObject(2, para2);
#执行查询sql语句，注意executeQuery()函数不需要输入参数
 ResultSet rs = pst.executeQuery();

#执行更新和插入语句
String sqlInsert = "insert into test_t (name) values (?)";
String sqlUpdate = "update teset_t set name = ?";
PreparedStatement pst = con.prepareStatement(sqlInsert);
//PreparedStatement pst = con.prepareStatement(sqlUpdate);
#设置问号占位符上的参数
pst.setObject(1, name);
#执行查询sql语句，注意executeQuery()函数不需要输入参数
pst.executeUpdate();  //返回值为int类型
```
注：使用preparedStatement类时，query和update、insert使用的执行sql语句的函数不一样
