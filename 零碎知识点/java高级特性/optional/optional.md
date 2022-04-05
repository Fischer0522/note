
# optional

## 简介

ptional 类主要解决的问题是臭名昭著的空指针异常（NullPointerException）使用它是为了避免代码中的 if (obj != null) { } 这样范式的代码，可以采用链式编程的风格。而且通过 Optional 中提供的 filter 方法可以判断对象是否符合条件，在符合条件的情况下才会返回，map 方法可以在返回对象前修改对象中的属性。



## Optional用处

本质上，Optional是一个包含有可选值的包装类，这意味着 Optional 类既可以含有对象也可以为空。

我们知道，任何访问对象方法或属性的调用都可能导致 NullPointerException，在这里，我举个简单的例子来说明一下

```java
String result = test.getName().getTime().getNum().getAnswer();
```

链式编程当中， 每一次get都可能存在为空的问题，通过optional进行空值检测，不需要显示的if！=null进行判断

## Optional的构造函数

- Optional.of(obj)：它要求传入的 obj 不能是 null 值的, 否则直接报NullPointerException 异常。
- Optional.ofNullable(obj)：它以一种智能的，宽容的方式来构造一个 Optional 实例。来者不拒，传 null 进到就得到 Optional.empty()，非 null 就调用 Optional.of(obj).
- Optional.empty()：返回一个空的 Optional 对象

## Optional的常用函数

- of：为非null的值创建一个Optional。of方法通过工厂方法创建Optional类。需要注意的是，创建对象时传入的参数不能为null。如果传入参数为null，则抛出NullPointerException。因此不经常用。
- ofNullable：为指定的值创建一个Optional，如果指定的值为null，则返回一个空的Optional。
- isPresent：如果值存在返回true，否则返回false。
- ifPresent：如果Optional实例有值则为其调用consumer，否则不做处理
- get：如果Optional有值则将其返回，否则抛出NoSuchElementException。因此也不经常用。
- orElse：如果有值则将其返回，否则返回指定的其它值。
- filter：如果有值并且满足断言条件返回包含该值的Optional，否则返回空Optional。
- map：如果有值，则对其执行调用mapping函数得到返回值。如果返回值不为null，则创建包含mapping返回值的Optional作为map方法返回值，否则返回空Optional。map(Function<T, U> mapper)：如果optional不为空，则将optional中的**对象 t 映射成另外一个对象 u**，并将 u 存放到一个新的optional容器中。

## Optional使用方法

- 主要针对用链式编程，用于进行空指针的处理

![在这里插入图片描述](optional.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RCQ18xMjE=,size_16,color_FFFFFF,t_70#pic_center.png)

简单用例：

不使用optional

```java
public String getName(User user){
	if(user == null){
		return "Unknown";
	}else return user.name();
}
```

使用optional改造

```java
public String getName(User user){
	return Optional.ofNullable(user)
							.map(u -> u.name)
							.orElse("Unknown");
}


```

- 源代码：

  ```java
  @GetMapping
      public ResponseEntity currentUser(@RequestHeader(value = "token")String token){
          Optional<String> subFromToken = jwtService.getSubFromToken(token);
          if(subFromToken.isPresent()){
              String id=subFromToken.get();
              UserData userData = userQueryService.finById(id).get();
              return ResponseEntity
                      .ok(userResponse(new UserWithToken(userData,token)));
          }
          else{
              throw new BizException(HttpStatus.UNAUTHORIZED,"token解析失败");
          }
  
      }
  ```

  

- 改造后：

  ```java
  @GetMapping
      public ResponseEntity currentUser(@RequestHeader(value = "token")String token){
          String id = jwtService.getSubFromToken(token)
                  .orElseThrow(()->new BizException(HttpStatus.UNAUTHORIZED,"token解析失败"));
              UserData userData = userQueryService.finById(id).get();
              return ResponseEntity
                      .ok(userResponse(new UserWithToken(userData,token)));
  
      }
## 开发中的应用



- 将实体类转化为optional，用于查询过程中避免空指针

```java
public Optional<Article> findById(String id) {
        return Optional.ofNullable(articleMapper.findById(id));
    }
```
- 查询结果封装为Optional，判断是否为空，返回对应的结果

```java
 public Optional<ArticleData> findById(String id, User user) {
    ArticleData articleData = articleReadService.findById(id);
    if (articleData == null) {
      return Optional.empty();
    } else {
      if (user != null) {
        fillExtraInfo(id, user, articleData);
      }
      return Optional.of(articleData);
    }
  }
```

- 在更新用户信息时进行数据重复的校验

  其中findByEmail查询的结果的返回形式为Optional，

  如果查询结果为null，那么直接走orElse，得到结果为true

  如果查询结果不为null，则为按照map中的判断形式得到一个boolean，由于一定会得到一个boolean，因此不会走orElse，直接将得到的boolean交给isEmailVaild

  ```java
  boolean isEmailValid =
                  userRepository.findByEmail(inputEmail)
                          .map(user -> user.equals(targetUser))
                          .orElse(true);
  ```

  

在Repository层使用Optional包装查询结果：

```java
 @Override
    public Optional<Article> findBySlug(String slug) {
        Article article=new Article();
        article.setSlug(slug);
        QueryWrapper<Article> wrapper=new QueryWrapper<>(article);
        Article article1=articleDao.selectOne(wrapper);
        if(article1==null){
            throw new BizException(HttpStatus.NOT_FOUND,"资源请求失败，要操作的文章可能已经不存在");
        }
 
        ArticleTagRelation articleTagRelation=new ArticleTagRelation();
        articleTagRelation.setArticleId(article1.getId());
        QueryWrapper<ArticleTagRelation>wrapper1=new QueryWrapper<>(articleTagRelation);
        List<String> tagIds=new LinkedList<>();
        for (ArticleTagRelation tagRelation : articleTagRelationDao.selectList(wrapper1)) {
            tagIds.add(tagRelation.getTagId());
        }
        if(!tagIds.isEmpty())
        {
            List<Tag> tags = tagDao.selectBatchIds(tagIds);
            article1.setTags(tags);
        }

        return Optional.ofNullable(article1);
    }


```

注：如果不是在下方添加Tag信息是使用到了article1，此处无需对空指针额外进行处理，而在此处，直接使用ofNullable将结果进行返回，此后的空指针问题便可以交给Service层取进行处理

在Service层使用`orElseThrow`方法，如果查询到则可以仅需进行链式处理，或者直接返回实体类形式的结果，如查询不到，则抛出指定的异常(Lambda表达式形式)

```java
 @DeleteMapping(path = "{id}")
    public ResponseEntity deleteComment(
            @PathVariable("slug") String slug,
            @PathVariable("id") String commentId,
            @RequestHeader(value = "token")String token){
        User user = jwtService.toUser(token).get();
        //此处直接使用实体类Article进行接收
        Article article = articleRepository.findBySlug(slug).orElseThrow(()->new BizException(HttpStatus.NOT_FOUND,"该评论所属的文章已经不存在"));
```



————————————————

参考链接：https://blog.csdn.net/DBC_121/article/details/104984093



