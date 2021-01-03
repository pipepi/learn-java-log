
# 投影 find({},{attr1:1【查询】,attr2:0【排除】})
db.emp.find({},{attr1:1,attr2:0})
# 排序 sor({attr1:1【升序】,attr2:-1【降序】}) 
db.emp.find({}).sor({sal:1})
# 批量插入
- 方案1 【7.2秒】
```shell
for(var i=0 ;i<=2000;i++){
db.numbers.insert({num:i});
}
```
- 方案2 【0.4秒】
```shell
var arr = [];
for(var i=1;i<20000;i++){
  arr.push({num:i});
}
db.numbers.insert(arr);
```
# 内嵌文档
- 文档属性的值为文档叫内嵌文档
- 按内嵌文档值查询时，key可以用.级联选择，但必须加引号，代表表达式。
  - db.users.find({'hobby.movies':"hello"})
# 查询操作符
- 比较
  - $gt
  - $gte
  - $lt
  - $lte
  - $eq
  - $ne
  - {$gt:40,$lt:50}
- 逻辑
  - $or:[{sal:{$lt:1000}},{sal:{$gt:2500}}] # 工资大于2500或小于1000

# 修改修改器/操作符
- obj类
  - $set
  - $unset
  - $min
  - $max
  - $inc
  - $sub
  - $mul
- array类
  - $push      直接添加，不考虑重复
  - $addToSet  如果重复/已有，就不添加
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
db.xxx.find({}).limit(10) # 查询前10条 
db.xxx.find({}).skip(10).limit(10) # 分页查询10条后的10条数据 ，skip,limit顺序无关，mongo自动调整

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
