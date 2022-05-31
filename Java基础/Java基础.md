# Java基础

## 接口

**概念**

实现接口->实现方法

接口不是类，是符合这个接口的类的一组需求

不是类因此不能通过new来实例化一个接口

但是可以直接声明，再通过他的实现类去实现

```java
UserService userService =new UserServiceImpl();
```

接口之间可以进行继承，用于扩展接口，形成接口链

接口中的方法自动设置为public，字段自动设置为public static final

一个类可实现多个接口，多个接口存在相同的方法，其中一个接口中提供了默认方法，实现时会存在冲突，需从中选择一个，覆盖解决冲突。同时，java中不存在多继承的情况，接口的引入补充了多继承的。

**默认方法**



如果一个类继承了一个超类，实现了一个接口，那么接口中所有冲突的默认方法都会被忽略，按照类优先原则处理

一个类去实现一个接口则需要实现接口中定义的出默认方法以外的全部方法，通过提供默认方法可以保证源代码兼容性

通常再框架中经常使用默认方法，在进行配置实现时，只需要首先需要配置的所需要的方法，其他的方法为默认方法无需实现

**Comparator接口**

进行自定义比较，需要自定义一个类去实现Comparator<T>类，并且实现他的compare方法,在其中自定义比较规则

```java
public class AgeCom implements Comparator<User> {
    @Override
    public int compare(User o1, User o2) {
        return o1.getAge()-o2.getAge();
    }
}

```

使用时，需要先建立一个实例

```java
AgeCom ageCom=new AgeCom();
Arrays.sort(user1,ageCom);
```

## Lambda表达式

Lambda表达式为一个可传递的代码块,解决传递代码块需要先创建对象的问题

特性：

- 代码块传递
- 延迟执行

不使用Lambda表达式时，想要传递一个代码块，则需要先new一个对象的实例，再将对象进行传递

在之前进行自定义字符串比较时，我们需要先定义一个类实现Compartor接口，并重写方法

```java
public class StringCom implements Comparator<String> {
    @Override
    public int compare(String o1, String o2) {
        return o1.length() -o2.length();
    }
}

```

使用Lambda表达式后：

指定排序规格，Java为强类型语言，因此需要声明变量类型



```java
Arrays.sort(strings,(s1,s3)->{return s1.length()-s3.length();});
```

**Lambda表达式的形式**：

参数，箭头，表达式

在没有参数时，需要提供空括号，正如无参方法一样

如果可以推导出一个变量的类型，那么则可以省略，如排序时sort第一个参数传入的为String[]，可以推断出比较规则是针对String而言的，因此可以省略

只有一个参数并且参数可以推断出时，可以省略变量类型和小括号

无需指定返回类型，返回类型总是可以通过上下文推导出

**函数式接口**

只有一个抽象方法的接口，可以为其提供Lambda表达式

Comparator其中只有一个抽象方法compare，可以提供Lambda表达式，在使用Arrays.sort方法时，提供一个Lambda表达式

例：ArrayList中有一个方法为removeIf，参数为一个Predicate的接口，

创建Mycontidion对象实现Predicate接口

手动实现：

```java
    @Override
    public boolean test(String s) {
        if(Strings.isNullOrEmpty(s)){
            return true;
        }
        return false;
    }
```

```java
List<String> list=new ArrayList<String>();
        list.add("1212");
        list.add("121211");
        list.add("");
        list.add("hello");
        System.out.println(list);
MyContidion myContidion=new MyContidion();
list.removeIf(myCondition);
```

Lambda表达式：

```
list.removeIf(n->{return n==null||n=="";});
```

**方法引用**

使用::分隔类名和方法`Tag::getName`

先看一个实际应用的例子

```
LambdaQueryWrapper<Tag> lqw=new LambdaQueryWrapper();
lqw.eq(Strings.isNotEmpty(tagName),Tag::getName,tagName);
```

其中，Tag为一个实体类，getName为Getter方法

Tag::getName为一个方法引用，指示编译器**生成**一个函数式接口的实例，覆盖了这个接口的抽象方法来调用给定的方法，此处生成一个Tag对象，调用getName()方法

与Lambda相比较：

```java
lqw.eq(Strings.isNotEmpty(tagName),Tag::getName,tagName);
lqw.eq(Strings.isNotEmpty(tagName),tag->{return tag.getName();},tagName);
```

与Lambda一样，自身非对象，但是为函数式接口赋值时会生成一个对象

方法引用主要有三种形式：

> - Object::instanceMethod
>
>   `System.out`(out属于System下的一个对象)
>
> - Class::instanceMethod
>
>   第一个类调用方法，第二个作为参数传入`(x,y)-> x.compareToIgnoreCase(y)`
>
> - Class::staticMethod
>
>   `Math::max`等同于`(x,y)->Math.max(x,y)` ，所有参数均传入静态方法

只有lambda表达式的体**只调用一个方法**而不做其他操作时，才能把Lambda表达式重写为方法引用

`s -> s.length()==0` 既有调用，又有比较，无法重写为方法引用

方法引用不能单独存在，总会转换为一个函数式接口的实例





### 闭包

Lambda表达式可以捕获自由变量的值，Lambda表达式即为Java中的闭包

- 可以捕获自由变量的值
- 捕获的值应为实际最终值，后续不能改变
- 捕获后只能进行引用，不能修改其值

**赋值给接口**

可以将Lambda表达式赋值给一个接口，通过Runnable接口进行执行

```java
Runnable action = () -> System.out.println("hello"); 
action.run();
```



## 内部类

特性：

- 内部类可以对包中的其他类隐藏
- 可以访问外围类的数据，包括私有字段
- 无类名，故无构造器

局部内部类：声明在方法之中，不能有访问说明符，只有该方法才知道这个类，对其他的一切均隐藏

### 匿名内部类

基础语法

```java
 SuperType in1=new SuperType (..){
   	内部类的方法与数据
}
```

声明一个匿名内部类，去实现接口SuperType或扩展的类并且通过in1来进行调用

其中，SuperType为要实现的接口或者要扩展的类，如果为接口，则要实现其中的方法，如果为类，内部类要扩展这个类

in1为对象名，用来调用匿名类方法的对象，SuperType为实现的接口或者要扩展的类只有大括号内的才为匿名内部类的东西，因此，确实是没有名字

例：

```java
public class User {
    private Integer age;
    private String username;
    public Predicate<String> predicate =new Predicate<String>(){
        @Override
        public boolean test(String s) {
            return false;
        }
    };
}
```

在匿名内部类中，实现了predicate接口，同时实现了其中的test方法

函数式编程的使用中：

```java
list.removeIf(new Predicate<String>(){
        @Override
        public boolean test(String s) {
            return s==null
        }
       });
```

与Lambda表达式相比较：

```
list.removeIf(testFind->{return s==null;});
```

与自己实现接口相比较

```java
public class MyContidion implements Predicate<String> {

    @Override
    public boolean test(String s) {
        if(Strings.isNullOrEmpty(s)){
            return true;
        }
        return false;
    }
}
```

```java
MyContidion myContidion=new MyContidion();
list.removeIf(myContidion);
```

## 代理

代理类可以在运行时创建新的类，去实现指定的接口。通过代理对象来调用接口的方法时，会由代理类进行处理

**代理对象**：

- 一个类加载器
- 一个class对象数组，每个元素对应实现的各个接口
- 一个调用处理器

对该接口进行代理之后，该接口的所有方法均会调用调用处理器的invoke方法进行处理，如toString等也一并处理

只有在对象数组中的接口才能够正确代理，而代理的主要处理工作便是通过调用处理器完成的

**调用处理器**

调用处理器为一个实现了InvocationHandler的类，其中只有一个invoke方法，在调用了代理对象的方法时，invoke方法会被调用，达到了代理处理的效果

**Demo**

定义一个接口in2

```java
public interface in2 {
    default void getName(int i){
        System.out.println(2);
    }
     void hello();
}
```

接口对应的实现类

```java
public class Myimpl implements in2,in1{

    @Override
    public void getName(int i) {
        System.out.println("getName已经被调用+"+i);
    }

    @Override
    public void hello() {
        System.out.println("say hello");
    }
}

```

调用处理器TraceHandler进行，调用过程的追踪的功能

```java
public class TraceHandler implements InvocationHandler {
    private Object target;
    TraceHandler(Object t){
        target = t;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //method为调用的方法，args为方法中的参数
        System.out.print(target);
        System.out.print("."+ method.getName()+"(");
        if(args != null) {
            for(int i = 0; i < args.length;i++){
                System.out.print(args[i]);
                if(i< args.length-1) System.out.print(",");;
            }
        }
        System.out.println(")");

        //if(target.getClass().equals(Myimpl.class))
        return method.invoke(target,args);//通过反射调用实际的方法

    }
}

```

Test：

```java
public class ProxyTest2 {
    public static void main(String[] args) {
        Myimpl myimpl = new Myimpl();
        TraceHandler handler = new TraceHandler(myimpl);
        Object proxy = Proxy.newProxyInstance(
                ClassLoader.getSystemClassLoader(),
                new Class[]{in1.class, in2.class,Comparable.class},
                handler);
        in2 test = (in2)proxy;
        test.getName(19);
        System.out.println(test);
        if(!test.equals("2")){
            System.out.println("调用toString:"+test);
        }
    }
}
```



将Myimpl的实例传给TraceHandler中的target，

- 将代理对象强转为in2型即可调用in2的方法，此时便调用了调用处理器中的handler方法，将调用过程打印出来，
- 并且最后通过反射的方法真正调用接口的实现类中的方法

调用getName和hello均会进行代理

**核心技术上的Demo**

```java
public class ProxyTest {
    public static void main(String[] args) {
        Object[] elements = new Object[1000];
        //element中的元素为代理Proxy，构造Proxy是传入了一个调用处理器，而具体的数值保存在调用处理器当中
        for(int i = 0; i < elements.length; i++){
            Integer value = i + 1;
            TraceHandler handler = new TraceHandler(value);
            Object proxy = Proxy.newProxyInstance(
                    ClassLoader.getSystemClassLoader(),
                    new Class[]{Comparable.class, in1.class, in2.class},//实现了Comparable接口，用于进行二分比较
                    handler
            );
            elements[i] = proxy;
        }
        Integer key = new Random().nextInt(elements.length) + 1;
        int result = Arrays.binarySearch(elements, key);
        if(result >= 0) {
            System.out.println(elements[result]);
        }
        TraceHandler handler = new TraceHandler(2);
        Object proxy = Proxy.newProxyInstance(
                ClassLoader.getSystemClassLoader(),
                new Class[]{Comparable.class, in1.class, in2.class},//只实现了Comparable接口，用于进行二分比较
                handler
        );
        Object test = proxy;
        System.out.println(test);

    }
}
```

该例中实现了一个二分搜索，通过查看源码，思路与上一个demo一致，

- 通过强制转换的方式将代理对象转换为Comparable，
- 调用Comparable接口中的compareTo方法，
- 最后调用了TraceHandler中的invoke方法，进行了代理
- 在invoke中最后通过反射的方式调用原本target的compareTo方法，实现了原本的二分查找

Integer实现了Comparable接口

```java
Comparable midVal = (Comparable)a[mid];
@SuppressWarnings("unchecked")
int cmp = midVal.compareTo(key);
```

## 泛型

泛型出现之前，通常采用继承Object，存在的问题：

- 需强制转换
- 无法检查错误，可以随意传值

### 类型变量的限定

对于一个泛型，在声明时完全处于未知的状态，如果需要调用一定的方法就需要对其进行限制(提供额外的信息)，通过关键词extends的方法进行限制

- extends只表示T是限定类型的子类型，并不是表明为真正的继承关系
- 所继承的可以是类也可是接口
- 使用通配符 & 来分隔多个类型变量(存在顺序问题)

```java
public class ArrayAlg {
    public  <T extends Comparable> Pair minmax(T[] a){
        T min = a[0];
        T max = a[0];
        for(int i = 0;i < a.length;i++){
            if(min.compareTo(a[i])>0){
                min = a[i];
            }
            if(max.compareTo(a[i])<0){
                max = a[i];
            }
        }
        return new Pair(min,max);

    }
}
```

**泛型与虚拟机**

虚拟机上面无泛型，只有普通类型，故存在类型擦除的过程

类型擦除：

- 有限定类型的替换为限定类型，无限定替换为Object
- 返回为泛型时且擦除为Object时，存在一个强制转换的过程，将被擦除的重新转换为传入的类型

```java
public class DateInterval extends Pair<LocalDate> {

    public void setSecond(LocalDate second) {
        if(second.compareTo(getFirst())>=0) {
            super.setSecond(second);
        }
    }
}
```

原本在继承的过程当中，子类和父类形参相同的相同的方法，按照多态的原则，在子类中应当由子类的方法覆盖掉父类的方法。但是此处由于类型擦除，父类无限定类型，被擦除成Object，子类为LocalDate，导致形参不一致，因此无法覆盖，与多态产生冲突

为解决冲突，编译器在子类中生成一个桥方法

```java
public void setSecond(Object second){
    setcond(LocalDate(second));
}
```

**类型擦除总结**

- 虚拟机无泛型
- 泛型会被替换为限定类型（无限定即为Object）
- 通过桥方法保持多态

### 限制

- 无基本类型，只能使用包装类型

- 不能创建参数化类型的数组，由于类型擦除使数组无法检查类型

  Pair\<String>[] 没问题，只是不能使用new Pair\<String>[10]进行初始化,但是不如直接使用ArrayList\<String>安全便捷

- 不能使用如 new T()等，因为类型擦除后会变为new Object()，或者是限定类型

- 不能构造泛型数组

  ```java
   public  <T extends Comparable> Pair minmax(T[] a){
          T[] mm = new T[100];
  
      }
  ```

  由于类型擦除，会一直构造Comparable[100]，而并非我们传入的类型T

- **泛型类**的静态上下文中类型变量无效

  ```java
  private static T getName();
  private static T name;
  ```

  `private static<T> T getname()`为泛型方法，注意区分

- 不能捕获后者抛出泛型类的实例，因此泛型类不能扩展Throwable，catch(T t)也不合法

- 注意擦除时的冲突问题

- ```
  public boolean equals(T value) {
      .....
  }
  ```

  调用时为Pair<String> 但擦除后仍为
  
  ```java
  public boolean equals(Object value)
  ```
  
  与原本的equals产生了冲突

**继承规则**

T之间存在继承关系，但是传入后的Pair\<T>之间不存在任何继承关系

### 通配符类型

允许类型参数发生变化

例`Pair<? extends Employee>`表示任何泛型Pair类型的类型参数为Employee的子类，如Pair\<Manager>,而不是Pair\<String>

此处的extends表示真正的继承关系，而不是像之前表限定作用

仅在方法的传递参数时使用通配符来解决类型的问题，在声明泛型类时仍使用正常的泛型\<T>
