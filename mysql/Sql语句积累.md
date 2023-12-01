# Sql语句积累

**一、我现在mysql表student中，有两个字端，语文分数chinese和数学分数math，我想给这个表按数学成绩排序，但是数学分数可能是空的，如果数学成绩是空，这个表就用语文成绩代替，这么写这个sql？**

你可以使用MySQL中的`ORDER BY`语句来按照数学成绩排序，同时使用`COALESCE`函数处理空值。`COALESCE`函数返回参数列表中的第一个非空表达式，因此你可以将它用于处理数学分数为空的情况，如果数学分数为空，就使用语文分数代替。以下是一个示例 SQL 查询：

```sql
SELECT *
FROM student
ORDER BY COALESCE(math, chinese) DESC;
```

在这个例子中，`COALESCE(math, chinese)`会返回`math`列的值，如果`math`列为空，则返回`chinese`列的值。然后，使用`ORDER BY`按照这个计算出的值进行降序排序。这样就可以实现按照数学成绩排序，如果数学成绩为空，就使用语文成绩代替。

```java
//lambdaQueryWrapper条件方法
lambdaQueryWrapper.orderByDesc(Student::getMath).orderByDesc(Student::getChinese);
```

> 需要注意的是，这里我们没有使用 COALESCE 函数，而是直接对 `math` 和 `chinese` 字段进行排序。这是因为在数据库中，如果某字段为空，在排序时会被当做最小值处理，因此不必显式使用 COALESCE 函数。如果确实需要使用 COALESCE 函数，你可以通过数据库函数来实现，而不是在 MyBatis-Plus 中进行。





**二、我现在mysql有两个表，第一个表是score表，有id，book_id，score三个字段，第二个表是book表，有id字段。我现在想先在第一个表中查询score小于90的相同book_id对应的数据库表记录数量，然后按这个数量排序，排序后按照book_id查另外一张book表id等于book_id的记录，根据这个排序来输出book的所有字段**

你可以使用以下 SQL 查询语句来实现你的需求：

```sql
SELECT b.*
FROM book b
JOIN (
    SELECT book_id, COUNT(*) AS score_count
    FROM score
    WHERE score < 90
    GROUP BY book_id
    ORDER BY score_count DESC
) s ON b.id = s.book_id
ORDER BY s.score_count DESC, b.id;
```

这个查询的步骤如下：

1. 内部查询（子查询）：使用 `SELECT book_id, COUNT(*) AS score_count` 统计 `score` 表中 `score` 小于 90 的记录的数量，并按照数量降序排序。
2. 使用 `JOIN` 连接 `book` 表和内部查询的结果，连接条件是 `b.id = s.book_id`。
3. 外部查询：选择 `book` 表中的所有字段。
4. 使用 `ORDER BY` 按照内部查询中的 `score_count` 降序排序，然后按照 `book_id` 升序排序。

这样你就可以得到按照 `score` 小于 90 的记录数量排序后，再按照 `book_id` 的顺序输出 `book` 表中的所有字段。

> 在 MyBatis-Plus 中，你可以使用 `LambdaQueryWrapper` 来构建与你提供的 SQL 查询相等的查询条件。以下是如何使用 `LambdaQueryWrapper` 表示你的 SQL 查询的示例：
>
> ```java
> import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
> import com.baomidou.mybatisplus.core.toolkit.Wrappers;
>
> // 先构建内部查询的 LambdaQueryWrapper
> LambdaQueryWrapper<Score> innerWrapper = Wrappers.lambdaQuery();
> innerWrapper.select(Score::getBookId)
>         .lt(Score::getScore, 90)
>         .groupBy(Score::getBookId)
>         .orderByDesc(Wrappers.lambdaQuery(Score::getBookId).count().desc());
>
> // 外部查询的 LambdaQueryWrapper
> LambdaQueryWrapper<Book> outerWrapper = Wrappers.lambdaQuery();
> outerWrapper.in(Book::getId, innerWrapper);
>
> // 最终查询
> List<Book> result = bookMapper.selectList(outerWrapper);
> ```
>
> 上述代码中的 `Score` 和 `Book` 分别是你的两个实体类，`Score` 对应 `score` 表，`Book` 对应 `book` 表。这里使用了 MyBatis-Plus 的 `Wrappers` 工具类来创建 `LambdaQueryWrapper`。
>
> 关键点包括：
>
> 1. 使用 `Wrappers.lambdaQuery()` 创建 `innerWrapper`，设置条件 `lt(Score::getScore, 90)` 来筛选 `score` 小于 90 的记录，使用 `groupBy` 和 `orderByDesc` 设置分组和排序条件。
>
> 2. 使用 `Wrappers.lambdaQuery()` 创建 `outerWrapper`，通过 `in` 设置外部查询的条件，将 `Book` 表的 `id` 匹配内部查询的结果。
>
> 3. 执行 `bookMapper.selectList(outerWrapper)` 来执行查询，获取最终结果。
>
> 请注意，这只是一个示例，实际使用时你需要根据具体的实体类和字段名进行调整。



三、**如果 `latest_comment_time` 列的某些记录可能包含空值（`NULL`），并且你仍希望对这些记录进行排序，你可以使用 `COALESCE()` 函数或 `IFNULL()` 函数来处理 `NULL` 值。**

**这两个函数的作用是，如果第一个表达式的值为 `NULL`，则返回第二个表达式的值。你可以将这两个函数应用到 `ORDER BY` 子句中，以确保在排序时 `NULL` 值被处理得当。**



```sql
SELECT *
FROM your_table
ORDER BY COALESCE(latest_comment_time, create_time) DESC;
```



```sql
SELECT *
FROM your_table
ORDER BY IFNULL(latest_comment_time, create_time) DESC;
```





































