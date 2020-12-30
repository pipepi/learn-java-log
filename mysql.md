# my.cnf
```shell
cp /usr/share/mysql/my-huge.cnf /etc/my.cnf
```
## 开启二进制操作日志，用于主从复制
```
[mysql]
port=3306
log-bin=/var/lib/mysql/logbin
```
## 操作日志

```
[mysql]
port=3306
log-err=/var/lib/mysql/logerr
```
