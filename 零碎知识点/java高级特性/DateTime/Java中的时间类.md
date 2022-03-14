## 时间类

### DateTime

- 取得当前时间

  ```java
  DateTime time= new DateTime();
  DateTime time=DateTime.now();
  ```

- 按格式输出字符串类型的时间

  ```java
  String time = dateTime.toString("yyyy-MM-dd hh:mm:ss.SSSa");
  //hh为12小时制，HH为24小时制
  ```

- 初始化时间

  ```java
  DateTime time = new DateTime(2018,4,23,23, 7,18,888);
  
  // 2018年4月23日23点7分18秒888毫秒
  ```

- 计算两个日期间隔的天数

  ```java
  LocalDate start=new LocalDate(2018,4,23);
  
  LocalDate end=new LocalDate(2019, 06, 16);
  
  int days = Days.daysBetween(start, end).getDays();
  ```

- 对日期进行修改增加dateTime1.plusDays(1) 、 plusHours(2) 等

  ```java
  DateTime dateTime = DateTime.parse("2018-04-23");
  
  dateTime = dateTime1.plusDays(1);
  
  dateTime = dateTime1.plusHours(2);
  
  dateTime = dateTime1.plusMinutes(3);
  
  dateTime = dateTime1.plusMonths(4);
  
  dateTime = dateTime1.plusSeconds(5);
  
  dateTime = dateTime1.plusWeeks(6);
  
  dateTime = dateTime1.plusYears(7);
  
  //减少日期
  
  DateTime dateTime = DateTime.parse("2018-04-23");
  
  dateTime = dateTime1.minusMillis(1);
  
  dateTime = dateTime1.minusHours(1);
  
  dateTime = dateTime1.minusSeconds(1);;
  
  
  ```

- 判断是否为闰月

  ```java
  DateTime time = new DateTime();
  
  org.joda.time.DateTime.Property month = time.monthOfYear();
  
  System.out.println("是否闰月:" + month.isLeap());
  ```

- DateTime与java.util.Date对象,当前系统TimeMillis转换  

  ```java
    DateTime dt6 = new DateTime(new Date());    
                  Date date = dateTime1.toDate();    
                  DateTime dt7 = new DateTime(System.currentTimeMillis());    
                  dateTime1.getMillis();   
                    
                  Calendar calendar = Calendar.getInstance();    
                  dateTime = new DateTime(calendar);  
  ```

---

参考链接：[joda-time的使用 - IT男 - ITeye博客](https://www.iteye.com/blog/ylq365-1769680)

