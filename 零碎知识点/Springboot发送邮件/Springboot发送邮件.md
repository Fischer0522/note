# Springboot发送邮件

## 依赖

springboot对其进行了整合，导入一个starter即可

```java
			<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

## 配置

正常情况来说直接使用yml配置文件即可，但是在我使用的时候会出现**25端口连接失败的问题**，但是在代码中配置又莫名解决了，

- yml配置

  ```yaml
  spring:
    mail:
      # 配置 SMTP 服务器地址
      host: smtp.qq.com
      # 发送者邮箱
      username: 1809sdas7@qq.com
      # 配置密码，注意不是真正的密码，而是刚刚申请到的授权码
      password: xxxxxxx
      # 端口号465或587
      port: 465
      # 默认的邮件编码为UTF-8
      default-encoding: UTF-8
      # 配置SSL 加密工厂
      properties:
        mail:
          smtp:
            port: 25
            socketFactoryClass: javax.net.ssl.SSLSocketFactory
          #表示开启 DEBUG 模式，这样，邮件发送过程的日志会在控制台打印出来，方便排查错误
          debug: true
      protocol: smtp
  ```

- 代码设置

  发送一段文本文件

  ```java
   JavaMailSenderImpl javaMailSender = new JavaMailSenderImpl();
          SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
          javaMailSender.setUsername("fischer0522@163.com");
          javaMailSender.setPassword("key");
          javaMailSender.setHost("smtp.163.com");
          simpleMailMessage.setFrom("fischer0522@163.com");
          simpleMailMessage.setTo("1809327837@qq.com");
          simpleMailMessage.setSubject("测试用例");
          simpleMailMessage.setText("收到收到");
          javaMailSender.send(simpleMailMessage);
  ```

为了避免配置文件与代码耦合，又不能直接使用spring的配置，因此在yml中自定义配置，在注入即可

```java
@Value("${mail.username}")
    private String username;
    @Value("${mail.host}")
    private String host;
    @Value("${mail.password}")
    private String password;

    
    @Test
    void contextLoads() {
        JavaMailSenderImpl javaMailSender = new JavaMailSenderImpl();
        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
        javaMailSender.setUsername(username);
        javaMailSender.setPassword(password);
        javaMailSender.setHost(host);
        simpleMailMessage.setFrom(username);
        simpleMailMessage.setTo("1809327837@qq.com");
        simpleMailMessage.setSubject("测试用例");
        simpleMailMessage.setText("收到收到");
        javaMailSender.send(simpleMailMessage);
    }
```

```yaml
mail:
  # 配置 SMTP 服务器地址
  host: smtp.163.com
  # 发送者邮箱
  username: fischer0522@163.com
  # 配置密码，注意不是真正的密码，而是刚刚申请到的授权码
  password: FWSsdsdasasdLYB
  # 端口号163默认为 465 994
  port: 465
  # 默认的邮件编码为UTF-8

```

