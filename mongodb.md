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
db.<collection name>.insert(doc_json) # 向集合中插入一个【{}】或多个【数据[]】文档  ；没添加_id字段【保证唯一】，mongodb自动生成
db.xxx.insertOne({})
db.xxx.insertMany([])

ObjectId() # 查询唯一id

db.<collection name>.find({key:value}) # 查询集合中符合查询条件的所有文档 ，返回数组[]
db.xxx.findOne({k:v}) # 查询第一个，返回对象{}
db.xxx.find({}).count() or db.xxx.find({}).length() # 查询文档数

db.xxx.update({}【查询条件】,{}【新对象】,{upsert:<bool>,multi:<bool>,writeConcern:<document>,collation:<document>}【修改选项】) # 按查询条件修改为新对象 。默认使用新对象替换旧对象【完全覆盖，可能丢失属性】。默认只修改第一个
db.xxx.update({}【查询条件】,{$set:{}}【新对象】) # 修改操作符【$set【修改或添加属性】: $unset【删除属性】:】，按属性修改
db.xxx.updateMany()
db.xxx.updateOne()
db.xxx.replaceOne()

db.xxx.remove({k:v},false【默认删除多个】|true【只删除第一个】) # 根据条件删除文档
db.xxx.remove({}) # 删除集合中全部文档，性能差 
db.xxx.drop() # 删除集合
db.dropDatabase() # 删除数据库
db.xxx.remove() # 非法
db.xxx.deleteOne()
db.xxx.deleteMany()
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