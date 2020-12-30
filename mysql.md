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
# 数据文件
- frm 存放表结构
- myd 存放数据
- myi 存放索引
# 结构
