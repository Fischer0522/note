# Springboot
## 基础篇

### 配置文件

**多环境配置**

一共准备三份application.yml配置文件，一个作为主控，另外两个一个作为开发环境的配置文件，另外一个作为线上的配置文件

- application.yml
- application-dev.yml 开发环境
- application-prod.yml 线上环境

 ```yaml
 spring:
   profiles:
     active: dev #启用开发环境
 ```

### 分层开发web

**MVC设计模式：**

- Model：实体javabean，代表存取数据的pojo，也可以带有逻辑，作用为在内存中暂时存储数据
- View：解析、处理、显示内容，进行模型的渲染
- Controller：处理视图响应，决定如何调用Model中的实体bean进行CRUD，建议在其中不要写业务逻辑代码

**工作流程：**

-   客户端发送请求由Tomcat接收，Tomcat将请求转发给DispacherServlet处理
- DispacherServlet匹配控制器中配置的映射路径，进行下一步处理
- ViewResolver将ModelAndView或Exception解析成View，View调用render()，将ModelAndView中的数据渲染成也页面

Repository和Dao层一样，都可以进行CRUD操作，相当于仓库管理员

- Dao层为存取工作

- Repository为存取和管理对象

  Repository=管理对象(对象缓存和在Repository的状态)+Dao

#### Thymeleaf

**基础语法：**

1. 引用命名空间

   首先加入依赖，在模板文件中引用命名空间，如下：

   ```html
   <html lang="zh" xmlns:th="http://www.thymeleaf.org">
   ```

2. 常用th标签

   - th:text

     ```html
     <div th:text="${name}">name</div>
     ```

     显示控制器传入的name值

   - th:object

==再议==

#### 控制器

处理由DispacherServlet分发过来的请求，把用户请求的数据通过业务处理封装成一个Model，返回给对应的View进行展示

**请求方法：**

- DELETE

  执行后返回的为void类型，对应的Http状态码为204 no content

- PUT

  对象需要更新，通过put方法发送请求

- PATCH

  为一个新引入的方法，是对PUT方法的补充，对已知局部资源进行更新

  如一个User有id，name，age，email等，只修改其中的部分信息，则可以使用PATCH方法，而不是使用PUT来进行一个完整的对象的提交，避免了资源的浪费

- OPTIONS

  用于获取当前的URL，请求成功，则在HTTP的头中中包含一个allow的头，其中为支持的方法，如GET，POST等，还允许客户端查看服务器的性能，如果遇到500错误，OPTIONS不进行第二次请求

- TRACE

  显示服务器收到的请求，主要用于测试或诊断

**参数获取方式：**

- 从路径中获取

  {}包含传入的参数

  ```java
  @RestController
  @RequestMapping(path = "/articles/{slug}")//
  public class ArticleApi {
  ```

  @PathVariable从中获取并注入形参

  ```java
  @GetMapping
      public ResponseEntity<?>article(
              @PathVariable("slug") String slug,
              @RequestHeader(value ="Authorization")String token)
  ```

- 通过Bean接收

  ```java
  public void addUser(User user)
  ```

- HttpServletRequest接收参数

  ```java
  public ResponseEntity tst(HttpServletRequest request){
          String xx = request.getParameter("xx");
      }
  ```

- @RequsetParam绑定入参

  ```java
  @GetMapping(path = "exact")
      public ResponseEntity getArticles(
              @RequestParam(value = "author",required = false) String author) 
  ```
  
  `value`指定要传入的参数名称
  
  `required=false`指定非必须的，不传入不会发生异常
  
- @RequestBody接收JSON数据
  
  ```java
  @PostMapping
      public ResponseEntity createArticle(
              @Valid @RequestBody NewArticleParam newArticleParam,
              @RequestHeader(value = "Authorization")String token)
  ```
  
- 上传文件
  
  ```java
  private String uploadFile(@RequestParam("file") MultipartFile file,HttpServletRequest request){
          String format=sdf.format(new Date());//以当前时间创建文件存储路径，便于进行管理
          File folder=new File(uploadPath+format);
      //得到实际的存储路径，不存在则递归创建
          if(!folder.isDirectory()){
              folder.mkdirs();
          }
      //防止上传的文件重名，通过UUID得到新的不重复的文件名
          String oldName=file.getOriginalFilename();
          String newName= UUID.randomUUID()
                  .toString()+oldName
                  .substring(oldName.lastIndexOf("."),oldName.length());
          try {
              //保存文件
              file.transferTo(new File(folder,newName));
          } catch (IOException e) {
              e.printStackTrace();
          }
      //返回文件的访问路径，用于给前端直接访问获取
          return request.getScheme()+"://"+request.getServerName()
                  +":"+request.getServerPort()+"/img"+format+newName;
  
      }
  
  ```
  
  直接使用MutipartFile中的transferTo进行文件保存即可，不必再去操作字节流
  
  MutipartFile[]数据可以处理多文件上传，遍历数组调用封装好的单文件处理方法即可
  
#### 模型

模型在MVC设计模式中即为一个实体Bean,代表一个存取数据的对象或者POJO,暂存数据在内存当中

可以时数据库表对应的实体类,也可以是经过了一定逻辑处理重新进行封装的实体类

## 进阶篇

### Springboot 进阶

#### AOP

  ==动态代理和AOP先欠下==

#### IOC

控制反转，即为将对象的创建权交给Spring框架

控制反转的实质为获得依赖对象的过程被反转，由本来的自己创建并管理变为通过IOC容器进行依赖注入

实际操作中只需要从IOC容器中获取要使用的对象，不需要关心创建的过程

**依赖注入的方式：**：

- 构造器注入
- setter注入
- field注入

[为什么IDEA不推荐你使用@Autowired ？ - 掘金 (juejin.cn)](https://juejin.cn/post/7026916446370267172?share_token=7ff337cc-7300-4bf0-9b0f-0d64ca146d47)

  #### Servlet

在springboot 开发时，controller能满足大部分业务需求，但是在某些场合依然需要用到servlet

- 注册servlet

  通过`@webSevlet进行servlet的注册`

  ```java
  @WebServlet(name = "/Servlet")
  public class Servlet extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  
      }
  
      @Override
      protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          this.doGet(request, response);
      }
  }
  ```

- 开启servlet

  在入口类上使用注解`@ServletComponentScan`使servlet生效

此后便可以访问`localhost:8080/Servlet`

#### 自动配置

springboot的启动类被@SpringbootApplication注解所修饰，而@SpringbootApplication注解又被以下三个注解所修饰

```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

即springboot提供了统一的注解来修饰这三个注解

`SpringApplication.run(ThymeleafApplication.class, args)`

通过调用静态方法run，实例化了一个SpringApplication对象，执行run方法

**自动配置的原理：

`@EnableAutoConfiguration`注解通过`@Import({AutoConfigurationImportSelector.class})`将所有符合`@Configuration`的配置加载到IOC容器当中

##### 元注解

元注解：包括@Retention、@Target、@Document、@Inherited四种。

- @Retention：指明在什么级别显示此注解

  -  @Retention(RetentionPolicy.SOURCE)  // 注解仅存在于源码中，编译阶段丢弃，在class字节码文件中不包含，如`@Override`等注解
  -  @Retention(RetentionPolicy.CLASS)   // 默认的保留策略，注解会在class字节码文件中存在，类加载时丢弃，因此运行时无法获得
  -  @Retention(RetentionPolicy.RUNTIME)  // 始终不会丢弃，运行期中也保留改注解，注解会在class字节码文件中存在，在运行时可以通过反射获取到

- @Target：

  告诉java自定义的注解放在什么地方，指定作用目标，可以传入一个值，也可传入一个数组指定多个目标，主要有以下几种：

  > ElementType.TYPE 用于类，接口，枚举但不能是注解
  >
  > ElementType.FIELD 作用于字段，包含枚举值
  >
  > ElementType.METHOD 作用于方法，不包含构造方法
  >
  > ElementType.PARAMETER 作用于方法的参数
  >
  > ElementType.CONSTRUCTOR 作用于构造方法
  >
  > ElementType.LOCAL_VERIABLE 作用于本地变量或者catch语句
  >
  > ElementType.ANNOTATION_TYPE 作用于注解
  >
  > ElementType.PACKAGE 作用于包

- @Documented：

  指明拥有这个注解的元素可以被javadoc此类的工具文档化。这种类型应该用于注解那些影响客户使用带注释的元素声明的类型。

- @Inherited：

  指明该注解类型被自动继承。如果用户在当前类中查询这个元注解类型并且当前类的声明中不包含这个元注解类型，那么也将自动查询当前类的父类是否存在Inherited元注解，这个动作将被重复执行知道这个标注类型被找到，或者是查询到顶层的父类。

  如果一个使用的`@Inherited`修饰的Annotation类型被用于一个Class，则这恶Annotation将被用于该Class的子类，即为自动继承

#### 单元测试

在测试类上添加`@Transactional`注解，测试完就会回滚，避免产生垃圾数据，如果要关闭回滚，则只需要使用`@Rollback(false)`即可

### REST

