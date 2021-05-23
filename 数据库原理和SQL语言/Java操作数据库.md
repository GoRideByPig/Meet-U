# 利用Java操作数据库的方法



## 一、基本操作



1. **第一步：在项目中创建`lib`目录，然后用`add as library`导入驱动。**
2. **第二步：**创建`db.properties`文件，在其中配置**驱动名称，URL，用户名和密码。**

```properties
driver=org.postgresql.Driver
//背
url=jdbc:postgresql://localhost:5432/postgres?useUnicode=true&characterEncoding=utf8&useSSL=true

username=postgres

password=123456
```

3. **第三步：创建`utils`目录，然后在其中创建`JDBCUtils`工具类**

（1）成员变量：就是在`db.properties`文件中配置的四个属性

```java
public class JDBCUtils
{
    private static String driver = null;
    private static String url = null;
    private static String username = null;
    private static String password = null;
}
```

（2）创建**静态代码块**，然后在其中把`db.properties`中的文件读取，之后加载驱动。

```java
static
{
    try
    {
        //背
        InputStream in = JDBCUtils.class.getClassLoader().getResourceAsStream("db.properties");
        Properties properties = new Properties();
        properties.load(in);
        
        driver = properties.getProperty("driver");
        url = properties.getProperty("url");
        username = properties.getProperty("username");
        password = properties.getProperty("password");
        
        //加载一次驱动
        Class.forName(driver);
        
    }catch(Exception e){
        e.printStackTrace();
    }
}
```

（3）**写上获取连接的方法：**

```java
public static Connection getConnection()
{
    return DriverManager.getConnection(url,username,password);
}
```

（4）**写上释放资源的方法**

```java
public static void release(Connection conn,Statement st,ResultSet rs)
{
    if(rs != null) rs.close();
    if(st != null) st.close();
    if(conn != null) conn.close();
}
//IDE中会提示抛异常
```

4. **使用方法：**

```java
public class Test
{
    public static void main(String[] args)
    {
        //建立连接，语句结果集对象
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        
        try{
            //获取链接
            conn = JDBCUtils.getConnection();
            //利用连接获取语句
            st = conn.createStatement();
            //编写sql语言
            String sql = "...";
            //利用语句对象执行语句,用结果集去接受
            rs = st.executeQuery();
            //利用迭代器取获取某一列的值
            while(rs.next())
            {
                System.out.println(rs.getString("name"));
            }
            
        }catch(SQLException e){
            print(trace);
        } finally{
            //在finally中释放资源
            JDBCUtils.release(conn,st,rs);
        }
    }
}
```



## 二、对象解释



1. > `DriverManager`对象解释

```java
//相当于激活这个类，拿到他的静态代码
Class.forName("com.mysql.jdbc.driver");

//原来的写法：注册驱动给
DriverManager.registerDriver(new com.mysql.jdbc.Driver);

//源码中的静态代码块已经用了上个代码，推荐使用反射写法


Connection connection = DriverManager.getConnection(url,username,password);
//connection代表数据库
//数据库设置自动提交
//事务提交
//事务回滚
connection.commit();
connection.rollback();
connection.setAutoCommit(true);

```



2. > `URL`对象解释

```java
String url = ?;
//jdbc:mysql://主机地址:端口号/数据库名？参数1&参数2&参数3
//jdbc:oracle:thin:@localhost:1521:sid
//公式为：协议：//主机地址：端口号/数据库名？参数1&参数2&参数3
```



3. > `Statement`对象：执行sql的对象

```java
Statement statement = connnection.createStatement();
statement.executeQuery();//执行查询
statement.execute();//执行所有sql，有个判断的功能，效率低
statement.executeUpdate();//执行更新，插入，删除操作，只是sql不同，返回一个受影响的行数
```

4. > ResultSet 查询的结果集：封装了所有的查询结果

   ```java
   ResultSet resultSet = statement.executeQuery(sql);
   
   result.getObject();//在不知道列类型的情况下使用
   result.getString();//知道列的类型就用指定的类型
   result.getInt();
   result.getDouble();
   ...
   ```

   遍历，指针

   ```java
   [1,1,1,1,1,1,1]
   resultSet.beforeFirst();//将指针移到第一个元素的位置
   resultSet.afterLast();//将指针移到最后一个元素的位置
   resultSet.next();//移动到下一个数据
   resultSet.previous();//移动到前一行
   resultSet.absolute(row);//移动到指定行
   ```

   

5. > 释放资源

   ```java
   //释放资源
   resultSet.close();
   statement.close();
   connection.close();
   ```

   

## 三、Statement对象详解



JDBC中的statement对象用于向数据库中发送sql语句，完成对数据库的增删改查

> CRUD操作-create

使用`executeUpdate(String sql)`方法完成数据添加操作，实例操作

```java
Statement st = conn.createStatement();
String sql = "insert into user(...) values (..,..)";
int num = st.executeUpdate(sql);

if(num > 0)
{
    System.out.println("插入成功");
}
```

> CRUD操作-delete

```java
Statement st = conn.createStatement();
String sql = "delete from user where id = 1";
int nums = st.executeUpdate(sql);
if(num > 0)
{
    System.out.println("删除成功！");
}
```

> CRUD操作-update

```java
Statement st = conn.createStatement();
String sql = "update user set name = '' where name='' ";
int num = st.executeUpdate(sql);
if(num > 0)
{
    System.out.println("修改成功！！！");
}
```



> CRUD操作-read

```java
Statement st = conn.createStatement();
String sql = "select * from user where id = 1";
ResultSet rs = st.executeQuery(sql);
while(rs.next())
{
    
}
```



