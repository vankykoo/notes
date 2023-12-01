# MySQL

## 一、MySQL的安装和配置

###连接MySQL数据库

①【mysql -h 主机名 -P 端口 -u 用户名 -p密码】

②登录前要保证服务启动

③【net stop mysql】退出mysql	【net start mysql】启动mysql

注意：

①-p密码不用有空格

②如果没有写-h 主机，默认就是本机

③如果没有写-P 端口，默认就是3306

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/%E7%AC%94%E8%AE%B0/11/22/MySQL%E4%B8%89%E5%B1%82%E7%BB%93%E6%9E%84.png)

## 二、数据库

###1）基本介绍

①安装MySQL数据库，就是在主机安装一个数据库管理系统（DBMS），这个管理程序可以管理多个数据库。DBMS（database manage system）

②一个数据库中可以创建多个表，以保存数据（信息）。

###2）数据在数据库中的存储方式：

表的一行称之为一条记录->在Java程序中，一行记录往往使用对象表示。

###3）SQL语句分类

①DDL：数据定义语句（create 表，库...）

②DML：数据操作语句（增加insert，修改updat，删除delete）

③DQL：数据查询语句（select）

④DCL：数据控制语句（管理数据库：比如用户权限 grant revoke）



###4）创建数据库

```mysql
#创建数据库
CREATE DATABASE [IF NOT EXISTS] db_name;
#创建数据库，并设置字符集【CHARACTER SET】，如果不指定，默认为utf8
CREATE DATABASE [IF NOT EXISTS] db_name CHARACTER SET utf8;
#创建数据库，并设置字符集和字符集的校对规则【COLLATE】，默认utf8_general_ci
CREATE DATABASE [IF NOT EXISTS] db_name CHARACTER SET utf8 COLLATE utf8_bin;

#使用数据库
use 数据库名;
```

※常用的校对规则

①【utf8_bin】：区分大小写

②【utf8_general_ci】：不区分大小写



###5）查看和删除数据库

```mysql
#显示数据库语句
SHOW DATABASES
#显示数据库创建语句
SHOW CREATE DATABASE db_name
#数据库删除语句
DROP DATABASE [IF EXISTS] db_name
```



###6）备份恢复数据库

```mysql
#备份数据库（在DOS执行）命令行
#【mysqldump -u 用户名 -p -B 数据库1 数据库2 数据库n > 文件名.sql】

#恢复数据库（进入mysql命令行再执行）
#【Source 文件名.sql】

#备份数据库的表
#【mysqldump -u 用户名 -p密码 数据库 表1 表2 表n > d:\\文件名.sql】
```



## 三、表

###1）创建表

```mysql
CREATE TABLE table_name(
  field1 datatype,
  field2 datatype,
  field3 datatype,
)CHARACTER SET 字符集 COLLATE 校对规则 ENGINE 存储引擎
```

**field**：指定列名	**datatype**：指定列类型（字段类型）

**CHARACTER SET**：如果不指定则为所在数据库字符集

**COLLATE**：如不指定则为所在数据库校对规则

**ENGINE**：设置引擎，默认为INNODB



###2）修改表

```mysql
#添加列/字段
ALTER TABLE t1 ADD image VARCHAR(32) NOT NULL DEFAULT '';
	-- 解释：在表t1中加入image列，不能为空
	
#显示表结构
DESC t1;
	-- 解释：显示表t1的列组成结构
	
#修改列，使其长度改为60
ALTER TABLE emp MODIFY job VARCHAR(60) NOT NULL DEFAULT '';
	-- 解释：修改表emp中job列的长度为60
	
#删除列
ALTER TABLE emp DROP sex;
	-- 解释：删除emp表中的sex列
	
#修改表名
ALTER TABLE emp TO employee;
	-- 解释：将表emp的名字改为employee
	
#修改字符集
ALTER TABLE employee CHARACTER SET utf8;
	-- 解释：将表employee的字符集改为utf8
	
#修改列名
ALTER TABLE employee CHANGE `name` `user_name` CARCHAR(64) NOT NULL DEFAULT '';
	-- 解释：将表employee中name列名字改为user_name，不能为空
	
#删除表
DROP TABLE table_name;
```



## 四、MySQL数据类型

###1）数据类型/列类型

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)

###2）整型

```mysql
#默认是有符号的
CREATE TABLE t1 (
  id TINYINT
);
#无符号
CREATE TABLE t2	(
  if TINYINT UNSIGNED
);
```

※在能够满足需求的情况下，尽量选择占用空间小的类型。



###3）bit类型

[1]使用方法

```mysql
#bit(m)  ---> m属于1-64直接
mysql>CREATE TABLE t5 (num bit(8));
mysql>INSERT INTO t5 values(255);#->  b'11111111'
```

[2]使用细节

①bit字段显示时，按照位的方式显示。

②查询的时候仍然可以用添加的数值查询

③如果一个值只有0和1，可以考虑使用bit(1)，可以节约空间

④位类型，M指定位数，默认值为1，范围1-64



###4）小数

[1]**FLOAT**/**DOUBLE**[UNSIGNED]：FLOAT是单精度，DOUBLE是双精度。

[2]**DECIMAL**[M,D]\[UNSIGNED]：

①可以支持更加精确的小数位，M是小数位数（精度）的总数，D是小数点（标度）后面的位数。

②如果D是0，则值没有小数点或分数部分。M最大为**65**，D最大是**30**。如果D被省略，默认是**0**。如果M被省略，默认是**10.**

③如果希望小数的精度高，推荐使用**decimal**



###5）字符串

[1]【CHAR(size)】：固定长度字符串，最大**255**字符

[2]【VARCHAR(size)】：0-65535，可变长度字符串，最大**65532**字节，【**utf8**编码最大**21844**字符，有1-3个字节用于**记录大小**】21844 = (65535 - 3) / 3		※减三是因为记录字节大小，除三是因为utf8一个字符占三个字节。

※使用细节

[1]细节一

* char(4)中的4表示**字符数**（最大是255），不是字节数，不管是中文还是字母都是发四个，按字符计算
* varchar(4)中的4表示**字符数**，不管是字母还是中文都以定义好的表的编码来存放数据。

[2]细节二

* char(4)是定长（固定的大小），即使插入’aa‘，也会占用分配的4个字符空间，所以可能会浪费空间资源。
* varchar(4)是变长，如果插入’aa‘，实际占用空间大小并不是4个字符，而是按照实际占用空间来分配（varchar本身还需要占用1-3个字节来记录存放内容长度）

[3]细节三

* 如果数据是定长，推荐使用**char**，比如md5的密码，邮编，手机号等等，查询速度会更快。
* 如果一个字段的长度是不确定的，使用**varchar**比如留言，文章。

[4]细节四

* 再存放文本时，也可以使用**TEXT**数据类型，可以将**TEXT**列视为**VARCHAR**列，注意：**TEXT**不能有默认值，大小**0-2^16**字节
* 如果希望存放更多字符，可以选择**MEDIUMTEXT(0-2^24)**或者**LONGTEXT(0-2^32)**。


###6）日期

①【DATE】：日期（年月日）

②【DATETIME】：日期+时间（年月日+时分秒）

③【TIMESTAMP】：实时日期+时间

※**TIMESTAMP**在**INSERT**和**UPDATE**时会自动更新。

```mysql
CREATE TABLE t1 (
  birthday DATE,
  day_time DATETIME,
  #时间戳后面要加上一段配置
  now_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP);
  
#添加数据,只需添加两个,时间戳自动更新
mysql> INSERT INTO t1 (birthday,day_time) VALUES('2023-2-25','2022-12-12 12:45:23')
```

## 五、CRUD增删改查

###1）添加数据

[1]语法

【**INSERT INTO** table_name (column...)  **VALUES**  (value...)】

```mysql
#添加数据到goods（有id、goods_name、price列）
INSERT INTO `goods` (id,goods_name,price) VALUES(10,'手机',2888);
```

[2]使用细节

①插入的数据应与字段的数据类型相同。

②数据的长度应在列的规定范围内。

③在values中列出的数据位置必须与被加入的列的排列位置相对应。

④字符和日期型数据应包含在单引号中，

⑤列可以插入空值（前提是该字段允许为空）

⑥可以以【**INSERT INTO** tab_name (列名...) **VALUES** (),(),()】形式添加多条记录。

⑦如果是给表中的所有字段添加数据，可以不写前面的字段名称。

⑧默认值的使用：当不给某个字段值时，如果有默认值就会添加默认值，否则报错。如果某个列没有指定NOT NULL，那么当添加数据时，没有给定值就会默认给null。如果我们希望指定某个列的默认值，可以在创建表时指定。

```mysql
CREATE TABLE t1 (
  id TINYINT,
  name VARCHAR(30),
  age INT NOT NULL DEFAULT 18 		-- 默认值为18
);
```



###2）修改数据

[1]使用细节：

①**UPDATE**语法可以用新值更新原有表行中的各列。

②**SET**子句指示要修改哪些列和要给哪些值。

③**WHERE**子句指定应更新哪些行。如没有**WHERE**子句，则更新所有的行（记录），慎用。

④如果需要修改多个字段，可以通过【**SET** 字段1=值1，字段2=值2...】

[2]使用语法

```mysql
#表employee中：
#id:1,name:'mike',job:'waiter',salary:3000
#id:2,name:'jack',job:'teacher',salary:5000

#mike的salary加1000
UPDATE employee SET salary = salary + 1000 WHERE `name` = 'mike';
#mike的salary加1000，工作改为postman
UPDATE employee 
			SET salary = salary + 1000 , job = 'postman' 
			WHERE `name` = 'mike';
```



###3）删除数据

[1]使用语法

【**DELETE FROM** tbl_name **WHERE** where_definition】

```mysql
#删除employee表中user_name为'mike'的记录
DELETE FROM employee WHERE user_name = 'mike';
#删除employee表中所有记录
DELETE FROM employee;
```



[2]使用细节：

①如果不使用**WHERE**子句，将删除表中所以数据。

②**DELETE**不能删除某一列的值（可使用**UPDATE**设为null或者''）

③使用**DELETE**语句仅删除记录，不删除表本身，如要删除表，使用【**DROP TABLE** 表名】



###4）查找SELECT

[1]基本语法

【**SELECT** [**DISTINCT**] * column1,column2,... **FROM** table_name】

①SELECT指定查询哪些列的数据

②column指定列名

③*号代表查询所以列

④FROM指定查询哪张表

⑤DISTINCT可选，指显示结果时，去掉重复数据。

```mysql
#查询employee表中所有记录
SELECT * FROM employee;
#查询employee表中所以记录对应的name和age
SELECT `name`,age FROM employee;
#查询employee表中所以记录对应的name和age，并去除重复记录，要求name和age都相同才去重。
SELECT DISTINCT `name`,age FROM employee;
```

[2]运用2

①使用表达式对查询的列进行运算

【**SELECT** * **column**|expression, column2|expression,... **FROM** table_name】

```mysql
#对三科成绩相加的的数来显示为一列
SELECT `name`,(chinese + english + math) FROM student;
#对三科成绩相加的的数来显示为一列,这列的列名显示为total_score
SELECT `name`,(chinese + english + math) AS total_score FROM student;
```

②在select语句中可使用**as**语句，对列名取别名来显示。

【**SELECT** column_name **as** 别名 **FROM** 表名;】

[3]WHERE子句中经常使用的运算符

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/select%E8%AF%AD%E5%8F%A5.png)

```mysql
#模糊查询LIKE
SELECT * FROM table_name WHERE `name` LIKE '李%';
	-- 解释：查找name中为'李'开头的记录
#BETWEEN...AND...的使用
SELECT * FROM table_name WHERE age BETWEEN 28 AND 35;
	-- 解释：查找age在28到35的记录，包含28和35
#IN的使用
SELECT * FROM table_name WHERE age IN (18,19,20);
	-- 解释：查找age为18，19，20的记录
```

[4]**ORDER BY** 子句排序查询结果

※细节

①**ORDER BY** 指定排序的列，排序的列既可以时表中的列名，也可以是**SELECT**语句后指定的列名。

②**ASC**为升序（默认），**DESC**为降序

③**ORDER BY**子句应位于**SELECT**语句的结尾。

```mysql
#语法
SELECT column1,column2,column3...
		FROM table_name
		ORDER BY column ASC/DESC;
		
#演示
SELECT * FROM student ORDER BY math; -- 按数学成绩升序排序
SELECT `name`,(chinese + english + math) AS total_score
		FROM student
        ORDER BY total_score DESC; -- 按总分从高到低排序（降序）
SELECT `name`,(chinese + english + math) AS total_score
		FROM student
		WHERE `name` LIKE '韩%'
		ORDER BY total_score;	-- 姓韩的学生按总分从低到高排序
```



## 六、函数

###1）合计/统计函数

①**COUNT**：统计行数

```mysql
#基本语法
SELECT COUNT(*/列名) FROM table_name [WHERE where_definition];

#案例演示
SELECT COUNT (*) FROM student WHERE math > 90;
	-- 解释：在student表中统计math分数大于90的记录行数。
#注意：如果*改为列名，则忽视列为null的情况。
```

②**SUM**：求和

```mysql
#基本语法
SELECT SUM(列名),SUM(列名)... FROM table_name [WHERE where_definition];

#案例演示
SELECT SUM(chinese + math + english) 
				FROM student 
				WHERE math > 80;
		-- 解释：查看student表中三科成绩总和且数学分数大于80的值。
```

③**AVG**：求平均

```mysql
#基本语法
SELECT AVG(列名),AVG(列名),... FROM table_name [WHERE where_definition];

#案例演示
SELECT AVG(math + chinese + english) 
			FROM student
			WHERE chinese > 70;
	-- 解释：查看student表中语文成绩大于70的三科成绩平均分
```

④**MAX/MIN**：求最值

```mysql
#基本语法
SELECT MAX/MIN(列名) FROM table_name [WHERE where_definition];

#案例演示
SELECT MAX(math)，MIN(chinese) FROM student;
	-- 解释：查看数学最高分和语文最低分的值。
```

⑤**GROUP BY**和**HAVING**：分组统计

**※GROUP BY**用于对查询的结果分组统计。

**※HAVING**子句用于限制分组显示结果。

```mysql
#基本语法
SELECT column1,column2,... 
		FROM table_name
		GROUP BY column HAVING;

#案例演示：显示平均工资低于2000的部门号和它的平均工资
SELECT AVG(sal),deptno 
                FROM emp 
                GROUP BY deptno HAVING AVG(sal) < 2000;
```



###2）字符串函数

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/MySQL%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%87%BD%E6%95%B0.png)



###3）数学函数

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/mysql_math%E5%87%BD%E6%95%B0.png)



###4）日期函数

**<u>※日期要带单引号！！</u>**

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/date%E5%87%BD%E6%95%B0.png)

```mysql
#UNIX_TIMESTAMP()：返回的是1970-1-1到现在的秒数
SELECT UNIX_TIMESTAMP() FROM DUAL; -- DUAL是亚元表
#FROM_UNIXTIME()：把一个秒数按规定格式转化。
SELECT FROM_UNIXTIME (161848100,'%Y-%m-%d &H:%i:%s') FROM DUAL;

#last_day(日期)---->返回日期月份的最后一天的日期。

#说明：在开发中，可以存放一个整数，然后表示时间，通过FROM_UNIXTIME()转换。
```



###5）加密函数

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/mysql%E5%8A%A0%E5%AF%86%E5%87%BD%E6%95%B0.png)



###6）流程控制函数

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/%E7%AC%94%E8%AE%B0/mysql%E6%B5%81%E7%A8%8B%E6%8E%A7%E5%88%B6%E5%87%BD%E6%95%B0.png)

```mysql
SELECT ename,(CASE 
		WHEN job = 'CLERK' THEN '职员'
		WHEN job = 'MANAGER' THEN '经理'
		WHEN job = 'SALESMAN' THEN '销售人员'
		ELSE job END) AS 'job'
		FROM emp;
SELECT ename,IFNULL(comm,0.0) AS 'comm' FROM emp;
#判断是否为空用IS而不是等号！
SELECT ename,IF(comm IS NULL,0.0,comm) AS 'comm' FROM emp;
```



###7）增强查询

```mysql
#查询ename第二个字符为'O'的所以员工的姓名和工资
SELECT ename,sal FROM emp WHERE ename LIKE '_O%';
#查询表emp的结构
DESC emp;
```

```mysql
#分页查询
#基本语法
SELECT ... 
      FROM ... 
      LIMIT start_,rows_;
	-- 表示从start+1行开始取，取出rows行，start从0开始计算。

#分组增强
#DISTINCT表示去重
SELECT COUNT(DISTINCT mgr) FROM emp;
	-- 避免统计重复mgr个数
```

```mysql
#多子句查询
#如果SELECT语句同时包含有GROUP BY,HAVING,LIMIT,ORDER BY，那么他们的顺序如下：
#ORDER BY在where后面
SELECT column1,column2,column3,... FROM table_name
		GROUP BY column		-- 分组查询
		HAVING condition	-- 限制条件
		ORDER BY column		-- 排序
		LIMIT start,rows;	-- 分页查询
```



###8）多表查询

※介绍：多表查询是指基于两个和两个以上的表查询。【多表查询的条件不能少于**表的个数-1**，否则会出现笛卡尔集。

[1]多表拼接

```mysql
SELECT * FROM emp,dept 
		WHERE emp.deptno = dept.deptno; -- 通过这个条件拼接
```

[2]自连接：在同一张表的连接查询【将同一张表看作两张表】

```mysql
SELECT worker.ename AS '职员名', boss.ename AS '上级名'
		FROM emp worker, emp boss	-- 取别名
		WHERE worker.mgr = boss.empno;	-- 拼接条件
```

[3]子查询

①子查询：指嵌入在其他sql语句中的SELECT语句，也叫嵌套查询

②单行子查询：指只返回一行数据的子查询语句。

③多行子查询：指返回多行数据的子查询。

```mysql
#单行子查询
SELECT * FROM emp
		WHERE deptno = (
        	SELECT deptno
        	FROM emp
        	WHERE ename = 'SMITH'
        )
        -- 解释：在emp表中查询和'SMITH'的deptno相同的员工资料
        
#多行子查询
SELECT ename,job,sal,deptno
		FROM emp
		WHERE job IN (
        	SELECT DISTINCT job
          	FROM emp
          	WHERE deptno = 10
        )AND deptno != 10;
     	-- 解释：查询和部门号为10的工作相同的雇员的名字、岗位、工资、部门号，但是不含10自己 
```

```mysql
#子查询临时表
SELECT goods_id,ecs_goods.cat_id,goods_name,shop_price
	FROM(	-- 把子查询当做一张临时表：每一类中价格最高的商品
      SELECT cat_id,MAX(shop_price) AS max_price
      FROM ecs_goods
      GROUP BY cat_id)
    )temp , ecs_goods		-- 把子查询命名为temp
    WHERE temp.cat_id = ecs_goods.cat_id
    AND temp.max_price = ecs_goods.shop_price;
```

```mysql
#ALL和ANY的使用
#ALL：指子查询中的所有数据
SELECT ename, sal, deptno
	FROM emp
	WHERE sal > ALL(
    	SELECT sal
      	FROM emp
      	WHERE deptno = 30
    )	-- 查找比部门号为30的所有员工的工资高的记录
    
#ANY：指子查询中的其中一个
SELECT ename, sal, deptno
	FROM emp
	WHERE sal > ANY(
    	SELECT sal
      	FROM emp
      	WHERE deptno = 30
    )	-- 查找比部门号为30的所有员工中其中一个员工的工资高的记录（比最低的高就行）
```

```mysql
#多列子查询：指查询返回多个列数据的子查询语句
#查询与ALLEN的部门号和工作（两列）完全相同的所以雇员，且不包含ALLEN本人
SELECT * FROM emp
		WHERE (deptno,job) = (
        	SELECT deptno,job
          	FROM emp
          	WHERE ename = 'ALLEN'
        )AND ename != 'ALLEN';
```

###9）表复制和去重

说明：有时为了对某个sql语句进行效率测试，我们需要海量数据时，可以使用此法为表创建海量数据。

```mysql
#表复制：
#一、把emp 表的记录复制到 my_tab01
INSERT INTO my_tab01 
	(id, `name`, sal, job,deptno)
	SELECT empno, ename, sal, job, deptno FROM emp;
#二、自我复制：每执行一次记录数乘二
INSERT INTO my_tab01
	SELECT * FROM my_tab01;
```

```mysql
#去重：删除一张表中重复的记录
#去除 my_tab02 中重复的记录：
# (1) 先创建一张临时表 my_tmp , 该表的结构和 my_tab02一样
create table my_tmp like my_tab02;
#(2) 把my_tmp 的记录 通过 distinct 关键字 处理后 把记录复制到 my_tmp
insert into my_tmp 
	select distinct * from my_tab02;
#(3) 清除掉 my_tab02 记录
delete from my_tab02;
#(4) 把 my_tmp 表的记录复制到 my_tab02
insert into my_tab02
	select * from my_tmp;
#(5) drop 掉 临时表my_tmp
drop table my_tmp;
```

###10）合并查询

介绍：【UNION ALL】和【UNION】用于合并多个SELECT语句的结果。

```mysql
#【UNION ALL】：合并查询结果，且不会去重
#【UNION】：合并查询结果，去重。
```



## 七、内连接

## 八、外连接

基本介绍：

①左外连接：左侧的表完全显示。

②右外连接：右侧的表完全显示。

```mysql
-- 使用右外连接（显示所有成绩，如果没有名字匹配，显示空)
-- 即：右边的表(exam) 和左表没有匹配的记录，也会把右表的记录显示出来
SELECT `name`, stu.id, grade
	FROM stu RIGHT JOIN exam
	ON stu.id = exam.id;
```

## 九、约束

### 1）PRIMARY KEY：主键

主键【用于唯一的标示表行的数据，当定义主键结束后，该列不能重复】

```mysql
#主键
CREATE TABLE t17
	(id INT PRIMARY KEY, -- 表示id列是主键：要求每个id唯一 
	`name` VARCHAR(32),
	email VARCHAR(32));
	
#复合主键
CREATE TABLE t18
	(id INT , 
	`name` VARCHAR(32), 
	email VARCHAR(32),
	PRIMARY KEY (id, `name`) -- 这里就是复合主键：要求id和name不能同时相同
	);
```

※使用细节：

①PRIMARY KEY不能重复而且不能为null。

②一张表最多只能有一个主键或者一个复合主键。

③主键的指定方式：

```mysql
#直接在字段名后指定
字段名 PRIMARY KEY;
#在表定义最后写
PRIMARY KEY(列名);
```

④使用【**DESC** 表名】，可以看到的**PRIMARY KEY**的情况



###2）NOT NULL：非空

如果在列上定义了NOT NULL,那么当插入数据时，必须为列提供数据。

```mysql
#定义方法
字段名 字段类型 NOT NULL;
```



###3）**UNIQUE**：唯一

※介绍：当定义了唯一约束后，该列值是不能重复的。

```mysql
#定义方法
字段名 字段类型 UNIQUE;
```

※细节：	①如果没有指定**NOT NULL**，则**UNIQUE**字段可以有多个**NULL**。

​		②一张表可以有多个UNIQUE字段。

​		③**UNIQUE NOT NULL**使用效果类似于PRIMARY KEY。



###4）FOREIGN KEY：外键

※介绍：用于定义主表和从表之间的关系：外键约束要定义在从表上，主表则必须具有主键约束或是**UNIQUE**约束，当定义外键约束后，要求外键列数据必须在主表的键列存在或是为**null**。

```mysql
#基本语法：
FOREIGN KEY 本表字段名 REFERENCES 主表名主键名/unique字段名;
#使用：
#主表
CREATE TABLE my_class (
	id INT PRIMARY KEY,
  	`name` VARCHAR(32) NOT NULL DEFAULT '');
 #从表
CREATE TABLE my_stu (
  id INT PRIMARY KEY,
  `name` VARCHAR(32) NOT NULL DEFAULT '',
  class_id INT,
  FOREIGN KEY (class_id) REFERENCES my_class(id));
```

###※细节：

①外键指向的表的字段要求是**PRIMARY KEY** 或者是 **UNIQUE**。

②表的类型是**INNODB**，这样的表才支持外键。

③外键字段的类型要和主键字段的类型一致（长度可以不同）

④外键字段的值，必须在主键字段中出现过，或者为**NULL**（前提是外键字段允许为NULL）

⑤一旦建立主外键的关系，数据就不能随意删除了。



###5）CHECK

※介绍：用于强制行数据必须满足的条件。

※Oracle和sql server均支持check，但是mysql5.7目前还不支持check，只做语法校验，但**不会生效**。

```mysql
CREATE TABLE t1 (
	id INT PRIMARY KEY,
  	`name` VARCHAR(32),
  	sex VARCHAR(6) CHECK (sex IN ('man','woman')),
  	sal DOUBLE CHECK (sal > 1000 AND sal < 2000));
```



###6）自增长

介绍：每次添加一个元素，会自动增加1.

```mysql
#演示
ALTER TABLE t1 AUTO_INCREMENT = 100; -- 可以指定自增长起始值
CREATE TABLE t1 (
  	id INT PRIMARY KEY AUTO_INCREMENT,
  	email VARCHAR(32) NOT NULL DEFAULT '',
  	`name` VARCHAR(32) NOT NULL DEFAULT '');
  	
#三种添加方式：  	
 INSERT INTO t1 VALUES (NULL,'tom@qq.com','tom');
 INSERT INTO t1 (email,`name`) VALUES ('jack@qq.com','jack');
 INSERT INTO t1 (id,email,`name`) VALUES (NULL,'mary@qq.com','mary');
```

※使用细节：

①一般来说自增长是和PRIMARY KEY 配合使用的。

②自增长也可以单独使用，需要配合一个UNIQUE。

③自增长修饰的字段为整数型的，可以小数，但是很少使用。

④自增长默认从1开始，也可以通过命令修改开始值。

⑤如果添加数据时，给自增长字段指定的值，则以指定的值为准，如果指定了自增长，一般来说，就按照自增长的规则来添加数据。

## 十、索引

###1）介绍

[1]原理：形成一个索引的数据结构，比如二叉树。

[2]索引的代价：	①磁盘占用

​				②对UPDATE,DELETE,INSERT语句的效率有影响。



###2）索引的类型

①主键索引，主键自动地为主索引（PRIMARY KEY）

②唯一索引（UNIQUE）

③普通索引（INDEX）

④全文索引（FULLTEXT）【适用于MyISAM】

​	※一般开发中，不使用mysql自带的全文索引，而是使用 全文搜索Solr和ElasticSearch（ES）



###3）添加索引

```mysql
#添加UNIQUE索引
CREATE UNIQUE INDEX 索引名 ON 表名 (列名);
#添加普通索引
CREATE INDEX 索引名 ON 表名 (列名);	-- 方式一
ALTER TABLE 表名 ADD INDEX 索引名 (列名);	-- 方式二
#添加主键索引
ALTER TABLE 表名 ADD PRIMARY KEY 索引名 (列名);
```

※如果某列的值是不会重复的，则优先考虑使用UNIQUE索引，否则使用普通索引。



###4）删除和查看

```mysql
#删除索引
DROP INDEX 索引名 ON 表名;
ALTER TABLE 表名 DROP PRIMARY KEY; -- 删除主键索引

#查看索引
SHOW INDEX FROM 表名;	-- 方式1
SHOW INDEXES FROM 表名;	-- 方式2
SHOW KEYS FROM 表名;	-- 方式3
DESC 表名;		-- 方式4
```



###5）使用建议

①较频繁地作为查询条件的字段应该创建索引

②唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件。

③更新非常频繁的字段不适合创建索引，

④不会出现在WHERE子句中的字段不该创建索引。

## 十一、事务

###1）基本介绍：

事务用于保证数据的一致性，它由一组相关的dml（update，insert，delete）语句组成，改组的dml语句要么全部成功，要么全部失败，



###2）事务和锁

当执行实务操作时（dml语句），mysql会在表上加锁，防止其他用户改表的数据。



###3）重要操作

①【start transaction】：开始一个事务

②【savepoint 保存点名】：设置保存点

③【rollback to 保存点名】：回退事务

④【rollback】：回退全部事务

⑤【commit】：提交事务，所有的操作生效，不能回退。



###4）操作说明

①回退事务：

保存点（savepoint）是事务中的点，用于取消部分事务，当结束事务时（commit），会自动地删除该事务所定义的所有保存点，当执行回退事务时，通过指定保存点可以回退到指定的点。

②提交事务：

使用commit语句可以提交事务，当执行了commit语句后，会确认事务的变化、结束事务、删除保存点、释放锁、数据生效。当使用commit语句结束事务后，其他会话（其他连接）将可以查看的事务变化后的新数据，也就是所以数据正式生效。



###5）使用细节

①如果不开始事务，默认情况下，dml操作是自动提交的，不能回滚。

②如果开始一个事务，你没有创建保存点，你可以执行rollback，默认就是回退到事务开始的状态。

③也可以在这个事务中（还没有提交）创建多个保存点。

④可以在事务没有提交前，选择回退到指定保存点。

⑤mysql的事务机制需要innodb的存储引擎才可以使用，myisam不好使。

⑥开始一个事务的两种方式：【start transaction】和【set autocommit = off】



###6）事务隔离级别

> 事物隔离级别**由数据库系统实现**，是数据库系统本身的一个功能  

[1]概念：MySQL隔离级别定义了事务与事务之间的隔离程度。

![四种隔离级别](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/mysql%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB.png)

[2]脏读、不可重复读、幻读

①脏读（dirty read）：脏读指的是**读到了其他事务未提交的数据**，未提交意味着这些数据可能会回滚，也就是可能最终不会存到数据库中，也就是不存在的数据。读到了并一定最终存在的数据，这就是脏读。![脏读](https://img-blog.csdnimg.cn/4b310fcad22941dcb5c2507ca0636a63.jpg)

②不可重复读（nonrepeatable read）：不可重复读指的是在一个事务内，最开始读到的数据和事务结束前的任意时刻读到的同一批数据出现不一致的情况。![在这里插入图片描述](https://img-blog.csdnimg.cn/19376af72b9c4d2c98723b43a866371e.jpg)

③幻读（phantom read）：幻读侧重的方面是某一次的 select 操作得到的结果所表征的数据状态无法支撑后续的业务操作。更为具体一些：select 某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。![幻读](https://img-blog.csdnimg.cn/db111988aea54e58a8bf6c36d3dad69f.jpg)

[3]操作

```mysql
#查看当前会话mysql的隔离级别
SELECT @@tx_isolation;

#查看系统当前隔离级别
SELECT @@global.tx_isolation;

#设置当前会话隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL 隔离级别;

#设置当前系统隔离级别
SET GLOBAL TRANSACTION ISOLATION LEVEL 隔离级别;
```

※mysql默认的事务隔离级别是repeatable read，一般情况下，没有特殊要求，<u>没有必要修改</u>。

※Oracle默认的事务隔离级别是Read committed。

[4]事务的ACID特性：

①原子性

②一致性

③隔离性

④持久性

[5]注意：

```mysql
#使用dos操作数据库：
#第一步：连接数据库（登录）
#第二步：创建/选择数据库
#第三步：开启事务，设置隔离级别
```



## 十二、其他

###1）表类型和存储引擎

[1]基本介绍：

①mysql的表类型由存储引擎（Storage Engines）决定，主要包括MyISAM、innoDB、Memory等。

②MySQL数据表主要支持六种类型，分别是：MyISAM、innoDB、Memory、CSV、ARCHIVE、MRG_MYISAM。

③这六种分为两类，一类是“事务安全型”（transaction-safe），比如innoDB；其余都属于第二类，称为“非事务安全型”（non-transaction-safe），比如myISAM、memory。

[2]基本特点

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/%E4%B8%BB%E8%A6%81%E7%9A%84%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E%E3%80%81%E8%A1%A8%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%89%B9%E7%82%B9.png)

[3]细节说明

①MyISAM不支持事务和外键，但访问速度快，对事物完整性没有要求。

②innoDB存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全，但是比起MyISAM存储引擎，innoDB写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引。

③MEMORY存储引擎使用存在内存中的内容来创建表。每个MEMORY表只实际对应一个磁盘文件。MEMORY类型的表访问非常快，因为它的数据是放在内存中的，并且默认使用hash索引，但是一旦MySQL服务关闭，表中的数据就会丢失，表的结构还在。

[4]如何选择表的存储引擎

①如果不需要事务，处理的只是基本的CRUD操作，那么MyISAM是不二选择，速度快。

②如果需要支持事务，选择INNODB

③MEMORY存储引擎就是将数据存储在内存中，由于没有磁盘IO的等待，速度极快。但由于是内存存储引擎，所做的任何修改在服务器重启后都将消失。



###2）视图

[1]视图介绍

①视图是根据基表（可以是多个基表）来创建的，视图是虚拟的表。

②视图也有列，数据来自基表。

③通过视图可以修改基表的数据。

④基表的改变，也会影响到视图的数据。



[2]操作

```mysql
#创建视图：【CREATE VIEW 视图名 AS SELECT 列名1,... FROM 表名;】
CREATE VIEW emp_view01 AS
		SELECT empno,ename,job,deptno FROM emp;
		
#查看视图数据：【SELECT 列名... FROM 视图名】
SELECT * FROM emp_view01;

#查看视图结构
DESC emp_view01;

#查看视图创建的指令
SHOW CREATE VIEW emp_view01;

#删除视图
DROP VIEW emp_view01;

#修改视图
UPDATE emp_view01
		SET job = 'MANAGER'
		WHERE empno = 7369;
		
#视图中创建视图
CREATE VIEW emp_view02 AS
		SELECT empno,ename FROM emp_view01;
```

[3]使用细节

①创建视图后，到数据库去看，对应视图只有一个视图结构文件（视图名.frm）

②视图的数据变化会影响到基表，基表的数据变化也会影响到视图

③视图中可以再使用视图，数据仍然来自基表。

[4]优点

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/%E8%A7%86%E5%9B%BE%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5.png)



###3）MySQL用户管理

[1]原因

做项目开发时，可以根据不同的开发人员，赋给他和相应的MySQL操作权限，所以MySQL数据库管理人员（root）可以根据需要创建不同用户，赋给相应的权限，供人员使用。

[2]操作

```mysql
#创建用户
CREATE USER 'vanky'@'localhost' IDENTIFIED BY '123456';
	-- 解释：'vanky'用户名，'localhost'登录IP，'123456'密码

#删除用户
DROP USER 'vanky'@'localhost';

#修改自己的密码
SET PASSWORD = PASSWORD('abcdef');
#修改其他人的密码，需要权限
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123123');
```

[3]权限管理操作

基本语法：

【**GRANT** 权限列表 **ON** 库.对象名 **TO** ’用户名‘@’登录位置‘ （**IDENTIFIED BY** ’密码‘）】

说明：

①权限列表用逗号分开，ALL代表所有权限。

②库.对象名：【*.\*】代表本系统中的所有数据库的所有对象（表，视图，存储过程）。

​			【库.*】表示某个数据库中的所有数据对象（表，视图，存储过程）。

③**IDENTIFIED BY**可以省略

如果用户存在，就是修改改用户的密码，如果该用户不存在，就是创建该用户。

④如果权限没有生效，可以执行【**FLUSH PRIVILEGES**;】

```mysql
#给'vanky'用户分配查看表和添加数据的权限
GRANT SELECT,INSERT 
	ON testdb.news
	TO 'vanky'@'localhost';
	
#回收用户在testdb.news的所有权限
#【REVOKE 权限列表 ON 库.对象名 FROM '用户名'@'登录位置';】
REVOKE SELECT,INSERT ON testdb.news FROM 'vanky'@'localhost';
REVOKE ALL ON testdb.news FROM 'vanky'@'localhost';

#删除用户
DROP USER 'vanky'@'localhost';
```

[4]权限

![](https://pic-1317004580.cos.ap-guangzhou.myqcloud.com/MySQL%E4%B8%AD%E7%9A%84%E6%9D%83%E9%99%90.png)

[5]使用细节

①在创建用户的时候，如果不指定host【CREATE USER ‘用户名’】，则为%，%表示所有IP都有连接权限。

②也可以通过【CREATE USER 'XXX'@'192.168.1.%'】 表示xxx用户在192.168.1.*的ip可以登录MySQL。

③在删除用户的时候，如果host不是%，需要明确指定【‘用户’@‘host值’】。























