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

通配符仅针对于某个实例，对该实例在声明泛型类时仍使用正常的泛型\<T>，只是在声明时不对传入的值进行具体约束

表述传入一个不确定的类型，并加以限定

- extends：表示参数化的类型可能是所指定的类型，或者是此类型的子类。
- super ：用 super 进行声明，表示参数化的类型可能是所指定的类型，或者是此类型的父类型，直至 Object
- 无限定

**类型擦除：**

`List<? extends T>`和`List<? super T>`类型擦除后得到的结果均为Tc

**extends**

例`Pair<? extends Employee>`表示任何泛型Pair类型的类型参数为Employee的子类，如Pair\<Manager>,而不是Pair\<String>

此处的extends表示真正的继承关系，而不是像之前表限定作用

此时的setter、getter方法为：

```java
	? extends Employee getFirst()
    void setFirst(? extends Employee)
```

```java
// T为Object，String是Object的子类，满足约束
ArrayList<? extends Object> list = new ArrayList<String>();//ok 
list.add("abc");// error
list.add(123)//满足通配符的要求，但是 Integer与String之间无直接关系，无法指向
```

可以理解为`ArrayList<? extends Object>`是`ArrayList<String>`的父类类型，可以接收`ArrayList<String>`。

- 为什么不能存元素呢？

规定的是上界为Object类型，那么Object子类都可以存，当然实际类型String的父类型也是可以的，那么就会出现子类引用指向父类对象的情况。 所以编译器禁止添加元素。

- 为什么能取元素呢？

规定的是上界，可以以上界的类型（父类类型）接收对象，不会出现问题。

**super**

而对于super的getter和setter：

```java
? super Manager getFirst()
void setFirst(? super Manager)
```

情况与extends刚好相反，变为getter无法确定保证类型，只能使用Object进行返回

即泛型下界，类型约束为T的基类。如：

```
//T为String，Object为String的基类，满足约束
ArrayList<? super String> list = new ArrayList<Object>();//ok 
list.add("abc");//ok
Object s = list.get(0);// Object
System.out.println(s);

复制代码
```

可以理解为`ArrayList<? super String>`是`ArrayList<Object>`的父类类型，可以接收`ArrayList<String>`。 在scala中类似的语义是泛型的逆变。

- 为什么可以存元素呢？

规定的是下界为String类型，即所有String及其子类都可以存（可能不太恰当，String是final的）。String及String的子类都是实际类型Object的子类。 所以，添加元素不会有问题。

- 为什么取出来的元素是Object类型

定义的为泛型下界，不能确定是哪一级父类，使用Object一定不会错。

**无限定**

`Pair<?>`

无法使用setter方法进行传入，getter方法返回的类型也为Object，可以用于一些无需setter和getter方法的简单操作，如判断是否为空等

**带有超类型限定的允许写入一个泛型对象，带有子类型限定的允许读取一个泛型对象**

**T 是一个 确定的 类型，通常用于泛型类和泛型方法的定义，？是一个 不确定 的类型，通常用于泛型方法的调用代码和形参，不能用于定义类和泛型方法。**

## 并发

将要执行的代码放在一个run方法之中，这个类要实现Runnable接口(其中只有一个run方法)，覆盖其run方法

```java
public class Myrun implements Runnable{
    @Override
    public void run() {
        System.out.println("hello");
    }
}
```

Runnable为函数式接口，因此也可使用Lambda表达式来完成

```java
Runnable r =()-> {
    System.out.println("hello");
};
Thread thread = new Thread(r);//创建并启动线程
thread.start();
```

当调用thread.start()后，线程处于可运行的状态，具体何时运行需要根据操作系统的调度              

stop方法被废弃之后无强制进程终止，可以通过interrupt方法请求中断一个进程

Thread.currentThread获取当前进程 isInterrupted检查进程是否中断，进程阻塞状态下会抛出InterruptException异常

此外，interrupted方法可以检查中断并且清除当前进程的中断状态

### 同步

对于临界资源的访问，若不使用原子性的操作，则会存在互相覆盖的问题，最后导致更新错误，因此需要使用锁来互斥的访问资源

#### 锁

获得锁后才能访问临界区，达到互斥访问

分类：

- ReetrantLock类
- Synchronized关键字

**ReetrantLock类**

配合try/finally

```java
myLock.lock();
    try{
        临界区访问
    } finally{
        myLock.unLock();
    }
```

**sychronized关键字**

基于：Java每个对象都有一个内部锁，并且这个锁有一个内部条件，该锁会管理试图进入声明sychronized方法的线程，这个条件管理调用了wait的线程

同步方法：

在方法声明时使用sychronized关键字，则锁会保护整个方法

```java
public synchronized void  transfer(int from,int to,double amount)
```

sychronized关键字配套的条件变量为wait和notifyAll，在声明sychronized关键字的方法内或者代码块内使用

同步块：

在对象内部声明一个lock对象(Object)，sychronized获取该lock对象，上锁

```java
public class Bank {
    
    private Object myLock = new Object();
    public void  transfer(int from,int to,double amount) throws InterruptedException{
        sychronized(myLock){
            doSomething...
        }
    }
    
}
```



#### 条件变量

陷入休眠队列，等待满足条件后才能执行，与锁配合使用，只有获得了锁之后才能去唤醒/休眠

```java
ReentrantLock banklock=new ReentrantLock();//创建一个锁

sufficientFunds=banklock.newCondition();//通过锁的newCondition方法获得一个条件对象
while (condition...)
           sufficientFunds.await();//陷入等待

sufficientFunds.signalAll();//在别的线程唤醒所有休眠的线程

```

if/while：

- signalAll唤醒所有的进程，无论是否满足条件，因此对于不满足条件的进程如果使用if，则会不满足条件而执行，因此应当使用while进行条件的重新检查

**volatile字段**

- 声明某个字段volatile后，编译器和虚拟机知道该字段可能被其他并发更新
- volatile并不提供原子性，如i++，无法代替锁，只能确保在该线程的更新在其他线程也是可见的，但是该更新过程可能会被其他线程中断
- 线程操作可见方式，并非同步方式

**原子操作**

只对变量进行赋值/读取操作时：

- AtomicInteger类：incrementAndGet/decrementAndGet 大量访问相同的原子值时，性能会大幅下降
- IntegerAdder类：多个线程单独累加，最后合计出总值，性能较高

#### 经典问题

**生产者消费者**

条件：

- 生产者消费者共用一个大小为10的缓冲区
- 非满时生产者可以生产放入其中
- 非空消费者可以从中取出进行消费

其中，使用了synchronized关键词，以及缓冲区List的对象锁，所有的线程去竞争缓冲区list的对象锁

因为只有声明了同步块才能使用条件变量，因此临界区为整个consume/produce函数

在条件变量判断时使用while而不是if，防止唤醒之后跳过检查的情况，在只有一个生产者进程时不存在任何问题，但是在开了五个生产者线程后，如果使用使用if，则会跳过条件判断，最后导致缓冲区中的元素大于10个

生产者类：

```java
public class Producer {
     volatile ArrayList<Integer> list;
    public Producer(ArrayList<Integer> list){
        this.list = list;
    }
    public void produce(){
        while (true){

        synchronized (list) {
            while(list.size()>=10) {//使用while而不使用if，防止唤醒后跳过判断
                try {
                    list.wait();//缓冲区满则休眠
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            Random random = new Random();
            int i = random.nextInt(100);
            list.add(i);
            System.out.print("生产了："+i+"  ");
            System.out.println("当前的容器为："+list);
            list.notify();//唤醒线程
        }
    }
    }
}
```

消费者类：

```java
public class Consumer {
    volatile ArrayList<Integer> list;
    public Consumer(ArrayList<Integer> list){
        this.list = list;
    }
    void consume() {
        while(true) {
            synchronized (list) {
                while(list.size()==0) {
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                Integer remove = list.remove(0);
                System.out.print("消费了:"+remove+" ");
                System.out.println("当前的容器为："+list);
                list.notify();

            }
        }
    }
}
```

启动类：

```java
public class Factory {
    public static void main(String[] args) {
        ArrayList list = new ArrayList(1);
        Runnable r1 = () -> {
            Consumer consumer = new Consumer(list);
            consumer.consume();
        };
        Runnable r2 = ()-> {
            Producer producer = new Producer(list);
            producer.produce();
        };
        for(int i = 0; i< 5; i++){
            //生产者消费者各创建5个进程
            Thread thread1 = new Thread(r1);
            Thread thread2 = new Thread(r2);
            thread1.start();
            thread2.start();
        }
    }
    
}
```



### 任务与线程池

**Runnable/Callable/Future**

Runnable 封装一个异步运行的任务，无参数无返回值，只有一个方法run

Callable为一个存在返回值的任务，只有一个方法call

Future保存异步计算的结果，将Future交给某个线程，这个Future对象的所有者在结果计算好之后就可以获得结果，可以通过get获取

执行callable的一种方法是FutureTask，它实现了Future和Runnable接口，用于构造线程执行任务，一个FutureTask对应这一个线程，因此，创建线程是传入同一个FutureTask只会创建一个线程：

```java
public class Main2 {
    public static void main(String[] args) throws InterruptedException{
        Callable c = ()->{
            String action = "hello";
            System.out.println(action);
            return action;
        };

        ExecutorService executorService = Executors.newFixedThreadPool(5);
        for(int i =0; i < 6;i++){
            FutureTask<Integer> integerFutureTask = new FutureTask<Integer>(c);//若放在循环之外则只会向线程池提交一个线程，从而只执行一次                    
            Thread thread = new Thread(integerFutureTask);
            Future<?> submit = executorService.submit(thread);
        }

        //thread.start();
    }
}
```

**执行器**

执行器类有静态工厂方法，用于创建线程池，返回一个ExecutorService

可以通过submit方法将Runnable或者Callable提交给线程池并且执行

线程池所作的工作：

1. 调用Executor的静态方法创建对应的线程池
2. 通过submit将Runnable或者Callable提交到线程池并且执行
3. 保存submit返回的Future对象，对结果进行处理
4. 若不想再提交时，使用shutdown关闭

**控制任务组**

提交：

- invokeAny：提交一个Callable集合中的所有对象至线程池，返回某一个执行结束的结果，通常是最快的那个
- invokeAll：提交一个Callable集合中的所有对象至线程池，该方法会阻塞，直至所有的任务都完成，返回一个Future集合

**小Demo**

```java
public static void testFuture()throws InterruptedException,ExecutionException{
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(3);
        list.add(8);
        list.add(12);
        List<Callable<Integer>> tasks = new ArrayList<>();
        for(int num:list){
            Callable<Integer> task =()->{ return num;};//该Callable仅做简单处理，将num返回
            tasks.add(task);
        }
        ExecutorService executorService = Executors.newCachedThreadPool();//创建线程池
        Integer integer = executorService.invokeAny(tasks);//返回某一个（最快）的结果
        System.out.println(integer);
        List<Future<Integer>> futures = executorService.invokeAll(tasks);//返回一个Future集合，处理所有结果
        int total = 0;
        for(Future<Integer> future: futures){
            total+=future.get();//所有结果累加得到最终的计算结果
        }
        System.out.println(total);
    }
```



**ForkJoin**

fork-join用于将一个任务分解成子任务

```java
if(problemSize<threshold){
    solve the problem
} else{
    break problem into subproblems
}
```

 demo:

二分法统计一个数组中有多少个元素满足特定的属性

需要一个拓展了RecursiveTask\<T>的类，再覆盖compute方法调用子任务，合并结果

```java
public class Counter extends RecursiveTask<Integer> {
    
    public static final int THRESHOLD = 1000;//当子数组长度小于1000时开始进行统计
    private double[] values;
    private int from;//左端点
    private int to;//右端点
    private DoublePredicate filter;//函数式接口，接受一个条件(传入一个Lambda表达式即可)

    public Counter(double[] values, int from, int to, DoublePredicate filter) {
        this.values = values;
        this.from = from;
        this.to = to;
        this.filter = filter;
    }
    @Override
    protected Integer compute() {
        if(to - from <THRESHOLD){//分割到阈值时，停止递归
            int count = 0;
            for(int i = from;i < to; i++){
                if(filter.test(values[i])) count++;//满足条件，进行统计
            }
            return count;
        } else{//二分递归提交任务
            int mid = (from+to)/2;
            Counter first = new Counter(values,from,mid,filter);
            Counter second = new Counter(values,mid,to,filter);
            invokeAll(first,second);
            return first.join()+second.join();//结果合并
        }
    }
}
```

启动类：

```java
public class ForkJoinTest {
    public static void main(String[] args) {
        final int SIZE = 10000000;
        double[] numbers = new double[SIZE];                
        for(int i = 0;i < SIZE;i++){
            numbers[i] = Math.random();
        }
        Counter counter = new Counter(numbers,0,numbers.length,x-> x>0.5);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        forkJoinPool.invoke(counter);
        System.out.println(counter.join());
    }
}
```

​          

### 异步计算

