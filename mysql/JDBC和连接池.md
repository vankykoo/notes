# JDBC和连接池

## 一、JDBC概述

###1）基本介绍

①JDBC为访问不同的数据库提供了统一的接口，为使用者屏蔽了细节问题。

②Java程序员使用JDBC，可以连接任何提供了JDBC驱动程序的数据库系统，从而完成对数据库的各种操作。

###2）原理图

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/JDBC%E5%8E%9F%E7%90%86%E5%9B%BE.png)

###3）JDBC的好处

※JDBC时Java提供的一套用于数据库操作的接口API，Java程序员只需要面向这套接口编程即可。不同的数据库厂商，需要针对这套接口，提供不同实现。

###4）快速入门

JDBC程序编写步骤：

①注册驱动：加载**Driver**类

②获取连接：得到**Connection**

③执行增删改查：发生SQL给mysql执行

④释放资源：关闭相关连接

## 二、JDBC API

###1）基本介绍

JDBC API时一系列的接口，它统一和规范了应用程序与数据库的连接、执行SQL语句，并得到返回结果等各类操作，相关类和接口在java.sql与javax.sql包下。



###2）获取数据库连接的方法

```java
//方式一：
Driver driver = new Driver();//创建Driver对象
String url = "jdbc:mysql://localhost:3306/db02";//db02为数据库名
Properties properties = new Properties();
properties.setProperty("user","root");//用户名
properties.setProperty("password","vanky");//密码
Connection connect = driver.connect(url,properties);//连接数据库

//方式二：
Class<?> aClass = Class.forName("com.mysql.jdbc.Driver");//获取Driver类
Driver driver = (Driver)aClass.newInstance;//创建Driver对象
String url = "jdbc:mysql://localhost:3306/db02";//db02为数据库名
Properties properties = new Properties();
properties.setProperty("user","root");//用户名
properties.setProperty("password","vanky");//密码
Connection connect = driver.connect(url,properties);//连接数据库

//方式三
Class<?> aClass = Class.forName("com.mysql.jdbc.Driver");//获取Driver类
Driver driver = (Driver)aClass.newInstance;//创建Driver对象
String url = "jdbc:mysql://localhost:3306/db02";//db02为数据库名
String user = "user";
String password = "vanky";
DriverManager.registerDriver(driver);//注册Driver驱动
Connection connection = DriverManager.getConnection(url,user,password);//连接

//方式四
Class.forName("com.mysql.jdbc.Driver");//在加载Driver类时，完成注册
String url = "jdbc:mysql://localhost:3306/db02";//db02为数据库名
String user = "user";
String password = "vanky";
Connection connection = DriverManager.getConnection(url,user,password);//连接

//方法五
//通过properties读取信息
Properties properties = new Properties();
properties.load(new FileInputStream("src\\mysql.properties"));
String user = properties.getProperty("user");
String password = properties.getProperty("password");
String driver = properties.getProperty("driver");
String url = properties.getProperty("url");
Class.forName(driver);//可以不写，建议写上
Connection connection = DriverManager.getConnection(url,user,password);
```



###3）ResultSet

①表示数据库结果集的数据表，通常通过执行查询数据库的语句生成。

②**ResultSet**对象保持一个光标指向其当前的数据行。最初，光标位于第一行之前。

③**next**方法将光标移动到下一行，并且由于在**ResultSet**对象中没有更多行时，返回false，因此可以在**while**循环中使用循环来遍历结果集。

```java
//通过properties读取信息
Properties properties = new Properties();
properties.load(new FileInputStream("src\\mysql.properties"));
String user = properties.getProperty("user");
String password = properties.getProperty("password");
String driver = properties.getProperty("driver");
String url = properties.getProperty("url");
//得到连接
Class.forName(driver);//可以不写，建议写上
Connection connection = DriverManager.getConnection(url,user,password);
//创建statement，执行sql语句
Statement statement = connection.createStatement();
String sql = "select id,name,sex,borndate from actor";
ResultSet resultSet = statement.executeQuery(sql);
//使用while取出并打印
while(resultSet.next()){
  int id = resultSet.getInt(1);//1表示第一列
  //可以写成	int id = resultSet.getInt("id");
  String name = resultSet.getString(2);
  String sex = resultSet.getString(3);
  Date date = resultSet.getDate(4);
  System.out.println(id + "\t" + name + "\t" + date)
}
//关闭连接
resultSet.close();
statement.close();
connection.close();
```



###4）Statement

**[1]基本介绍：**

①**Statement**对象用于执行静态sql语句并返回其生成的结果的对象。

②在连接建立后，需要对数据库进行访问，执行命令或是sql语句，可以通过**Statement**[存在SQL注入]，**PreparedStatement**[预处理]，**CallableStatement**[存储过程]。

③SQL注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的SQL语句段或命令，恶意攻击数据库。万能密码【**or'1'='1**】

④需要用**PreparedStatement**取代**Statement**来防范SQL注入。



###5）PreparedStatement

**[1]基本介绍**

①PreparedStatement执行的SQL语句中的参数用问号（？）来表示，调用PreparedStatement对象的setXxx()方法来设置这些参数，setXxx()方法有两个参数，第一个是要设置的sql语句中的参数的索引（从1开始），第二个是设置的sql语句中的参数。

②调用executeQuery()，返回ResultSet对象。

③调用executeUptate()，执行更新，包括增、删、改。

[**2]预处理的好处**

①不用再使用+拼接sql语句，减少语法错误。

②有效地解决了sql注入问题。

③大大减少了编译次数，效率较高。

```java
//用connection得到PreparedStatement
String sql = "select id,`name` from student where id =? and name =?";
PreparedStatement preparedStatement = connection.prepareStatement(sql);
//给？赋值
preparedStatement.setString(1,admin_id);
preparedStatement.setString(2,admin_name);

//使用PreparedStatement的executeQuery方法
ResultSet resultSet = preparedStatement.executeQuery(sql);
if(resultSet.next()){
  String id = resultSet.getNString(1);
  String name = resultSet.getNString(2);
  System.out.println("id=" + id + " " + "name=" + name);
}
```



###6）API小结【重要】

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/API1.png)

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/API2.png)



## 三、JDBCUtils

###1）创建JDBCUtils工具类

①介绍：JDBCUtils工具类主要用于获取连接和释放资源（关闭资源）。

②代码演示：

```java
public class JDBCUtils{
  private static String user; //用户名
  private static String password; //密码
  private static String url; //url
  private static String driver; //驱动名
  
  //使用静态代码块初始化
  static {
        try {
            Properties properties = new Properties();
            properties.load(new FileInputStream("src\\mysql.properties"));
            //读取相关的属性值
            user = properties.getProperty("user");
            password = properties.getProperty("password");
            url = properties.getProperty("url");
            driver = properties.getProperty("driver");
        } catch (IOException e) {
            //在实际开发中，我们可以这样处理
            //1. 将编译异常转成 运行异常
            //2. 调用者，可以选择捕获该异常，也可以选择默认处理该异常，比较方便.
            throw new RuntimeException(e);
        }
    }
  
  //连接数据库方法，返回Connection
  public static Connection getConnection() {
        try {
            return DriverManager.getConnection(url, user, password);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
  
  //关闭资源的方法
  public static void close(ResultSet set,Statement statement,Connection connection){
    try {
            if (set != null) {
                set.close();
            }
            if (statement != null) {
                statement.close();
            }
            if (connection != null) {
                connection.close();
            }
        } catch (SQLException e) {
            //将编译异常转成运行异常抛出
            throw new RuntimeException(e);
        }
  }
}
```

※**静态代码块**: 格式：static{} 特点：需要通过static关键字修饰，随着类的加载而加载，并且自动触发、只执行一次 优先加载 使用场景：在类加载的时候做一些静态数据初始化的操作，以便后续使用。



###2）JDBCUtils的使用

```java
public void testSelect() {
  //1. 得到连接
  Connection connection = null;
  //2. 组织一个sql
  String sql = "select * from actor where id = ?";
  PreparedStatement preparedStatement = null;
  ResultSet set = null;
  //3. 创建PreparedStatement 对象
  try {
    connection = JDBCUtils.getConnection();
    System.out.println(connection.getClass()); 			      //com.mysql.jdbc.JDBC4Connection
    preparedStatement = connection.prepareStatement(sql);
    preparedStatement.setInt(1, 5);//给?号赋值
    //执行, 得到结果集；如果是修改就用executeUpdate()方法。
    set = preparedStatement.executeQuery();
    //遍历该结果集
    while (set.next()) {
      int id = set.getInt("id");
      String name = set.getString("name");
      String sex = set.getString("sex");
      Date borndate = set.getDate("borndate");
      String phone = set.getString("phone");
      System.out.println(id + "\t" + name + "\t" + sex + "\t" + borndate + "\t" + phone);
    }
  } catch (SQLException e) {
    e.printStackTrace();
  } finally {
    //关闭资源，没有就填null
    JDBCUtils.close(set, preparedStatement, connection);
  }
}
```

## 四、事务

###1）基本介绍

①JDBC程序中当一个Connection对象创建时，默认情况下是自动提交事务：每次执行一个sql语句时，如果执行成功，就会向数据库自动提交，而不能回滚。

②JDBC程序中为了让多个sql语句作为一个整体执行，需要使用事务。

③调用**Connection的【setAutoCommit(false)】**可以<u>取消自动提交事务</u>。

④在所有的sql语句都成功执行后，调用**Connection的【commit()】**方法<u>提交事务</u>。

⑤在其中某个操作失败或出现异常时，可以调用**Connection的【rollback()】**方法<u>回滚事务</u>。



###2）代码演示

```java
public void useTransaction() {
  //操作转账的业务
  //1. 得到连接
  Connection connection = null;
  //2. 组织一个sql
  String sql = "update account set balance = balance - 100 where id = 1";
  String sql2 = "update account set balance = balance + 100 where id = 2";
  PreparedStatement preparedStatement = null;
  //3. 创建PreparedStatement 对象
  try {
    connection = JDBCUtils.getConnection(); // 在默认情况下，connection是默认自动提交
    //※※※※※※※※※  将 connection 设置为不自动提交  ※※※※※※※
    connection.setAutoCommit(false); //开启了事务
    preparedStatement = connection.prepareStatement(sql);
    preparedStatement.executeUpdate(); // 执行第1条sql

    int i = 1 / 0; //抛出异常
    preparedStatement = connection.prepareStatement(sql2);
    preparedStatement.executeUpdate(); // 执行第3条sql

    //※※※※※※※※※  这里提交事务  ※※※※※※※※※
    connection.commit();

  } catch (SQLException e) {
    //※※※※※※※※※  这里我们可以进行回滚，即撤销执行的SQL  ※※※※※※※※※
    //默认回滚到事务开始的状态.
    System.out.println("执行发生了异常，撤销执行的sql");
    try {
      connection.rollback();
    } catch (SQLException throwables) {
      throwables.printStackTrace();
    }
    e.printStackTrace();
  } finally {
    //关闭资源
    JDBCUtils.close(null, preparedStatement, connection);
  }
}
```



## 五、批处理

###1）基本介绍

①当需要成批插入或者更新记录时，可以采用Java的批量更新机制，这一机制允许多条语句一次性提交给数据库批量处理。通常情况下比单独提交处理更有效率。

②JDBC的批量处理语句包括下面方法：

* 【addBatch()】：添加需要批量处理的sql语句或参数。
* 【executeBatch()】：执行批量处理语句。
* 【clearBatch()】：清空批处理包的语句。

③JDBC连接MySQL时，如果要使用批处理功能，请再properties的url后加上参数【rewriteBatchedStatements=true】。

④批处理往往和PreparedStatement一起搭配使用，既可以减少编译次数，又减少运行次数，效率大大提高。

###2）代码实现

```java
public void batch() throws Exception {

  Connection connection = JDBCUtils.getConnection();
  String sql = "insert into admin2 values(null, ?, ?)";
  PreparedStatement preparedStatement = connection.prepareStatement(sql);
  for (int i = 0; i < 5000; i++) {//5000执行
    preparedStatement.setString(1, "jack" + i);
    preparedStatement.setString(2, "666");
    //将sql 语句加入到批处理包中 -> 看源码
    /*
            //1. //第一就创建 ArrayList - elementData => Object[]
            //2. elementData => Object[] 就会存放我们预处理的sql语句
            //3. 当elementData满后,就按照1.5扩容
            //4. 当添加到指定的值后，就executeBatch
            //5. 批量处理会减少我们发送sql语句的网络开销，而且减少编译次数，因此效率提高
      */
    preparedStatement.addBatch();
    //当有1000条记录时，在批量执行
    if((i + 1) % 1000 == 0) {//满1000条sql
      preparedStatement.executeBatch();
      //清空一把
      preparedStatement.clearBatch();
    }
  }
  //关闭连接
  JDBCUtils.close(null, preparedStatement, connection);
}
```



## 六、数据库连接池

###1）传统获取Connection问题分析

①传统的JDBC数据库连接使用DriverManager来获取，每次向数据库建立连接的时候都要将Connection加载到内存中，再验证IP地址，用户名和密码（0.05s~1s时间）。需要数据库连接的时候，就向数据库要求一个频繁进行数据库连接操作将占用很多系统的资源，容易造成服务器崩溃。

②每一次数据库连接，使用完后都得断开，如果程序出现异常而未能关闭，将导致数据库内存泄露，最终将导致重启数据库。

③传统获取连接的方式，不能控制创建的连接数量，如果连接过多，也可能导致内存泄露，MySQL崩溃。

###2）基本介绍

①预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕后再放回去。

②数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个。

③当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。

![数据库连接池示意图](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

###3）数据库连接池的种类

①C3P0：速度相对较慢，稳定性不错。

②DBCP：速度相对较快，不稳定

③Proxool：又监控连接池状态的功能，稳定性较差

④BoneCP：速度快

⑤Druid（德鲁伊）：集DBCP、C3P0、Proxool优点于一身。

※JDBC的数据库连接池使用javax.sql.DataSource来表示，DataSource只是一个接口，该接口通常由第三方提供实现。



###4）C3P0的使用

```java
//方式一
//1. 创建一个数据源对象
ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();

//2. 通过配置文件mysql.properties 获取相关连接的信息
Properties properties = new Properties();
properties.load(new FileInputStream("src\\mysql.properties"));
//读取相关的属性值
String user = properties.getProperty("user");
String password = properties.getProperty("password");
String url = properties.getProperty("url");
String driver = properties.getProperty("driver");
//给数据源 comboPooledDataSource 设置相关的参数
//注意：连接管理是由 comboPooledDataSource 来管理
comboPooledDataSource.setDriverClass(driver);
comboPooledDataSource.setJdbcUrl(url);
comboPooledDataSource.setUser(user);
comboPooledDataSource.setPassword(password);
//设置初始化连接数
comboPooledDataSource.setInitialPoolSize(10);
//最大连接数
comboPooledDataSource.setMaxPoolSize(50);

  Connection connection = comboPooledDataSource.getConnection(); //这个方法就是从 DataSource 接口实现的
  connection.close();
```

```java
//方式二
//对ComboPooledDataSource配置相关参数放在xml文件下。
ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource("vanky");

Connection connection = comboPooledDataSource.getConnection();
connection.close();
```



###5）Druid的使用

```java
//1. 加入 Druid jar包
//2. 加入 配置文件 druid.properties , 将该文件拷贝项目的src目录
//3. 创建Properties对象, 读取配置文件
Properties properties = new Properties();
properties.load(new FileInputStream("src\\druid.properties"));

//4. 创建一个指定参数的数据库连接池, Druid连接池
DataSource dataSource =DruidDataSourceFactory.createDataSource(properties);

Connection connection = dataSource.getConnection();

connection.close();
```



###6）基于druid数据库连接池的工具类

```java
public class JDBCUtilsByDruid {

  private static DataSource ds;

  //在静态代码块完成 ds初始化
  static {
    Properties properties = new Properties();
    try {
      properties.load(new FileInputStream("src\\druid.properties"));
      ds = DruidDataSourceFactory.createDataSource(properties);
    } catch (Exception e) {
      e.printStackTrace();
    }

  }

  //编写getConnection方法
  public static Connection getConnection() throws SQLException {
    return ds.getConnection();
  }

  //关闭连接, 老师再次强调： 在数据库连接池技术中，close 不是真的断掉连接
  //而是把使用的Connection对象放回连接池
  public static void close(ResultSet resultSet, Statement statement, Connection connection) {

    try {
      if (resultSet != null) {
        resultSet.close();
      }
      if (statement != null) {
        statement.close();
      }
      if (connection != null) {
        connection.close();
      }
    } catch (SQLException e) {
      throw new RuntimeException(e);
    }
  }
}
```



## 七、Apache-DBUtils

###1）原理图

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/ApDBUtils%E7%A4%BA%E6%84%8F%E5%9B%BE.png)



###2）基本介绍

①commons-dbutils时Apache组织提供的一个开源JDBC工具类库，它是对JDBC的封装，使用dbutils能极大简化jdbc编码的工作量。

②使用DbUtils类

* QueryRunner类：该类封装了sql的执行，是线程安全的，可以实现增删改查、批处理。
* 使用QueryRunner类实现查询。
* ResultSetHandler接口：该接口用于处理java.sql.ResultSet，将数据按要求转换为另一种形式。

※QueryRunner类方法

①【ArrayHandler】：把结果集中的第一行数据转成对象数据。

②【ArrayListHandler】：把结果集中的每一行数据都转成一个数据，再存放到List中。

③【BeanHandler】：将结果集中的第一行数据封装到一个对应的JavaBean实例中。

④【BeanListHandler】：将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里。

⑤【ColumnListHandler】：将结果集中某一列的数据存放到List中。

⑥【KeyedHandler(name)】：将结果集中的每行数据都封装到Map里，再把这些map再存到 一个map，其key为指定的key。

⑦【MapHandler】：将结果集中的每一行数据封装到一个Map里，key是列名，value就是对应的值。

⑧【MapListHandler】：将结果集中的每一行数据都封装到一个Map里，然后再存放到List。



###3）DBUtils的使用

```java
//使用apache-DBUtils 工具类 + druid 完成对表的crud操作
public void testQueryMany() throws SQLException { //返回结果是多行的情况

  //1. 得到 连接 (druid)
  Connection connection = JDBCUtilsByDruid.getConnection();
  //2. 使用 DBUtils 类和接口 , 先引入DBUtils 相关的jar , 加入到本Project
  //3. 创建 QueryRunner
  QueryRunner queryRunner = new QueryRunner();
  //4. 就可以执行相关的方法，返回ArrayList 结果集
  //String sql = "select * from actor where id >= ?";
  //   注意: sql 语句也可以查询部分列
  String sql = "select id, name from actor where id >= ?";
  // 老韩解读
  //(1) query 方法就是执行sql 语句，得到resultset ---封装到 --> ArrayList 集合中
  //(2) 返回集合
  //(3) connection: 连接
  //(4) sql : 执行的sql语句
  //(5) new BeanListHandler<>(Actor.class): 在将resultset -> Actor 对象 -> 封装到 ArrayList
  //    底层使用反射机制 去获取Actor 类的属性，然后进行封装
  //(6) 1 就是给 sql 语句中的? 赋值，可以有多个值，因为是可变参数Object... params
  //(7) 底层得到的resultset ,会在query 关闭, 关闭PreparedStatment
  /**
         * 分析 queryRunner.query方法:
         * public <T> T query(Connection conn, String sql, ResultSetHandler<T> rsh, Object... params) throws SQLException {
         *         PreparedStatement stmt = null;//定义PreparedStatement
         *         ResultSet rs = null;//接收返回的 ResultSet
         *         Object result = null;//返回ArrayList
         *
         *         try {
         *             stmt = this.prepareStatement(conn, sql);//创建PreparedStatement
         *             this.fillStatement(stmt, params);//对sql 进行 ? 赋值
         *             rs = this.wrap(stmt.executeQuery());//执行sql,返回resultset
         *             result = rsh.handle(rs);//返回的resultset --> arrayList[result] [使用到反射，对传入class对象处理]
         *         } catch (SQLException var33) {
         *             this.rethrow(var33, sql, params);
         *         } finally {
         *             try {
         *                 this.close(rs);//关闭resultset
         *             } finally {
         *                 this.close((Statement)stmt);//关闭preparedstatement对象
         *             }
         *         }
         *
         *         return result;
         *     }
         */
  List<Actor> list =
    queryRunner.query(connection, sql, new BeanListHandler<>(Actor.class), 1);
  System.out.println("输出集合的信息");
  for (Actor actor : list) {
    System.out.print(actor);
  }


  //释放资源
  JDBCUtilsByDruid.close(null, null, connection);
```

```java
//演示 apache-dbutils + druid 完成 返回的结果是单行记录(单个对象)
Connection connection = JDBCUtilsByDruid.getConnection();
QueryRunner queryRunner = new QueryRunner();
//可以执行相关的方法，返回单个对象
String sql = "select * from actor where id = ?";
// 因为我们返回的单行记录<--->单个对象 , 使用的Hander 是 BeanHandler
Actor actor = queryRunner.query(connection, sql, new BeanHandler<>(Actor.class), 10);
System.out.println(actor);
// 释放资源
JDBCUtilsByDruid.close(null, null, connection);

/*-----------------------------------------------------------------------*/
//演示apache-dbutils + druid 完成查询结果是单行单列-返回的就是object
Connection connection = JDBCUtilsByDruid.getConnection();
QueryRunner queryRunner = new QueryRunner();
//执行相关的方法，返回单行单列 , 返回的就是Object
String sql = "select name from actor where id = ?";
//老师解读： 因为返回的是一个对象, 使用的handler 就是 ScalarHandler
Object obj = queryRunner.query(connection, sql, new ScalarHandler(), 4);
System.out.println(obj);
JDBCUtilsByDruid.close(null, null, connection);
```

```java
//演示apache-dbutils + druid 完成 dml (update, insert ,delete)
Connection connection = JDBCUtilsByDruid.getConnection();
QueryRunner queryRunner = new QueryRunner();
//4. 这里组织sql 完成 update, insert delete
//String sql = "update actor set name = ? where id = ?";
//String sql = "insert into actor values(null, ?, ?, ?, ?)";
String sql = "delete from actor where id = ?";
//解读
//(1) 执行dml 操作是 queryRunner.update()
//(2) 返回的值是受影响的行数 
//int affectedRow = queryRunner.update(connection, sql, "林青霞", "女", "1966-10-10", "116");
int affectedRow = queryRunner.update(connection, sql, 1000 );
System.out.println(affectedRow > 0 ? "执行成功" : "执行没有影响到表");
// 释放资源
JDBCUtilsByDruid.close(null, null, connection);
```



## 八、DAO增删改查-BasicDao

###1）基本说明

①DAO：data access object数据访问对象。

②这样的通用类，称为BasicDao，是专门和数据库交互的，即完成对数据库（表）的crud操作。

③在BasicDao的基础上，实现一张表对应一个DAO，更好地完成功能，比如：Customer表-------Customer.java类（JavaBean）------CustomerDao.java。

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/BasicDAO1.png)

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/BasicDAO2.png)



###2）BasicDao

```java
public class BasicDAO<T> { //泛型指定具体类型

    private QueryRunner qr =  new QueryRunner();

    //开发通用的dml方法, 针对任意的表
    public int update(String sql, Object... parameters) {

        Connection connection = null;

        try {
            connection = JDBCUtilsByDruid.getConnection();
            int update = qr.update(connection, sql, parameters);
            return  update;
        } catch (SQLException e) {
           throw  new RuntimeException(e); //将编译异常->运行异常 ,抛出
        } finally {
            JDBCUtilsByDruid.close(null, null, connection);
        }

    }

    //返回多个对象(即查询的结果是多行), 针对任意表

    /**
     *
     * @param sql sql 语句，可以有 ?
     * @param clazz 传入一个类的Class对象 比如 Actor.class
     * @param parameters 传入 ? 的具体的值，可以是多个
     * @return 根据Actor.class 返回对应的 ArrayList 集合
     */
    public List<T> queryMulti(String sql, Class<T> clazz, Object... parameters) {

        Connection connection = null;
        try {
            connection = JDBCUtilsByDruid.getConnection();
            return qr.query(connection, sql, new BeanListHandler<T>(clazz), parameters);

        } catch (SQLException e) {
            throw  new RuntimeException(e); //将编译异常->运行异常 ,抛出
        } finally {
            JDBCUtilsByDruid.close(null, null, connection);
        }

    }

    //查询单行结果 的通用方法
    public T querySingle(String sql, Class<T> clazz, Object... parameters) {

        Connection connection = null;
        try {
            connection = JDBCUtilsByDruid.getConnection();
            return  qr.query(connection, sql, new BeanHandler<T>(clazz), parameters);

        } catch (SQLException e) {
            throw  new RuntimeException(e); //将编译异常->运行异常 ,抛出
        } finally {
            JDBCUtilsByDruid.close(null, null, connection);
        }
    }

    //查询单行单列的方法,即返回单值的方法

    public Object queryScalar(String sql, Object... parameters) {

        Connection connection = null;
        try {
            connection = JDBCUtilsByDruid.getConnection();
            return  qr.query(connection, sql, new ScalarHandler(), parameters);

        } catch (SQLException e) {
            throw  new RuntimeException(e); //将编译异常->运行异常 ,抛出
        } finally {
            JDBCUtilsByDruid.close(null, null, connection);
        }
    }
}
```



































