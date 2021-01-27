# sql总结
- 查询表名称和注解
```sql
select TABLE_NAME,TABLE_COMMENT from information_schema.tables where TABLE_TYPE = 'BASE TABLE' and table_schema = 'digital_marketing_platform';
```
# ？已分表数据分页查询
- 允许精度误差
  - 按查询条件去不同库中获取对应百分比的量，进行汇总
  
# ？每天5000万新增，没有修改，如何选型和规划数据库
- infobright存储引擎，
  - 不支持update
  - 不支持insert
  - 不支持delete
  - 支持 load data infile
  - 高压缩的存储引擎，查询速度快
  - 实现【数据汇总】
    - 按时间划分，利用缓存【redis/mongodb】存储一定时间段的数据，写入文件中
    - 多线程把文件加载到infobright存储引擎中
    - 脚本定期清理文件
# ？月增千万的订单表如何拆分
- 使用场景
  - 用户查询自己的订单
  - 商家查询订单
- 分表原则
  - 尽量避免最大事务边界【查询的sql对应的后端数据库同时执行的量】
- 方案
  - 分表数n 【3-5年额度量的表数量，例如8】
  - 切分点 订单hash对n取模
  - 异构索引表 【根据用户或商家设计中间表】
    - 按用户/商家对异构索引表进行分表
    - 订单表id设计 【时间戳+user+shop+随机数】 【后期维护，可以时间戳归档到历史订单表中】
  
# mysql架构演变
- 单库
- 垂直分表 【初期数据库设计，分离高频查询数据和很少查询的数据】
- 主从 【解决访问量问题，不能解决查询效率问题】
  - 问题
    - 延迟问题 【写入的时候，存一份到redis中】
    - 一致性问题 【pt-table-sync 工具】
- 水平分表 【解决查询效率问题】【范围，时间，hash】
- 垂直分库 【解决查询效率问题】【根据业务分库，微服务】
- 水平分库 【同业务多数据库】

# ?如何进行sql优化
- 开启慢查询日志，根据慢查询日志定位sql
- explain分析sql，查看索引的使用情况
- 结合索引和业务制定优化方案
# 日志文件
- error log 
- query log
- low query log
- bin log
- 隐藏日志
  - 事务日志redo-undu 【只属于innodb，对应会有Mysql物理文件】
  - 重放日志relay log 【只在主从复制中的从节点，用于数据复制】
# 配置
## 配置 my.cnf
```shell
cp /usr/share/mysql/my-huge.cnf /etc/my.cnf
```
## 配置字符集
```
待续
```
## 开启二进制操作日志，用于主从复制
```
[mysql]
port=3306
log-bin=/var/lib/mysql/logbin
```
## 操作日志，默认关闭

```
[mysql]
port=3306
log-err=/var/lib/mysql/logerr
```
## 操作日志，默认关闭，记录慢查询
```
待续
```
## 数据文件
- frm 存放表结构
- myd 存放数据
- myi 存放索引
# 结构
- level 1 连接层
  - connection pool
- level 2 服务层
  - managementService & Utilities
  - caches & buffers
  - sql interface & Parser & Optimizer
- level 3 引擎
  - Pluggable Stroage Engines
    ```sql
    show engines;
    ----------------------------------------------------------------------------------------------------
    "Engine";"Support";"Comment";"Transactions";"XA";"Savepoints"
    "PERFORMANCE_SCHEMA";"YES";"Performance Schema";"NO";"NO";"NO"
    "MyISAM";"YES";"MyISAM storage engine";"NO";"NO";"NO"
    "MRG_MYISAM";"YES";"Collection of identical MyISAM tables";"NO";"NO";"NO"
    "MEMORY";"YES";"Hash based, stored in memory, useful for temporary tables";"NO";"NO";"NO"
    "CSV";"YES";"CSV storage engine";"NO";"NO";"NO"
    "BLACKHOLE";"YES";"/dev/null storage engine (anything you write to it disappears)";"NO";"NO";"NO"
    "ARCHIVE";"YES";"Archive storage engine";"NO";"NO";"NO"
    "FEDERATED";"NO";"Federated MySQL storage engine";NULL;NULL;NULL
    "InnoDB";"DEFAULT";"Supports transactions, row-level locking, and foreign keys";"YES";"YES";"YES"
    ----------------------------------------------------------------------------------------------------
    show variables like '%storage_engine%';

    ```
    - InnoDB
    - Myisam
    - InnoDB vs Myisam
      - 主/外键（inno支持主键和外键，myisam不支持）
      - 事务（inno支持事务，myisam不支持）
      - 行锁（inno支持行级锁【适合**高并发**】，myisam只支持表锁）
      - 缓存（inno会缓存数据，myisam只缓存地址引用）
      - 表空间（inno 大，myisam 小）
      - 关注点（inno 事务，myisam 性能）
    - ali使用 
      - 早期 percona **xtradb**存储引擎
      - 后期 AliSql+AliRedis
      
- level 4 存储
  - FileSystem & Files & Logs
# 索引优化分析
- sql慢
  - 执行时间长或等待时间长
    - 查询语句写的烂
    - 索引失效（单值，复合）
    - 关联太多join
    - 服务器调优和各个参数设置（缓存，线程数等）
