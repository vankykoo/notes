# Java 8 新特性

## 一、Lambda表达式

###1）介绍

①Lambda表达式也成为闭包，是Java8发布的新特性

②Lambda表达式允许把函数作为一个方法的参数。

※lambda表达式可以避免匿名内部类定义过多，让代码简洁



###2）语法

```java
(parameters) -> expression[表达式]
(parameters) -> statements[语句]
(parameters) ->{ statements; }
```

> - 可省略类型声明：不需要声明形式参数类型
> - 可省略参数括号：一个参数无需定义括号，但多个参数需要定义括号
> - 可省略花括号：如果主体只包含了一个语句就不需要使用花括号
> - 可省略返回关键字：如果主体只包含了一个返回值语句则会自动返回



###3）函数式接口

Functional Interface是指任何接口，如果只包含唯一一个抽象方法，那么它是一个函数式接口，可以通过lambda表达式创建该接口的对象。



###4）演示

①常规情况下

```java
public class LambdaDemo {
	
    public static void main(String[] args) {
    //3.执行计算
        MathOperation addition = new Addition();
        addition.operation(1,2);
    }
}
    //1.定义一个函数式接口方法
interface MathOperation{
    void operation(int a, int b);
}
    //2.定义实现类
class Addition implements MathOperation{
    @Override
    public void operation(int a, int b) {
        System.out.println(a + b);
    }
}
```



②使用lambda表达式

```java
public class LambdaDemo {
    public static void main(String[] args) {
        //2.Lambda简化，只有一块语句，省略接口和方法，只留下语句实现
        MathOperation addition = (int a , int b) -> {
            System.out.println(a + b);
        };
        //3.执行计算
        addition.operation(1, 2);
    }
}
        //1.定义一个函数式接口
interface MathOperation{
    void operation(int a, int b);
}


//也可以化简
public class LambdaDemo {
    public static void main(String[] args) {
        //2.Lambda简化，省略参数类型，代码块只有一句，可以省略花括号
        MathOperation addition = (a , b) -> System.out.println(a + b);
        //3.执行计算
        addition.operation(1,2);
    }
}
        //1.定义一个函数式接口
interface MathOperation{
    void operation(int a, int b);
}
```



## 二、Interface

### 1）在Java8，接口和抽象类有什么区别

* interface和class的区别，主要有：
  * 接口多实现，类单继承
  * 接口的方法是public abstract 修饰，变量是public static final修饰。abstract class可以用其他修饰符
* interface的方法更像是一个拓展插件。而abstract class的方法是要继承的。

interface新增default和static修饰的方法，为了解决接口的修饰与现有的实现不兼容的问题，并不是为了要替代abstract class。在使用上，该用abstract class的地方还是要用abstract class，不要因为interface的新特性而将之替换。



###2）functional interface函数式接口

定义：也称SAM接口，即Single Abstract Method interfaces，有且只有一个抽象方法，但可以有多个非抽象方法的接口。

在Java8中专门有一个包放函数式接口java.util.function，该包下的所有接口都有@FunctionalInterface注解，提供函数式编程。

在其他包中也有函数式接口，其中一些没有@FunctionalInterface注解，但是只要符合函数式接口的定义就是函数式接口，与是否有@FunctionalInterface注解无关，注解只是在编译时起到强制规范定义的作用。其在Lambda表达式中有广泛的应用。





##三、Stream

###1）介绍

java新增了java.util.stream包，它和之前的流大同小异。之前解除最多的是资源流，比如java.io.FileInputStream，通过流把文件从一个地方输入到另一个地方，它只是内容搬运工，对文件内容不做任何CRUD。

Stream依然不存储数据，不同的是它可以检索（Retrieve）和逻辑处理集合数据、包括筛选、排序、统计、计数等。可以想象成是Sql语句。

它的源数据可以是Collection、Array等。由于它的方法参数都是函数式接口类型，所以一般和Lambda配合使用。



### 2）流类型

1、stream串行流

2、parallelStream并行流，可多线程执行。



### 3）常用方法

```java
/**
* 返回一个串行流
*/
default Stream<E> stream()

/**
* 返回一个并行流
*/
default Stream<E> parallelStream()

/**
* 返回T的流
*/
public static<T> Stream<T> of(T t)

/**
* 返回其元素是指定值的顺序流。
*/
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}


/**
* 过滤，返回由与给定predicate匹配的该流的元素组成的流
*/
Stream<T> filter(Predicate<? super T> predicate);

/**
* 此流的所有元素是否与提供的predicate匹配。
*/
boolean allMatch(Predicate<? super T> predicate)

/**
* 此流任意元素是否有与提供的predicate匹配。
*/
boolean anyMatch(Predicate<? super T> predicate);

/**
* 返回一个 Stream的构建器。
*/
public static<T> Builder<T> builder();

/**
* 使用 Collector对此流的元素进行归纳
*/
<R, A> R collect(Collector<? super T, A, R> collector);

/**
 * 返回此流中的元素数。
*/
long count();

/**
* 返回由该流的不同元素（根据 Object.equals(Object) ）组成的流。
*/
Stream<T> distinct();

/**
 * 遍历
*/
void forEach(Consumer<? super T> action);

/**
* 用于获取指定数量的流，截短长度不能超过 maxSize 。
*/
Stream<T> limit(long maxSize);

/**
* 用于映射每个元素到对应的结果
*/
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

/**
* 根据提供的 Comparator进行排序。
*/
Stream<T> sorted(Comparator<? super T> comparator);

/**
* 在丢弃流的第一个 n元素后，返回由该流的 n元素组成的流。
*/
Stream<T> skip(long n);

/**
* 返回一个包含此流的元素的数组。
*/
Object[] toArray();

/**
* 使用提供的 generator函数返回一个包含此流的元素的数组，以分配返回的数组，以及分区执行或调整大小可能需要的任何其他数组。
*/
<A> A[] toArray(IntFunction<A[]> generator);

/**
* 合并流
*/
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
```



### 4）实战

```java
@Test
public void test() {
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
    //返回符合条件的stream
    Stream<String> stringStream = strings.stream().filter(s -> "abc".equals(s));
    //计算流符合条件的流的数量
    long count = stringStream.count();

    //forEach遍历->打印元素
    strings.stream().forEach(System.out::println);

    //limit 获取到1个元素的stream
    Stream<String> limit = strings.stream().limit(1);
    //toArray 比如我们想看这个limitStream里面是什么，比如转换成String[],比如循环
    String[] array = limit.toArray(String[]::new);

    //map 对每个元素进行操作返回新流
    Stream<String> map = strings.stream().map(s -> s + "22");

    //sorted 排序并打印
    strings.stream().sorted().forEach(System.out::println);

    //Collectors collect 把abc放入容器中
    List<String> collect = strings.stream().filter(string -> "abc".equals(string)).collect(Collectors.toList());
    //把list转为string，各元素用，号隔开
    String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(","));

    //对数组的统计，比如用
    List<Integer> number = Arrays.asList(1, 2, 5, 4);

    IntSummaryStatistics statistics = number.stream().mapToInt((x) -> x).summaryStatistics();
    System.out.println("列表中最大的数 : "+statistics.getMax());
    System.out.println("列表中最小的数 : "+statistics.getMin());
    System.out.println("平均数 : "+statistics.getAverage());
    System.out.println("所有数之和 : "+statistics.getSum());

    //concat 合并流
    List<String> strings2 = Arrays.asList("xyz", "jqx");
    Stream.concat(strings2.stream(),strings.stream()).count();

    //注意 一个Stream只能操作一次，不能断开，否则会报错。
    Stream stream = strings.stream();
    //第一次使用
    stream.limit(2);
    //第二次使用
    stream.forEach(System.out::println);
    //报错 java.lang.IllegalStateException: stream has already been operated upon or closed

    //但是可以这样, 连续使用
    stream.limit(2).forEach(System.out::println);
}
```



### 5）延迟执行

在执行返回Stream的方法时，并不立刻执行，而是等返回一个非Stream的方法后才执行。因为拿到Stream并不能直接用，而是需要处理成一个常规类型。这里的Stream可以想象成是二进制流（2个完全不一样的东西），拿到也看不懂。

```java
@Test
public void laziness(){
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
  Stream<Integer> stream = strings.stream().filter(new Predicate() {
      @Override
      public boolean test(Object o) {
        System.out.println("Predicate.test 执行");
        return true;
        }
      });

   System.out.println("count 执行");
   stream.count();
}
/*-------执行结果--------*/
count 执行
Predicate.test 执行
Predicate.test 执行
Predicate.test 执行
Predicate.test 执行
```

按执行顺序应该是先打印4次`Predicate.test 执行`，再打印`count 执行`。实际结果恰恰相反。说明filter中的方法并没有立刻执行，而是等调用`count()`方法之后才执行。



上面都是串行`Stream`的实例。并行`parallelStream`在使用方法上和串行一样。主要区别是`parallelStream`可多线程执行，是基于`ForkJoin`框架实现的，可以了解一些`ForkJoin`框架和`ForkJoinPool`。

```java
//体验一下并行流的多线程执行
@Test
public void parallelStreamTest(){
   List<Integer> numbers = Arrays.asList(1, 2, 5, 4);
   numbers.parallelStream() .forEach(num->System.out.println(Thread.currentThread().getName()+">>"+num));
}
//执行结果
main>>5
ForkJoinPool.commonPool-worker-2>>4
ForkJoinPool.commonPool-worker-11>>1
ForkJoinPool.commonPool-worker-9>>2
```



### 6）小结

从源码的实例中可以总结出一些stream的特点：

* 通过简单的链式编程，使得它可以方便地对遍历处理后的数据进行再处理。
* 方法参数都是函数式接口类型。
* 一个Stream只能操作一次，操作完就关闭了，继续使用这个stream会报错。
* Stream不保存数据，不改变数据源。





## Optional

###1）介绍

在阿里巴巴开发手册关于Optional的介绍中这样写到：

> 防止 NPE，是程序员的基本修养，注意 NPE 产生的场景：
>
> 1） 返回类型为基本数据类型，return 包装数据类型的对象时，自动拆箱有可能产生 NPE。
>
> 反例：public int f() { return Integer 对象}， 如果为 null，自动解箱抛 NPE。
>
> 2） 数据库的查询结果可能为 null。
>
> 3） 集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。
>
> 4） 远程调用返回对象时，一律要求进行空指针判断，防止 NPE。
>
> 5） 对于 Session 中获取的数据，建议进行 NPE 检查，避免空指针。
>
> 6） 级联调用 obj.getA().getB().getC()；一连串调用，易产生 NPE。
>
> 正例：使用 JDK8 的 Optional 类来防止 NPE 问题。

他建议使用`Optional`解决`NPE（java.lang.NullPointerException）`问题，它就是为`NPE`而生的，其中可以包含空值或非空值。下面通过源码逐步揭开Optional的红盖头。

假如有一个`Zoo`类，里面有个属性`Dog`，需求要获取`Dog`的`age`。

```java
class Zoo {
   private Dog dog;
}

class Dog {
   private int age;
}
```

传统解决`NPE`的办法如下：

```java
Zoo zoo = getZoo();
if(zoo != null){
   Dog dog = zoo.getDog();
   if(dog != null){
      int age = dog.getAge();
      System.out.println(age);
   }
}
```

`Optional`实现方式：

```java
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).ifPresent(age ->   System.out.println(age)
);
```





### 2）常用方法

```java
/**
* Common instance for {@code empty()}. 全局EMPTY对象
*/
private static final Optional<?> EMPTY = new Optional<>();

/**
* Optional维护的值
*/
private final T value;

/**
* 如果value是null就返回EMPTY，否则就返回of(T)
*/
public static <T> Optional<T> ofNullable(T value) {
   return value == null ? empty() : of(value);
}
/**
* 返回 EMPTY 对象
*/
public static<T> Optional<T> empty() {
   Optional<T> t = (Optional<T>) EMPTY;
   return t;
}
/**
* 返回Optional对象
*/
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}
/**
* 私有构造方法，给value赋值
*/
private Optional(T value) {
  this.value = Objects.requireNonNull(value);
}
/**
* 所以如果of(T value) 的value是null，会抛出NullPointerException异常，这样貌似就没处理NPE问题
*/
public static <T> T requireNonNull(T obj) {
  if (obj == null)
         throw new NullPointerException();
  return obj;
}
```

`ofNullable`方法和`of`方法唯一区别就是当`value`为null时，`ofNullable`返回的是`EMPTY`，`of`会抛出`NullPointerException`异常。如果需要把`NullPointerException`暴露出来就用`of`，否则用`ofNullable`。



### 3）map()相关方法

```java
/**
* 如果value为null，返回EMPTY，否则返回Optional封装的参数值
*/
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
}
/**
* 如果value为null，返回EMPTY，否则返回Optional封装的参数值，如果参数值返回null会抛 NullPointerException
*/
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
}
```

`map()`和`flatMap()`的区别：

* 参数不同

  ```java
  //flatMap()
  class ZooFlat {
          private DogFlat dog = new DogFlat();

          public DogFlat getDog() {
              return dog;
          }
      }

  class DogFlat {
          private int age = 1;
          public Optional<Integer> getAge() {
              return Optional.ofNullable(age);
          }
  }

  ZooFlat zooFlat = new ZooFlat();
  Optional.ofNullable(zooFlat).map(o -> o.getDog()).flatMap(d -> d.getAge()).ifPresent(age ->
      System.out.println(age)
  );
  ```

  ```java
  //map()
  Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).ifPresent(age ->   System.out.println(age)
  );
  ```

* `flatMap()`参数返回值如果是null会抛出`NullPointerException`，而`map()`返回`EMPTY`。



### 4）判断value是否为null

```java
/**
* value是否为null
*/
public boolean isPresent() {
    return value != null;
}
/**
* 如果value不为null执行consumer.accept
*/
public void ifPresent(Consumer<? super T> consumer) {
   if (value != null)
    consumer.accept(value);
}

```



### 5）获取value

```java
/**
* Return the value if present, otherwise invoke {@code other} and return
* the result of that invocation.
* 如果value != null 返回value，否则返回other的执行结果
*/
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

/**
* 如果value != null 返回value，否则返回T
*/
public T orElse(T other) {
    return value != null ? value : other;
}

/**
* 如果value != null 返回value，否则抛出参数返回的异常
*/
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
}
/**
* value为null抛出NoSuchElementException，不为空返回value。
*/
public T get() {
  if (value == null) {
      throw new NoSuchElementException("No value present");
  }
  return value;
}

```



### 6）过滤值

```java
/**
* 1. 如果是empty返回empty
* 2. predicate.test(value)==true 返回this，否则返回empty
*/
public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
}
```



### 7）小结

如果不想看见`NPE`，就不要用`of()`、`get()`、`flatMap()`。

```java
//综合使用 Optional高频方法
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).filter(v -> v==1).orElse(3);
```



## Date-Time API

这是对java.util.Date强有力的补充，解决了Date类的大部分痛点：

①非线程安全

②时区处理麻烦

③各种格式化、和时间计算繁琐

④设计有缺陷，Date类同时包含日期和时间；还有一个java.sql.Date，容易混淆。



## 一、java.time 主要类

java.util.Date既包含日期又包含时间，而java.time把它们进行了分离

```java
LocalDateTime.class //日期+时间 format: yyyy-MM-ddTHH:mm:ss.SSS
LocalDate.class //日期 format: yyyy-MM-dd
LocalTime.class //时间 format: HH:mm:ss
```



## 二、格式化

Java 8之前：

```java
public void oldFormat(){
    Date now = new Date();
    //format yyyy-MM-dd
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    String date  = sdf.format(now);
    System.out.println(String.format("date format : %s", date));

    //format HH:mm:ss
    SimpleDateFormat sdft = new SimpleDateFormat("HH:mm:ss");
    String time = sdft.format(now);
    System.out.println(String.format("time format : %s", time));

    //format yyyy-MM-dd HH:mm:ss
    SimpleDateFormat sdfdt = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String datetime = sdfdt.format(now);
    System.out.println(String.format("dateTime format : %s", datetime));
}
```

Java 8之后：

```java
public void newFormat(){
    //format yyyy-MM-dd
    LocalDate date = LocalDate.now();
    System.out.println(String.format("date format : %s", date));

    //format HH:mm:ss
    LocalTime time = LocalTime.now().withNano(0);
    System.out.println(String.format("time format : %s", time));

    //format yyyy-MM-dd HH:mm:ss
  //1、获取时间
  //2、定义格式
  //3、按格式转化
  //4、按格式输出
    LocalDateTime dateTime = LocalDateTime.now();
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String dateTimeStr = dateTime.format(dateTimeFormatter);
    System.out.println(String.format("dateTime format : %s", dateTimeStr));
}
```



## 三、字符串转日期格式

Java 8之前：

```java
//已弃用
Date date = new Date("2021-01-26");
//替换为
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
Date date1 = sdf.parse("2021-01-26");
```

Java 8之后：

```java
LocalDate date = LocalDate.of(2021, 1, 26);//上下两句等同
LocalDate.parse("2021-01-26");

LocalDateTime dateTime = LocalDateTime.of(2021, 1, 26, 12, 12, 22);
LocalDateTime.parse("2021-01-26 12:12:22");

LocalTime time = LocalTime.of(12, 12, 22);
LocalTime.parse("12:12:22");
```

※Java8之前转换都需要借助`SimpleDateFormat`类，而Java8之后只需要`LocalDate`、`LocalTime`、`LocalDateTime`的`of`或`parse`方法。





## 四、日期计算

下面仅以一周后日期为例，其他单位大同小异。另外，这些单位都在 *`java.time.temporal.ChronoUnit`* 枚举中定义。

**java8之前：**

```java
public void afterDay(){
     //一周后的日期
     SimpleDateFormat formatDate = new SimpleDateFormat("yyyy-MM-dd");
     Calendar ca = Calendar.getInstance();
     ca.add(Calendar.DATE, 7);
     Date d = ca.getTime();
     String after = formatDate.format(d);
     System.out.println("一周后日期：" + after);

   //算两个日期间隔多少天，计算间隔多少年，多少月方法类似
     String dates1 = "2021-12-23";
   String dates2 = "2021-02-26";
     SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
     Date date1 = format.parse(dates1);
     Date date2 = format.parse(dates2);
  //以毫秒为单位
     int day = (int) ((date1.getTime() - date2.getTime()) / (1000 * 3600 * 24));
     System.out.println(dates1 + "和" + dates2 + "相差" + day + "天");
     //结果：2021-02-26和2021-12-23相差300天
}

```

**Java8之后：**

```java
public void pushWeek(){
     //一周后的日期
     LocalDate localDate = LocalDate.now();
     //方法1
     LocalDate after = localDate.plus(1, ChronoUnit.WEEKS);
     //方法2
     LocalDate after2 = localDate.plusWeeks(1);
     System.out.println("一周后日期：" + after);

     //算两个日期间隔多少天，计算间隔多少年，多少月
     LocalDate date1 = LocalDate.parse("2021-02-26");
     LocalDate date2 = LocalDate.parse("2021-12-23");
     Period period = Period.between(date1, date2);
     System.out.println("date1 到 date2 相隔："
                + period.getYears() + "年"
                + period.getMonths() + "月"
                + period.getDays() + "天");
		 //打印结果是 “date1 到 date2 相隔：0年9月27天”
     //这里period.getDays()得到的天是抛去年月以外的天数，并不是总天数
     //如果要获取纯粹的总天数应该用下面的方法
     long day = date2.toEpochDay() - date1.toEpochDay();
     System.out.println(date1 + "和" + date2 + "相差" + day + "天");
     //打印结果：2021-02-26和2021-12-23相差300天
}
```





## 五、获取指定日期

**Java8之前：**

```java
public void getDay() {

        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        //获取当前月第一天：
        Calendar c = Calendar.getInstance();
        c.set(Calendar.DAY_OF_MONTH, 1);
        String first = format.format(c.getTime());
        System.out.println("first day:" + first);

        //获取当前月最后一天
        Calendar ca = Calendar.getInstance();
        ca.set(Calendar.DAY_OF_MONTH, ca.getActualMaximum(Calendar.DAY_OF_MONTH));
        String last = format.format(ca.getTime());
        System.out.println("last day:" + last);

        //当年最后一天
        Calendar currCal = Calendar.getInstance();
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(Calendar.YEAR, currCal.get(Calendar.YEAR));
        calendar.roll(Calendar.DAY_OF_YEAR, -1);
        Date time = calendar.getTime();
        System.out.println("last day:" + format.format(time));
}
```

**Java8之后：**

```java
public void getDayNew() {
    LocalDate today = LocalDate.now();
    //获取当前月第一天：
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
    // 取本月最后一天
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
    //取下一天：
    LocalDate nextDay = lastDayOfThisMonth.plusDays(1);
    //当年最后一天
    LocalDate lastday = today.with(TemporalAdjusters.lastDayOfYear());
    //2021年最后一个周日，如果用Calendar是不得烦死。
    LocalDate lastMondayOf2021 = LocalDate.parse("2021-12-31").with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
}
```



## 七、JDBC和java8

现在jdbc时间类型和java8时间类型的对应关系是

1、`Date` --->  `LocalDate`

2、`Time` --->`LocalTime`

3、`Timestamp` ---> `LocalDateTime`

而之前统统对应`Date`，也只有`Date`





## 八、时区

> 时区：正式的时区划分为每隔经度 15° 划分一个时区，全球共 24 个时区，每个时区相差 1 小时。但为了行政上的方便，常将 1 个国家或 1 个省份划在一起，比如我国幅员宽广，大概横跨 5 个时区，实际上只用东八时区的标准时即北京时间为准。

java.util.Date对象实质上存的是1970年1月1日0点(GMT)至Date对象所表示时刻所经过的毫秒数。也就是说不管在哪个时区new Date，它记录的毫秒数都一样，和时区无关。但在使用上应该把它转换成当地时间，这就涉及到了时间的国际化。java.util.Date本身并不支持国际化，需要借助TimeZone。

```java
//北京时间：Wed Jan 27 14:05:29 CST 2021
Date date = new Date();

SimpleDateFormat bjSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//北京时区
bjSdf.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
System.out.println("毫秒数:" + date.getTime() + ", 北京时间:" + bjSdf.format(date));

//东京时区
SimpleDateFormat tokyoSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
tokyoSdf.setTimeZone(TimeZone.getTimeZone("Asia/Tokyo"));  // 设置东京时区
System.out.println("毫秒数:" + date.getTime() + ", 东京时间:" + tokyoSdf.format(date));

//如果直接print会自动转成当前时区的时间
System.out.println(date);
//Wed Jan 27 14:05:29 CST 2021
```



在新特性中引入了`java.time.ZonedDateTime`来表示带时区的时间。它可以看成是`LocalDateTime + ZoneId`。

```java
//当前时区时间
ZonedDateTime zonedDateTime = ZonedDateTime.now();
System.out.println("当前时区时间: " + zonedDateTime);

//东京时间
ZoneId zoneId = ZoneId.of(ZoneId.SHORT_IDS.get("JST"));
ZonedDateTime tokyoTime = zonedDateTime.withZoneSameInstant(zoneId);
System.out.println("东京时间: " + tokyoTime);

// ZonedDateTime 转 LocalDateTime
LocalDateTime localDateTime = tokyoTime.toLocalDateTime();
System.out.println("东京时间转当地时间: " + localDateTime);

//LocalDateTime 转 ZonedDateTime
ZonedDateTime localZoned = localDateTime.atZone(ZoneId.systemDefault());
System.out.println("本地时区时间: " + localZoned);

//打印结果
当前时区时间: 2021-01-27T14:43:58.735+08:00[Asia/Shanghai]
东京时间: 2021-01-27T15:43:58.735+09:00[Asia/Tokyo]
东京时间转当地时间: 2021-01-27T15:43:58.735
当地时区时间: 2021-01-27T15:53:35.618+08:00[Asia/Shanghai]
```



## 总结

通过上面比较新老`Date`的不同，当然只列出部分功能上的区别，更多功能还得自己去挖掘。总之，date-time-api给日期操作带来了福利。在日常工作中遇到date类型的操作，第一考虑的是date-time-api，实在解决不了再考虑老的`Date` 。





















































