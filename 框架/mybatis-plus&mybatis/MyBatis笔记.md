#MyBatis

## 零、杂碎知识点

###1）**面向接口编程的好处**

降低了程序的耦合性 .其能够最大限度的解耦，所谓解耦既是解耦合的意思，它和耦合相对。 耦合就是联系，耦合越强，联系越紧密。 在程序中紧密的联系并不是一件好的事情，因为两种事物之间联系越紧密，你更换其中之一的难度就越大，扩展功能和debug的难度也就越大。

易扩展. 我们知道程序设计的原则是对修改关闭,对新增开放.面向接口编程扩展功能只需要创建新实现类重写接口方法进行升级扩展就可以,达到了在不修改源码的基础上扩展的目的.



###2）**SqlSessionFactory、sqlSession**

- 加载MyBatis的配置文件：SqlSessionFactory可以读取MyBatis的配置文件，并解析其中的配置信息，包括数据库连接信息、映射文件信息、插件信息等。
- 创建SqlSession对象：SqlSessionFactory可以根据配置信息创建SqlSession对象，SqlSession是MyBatis中最重要的对象之一，用于执行SQL语句、提交事务等操作。
- 管理数据库连接：SqlSessionFactory可以管理数据库连接池，避免频繁地连接和断开数据库，提高应用程序的性能。
- 提供事务管理：SqlSessionFactory可以为SqlSession提供事务管理功能，保证数据库操作的原子性、一致性和持久性。



**SqlSession是应用程序与持久存储层之间执行交互操作的一个单线程对象，也是MyBatis执行持久化操作的关键对象**。 SqlSession对象完全包含以数据库为背景的所有执行SQL操作的方法，它的底层封装了JDBC连接，可以用SqlSession实例来直接执行已映射的SQL语句。



## 一、configuration

```xml
 <?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
    <settings>
        <setting name="logImpl" value="SLF4J"/>
    </settings>
    
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="vanky"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

* default表示默认使用的环境。
* 默认环境：当你使用mybatis创建SqlSessionFactory对象的时候，没有指定环境的话，默认使用哪个环境。
* 一般一个环境environment会对应一个SqlSessionFactory对象。
* **transactionManager标签：**
  * 作用：配置事务管理器，指定mybatis具体使用什么方式去管理事务。
  * type属性有两个值：
    * JDBC：使用原生的JDBC代码来管理事务
    * MANAGED：mybatis不再负责事务的管理，将事务管理交给其它的JAVAEE容器来管理。如：spring
  * 不区分大小写。
  * 在mybatis中提供了一个事务管理器接口：Transaction
    * 在该接口下有两个实现类：JdbcTransaction、ManagedTransaction
    * 如果type=“JDBC“，那么底层会实例化JdbcTransaction。
    * 如果type=”MANAGED“，那么底层会实例化ManagedTransaction。
* **dataSource配置**
  * dataSource：数据源
  * 作用：为程序提供Connection对象。（给程序提供Connection对象的都叫做数据源）
  * 数据源实际上是一套规范，JDK中有这套规范：javax.sql.DataSource（这个数据源的规范，这套接口实际上是JDK规定的）。 
  * 我们自己也可以编写数据源组件，只要实现javax.sql.DataSource接口就行，实现接口当中所有的方法，这样就有了自己的数据源，比如可以写一个属于自己的数据库连接池，（数据库连接池是提供连接对象的，所以数据库连接池就是一个数据源）
  * 常见的数据源组件（常见的数据库连接池）：
    * 德鲁伊druid连接池
    * C3P0、dbcp
  * type属性用来指定数据源的类型，指定具体使用什么方式来获取Connection对象：
    * type属性有三个值：必须三选一。
    * type="[UNPOOLED | POOLED | JNDI]"
    * UNPOOLED：不使用数据库连接池技术，每一次请求过来之后，都创建新的Connection对象。
    * JNDI：集成其它第三方的数据库连接池。
      * JNDI是一套规范，大部分web容器都实现了JNDI规范，如：tomcat、Jetty、WebSphere，这些服务器（容器）都实现了JNDI规范。
      * JNDI是：Java命名目录接口，Tomcat服务器实现了这个规范。
  * `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
  * `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
  * `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）。
  * `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。

>  连接池优点：
>
>  1、每一次获取连接都从连接池中拿，效率高，
>
>  2、因为每一次只能从池中拿，所以连接对象的创建数量是可控的。
>
>  如果连接池内连接的数量过多，会出现宕机现象。
>
>  **宕机：指操作系统无法从一个严重系统错误中恢复过来，或系统硬件层面出问题，以致系统长时间无响应，而不得不重新启动计算机的现象**。

* java.util.Properties类是一个Map集合，key和value都是String类型。
  * 在properties标签中可以配置很多属性。
  * <properties resource="jdbc.properties"/>






## 二、在WEB应用中使用MyBatis

* Dao对象中的任何一个方法和业务不挂钩，没有任何业务逻辑在里面。

  * DAO中的方法就是做CRUD的。所以方法名大部分是：insertXXX，deleteXXX，updateXXX，selectXXX。

* 为什么service要分为接口和实现类：

  * **面向接口开发。**
  * 多人分模块开发时，写service(业务层)的人将接口定义好提交到SVN，其它层的人直接可以调用接口方法，而写service层的人也可以通过实现类写具体方法逻辑。达到多人同时开发。

  ​

> SVN全称：Subversion，是一个开放源代码的**版本控制系统**
>
> Svn是一种**集中式文件版本管理系统**。集中式代码管理的**核心是服务器**，所有开发者在开始新一天的工作之前必须从服务器获取代码，然后开发，最后解决冲突，提交。
>
> 集中式文件版本控制器：**将所有的文件都交由服务器来进行统一的管理。既然是有服务器的，那么就需要联网进行操作了。**
>
> 为什么要使用SVN
>
> 我们写一个项目一般都是一个团队来写，**如果我们没有用SVN的话，那么我们只能在团队中互相拷贝对方的代码来完成我们的项目**。
>
> SVN还有如下的好处：
>
> - **轻松比较不同版本间的细微差别【修改了代码，就有版本号，还能知道修改前后的数据】**
> - **及时了解团队中其他成员的进度【如果没有把代码提交到服务器中，就是做得比较慢了】**
> - **广域网共享【连上局域网就可以代码共享了】**
> - **协同工作，大大提高团队工作效率**
>
> 快速了解SVN
>
> 

* **SqlSessionUtils**

  ```java
  public class SqlSessionUtil {
      private SqlSessionUtil(){}

      private static SqlSessionFactory sqlSessionFactory;

      static{
          try {
              sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
          } catch (IOException e) {
              throw new RuntimeException(e);
          }
      }

      private static ThreadLocal<SqlSession> local = new ThreadLocal<>();

      /**
       * 返回会话对象
       * @return
       */
      public static SqlSession openSession(){
          SqlSession sqlSession = local.get();
          if (sqlSession == null){
              sqlSession = sqlSessionFactory.openSession();
              //将sqlSession对象绑定到当前线程上
              local.set(sqlSession);
          }
          return sqlSession;
      }

      /**
       * 关闭sqlSession（从当前线程中移除SqlSession对象。）
       * @param sqlSession
       */
      public static void close(SqlSession sqlSession){
          if (sqlSession != null) {
              sqlSession.close();
              local.remove();
          }
      }
  }
  ```

  * 为什么把SqlSession对象放到ThreadLocal当中呢？**为了保证一个线程对应一个SqlSession。**



* **MyBatis的三个作用域及其生命周期**

  **SqlSessionFactoryBuilder**

  这个类可以被实例化、使用和丢弃，**一旦创建了 SqlSessionFactory，就不再需要它了。** 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

  ​

  **SqlSessionFactory**

  **SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在**，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

  ​

  **SqlSession**

  **每个线程都应该有它自己的 SqlSession 实例**。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 **绝对不能将 SqlSession 实例的引用放在一个类的静态域**，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。



### javassist

* mybatis中，提供了相关的机制，可以动态为我们生成dao接口实现类，（代理类：dao接口的代理）
* mybatis当中实际上采用了代理模式，在内存中生成dao接口的代理类，然后创建代理类的实例。
* 使用mybatis的这种代理机制的前提：SqlMapper.xml文件当中namespace必须是dao接口的全限定名称，id必须是dao接口中的方法名。

`private Account account = SqlSessionUtil.openSession().getMapper(AccountDao.class);`

* **sqlSeesion.getMapper(XXXDao.class);**



* dao包改为mapper包。只需要一个接口和对应的xml文件。

  * **注意方法名和id对应。namespace和接口名对应。**

  ```java
  public interface CarMapper {

      /**
       * 新增car
       * @param car
       * @return
       */
      int insert(Car car);

      /**
       * 根据id删除car
       * @param id
       * @return
       */
      int deleteById(Long id);

      /**
       * 修改汽车信息
       * @param car
       * @return
       */
      int update(Car car);

      /**
       * 根据Id查询汽车信息
       * @param Id
       * @return
       */
      Car selectById(Long Id);

      /**
       * 获取所有汽车信息
       * @return
       */
      List<Car> selectAll();
  }
  ```

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

  <mapper namespace="com.powernode.mybatis.mapper.CarMapper">
      <insert id="insert">
          insert into t_car values (null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
      </insert>

      <delete id="deleteById">
          delete from t_car where id = #{id}
      </delete>

      <update id="update">
          update t_car set
              car_num = #{carNum},
              brand = #{brand},
              guide_price = #{guidePrice},
              produce_time = #{produceTime},
              car_type = #{carType}
          where
              id = #{id}
      </update>

      <select id="selectById" resultType="com.powernode.mybatis.pojo.Car">
          select
              id,
              car_num as carNum,
              brand,
              guide_price as guidePrice,
              produce_time as produceTime,
              car_type as carType
          from t_car where id = #{id}
      </select>

      <select id="selectAll" resultType="com.powernode.mybatis.pojo.Car">
          select
              id,
              car_num as carNum,
              brand,
              guide_price as guidePrice,
              produce_time as produceTime,
              car_type as carType
          from t_car
      </select>

  </mapper>
  ```

  ​


## 三、MyBatis小技巧

### 1）、#{}和${}的区别

* \#{}
  * 底层使用PreparedStatement。
  * 特点：先进行SQL语句的编译，然后给SQL语句的占位符(问号?)传值。可以避免SQL注入的风险。
* ${}
  * 底层使用Statement
  * 特点：先进行SQL语句的拼接，然后再对sql语句进行编译，存在SQL注入的风险。

※优先使用#{}，这是原则，避免SQL注入的风险。

※如果需要SQL语句的关键字放到SQL语句中，只能使用${}，因为#{}是以值的的形式放到sql语句中的。



* 向SQL语句当中拼接表名，就需要使用${}
  * 现实业务中，可能会存在分表存储数据的情况，因为一张表存储数据量太大，查询效率低。
  * 可以将这些数据有规律的分表存储，这样在查询的时候效率就比较高。



* **批量删除**
  * 第一种写法or：`delete from t_car where id=1 or id=2 or id=3;`
  * 第二种写法in：`delete from t_car where id in(1,2,3);`
    * 使用in时，应该使用${} 拼接
    * `delete from t_car where id in (${ids})`



* **模糊查询**

  需求：根据汽车品牌进行模糊查询

  `select * from t_car where brand like '%奔驰%';`

  * 方案一：`'%${brand}%'`

  * 方案二：concat函数，这是mysql数据库中的一个函数，专门进行字符串拼接。

    ​		`concat('%',#{brand},'%')`

  * 方案三：`"%"#{brand}"%"`



### 2）、typeAliases起别名

* 所有别名不区分大小写.
* namespace不能使用别名.

```xml
<!--起别名-->
<typeAliases>
  
  <!--自己指定别名-->
  <typeAlias type="com.powernode.mybatis.pojo.Car" alias="aaa"/>
  <typeAlias type="com.powernode.mybatis.pojo.Log" alias="bbb"/>

  <!--使用默认别名（类的简名），如Car，Log-->
  <typeAlias type="com.powernode.mybatis.pojo.Log"/>
  <typeAlias type="com.powernode.mybatis.pojo.Car"/>

  <!--包下所有类自动起别名，使用简名作用别名-->
  <package name="com.powernode.mybatis.pojo"/>
</typeAliases>
```



### 3）、mapper标签

* `<mapper resource="CarMapper.xml"/>` 要求类的根路径下必须有：CarMapper.xml
* `<mapper url="file:///d:/CarMapper.xml"` 要求d:/下有CarMapper.xml文件
* `<mapper class="全限定接口名，带有包名"/>`



* mapper标签的三个属性：
  * **resource**：这种方式是从类的根路径下开始查找资源，采用这种方式的话，配置文件需要放到类路径当中。
  * **url**：这种方式是一种绝对路径，这种方式不要求配置文件放到类路径中，只要提供一个绝对路径就行，这种方式用的极少，因为移植性差。
  * **class**：这个位置提供的是mapper接口的全限定接口名，必须带有包名。
    * `<mapepr class="com.powernode.mybatis.mapper.CarMapper"/>`
    * 如果class指定的是`com.powernode.mybatis.mapper.CarMapper`，那么mybatis框架会自动到`com/powernode/mybatis/mapper`目录下查找CarMapper.xml文件。
    * 注意：也就是说，如果采用这种方式，必须保证CarMapper.xml文件和CarMapper接口必须在同一个目录下，并且名字一致。CarMapper接口----->CarMapper.xml

※注意：在IDEA的resources目录下新建多重目录时，应该使用`com/powernode/mybatis/mapper`，而不是`com.powernode.mybatis.mapper`



* 实际开发中，一般使用package来映射。前提是XML文件必须和接口放一起，并且名字一致。

  `<package name="com.powernode.mybatis.mapper"/>`



### 4）、返回并使用自增属性

* useGeneratedKeys="true" ：使用自动生成的主键值。
* keyProperty="id" ：指定主键值赋值给对象的哪个属性，这个就表示将主键值赋值给Car对象的id属性。

```xml
<insert id="insertCarUseGeneratedKeys" useGeneratedKeys="true" keyProperty="id">
        insert into t_car values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    </insert>
```





## 四、MyBatis参数

### 1）、单个简单类型参数

* byte  short  int long  float  double  char
* Byte Short  Integer  Long  Float  Double  Character
* String
* java.util.Date
* java.sql.Date


* parameterType属性的作用：告诉MyBatis框架，当前方法的参数是什么类型的。【mybatis框架自身带有自动推断机制，所以大部分情况下parameterType属性都可以省略不写。】

```xml
<select id="selectById" resultType="student" parameterType="java.lang.Long">
        select * from t_student where id = #{id}
</select>
```



### 2）、Map

\#{}内为map的key值。



###3）、实体类

\#{}内为实体类属性名称。



### 4）、多个参数

* 方法一：

  ```java
  	 //接口中：
  /**
       * 根据名字和性别查询.
       * 如果是多个参数，mybatis框架底层会自动创建一个Map集合，并且Map集合是以以下方式存储参数的：
       *      map.put("arg0",name);
       *      map.put("arg1",sex);
       *
       * 或者：map.put("param1",name);
       *      map.put("param2",sex);
       * @param name
       * @param sex
       * @return
       */
      List<Student> selectByNameAndSex(String name, Character sex);
  ```

  ```xml
  <select id="selectByNameAndSex" resultType="student">
          select * from t_student where name = #{arg0} and sex = #{arg1}
  </select>
  ```



* 方法二：@Param注解

  ```java
  	//接口中：
  /**
       * 使用@Param注解：
       *  map.put("name",name);
       *  map.put("sex",sex);
       *  arg0和arg1会失效
       *  param1和param2仍有效。
       * @param name
       * @param sex
       * @return
       */
      List<Student> selectByNameAndSex2(@Param("name") String name, @Param("sex") Character sex);
  ```

  ```xml
  <select id="selectByNameAndSex2" resultType="student">
          select * from t_student where name = #{name} and sex = #{sex}
      </select>
  ```

  ​




## 五、查询

### 1）、返回Map

```java
/**
     * 根据id查询汽车信息，放到map集合中。
     * @param id
     * @return
     */
    Map<String,Object> selectByIdRetMap(Long id);
```

数据库表头名为KEY，值为Value。

`{car_num=8899, id=14, guide_price=50.20, produce_time=2018-12-14, brand=丰田酷路泽, car_type=燃油车}`



### 2）、返回一个大Map

可以设置map的key为记录中的某个属性

```java
/**
     * 查询所有，返回一个大Map集合。
     * Map集合的key是每条记录的主键值。
     * Map集合的value是每条记录。
     * @return
     */
    @MapKey("id")   //将查询结果的id值作为整个大Map集合的Key。
    Map<Long,Map<String,Object>> selectAllRetMap();
```

> **结果：**
>
> {
>
> 1={car_num=1001, id=1, guide_price=10.00, produce_time=2020-10-11, brand=宝马520Li, car_type=燃油车}, 
>
> 2={car_num=1002, id=2, guide_price=55.00, produce_time=2020-11-11, brand=奔驰E300L, car_type=新能源}, 
>
> 4={car_num=1234, id=4, guide_price=30.30, produce_time=1999-10-11, brand=凯美瑞, car_type=燃油车}, 
>
> 5={car_num=1003, id=5, guide_price=30.00, produce_time=2020-10-11, brand=丰田霸道, car_type=燃油车}, 
>
> 10={car_num=1111, id=10, guide_price=10.00, produce_time=2020-11-11, brand=比亚迪汉, car_type=电车}, 
>
> 11={car_num=1111, id=11, guide_price=10.00, produce_time=2020-11-11, brand=比亚迪汉Plus, car_type=电车},
>
>  12={car_num=3333, id=12, guide_price=30.00, produce_time=2020-10-11, brand=比亚迪秦, car_type=新能源}, 
>
> 13={car_num=9879, id=13, guide_price=35.00, produce_time=2014-12-21, brand=皇冠新版, car_type=新能源}, 
>
> 14={car_num=8899, id=14, guide_price=50.20, produce_time=2018-12-14, brand=丰田酷路泽, car_type=燃油车}
>
> }



### 3）、结果映射ResultMap

* 在sql映射配置文件中，专门定义一个结果映射，在这个结果映射当中指定数据库表的字段名和Java类的属性名的对应关系。
  * type属性：用来指定pojo类的类名（可以用别名）
  * id属性：指定resultMap的唯一标识，这个id要在select标签中使用。
* 细节：
  * 如果数据库表中有主键，建议在resultMap中配置一个id标签，提高效率。
  * property：填写pojo类的属性名。
  * column填写数据库表的字段名。
  * 如果property和column的值一样。可以省略。



### 4）、开启驼峰命名自动映射

* 必须符合命名规范：

  * Java命名规范：首字母小写，后面每个单词的首字母大写，遵循驼峰命名方式。
  * SQL命名规范：全部小写，单词之间用下划线分隔。
  * carNum<------>car_num                carType<----------->car_type

* 步骤：

  * 在mybatis-config.xml下配置settings。

    ```xml
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    ```

  * 在sql映射配置文件中，使用resultType即可：

    ```xml
    <select id="selectAllByMapUnderscoreToCamelCase" resultType="car">
      select * from t_car
    </select>
    ```

    ​



### 5）、查询总记录条数：

CarMapper.java 

```java
/**
	 * 
     * 查询总记录条数
     * @return
     */
Long selectTotal();
```

CarMapper.xml

```xml
<select id="selectTotal" resultType="long">
  select count(*) from t_car
</select>
```





## 六、动态SQL

### 1）、if标签

* if标签中test属性是必须的。
* 如果test是true，则if标签中的sql语句会拼接，反之，则不会拼接。
* test属性中可以使用的是：
  * 当使用了@Param注解，那么test中要出现的是@Param注解指定的参数名，@Param("brand")，那么这里只能用brand。
  * 当没有使用@Param注解，那么test中要出现的是：param1，param2，param3  / arg0，arg1，arg2.....
  * 当使用了POJO，那么test中出现的是POJO类的属性名。



CarMapper.java

```java
/**
* 多条件查询
* @return
*/
List<Car> selectByMutiCondition(@Param("brand")String brand, @Param("guidePrice")Double guidePrice, @Param("carType") String carType);
```

CarMapper.xml

```xml
<select id="selectByMutiCondition" resultType="car">
  		<!--加入一个恒成立条件便于拼接-->
        select * from t_car where 1 = 1
        <if test="brand != null and brand != ''">
            and brand like "%"#{brand}"%"
        </if>
        <if test="guidePrice != null and guidePrice != ''">
            and guide_price > #{guidePrice}
        </if>
        <if test="carType != null and carType != ''">
            and car_type like "%"#{carType}"%"
        </if>
    </select>
```



###2）、where标签

* 作用
  * 所有条件都为空时，where标签保证不会生成where子句。
  * 自动去除某些条件**前面**多余的and或or。

CarMapper.java

```java
	/**
     * 使用where标签
     * @param brand
     * @param guidePrice
     * @param carType
     * @return
     */
List<Car> selectByMutiConditionWithWhere(@Param("brand")String brand, @Param("guidePrice")Double guidePrice, @Param("carType") String carType);
```

CarMapper.xml

```xml
<select id="selectByMutiConditionWithWhere" resultType="car">
  select * from t_car
  <where>
    <if test="brand != null and brand != ''">
      and brand like "%"#{brand}"%"
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      and guide_price > #{guidePrice}
    </if>
    <if test="carType != null and carType != ''">
      and car_type like "%"#{carType}"%"
    </if>
  </where>
</select>
```



### 3）、trim标签

* 属性
  * prefix：加前缀
  * suffix：加后缀
  * prefixOverrides：删除前缀
  * suffixOverrides：删除后缀
* 细节
  * prefix="where"：是在trim标签所有内容的前面添加where
  * suffixOverrides="and|or"：把trim标签中内容的后缀and或or去掉。



CarMapper.java

```java
	/**
     * 使用trim标签
     * @param brand
     * @param guidePrice
     * @param carType
     * @return
     */
List<Car> selectByMutiConditionWithTrim(@Param("brand")String brand, @Param("guidePrice")Double guidePrice, @Param("carType") String carType);
```

CarMapper.xml

```xml
<select id="selectByMutiConditionWithTrim" resultType="car">
  select * from t_car
  <trim prefix="where" suffixOverrides="and">
    <if test="brand != null and brand != ''">
      brand like "%"#{brand}"%" and
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      guide_price > #{guidePrice} and
    </if>
    <if test="carType != null and carType != ''">
      car_type = #{carType}
    </if>
  </trim>
</select>
```



### 4）、set标签

使用在update语句当中，用来生成set关键字，同时去掉最后多余的","



CarMapper.xml

```xml
<update id="updateBySet">
  update t_car
  <set>
    <if test="carNum != null and carNum != ''">car_num = #{carNum},</if>
    <if test="brand != null and brand != ''">brand = #{brand},</if>
    <if test="guidePrice != null and guidePrice != ''">guide_price = #{guidePrice},</if>
    <if test="produceTime != null and produceTime != ''">produce_time = #{produceTime},</if>
    <if test="carType != null and carType != ''">car_type = #{carType}</if>
  </set>
  where
 	 id = #{id}
</update>
```

test.java

```java
@Test
public void testUpdateBySet(){
  SqlSession sqlSession = SqlSessionUtil.openSession();
  CarMapper mapper = sqlSession.getMapper(CarMapper.class);
  Car car = new Car(10L,null,null,null,null,"新能源");
  int count = mapper.updateBySet(car);
  System.out.println(count);
  sqlSession.commit();
  sqlSession.close();
}
```

* 运行日志
  * Preparing: `update t_car SET car_type = ? where id = ?`
  * Parameters: 新能源(String), 10(Long)



### 5）、choose  when  otherwise标签

* 只有一个标签会被选择
  * when：相当于if或else if
  * otherwise：相当于else



CarMapper.xml

```xml
<select id="selectByChoose" resultType="car">
  select * from t_car
  <where>
    <choose>
      <when test="brand != null and brand != ''">
        brand = #{brand}
      </when>
      <when test="guidePrice != null and guidePrice != ''">
        guide_price > #{guidePrice}
      </when>
      <otherwise>
        car_type like "%"#{carType}"%"
      </otherwise>
    </choose>
  </where>
</select>
```



### 6）、foreach标签

* foreach标签属性
  * collection：指定数组或集合
  * item：代表数组或集合中的元素。
  * separator：循环之间的分隔符
  * open：foreach循环拼接的所有sql语句的最前面以什么开始
  * close：foreach循环拼接的所有sql语句的最后面以什么结束

####①、批量删除

* in
  * CarMapper.xml

```xml
<delete id="deleteByForeach">
  delete from t_car where id in
  <foreach collection="ids" item="id" separator="," open="(" close=")">
    #{id}
  </foreach>
</delete>
```

* or

  * CarMapper.xml

  * ```xml
    <delete id="deleteByIds2">
            delete
            from t_car
             <where>
                <foreach collection="ids" item="id" separator="or">
                    id=#{id}
                </foreach>
             </where>
        </delete>
    ```

  * 分隔符改为or





#### ②、批量添加

CarMapper.xml

```xml
<insert id="insertBatch">
  insert into t_car values
  <foreach collection="cars" separator="," item="car">
    (null,#{car.carNum},#{car.brand},#{car.guidePrice},#{car.produceTime},#{car.carType})
  </foreach>
</insert>
```

test.java

```java
@Test
public void testInsertBatch(){
  SqlSession sqlSession = SqlSessionUtil.openSession();
  CarMapper mapper = sqlSession.getMapper(CarMapper.class);
  Car car1 = new Car(null,"1222","卡宴1",80.0,"2019-01-19","燃油车");
  Car car2 = new Car(null,"1223","卡宴2",80.0,"2019-01-19","燃油车");
  Car car3 = new Car(null,"1224","卡宴3",80.0,"2019-01-19","燃油车");
  List<Car> cars = new ArrayList<>();
  cars.add(car1);
  cars.add(car2);
  cars.add(car3);
  int count = mapper.insertBatch(cars);
  System.out.println(count);
  sqlSession.commit();
  sqlSession.close();
}
```



* 执行日志
  * Preparing: insert into t_car values (null,?,?,?,?,?) , (null,?,?,?,?,?) , (null,?,?,?,?,?)
  * Parameters: 1222(String), 卡宴1(String), 80.0(Double), 2019-01-19(String), 燃油车(String), 1223(String), 卡宴2(String), 80.0(Double), 2019-01-19(String), 燃油车(String), 1224(String), 卡宴3(String), 80.0(Double), 2019-01-19(String), 燃油车(String)



### 7）、sql标签和include标签

* 作用：代码复用，易维护
  * sql标签：声明sql片段
  * include标签：将声明的sql片段包含到某个sql语句当中。

```xml
<!--定义复用sql语句-->
<sql id="selectColumnSql">
  id,
  car_num as carNum,
  brand,
  guide_price as guidePrice,
  produce_time as produceTime,
  car_type as carType
</sql>
```

```xml
<!--引用sql语句-->

<!--引用前-->
<select id="selectById" resultType="car">
  select
  id,
  car_num as carNum,
  brand,
  guide_price as guidePrice,
  produce_time as produceTime,
  car_type as carType
  from
  t_car
  where
  id = #{id}
</select>

<!--引用后-->
<select id="selectById" resultType="car">
  select
  	<include refid="selectColumnSql"/>
  where
  	id = #{id}
</select>
```



## 七、高级映射及延迟加载

### 1）多对一

* 多个学生对应一个班级。
  * 多的一方：Student
  * 一的一方：Clazz
* 分主表和副表
  * 谁在前谁就是主表
    * 多对一：多在前，多就是主表。
    * 一对多：一在前，一就是主表。
* 多对一主表是t_student，那么JVM中的主对象是Student对象。

####①第一种方式：

一条sql语句，级联属性映射。

* StudentMapper.java

  ```java
  /**
       * 根据id查询学生信息。
       * @param sid
       * @return
       */
  Student selectById(Integer sid);
  ```

* StudentMapper.xml

  ```xml
  <resultMap id="studentResultMap" type="Student">
    <id property="sid" column="sid"/>
    <result property="sname" column="sname"/>
    <result property="clazz.cid" column="cid"/>
    <result property="clazz.cname" column="cname"/>
  </resultMap>

  <select id="selectById" resultMap="studentResultMap">
    select
   	s.sid,s.sname,c.cid,c.cname
    from
    	t_stu s left join t_clazz c on s.cid = c.cid
    where
    	s.sid = #{sid}
  </select>
  ```

  ​

#### ②第二种方式

* 一条语句，采用association进行关联。

  * association：一个对象关联另一个对象。
  * property：提供要映射的pojo类的属性名。
  * javaType：用来要映射的java类型。

* StudentMapper.java

  ```java
  /**
       * 一条SQL语句，使用association
       * @param sid
       * @return
       */
  Student selectByIdAssociation(Integer sid);
  ```

* StudentMapper.xml

  ```xml
  <resultMap id="studentResultMapAssociation" type="student">
    <id property="sid" column="sid"/>
    <result property="sname" column="sname"/>
    <association property="clazz" javaType="Clazz">
      <id property="cid" column="cid"/>
      <result property="cname" column="cname"/>
    </association>
  </resultMap>

  <select id="selectByIdAssociation" resultMap="studentResultMapAssociation">
    select
    s.sid,s.sname,c.cid,c.cname
    from
    t_stu s left join t_clazz c on s.cid = c.cid
    where
    s.sid = #{sid}
  </select>
  ```

  ​

#### ③第三种方法

* 两条SQL语句，分布查询。

  * 可复用

  * 支持延迟加载/懒加载

    * 延迟加载核心原理：用的时候再执行查询语句，不用的时候不查询。

    * 作用：提高性能，尽可能的不查，或者少查。

    * 开启延迟加载方法：association标签中添加fetchType="lazy"

    * 默认情况下是没有开启延迟加载的。

    * 在association中开启的延迟加载只是局部的设置，只对当前的association关联的sql语句起作用。

    * 可以在mybatis-config.xml中开启**全局延迟加载**机制。

      * 可以在association标签中添加fetchType="eager"来关闭局部的的延迟加载。

      ```xml
      <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
      </settings>
      ```



* 第一步：根据id查询学生信息

  * StudentMapper.java

    ```java
    /**
         * 分布查询第一步；根据id查询学生信息
         * @param id
         * @return
         */
    Student selectByIdStep1(Integer id);
    ```

  * StudentMapper.xml

    * association中的**select属性**填写第二步查找的sql语句id。
    * **cid属性**填写需要传给第二个sql语句的参数

    ```xml
    <resultMap id="studentResultMapStep" type="student">
      <id property="sid" column="sid"/>
      <result property="sid" column="sid"/>
      <association property="clazz"
                   select="com.powernode.mybatis.mapper.ClazzMapper.selectByIdStep2"
                   column="cid"
                   fetchType="lazy"/>
    </resultMap>

    <select id="selectByIdStep1" resultMap="studentResultMapStep">
      select * from t_stu where sid = #{sid}
    </select>
    ```

* 第二步：根据cid查找班级信息

  * ClazzMapper.java

  * ```java
        /**
         * 分布查询第二步，根据cid获取班级信息
         * @param cid
         * @return
         */
        Clazz selectByIdStep2(Integer cid);
    ```

  * ClazzMapper.xml

    ```xml
    <select id="selectByIdStep2" resultType="clazz">
      select * from t_clazz where cid = #{cid}
    </select>
    ```

* 运行日志：

  Preparing: select * from t_stu where sid = ?
  Parameters: 5(Integer)
  Preparing: select * from t_clazz where cid = ?
  Parameters: 1001(Integer)
  ​
  Student{sid=5, sname='钱七', clazz=Clazz{cid=1001, cname='二班'}}



### 2）一对多

* 一个班级对应多个学生：使用集合或数组容纳多个元素。
  * 一是主表
  * 多是副表



#### 方法一：

* **使用collection标签。**

  * property表示pojo类中集合的名称。
  * ofType表示集合中元素的类型。

* ClazzMapper.java

  ```java
  /**
       * 根据cid查询班级学生信息.
       * @param cid
       * @return
       */
  Clazz selectByCollection(Integer cid);
  ```

* ClazzMapper.xml

  ```xml
  <resultMap id="ClazzResultMap" type="clazz">
    <id property="cid" column="cid"/>
    <result property="cname" column="cname"/>
    <collection property="stus" ofType="student">
      <id property="sid" column="sid"/>
      <result property="sname" column="sname"/>
    </collection>
  </resultMap>

  <select id="selectByCollection" resultMap="ClazzResultMap">
    select
    	c.cid,c.cname,s.sid,s.sname
    from
   	t_clazz c left join t_stu s
    on
    	c.cid = s.cid
    where
    	c.cid = #{cid}
  </select>
  ```



#### 方法二：

* 分布查询

* 第一步：通过cid查询班级信息

  * ClazzMapper.java

    ```java
    /**
         * 分布查询第一步，查询班级cid和cname。
         * @param cid
         * @return
         */
    Clazz selectByStep1(Integer cid);
    ```

  * ClazzMapper.xml

    ```xml
    <resultMap id="clazzResultMapStep" type="clazz">
      <id property="cid" column="cid"/>
      <result property="cname" column="cname"/>
      <collection property="stus"
                  select="com.powernode.mybatis.mapper.StudentMapper.selectByCidStep2"
                  column="cid"/>
    </resultMap>

    <select id="selectByStep1" resultMap="clazzResultMapStep">
      select cid,cname from t_clazz where cid = #{cid}
    </select>
    ```

* 第二步：将cid传入第二步sql语句中。

  * StudentMapper.java

    ```java
    /**
         * 一对多分布查询第二步。
         * @param cid
         * @return
         */
    List<Student> selectByCidStep2(Integer cid);
    ```

  * StudentMapper.xml

    ```xml
    <select id="selectByCidStep2" resultType="student">
      select * from t_stu where cid = #{cid}
    </select>
    ```

    ​



### 3）多对多

分成多个**一对多**。



## 八、缓存

### 1）介绍

缓存（cache）：提前把数据放到缓存中（内存中），下一次用的时候，直接从缓存中拿。

* 计算机缓存方式：
  * 内存：临时存储数据的空间，断电后消失。
  * 硬盘：持久化的数据在硬盘当中，文件也是存储在硬盘中的，是一个持久化设备。
* MyBatis的缓存机制：
  * 执行DQL（select语句）的时候，将查询结果放到缓存（内存）中，如果下一次还是执行完全相同的dql语句，直接从缓存中拿数据。不再查数据库了，不再去硬盘上找数据了。
  * 目的：提高执行效率
  * 原理：减少IO（读写文件）的方式来提高效率。
  * ※缓存只针对DQL语句，即只对应select语句。
* mybatis缓存包括：
  * 一级缓存：将查询到的数据存储到SqlSession中。
  * 二级缓存：将查询到的数据存储到SqlSessionFactory中。
  * 集成第三方缓存：如EhCache【java语言开发的】，Memcache【c语言开发的】。。




### 2）一级缓存

* 一级缓存默认是开启的，不需要做任何配置。
* 只要使用同一个SqlSession对象执行同一条Sql语句，就会走 缓存。



* 什么时候不走缓存？
  * SqlSession对象不是同一个时。
  * 查询条件不一样时。
* 什么时候一级缓存失效？
  * 执行了SqlSession的clearCache()方法，手动清空缓存。
  * 执行了insert或delete或update语句，不管操作哪张表，都会清空一级缓存。



### 3）二级缓存

* 二级缓存的范围是：SqlSessionFactory
* 失效：只要两次查询之间出现了增删改操作，二级缓存就会失效。


* 使用二级缓存需要具备的条件：
  * `<setting name="cacheEnabled" value="true"> ` 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存，默认就是true，无需设置。
  * 在需要使用二级缓存的SqlMapper.xml文件中**添加配置:** `<cache/>`
  * 使用二级缓存的实体类对象必须是可序列化的，也就是必须**实现java.io.Serializable接口**。
  * **SqlSession对象关闭或提交**之后，一级缓存中的数据才会被写入到二级缓存中，此时二级缓存才可用。
* cache标签中的属性
  * eviction：指定从缓存中移除某个对象的淘汰算法，默认采用LRU策略。
    *   a. LRU：Least Recently Used。最近最少使用。优先淘汰在间隔时间内使用频率最低的对象。(其实还有一种淘汰算法LFU，最不常用。)

    *   b. FIFO：First In First Out。一种先进先出的数据缓存器。先进入二级缓存的对象最先被淘汰。

    *   c. SOFT：软引用。淘汰软引用指向的对象。具体算法和JVM的垃圾回收算法有关。

    *   d. WEAK：弱引用。淘汰弱引用指向的对象。具体算法和JVM的垃圾回收算法有关。
  * flushInterval：二级缓存的刷新时间间隔。单位毫秒，如果没有设置，就代表不刷新缓存，只要内存足够大，一直会向二级缓存中缓存数据。除非执行了增删改。
  * readOnly：
    * true：  多条相同的sql语句执行之后返回的对象是共享的同一个。性能好。但是多线程并发可能会存在安全问题。
    * false：多条相同的sql语句执行之后返回的对象是副本，调用了clone方法。性能一般。但安全。
  * size：设置二级缓存中最多可存储的java对象数量，默认值1024.



* 未使用二级缓存

  ```java
  @Test
  public void testSelectById2() throws IOException {
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
    SqlSession sqlSession1 = sqlSessionFactory.openSession();
    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    CarMapper mapper1 = sqlSession1.getMapper(CarMapper.class);
    CarMapper mapper2 = sqlSession2.getMapper(CarMapper.class);
    
    //这行代码执行结束之后，实际上数据缓存到一级缓存当中了，（sqlSession是一级缓存）
    Car car1 = mapper1.selectById2(19L);
    System.out.println(car1);
    
    //这行代码执行结束之后，实际上数据缓存到一级缓存当中了，（sqlSession是一级缓存）
    Car car2 = mapper2.selectById2(19L);
    System.out.println(car2);
    
    //程序执行到这里的时候，会将sqlSession1这个一级缓存的数据写入到二级缓存当中。
    sqlSession1.close();
    //程序执行到这里的时候，会将sqlSession2这个一级缓存的数据写入到二级缓存当中。
    sqlSession2.close();
  }
  ```

* 使用二级缓存

  ```java
  @Test
  public void testSelectById2() throws IOException {
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
    SqlSession sqlSession1 = sqlSessionFactory.openSession();
    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    CarMapper mapper1 = sqlSession1.getMapper(CarMapper.class);
    CarMapper mapper2 = sqlSession2.getMapper(CarMapper.class);

    //这行代码执行结束之后，实际上数据缓存到一级缓存当中了，（sqlSession是一级缓存）
    Car car1 = mapper1.selectById2(19L);
    System.out.println(car1);
    //程序执行到这里的时候，会将sqlSession1这个一级缓存的数据写入到二级缓存当中。
    sqlSession1.close();

    //这行代码执行结束之后，实际上数据缓存到一级缓存当中了，（sqlSession是一级缓存）
    Car car2 = mapper2.selectById2(19L);
    System.out.println(car2);
    //程序执行到这里的时候，会将sqlSession2这个一级缓存的数据写入到二级缓存当中。
    sqlSession2.close();
  }
  ```

  ​


### 4）集成EhCache

* 集成EhCache是为了代替mybatis自带的二级缓存，一级缓存是无法替代的。

* 步骤：

  * **第一步：引入mybatis整合的ehcache的依赖。**

    ```xml
    <!--mybatis集成ehcache的组件-->
    <dependency>
      <groupId>org.mybatis.caches</groupId>
      <artifactId>mybatis-ehcache</artifactId>
      <version>1.2.2</version>
    </dependency>
    ```

  * **第二步：在类的根路径下新建ehcache.xml文件，并提供以下配置信息**

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
             updateCheck="false">
        <!--磁盘存储:将缓存中暂时不使用的对象,转移到硬盘,类似于Windows系统的虚拟内存-->
        <diskStore path="e:/ehcache"/>
      
        <!--defaultCache：默认的管理策略-->
        <!--eternal：设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断-->
        <!--maxElementsInMemory：在内存中缓存的element的最大数目-->
        <!--overflowToDisk：如果内存中数据超过内存限制，是否要缓存到磁盘上-->
        <!--diskPersistent：是否在磁盘上持久化。指重启jvm后，数据是否有效。默认为false-->
        <!--timeToIdleSeconds：对象空闲时间(单位：秒)，指对象在多长时间没有被访问就会失效。只对eternal为false的有效。默认值0，表示一直可以访问-->
        <!--timeToLiveSeconds：对象存活时间(单位：秒)，指对象从创建到失效所需要的时间。只对eternal为false的有效。默认值0，表示一直可以访问-->
        <!--memoryStoreEvictionPolicy：缓存的3 种清空策略-->
        <!--FIFO：first in first out (先进先出)-->
        <!--LFU：Less Frequently Used (最少使用).意思是一直以来最少被使用的。缓存的元素有一个hit 属性，hit 值最小的将会被清出缓存-->
        <!--LRU：Least Recently Used(最近最少使用). (ehcache 默认值).缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存-->
        <defaultCache eternal="false" maxElementsInMemory="1000" overflowToDisk="false" diskPersistent="false"
                      timeToIdleSeconds="0" timeToLiveSeconds="600" memoryStoreEvictionPolicy="LRU"/>

    </ehcache>
    ```

  * **第三步：修改SqlMapper.xml文件中的 `<cache/>` 标签，添加type属性。**

    ```xml
    <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
    ```

    ​





## 九、MyBatis逆向工程

### 1）介绍

* 逆向工程：根据数据库表逆向生成Java的pojo类，SqlMapper.xml文件，以及Mapper接口类等。
* 需要提供的信息：
  * pojo类名、包名、生成位置。
  * SqlMapper.xml文件名以及生成位置。
  * Mapper接口名以及生成位置。
  * 连接数据库的信息。
  * 指定哪些表参与逆向工程。



### 2）使用步骤 

* 第一步：在pom.xml文件中添加逆向工程插件。

  ```xml
  <!--定制构建过程-->
  <build>
    <!--可配置多个插件-->
    <plugins>
      <!--其中的一个插件：mybatis逆向工程插件-->
      <plugin>
        <!--插件的GAV坐标-->
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.4.1</version>
        <!--允许覆盖-->
        <configuration>
          <overwrite>true</overwrite>
        </configuration>
        <!--插件的依赖-->
        <dependencies>
          <!--mysql驱动依赖-->
          <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
          </dependency>
        </dependencies>
      </plugin>
    </plugins>
  </build>
  ```

* 第二步：配置generatorConfig.xml

  * 文件名必须是：generatorConfig.xml
  * 该文件必须放在类的根路径下。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE generatorConfiguration
          PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
          "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

  <generatorConfiguration>
      <!--
          targetRuntime有两个值：
              MyBatis3Simple：生成的是基础版，只有基本的增删改查。
              MyBatis3：生成的是增强版，除了基本的增删改查之外还有复杂的增删改查。
      -->
      <context id="DB2Tables" targetRuntime="MyBatis3">
          <!--防止生成重复代码-->
          <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin"/>
        
          <commentGenerator>
              <!--是否去掉生成日期-->
              <property name="suppressDate" value="true"/>
              <!--是否去除注释-->
              <property name="suppressAllComments" value="true"/>
          </commentGenerator>

          <!--连接数据库信息-->
          <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                          connectionURL="jdbc:mysql://localhost:3306/powernode"
                          userId="root"
                          password="root">
          </jdbcConnection>

          <!-- 生成pojo包名和位置 -->
          <javaModelGenerator targetPackage="com.powernode.mybatis.pojo" targetProject="src/main/java">
              <!--是否开启子包-->
              <property name="enableSubPackages" value="true"/>
              <!--是否去除字段名的前后空白-->
              <property name="trimStrings" value="true"/>
          </javaModelGenerator>

          <!-- 生成SQL映射文件的包名和位置 -->
          <sqlMapGenerator targetPackage="com.powernode.mybatis.mapper" targetProject="src/main/resources">
              <!--是否开启子包-->
              <property name="enableSubPackages" value="true"/>
          </sqlMapGenerator>

          <!-- 生成Mapper接口的包名和位置 -->
          <javaClientGenerator
                  type="xmlMapper"
                  targetPackage="com.powernode.mybatis.mapper"
                  targetProject="src/main/java">
              <property name="enableSubPackages" value="true"/>
          </javaClientGenerator>

          <!-- 表名和对应的实体类名-->
          <table tableName="t_car" domainObjectName="Car"/>

      </context>
  </generatorConfiguration>
  ```

* 第三步：运行插件的generate功能。





* 使用：

  QBC风格：Query By Criteria，一种查询方式，比较面向对象，看不到sql语句。

```java
	@Test
    public void testSelect(){
        SqlSession sqlSession = SqlSessionUtil.openSession();
        CarMapper mapper = sqlSession.getMapper(CarMapper.class);
        Car car = mapper.selectByPrimaryKey(20L);
        System.out.println(car);
		//2.查询所有，查询条件为空
        List<Car> cars = mapper.selectByExample(null);
        cars.forEach(car1 -> System.out.println(car1));
        System.out.println("======================================");

        //3.按照条件查询
        CarExample carExample = new CarExample();
        //使用createCriteria方法添加查询条件。
        carExample.createCriteria()
                        .andBrandLike("卡宴")
                        .andGuidePriceGreaterThan(new BigDecimal(20.0));
        //添加or
        carExample.or().andCarTypeEqualTo("新能源");
        //执行查询
        List<Car> cars2 = mapper.selectByExample(carExample);
        cars2.forEach(car2 -> System.out.println(car2));

        sqlSession.close();
    }
```

* 第二种sql语句：
  * `select id, car_num, brand, guide_price, produce_time, car_type from t_car`
* 第三种sql语句：
  * `select id, car_num, brand, guide_price, produce_time, car_type from t_car WHERE ( brand like ? and guide_price > ? ) or( car_type = ? )`






## 十、PageHelper

### 1）介绍分页机制

* 属性

  * 页码：pageNum（用户发送请求，携带页码pageNum给服务器）
  * 每页显示的记录条数：pageSize

  前端提交表单的数据格式：

  * `uri?pageNum=1&pageSize=10`

* mysql中limit关键字：

  * 格式：`limit 开始下标,  显示的记录条数`
  * 例如：select * from t_car limit 0, 3;
  * 注意：`select * from t_car limit 2;`等效于`select * from t_car limit 0, 2;`
  * mysql第一条记录的下标为0.

* 假设每页显示pageSize条记录：

  那么第pageNum页：**`limit  (pageNum - 1) * pageSize, pageSize`**




### 2）PageHelper使用步骤

* 第一步：pom.xml引入依赖

  ```xml
  <dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.3.1</version>
  </dependency>
  ```

* 第二步：mybatis-config.xml配置插件。

  ```xml
  <plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
  </plugins>
  ```

* 第三步：使用

  ```java
  	@Test
      public void testSelectAll(){
          SqlSession sqlSession = SqlSessionUtil.openSession();
          CarMapper mapper = sqlSession.getMapper(CarMapper.class);

          int pageNum = 1;
          int pageSize = 3;
          //开启分页功能。
          PageHelper.startPage(pageNum,pageSize);
          List<Car> cars = mapper.selectAll();

          cars.forEach(car -> System.out.println(car));

          sqlSession.close();
      }
  ```



* PageInfo对象

  * PageInfo对象是PageHelper提供的，用来封装分页相关的信息的对象。

    ```java
    @Test
        public void testSelectAll(){
            SqlSession sqlSession = SqlSessionUtil.openSession();
            CarMapper mapper = sqlSession.getMapper(CarMapper.class);

            int pageNum = 1;
            int pageSize = 3;
            //开启分页功能。
            PageHelper.startPage(pageNum,pageSize);
            List<Car> cars = mapper.selectAll();

            PageInfo<Car> carPageInfo = new PageInfo<>(cars,3);
            System.out.println(carPageInfo);
            //cars.forEach(car -> System.out.println(car));

            sqlSession.close();
        }
    ```

  **输出结果：**

  PageInfo{pageNum=1, pageSize=3, size=3, startRow=1, endRow=3, total=17, pages=6, 

  list=Page{count=true, pageNum=1, pageSize=3, startRow=0, endRow=3, total=17, pages=6, reasonable=false, pageSizeZero=false}

  [Car{id=20, carNum='1224', brand='卡宴3', guidePrice=80.0, produceTime='2019-01-19', carType='燃油车'}, Car{id=21, carNum='2412', brand='帕拉梅拉', guidePrice=90.0, produceTime='2023-6-12', carType='新能源'}, Car{id=22, carNum='1250', brand='蔚来0', guidePrice=30.4, produceTime='2021-12-13', carType='纯电动'}], 

  prePage=0, nextPage=2, isFirstPage=true, isLastPage=false, hasPreviousPage=false, hasNextPage=true, navigatePages=3, navigateFirstPage=1, navigateLastPage=3, navigatepageNums=[1, 2, 3]}





## 十一、MyBatis注解式开发

### 1）介绍

使用注解来映射简单语句会使代码更加简洁，但对于稍微复杂一点的语句，Java注解不仅力不从心，还会让你本就复杂的sql语句更加混乱，因此，如果需要做一些复杂的操作，最好用xml来映射语句。



### 2）@Insert

CarMapper.java

```java
@Insert("insert into t_car values (null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})")
int insert(Car car);
```



###3）@Delete

CarMapper.java

```java
@Delete("delete from t_car where id = #{id}")
int deleteById(Integer id);
```





### 4）@Update

CarMapper.java

```java
@Update("update t_car set car_num = #{carNum},brand = #{brand},guide_price = #{guidePrice},produce_time = #{produceTime},car_type = #{carType} where id = #{id}")
int updateById(Car car);
```



### 5）@Select

CarMapper.java

```java
@Select("select * from t_car where id = #{id}")
Car selectById(Integer id);
```



### 6）@Results和@Result

java对象属性与表格属性映射。

```java
@Select("select * from t_car where id = #{id}")
@Results({
  @Result(property = "id", column = "id"),
  @Result(property = "carNum", column = "car_num"),
  @Result(property = "brand", column = "brand"),
  @Result(property = "guidePrice", column = "guide_price"),
  @Result(property = "produceTime", column = "produce_time"),
  @Result(property = "carType", column = "car_type")
})
Car selectById(Integer id);
```





## 十二、Spring6集成mybatis

### 1）步骤：

####第一步：准备数据库表

- - 使用t_act表（账户表）

####第二步：IDEA中创建一个模块，并引入依赖

- - spring-context
  - spring-jdbc
  - mysql驱动
  - mybatis
  - mybatis-spring：**mybatis提供的与spring框架集成的依赖**
  - 德鲁伊连接池
  - junit

- ```xml
      <repositories>
          <!--        spring里程碑版本的仓库-->
          <repository>
              <id>repository.spring.milestone</id>
              <name>Spring Milestone Repository</name>
              <url>https://repo.spring.io/milestone</url>
          </repository>
      </repositories>

      <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>6.1.0-M2</version>
          </dependency>

          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jdbc</artifactId>
              <version>6.0.0-M2</version>
          </dependency>

          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>8.0.30</version>
          </dependency>

          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.5.10</version>
          </dependency>

          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis-spring</artifactId>
              <version>2.0.7</version>
          </dependency>

          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>druid</artifactId>
              <version>1.2.13</version>
          </dependency>

          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.13.2</version>
              <scope>test</scope>
          </dependency>
      </dependencies>
  ```

- ​

####第三步：基于三层架构实现，所以提前创建好所有的包

- - com.powernode.bank.mapper
  - com.powernode.bank.service
  - com.powernode.bank.service.impl
  - com.powernode.bank.pojo

####第四步：编写pojo

- - Account，属性私有化，提供公开的setter getter和toString。

####第五步：编写mapper接口

- - AccountMapper接口，定义方法

####第六步：编写mapper配置文件

- - 在配置文件中配置命名空间，以及每一个方法对应的sql。

####第七步：编写service接口和service接口实现类

- - AccountService
  - AccountServiceImpl

####第八步：编写jdbc.properties配置文件

- - 数据库连接池相关信息

- ```properties
  jdbc.driver=com.mysql.cj.jdbc.Driver
  jdbc.url=jdbc:mysql://localhost:3306/spring6
  jdbc.username=root
  jdbc.password=vanky
  ```

- ​

####第九步：编写mybatis-config.xml配置文件

- - 该文件可以没有，大部分的配置可以转移到spring配置文件中。
  - 如果遇到mybatis相关的系统级配置，还是需要这个文件。

- ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>

      <settings>
          <setting name="logImpl" value="STDOUT_LOGGING"/>
      </settings>

  </configuration>
  ```

- ​

####第十步：编写spring.xml配置文件

- - 组件扫描
  - 引入外部的属性文件
  - 数据源
  - SqlSessionFactoryBean配置


- - - 注入mybatis核心配置文件路径
    - 指定别名包
    - 注入数据源


- - Mapper扫描配置器


- - - 指定扫描的包


- - 事务管理器DataSourceTransactionManager


- - - 注入数据源


- - 启用事务注解


- - - 注入事务管理器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <context:component-scan base-package="com.powernode.bank"></context:component-scan>

    <context:property-placeholder location="jdbc.properties"></context:property-placeholder>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--配置SqlSessionFactoryBean-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
        <!--指定mybatis核心配置文件-->
        <property name="configLocation" value="mybatis-config.xml"/>
        <!--指定别名-->
        <property name="typeAliasesPackage" value="com.powernode.bank.pojo"/>
    </bean>

    <!--Mapper扫描配置器，主要扫描Mapper接口，生成代理类-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.powernode.bank.mapper"/>
    </bean>

    <!--事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--启用事务注解-->
    <tx:annotation-driven transaction-manager="txManager"/>

</beans>

```



####第十一步：编写测试程序，并添加事务，进行测试
































































































































