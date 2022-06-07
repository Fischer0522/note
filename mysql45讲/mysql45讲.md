# mysql45讲



## 基础篇

### 基础架构

**mysql基本架构示意图**：

![img](mysql45%E8%AE%B2.assets/0d2070e8f84c4801adbfa03bda1f98d9.png)

**连接器**

```mys
mysql -h$ip -P$port -u$user -p
```

- 若不指定ip与端口，则默认为localhost的默认端口(3306)
- tcp握手后便进行身份验证，校验用户与密码
- 用户密码认证通过后，会从权限表中查询该连接对应的权限，并且，此次连接后，即使管理员修改了该用户的权限，此次连接的权限也不变，建立新的连接后才会使用新的权限设置

**长连接/短链接**

长连接为连接成功，客户端持续请求都使用这一个连接

短链接为执行很少几次的查询断开连接，下次查询重新建立连接

建议使用长连接，但是长链接过多时会导致资源占用过多而不被释放，最后导致OOM重启

解决方法：

- 定期断开长连接
- mysql_reset_connection重新初始化连接资源(mysql 5.7及以上)

**查询缓存**

mysql将执行过的语句和查询的结果以key-value的形式存储

- 查询后若缓存命中则直接返回结果
- 每次数据更新导致缓存被清空，因此命中率较低
- 适用于静态查询表
- mysql8.0缓存被删除

**分析器**

词法分析->语法分析：

- 词法分析识别出其中的字符串是什么
- 语法分析检验语法

**优化器**

- 多个索引的情况下选择索引的使用
- join的先后顺序

经过优化器之后，便可以确定执行方案，交给执行器进行执行

**执行器**

首先判断权限，是否有查询该表的权限

之后根据表的引擎定义，使用这个引擎对应的接口

### 日志系统

与查询不同的是，更新语句还涉及日志模块，分别为redo log 和 bin log

**redo log**

MYSQL中的Write-Ahead Logging技术：先写日志，再写磁盘

更新是InnoDB先将记录写道rodo log中，再空闲时再从redo log中写入到硬盘之中

与此类似，InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么这块“粉板”总共就可以记录 4GB 的操作。从头开始写，写到末尾就又回到开头循环写

![img](mysql45%E8%AE%B2.assets/16a7950217b3f0f4ed02db5db59562a7.png)

write pos处写入，check point处擦除，当write pos追上check point时，即为已经写满，需要将一部分写入磁盘，推荐check point

有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 crash-safe。

**bin log**

最开始 MySQL 里并没有 InnoDB 引擎。MySQL 自带的引擎是 MyISAM，但是 MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档。

binlog与redolog的不同：

- redolog InnoDB特有，binlog为mysql server层实现的 任何引擎都可以使用
- redolog为物理日志，记录在某个数据页上做了什么修改，binlog为逻辑日志，记录语句的原始逻辑 statement格式记录sql语句，row格式记录行的内容
- redo log循环写，空间大小固定会用完，binlog可切换新的文件进行追加
- 备份时即为备份 binlog 用于归档

**执行过程：两阶段提交**

```mysql
mysql> update T set c=c+1 where ID=2;
```

![img](mysql45%E8%AE%B2.assets/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png)

通过两阶段提交，全部写入成功后再提交事务，保证两份日志的逻辑一致

若不使用两阶段提交：

- 先redo log后bin log：crash后redo log进行数据恢复，值被修改，但是备份中的值未被修改
- 先bin log后 redo log：crash后成功备份但是当前值为改变

 
