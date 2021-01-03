# 常用命令
- 基本命令，查询切换数据库或集合
```shell
show dbs  or show databases # 显示数据库列表
use <dbname> # 使用【进入】或创建【在第一次添加文档的时候，自动创建不存在的数据库和集合】数据库
db # 查询当前所在的数据库
show collections # 显示数据库中的所有集合
```
- CRUD命令
```shell
db.<collection name>.insert(doc_json) # 向集合中插入文档
db.<collection name>.find() # 查询集合中的所有文档
```
# 存储结构
- 数据库【库】
- 集合【表】
- 文档【记录】
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
