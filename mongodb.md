# 启动命令
```shell
mongod --dbpath xxxx --port 27017
```
# mongod.cfg 【放到安装mongo的根目录，与bin目录同级】
```yaml
systemLog:
  destination: file
  path: c:\data\log\mongod.log
storage:
  dbPath: c:\data\db
```
