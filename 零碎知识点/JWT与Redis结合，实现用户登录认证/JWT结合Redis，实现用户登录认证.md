# JWT结合Redis，实现用户登录认证

## JJWT

JJWT是一个提供端到端的JWT创建和验证的Java库。永远免费和开源(Apache License，版本2.0)，JJWT很容易使用和理解。它被设计成一个以建筑为中心的流畅界面，隐藏了它的大部分复杂性。

依赖坐标如下：

```xml
 <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.7.0</version>
</dependency>
<!--注：不可导成jjwt-api-->
```

## JWT的生成与解析

首先设置一个Key，用于生成签名，这里直接写在配置文件当中，在初始化时在进行注入

接口类JwtService：

```java
package com.fischer.jwt;

public interface JwtService {
    String toToken(User user);

    Optional<String> getSubFromToken(String token);
}

```

实现类DefaultJwtService：

- 生成Token：
  - 在subject设置为user的Id，用于进行用户的标识，解析后即可得到该用户的id
  - 过期时间设置为一天之后
  - 使用之前配置的secret Key加密生成签名（secret应当妥善保管不泄露）

```java
package com.fischer.jwt;

@Component
public class DefaultJwtService implements JwtService{
    private String secret;
    private int sessionTime;
    @Autowired
    public DefaultJwtService(@Value("${jwt.secret}") String secret,
                             @Value("${jwt.sessionTime}") int sessionTime) {
        this.secret = secret;
        this.sessionTime = sessionTime;
    };
    @Override
    public String toToken(User user) {
        String token = Jwts.builder()
                .setSubject(user.getId())
                .setExpiration(expireTimeFromNow())
                .signWith(SignatureAlgorithm.HS256, secret)
                .compact();
        return token;
    }
    @Override
    public Optional<String> getSubFromToken(String token) {
        try {
            Jws<Claims> claimsJws = Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
            //
            return Optional.ofNullable(claimsJws.getBody().getSubject());
        } catch (Exception e){
            return Optional.empty();
        }
    }

    private Date expireTimeFromNow(){
        return new Date(System.currentTimeMillis()+sessionTime*1000);
    }
}
```

得到的Token：

```token
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0IiwiZXhwIjoxNjQ2OTEzOTk4fQ.HEfBEeS1-3ZOJsMeWk6xMzEORsLvfxQutNbDiReZOtg
```

解析结果便可以得到id，以下是第三方网站的解析结果：

![image-20220309202230913](JWT结合Redis，实现用户登录认证.assets/image-20220309202230913.png)



## 配合Redis实现登录认证

在redis中使用层次键值对的方式记录是否登录，loginUser管理所有已经登录的用户，只要能查询到key即证明已经成功登录，因此value可以随意设置，此处设置为1

<img src="JWT结合Redis，实现用户登录认证.assets/image-20220312111957772.png" alt="image-20220312111957772" style="zoom:67%;" />



==实体类：==

```java
@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String id;
    private String email;
    private String username;
    private String password;
    private String bio;
    private String image;

    public User(String email,String username,String password,String bio,String image)
    {
        this.id= UUID.randomUUID().toString();
        this.email=email;
        this.username=username;
        this.password=password;
        this.bio=bio;
        this.image=image;
    }
}
```



==登录表单：==

```java
@Getter
@NoArgsConstructor
@JsonRootName("user")
public class LoginParam {
    @Email(message = "请填写格式正确的邮箱")
    @NotBlank(message = "邮箱不能为空")
    private String email;
    @NotBlank(message = "密码不能为空")
    private String password;

}
```

### 拦截器

==拦截器配置类：==

- 因为拦截器的加载时间在Springcontext之前，直接在拦截器中注入则会为null，产生空指针异常，因此我们需在配置类加载的同时加载拦截器
- 放行`user/login`,其他全部进行拦截

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    //加载拦截器
    @Bean
    public MyInterceptor myInterceptor(){
        return new MyInterceptor();

    }
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(myInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/user/login");

    }
}
```

==拦截器：==

- 注入JwtService用于解析token，StringRedisTemplate用于操作Redis，存放和查询token对应的id，判断是否登录
- 处理以下三种情况的异常：
  - 未携带token
  - token解析失败(此处只处理了针对负载payload解析失败的情况，对于secret key部分的解析校验有待后续完善补充)
  - 查询不到该token对应的id（未登录）的三种情况，抛出异常

  能正确查询到则放行，进行资源访问

```java
public class MyInterceptor implements HandlerInterceptor {
    @Autowired
    private JwtService jwtService;
    @Autowired
    private StringRedisTemplate redisTemplate;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("token");

        if (token==null){

            throw new BizException(HttpStatus.UNAUTHORIZED,"未携带Token");
        }
        Optional<String> temp = jwtService.getSubFromToken(token);

        if(!temp.isPresent()){
            throw new BizException(HttpStatus.UNAUTHORIZED,"Token解析失败！请联系管理员");
        }
        String userId="loginUser:"+temp.get();
        String s = redisTemplate.opsForValue().get(userId);
        if(s==null){
        throw new BizException(HttpStatus.UNAUTHORIZED,"未登录!请先登录");
        }
        return true;

    }
}
```



### Controller

#### 登录

```java
package com.fischer.api;
@RestController
@RequestMapping("user")
@Validated
public class RegisterApi {
    @Autowired
    JwtService jwtService;

    @PostMapping
    public ResponseEntity userLogin(@Valid @RequestBody LoginParam loginParam){
        //实际开发中应为从数据库中查询信息进行校验，此处直接简单模拟
        String email="1809327837@qq.com";
        String password="1234";
        User user=new User(
                email,"fischer",password,"hey","noimage");

        if(email.equals(loginParam.getEmail())&&password.equals(loginParam.getPassword())){
            System.out.println("登录成功");

            //登录成功后，签发token
            String token = jwtService.toToken(user);
            //将用户信息携带token返回给前端，用于进行其他操作时的认证
            UserWithToken userWithToken=new UserWithToken(user,token);
            return ResponseEntity.ok(userWithToken);

        }
        else{
            //正常开发过程中可以选择抛出异常，给异常处理类去捕捉处理，此处随意返回一下
            return ResponseEntity.ok("用户名或者密码错误，登陆失败");
        }
    }
}
```



具体过程如下：

- 从前端接收到表单数据，首先使用Validation进行表单数据校验
- 从数据库按email查询出user，进行信息校验
- 若通过校验，则调用JwtService签发token
- 将User信息与token返回给前端，用于在进行登录后的其他操作时的认证与授权

返回结果：

![image-20220312121313074](JWT结合Redis，实现用户登录认证.assets/image-20220312121313074.png)

登录之后得到的Token：

```java
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxODE2NDdmYS1kMmQ2LTQ3ZDctYTJhNi0xNTExYTYwZGRhYjgiLCJleHAiOjE2NDcxNzYwNTF9.0GweP8eDwqC0khJQyvPbNDHiefl1zdVAnIa5F6vhymA

```

解析结果：

![image-20220312225455459](JWT结合Redis，实现用户登录认证.assets/image-20220312225455459.png)

#### 登录后访问

```java
@GetMapping
    public ResponseEntity getId(HttpServletRequest request){
        String token = request.getHeader("token");
        System.out.println(token);
        Optional<String> id = jwtService.getSubFromToken(token);
        return ResponseEntity.ok(id.get());

    }
```

携带登录后返回的token进行访问，拦截器正常放行，访问到了getId，结果如下：

![image-20220312231647791](JWT结合Redis，实现用户登录认证.assets/image-20220312231647791.png)

==未携带Token时的情况：==

![image-20220312232354749](JWT结合Redis，实现用户登录认证.assets/image-20220312232354749.png)

==携带错误的Token：==

![image-20220312232653039](JWT结合Redis，实现用户登录认证.assets/image-20220312232653039.png)

#### 注销

```java
@DeleteMapping
    public ResponseEntity logout(HttpServletRequest request){
        String token = request.getHeader("token");
        Optional<String> id = jwtService.getSubFromToken(token);
        String deleteId="loginUser:"+id.get();
        redisTemplate.delete(deleteId);
        return ResponseEntity.noContent().build();

    }
```

![image-20220312225254903](JWT结合Redis，实现用户登录认证.assets/image-20220312225254903.png)

![image-20220312225515431](JWT结合Redis，实现用户登录认证.assets/image-20220312225515431.png)

Redis中已经不存在以该用户的key，成功删除

再次携带该token尝试访问登录后才能访问的内容，

在拦截器中解析token后得到用户的id，发现Redis中已经不存在该用户的id，抛出未授权异常，并且能够被自定义的全局处理类给正确捕捉到，返回给前端

结果如下：

![image-20220312225728179](JWT结合Redis，实现用户登录认证.assets/image-20220312225728179.png)

