# Lambda表达式

## 语法

```java
(parameters) -> expression
或
(parameters) ->{ statements; }
```

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。

## 简单实例

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

## 实际开发中的应用

```java
 articles.stream()
                .map(articleData1 -> articleData1.getProfileData().getId())
                .collect(toList()));
```

- 此处的操作为articleData1，对其进行getProfileData().getId())，并返回结果
- Stream.map映射每个元素到对应的结果，用于进行数据类型的转换，在Optional中也有类似的map

## 变量作用域

- lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

```java
public class Java8Tester {
 
   final static String salutation = "Hello! ";
   
   public static void main(String args[]){
      GreetingService greetService1 = message -> 
      System.out.println(salutation + message);
      greetService1.sayMessage("Runoob");
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
}
```

结果为

```java
Hello! Runoob
```

## Lambda表达式化简匿名内部类

![image-20220224104859514](Lambda表达式.assets/image-20220224104859514.png)

```java
public class lambdaDemo1 {
    public static void main(String[] args) {
        Ainimal a=new Ainimal() {
            @Override
            public void run() {
                System.out.println("乌龟跑的慢");
            }
        };
        a.run();
        /*匿名类写法，然后将对象传入go之中
		Swimming s1=new Swimming() {
            @Override
            public void swim() {
                System.out.println("划水");
            }
        };*/

		//简化写法
		Swimming s1=()-> {
            System.out.println("老师划水");
        };
        go(s1);
        
        /*匿名类写法，直接在go之中new一个匿名类的无名对象
		go(new Swimming() {
            @Override
            public void swim() {
                System.out.println("学生划水");
            }
        });*/

		//简化写法
        go(()-> {
                    System.out.println("学生划水");
                }
        );
    /*错误示范 非接口类无法使用Lambda进行简化
    Ainimal a1=()->{
        System.out.println("乌龟跑得慢");
    }
*/
    }
    public static void go(Swimming s)
    {
        System.out.println("开始");
        s.swim();
        System.out.println("结束");
    }
}
 abstract class Ainimal
{
    public abstract void run();
}
@FunctionalInterface// 加上注解必须是函数接口，里面只有一个抽象方法
interface  Swimming{//接口默认为抽象类
    void swim();
}

```

- 通过Lambda表达式化简比较器

```java
Arrays.sort(arrs, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                //制订比较规则
                return -(o1-o2);
            }
        });
        Arrays.sort(arrs, (Integer o1, Integer o2)-> {
                //制订比较规则
                return -(o1-o2);
        });
```

