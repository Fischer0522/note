# Java基础

## 接口

**概念**

接口不是类，是符合这个接口的类的一组需求

不是类因此不能通过new来实例化一个接口

但是可以直接声明，再通过他的实现类去实现

```java
UserService userService =new UserServiceImpl();
```

接口之间可以进行继承，用于扩展接口，形成接口链

接口中的方法自动设置为public，字段自动设置为public static final

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

**有点晦涩，需进一步学习**

## Lambda表达式

Lambda表达式为一个可传递的代码块

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

对于只有一个抽象方法的接口，当需要这种接口的对象时，就可以提供一个Lambda表达式，这种接口称为函数式接口

Comparator其中只有一个抽象方法compare，可以提供Lambda表达式，在使用Arrays.sort方法时，提供一个Lambda表达式

例：ArrayList中有一个方法为removeIf，参数为一个Predicate的接口，

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

Tag::getName为一个方法引用，指示编译器生成一个函数式接口的实例，覆盖了这个接口的抽象方法来调用给定的方法，此处生成一个Tag对象，调用getName()方法

与Lambda相比较：

```java
lqw.eq(Strings.isNotEmpty(tagName),Tag::getName,tagName);
lqw.eq(Strings.isNotEmpty(tagName),tag->{return tag.getName();},tagName);
```

与Lambda一样，自身非对象，但是为函数式接口赋值时会生成一个对象

方法引用主要有三种形式：

> - Object::instanceMethod
> - Class::instanceMethod
> - Class::staticMethod

只有lambda表达式的体只调用一个方法而不做其他操作时，才能把Lambda表达式重写为方法引用

`s -> s.length()==0` 既有调用，又有比较，无法重写为方法引用