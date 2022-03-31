# LocalDate

Java8提供了新的时间处理类，已经可以取代Date以及JodaTime，此处只介绍LocaDate，其他几种也大差不差，按照idea提示使用即可

## 关键类

Instant：瞬时实例。

LocalDate：本地日期，不包含具体时间 例如：2014-01-14 可以用来记录生日、纪念日、加盟日等。

LocalTime：本地时间，不包含日期。

LocalDateTime：组合了日期和时间，但不包含时差和时区信息。

ZonedDateTime：最完整的日期时间，包含时区和相对UTC或格林威治的时差。

## 获取当前日期

```java
//获取今天的日期
public void getCurrentDate(){
    LocalDate today = LocalDate.now();
    System.out.println("Today's Local date : " + today);
}
```

## 获取年月日信息

```java
//获取年、月、日信息
public void getDetailDate(){
    LocalDate today = LocalDate.now();
    int year = today.getYear();
    int month = today.getMonthValue();
    int day = today.getDayOfMonth();
    String s=localDate.getDayOfWeek()//获取字符串类型的，如：SUNDAY
	int weekDay= localDate.getDayOfWeek().getValue()//转为int类型
    System.out.printf("Year : %d  Month : %d  day : %d t %n", year, month, day);
}
```

## 处理特定日期

LocalDate.of()创建任意日期，传入年月日作为参数

```java
 LocalDate localDate=LocalDate.of(2000,1,1);
```

## 日期判断

传入一个ChronoLocalDate

- `isEqual`
- `isBefore`
- `isAfter`

## 截取时间

YearMonth只保留年月，格式为`2022-03`

MonthDay则为月日，格式为`--03-30`

```java
YearMonth now = YearMonth.now();
        System.out.println(now);
        MonthDay now1 = MonthDay.now();
        System.out.println(now1);
```

## 操作当前时间

增加时间：传入一个long类型的数值

- `plusYears`
- `plusMonths`
- `plusDays`
- `plusHours`

同理还有减少时间：

- `minusYears`
- `minusMonths`
- `minusDays`
- `minusHours`

## 时间间隔

有一个常见日期操作是计算两个日期之间的天数、周数或月数。在Java 8中可以用java.time.Period类来做计算。下面这个例子中，我们计算了当天和将来某一天之间的月数。

下面的例子：现在是一月份，距离到五月份，中间相隔3月

```java
//计算两个日期之间的天数和月数
public void calcDateDays(){
    LocalDate today = LocalDate.now();

    LocalDate java8Release = LocalDate.of(2018, Month.MAY, 14);

    Period periodToNextJavaRelease = Period.between(today, java8Release);

    System.out.println("Months left between today and Java 8 release : "
                                           + periodToNextJavaRelease.getMonths() );
}

```

## 格式化

使用DateTimeFormatter，进行格式化，避免了SimpleDateFormat的线程不安全问题，主要可以通过以下两种方法进行格式化

- 使用标准形式

  ```java
  String day="2018年02日10";
          System.out.println(LocalDate.parse(day, DateTimeFormatter.BASIC_ISO_DATE).toString());
  ```

  标准形式如下：

  ![image-20220330213151091](LocalDate家族.assets/image-20220330213151091.png)

使用ofPattern自定义格式：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate date=LocalDate.now();
System.out.println(formatter.format(date));
```

值得注意的是，提供的信息要比指定的格式中的信息多，如想格式化到小时则需要使用LocalDateTime，LocalDate信息不足则会抛出异常

