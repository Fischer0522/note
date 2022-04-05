# docker部署springboot+mysql+redis

最近将自己的后端项目部署到云服务器上，因此整理了从安装docker到项目部署的全过程，仅供参考

**docker命令：**

[Docker 命令大全 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-command-manual.html)

## docker的下载与安装

安装的具体过程主要参考官方即可，此处我选的是centos版本

[Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/)

具体安装步骤如下：

- 移除旧版本的内容，如果没安装此步跳过也可

  ```shell
   sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-engine

- gcc相关配置：

  ```shell
  yum -y install gcc
  ```

  ```shell
  yum -y install gcc-c++
  ```

- 安装`yum-utils`以及稳定的仓库，由于官网给出的仓库连接较慢，因此我们此处使用阿里云的镜像

  ```shell
   sudo yum install -y yum-utils
   sudo yum-config-manager \
      --add-repo \
      http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```

- 安装docker

  ```shell
  sudo yum install docker-ce docker-ce-cli containerd.io
  ```

- 启动docker

  ```shell
  sudo systemctl start docker
  ```

- 测试docker

  ```shell
  sudo docker run hello-world
  ```

  由于本地不存在hello world的镜像资源，因此会自动去仓库拉取，当能够看到hello world正常运行时，证明docker已经成功安装

![image-20220326101242769](docker部署springboot+mysql+redis.assets/image-20220326101242769.png)

## Docker应用部署 

### mysql

1. 搜索mysql镜像

```shell
docker search mysql
```

2. 拉取mysql镜像

```shell
docker pull mysql:5.6
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```shell
docker run -id \
-p 3307:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql5.7_utf8mb4

```

- 参数说明：
  - **-p 3307:3306**：将容器的 3306 端口映射到宿主机的 3307 端口。
  - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
  - **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
  - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。



4. 进入容器，操作mysql

```shell
docker exec –it c_mysql /bin/bash
```

5. 使用外部机器连接容器中的mysql，此处使用的是navicat

![image-20220326102639831](docker部署springboot+mysql+redis.assets/image-20220326102639831.png)

此后边与操作本地的mysql无异

### redis

1. 搜索redis镜像

```shell
docker search redis
```

2. 拉取redis镜像

```shell
docker pull redis:6.0.8
```

3. 配置redis.conf文件

   从redis官网找一个redis.conf文件

   - 设置密码

     ```
     requirepass 1234
     ```

   - 允许redis外地连接

     注释掉`#bind 127.0.0.1` 

   - daemonize no

     设置daemonize 为no，否则会和docker run -d冲突，导致容器启动失败

4. 创建容器，设置端口映射

```shell
docker run -p 6379:6379 --name c_redis --privileged=true -v /root/app/redis/redis.conf:/etc/redis/redis.conf -v /root/app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
```

- --privileged=true使container内的root拥有真正的root权限
- -v指定自己的redis配置文件映射到默认的redis.conf
- -v 设置数据卷，防止docker容器挂掉
- redis-server服务的启动去读取容器中已经被修改过的conf文件

5. 使用外部机器连接redis

```shell
redis-cli.exe -h 42.192.180.126 -p 6379
```

此处使用RDM添加键进行测试不知为何没有效果，最后仍采用redis-cli进行操作测试

```shell

```



## 项目部署

### demo

首先编写一个小demo用于测试

- 依赖

  ```xml
  <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <optional>true</optional>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
          <!-- https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter -->
          <dependency>
              <groupId>com.baomidou</groupId>
              <artifactId>mybatis-plus-boot-starter</artifactId>
              <version>3.5.1</version>
          </dependency>
          <dependency>
              <groupId>org.apache.commons</groupId>
              <artifactId>commons-pool2</artifactId>
          </dependency>
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>druid</artifactId>
              <version>1.2.8</version>
          </dependency>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <scope>runtime</scope>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-redis</artifactId>
          </dependency>
  
  
      </dependencies>
  
  ```

- pojo:

  ```java
  @AllArgsConstructor
  @NoArgsConstructor
  @Data
  @TableName(value = "users")
  public class User {
      private String id;
      private String email;
      private String username;
      private String password;
      private String bio;
      private String image;
  }
  
  ```

- 供mybatis-plus用于查询的UserDao

  ```java
  @Mapper
  public interface UserDao extends BaseMapper<User> {
  }
  
  ```

- controller:

  分别对mysql和redis进行测试，做一个简单的查询：

  ```java
  @RestController
  public class UserApi {
  
      @Autowired
      UserDao userDao;
      @Autowired
      StringRedisTemplate redisTemplate;
  
      @GetMapping("user")
      public ResponseEntity getUser(){
          List<User> users = userDao.selectList(null);
          return ResponseEntity.ok(users);
      }
  
      @GetMapping("name")
      public ResponseEntity getname(){
          String s = redisTemplate.opsForValue().get("name");
          return ResponseEntity.ok(s);
  
      }
  }
  
  ```

### 打包

在本地测试正常后，使用maven进行打包，同时编写dockerfile用于生成镜像资源：

==注：不要忘记将配置文件中的localhost换成部署的服务器的ip==

- 打包：看到build success即可，在target目录中会生成一个jar包

<img src="docker部署springboot+mysql+redis.assets/image-20220326103724950.png" alt="image-20220326103724950" style="zoom:67%;" />



<img src="docker部署springboot+mysql+redis.assets/image-20220326103744151.png" alt="image-20220326103744151" style="zoom:67%;" />

- dockerfile

```do
FROM java:8
COPY *.jar /app.jar

CMD ["--server.port=8080"]

#定义时区参数
ENV TZ=Asia/Shanghai

#设置时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo '$TZ' > /etc/timezone

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]

```

- 依托于父镜像java8，可提前使用docker pull进行下载，不提前下载也会自动下载
- 将同目录下所有的jar包用于构建镜像
- docker默认使用的为UTC时区，会和本地差8小时，并且由于docker的隔离性，单独对宿主机设置时间并无效果，因此需要在dockerFile中设置相应的时区
- 暴露8080端口用于访问
- 运行java -jar

### 部署

将jar包与dockerfile上传到服务器的同一个目录下，运行：

```shell
docker bulid -t demo .
```

demo为名字

最后一定要加上`.`

![image-20220326104514351](docker部署springboot+mysql+redis.assets/image-20220326104514351.png)

此时便可以看到镜像已经制作成功

![image-20220326104539212](docker部署springboot+mysql+redis.assets/image-20220326104539212.png)

运行：

```
docker run -d -p 9000:8080 -v /root/blog/static/img:/root/blog/static/img --link c_mysql --link c_redis --name blogdemo1 blogdemo1 
```

- -p指定端口映射
- --link使该容器与redis和mysql相连
- -v 挂载数据卷，将上传的图片保存到宿主机当中

运行该容器并且与mysql和redis容器通过`--link`命令进行互联，至此，容器已经可以成功运行

![image-20220326104901538](docker部署springboot+mysql+redis.assets/image-20220326104901538.png)

![image-20220326104918513](docker部署springboot+mysql+redis.assets/image-20220326104918513.png)

redis与mysql均可以正常运行

