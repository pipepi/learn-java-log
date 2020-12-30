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
      - 行锁（inno支持行级锁，myisam只支持表锁）
      - 缓存（inno会缓存数据，myisam只缓存地址引用）
      - 表空间（inno 大，myisam 小）
      - 关注点（inno 事务，myisam 性能）
      
- level 4 存储
  - FileSystem & Files & Logs
