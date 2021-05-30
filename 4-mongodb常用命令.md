# mongodb常用操作命令

## 下载

mongodb下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[下载地址](https://www.mongodb.com/try/download/community/)  



- 常用操作命令
```
use test 新建test库
show dbs 查询所以库,如果没有collections就不会显示
db 显示当前所在的库
db.dropDatabase删除当的前库
db.shutdownServer()关闭数据库 use admin可以在127.0.0.1登陆进来才关闭


db.test1.insert()在test1中插入数据
db.test1.find()查询test1中的数据
db.test1.remove()在test1中删除数据
db.test1.drop()删除test1

db.test1.count()查询test1行数
db.test1.dataSize()查询test1大小
db.test1.stats()查询test1状态信息
db.test1.totalIndexSize()查询test1索引大小默认16K


db.serverStatus().connections;当前数据库的连接信息


for(i=1;i<=3000;i++) db.test1.insert({uid:i,name:'hukey',age:23})
db.test1.createIndex({"uid":1}); #建立索引  1表示升序,-1表示倒序
db.test1.find({"uid":"test1"}).explain(); #查看索引
db.test1.dropIndex({"uid":1}); #删除索引
for(i=1;i<=300;i++) db.test1.insert({uid:i,name:"test"+i,age:i})

```
- 管理命令
```
use admin
db.createUser({user:"admin",pwd:"123456",roles:["root"]})  创建一个超级用户
db.getUser("root")
db.getUsers()
```



- MongoDB基本的角色
```
1.数据库用户角色：read、readWrite;
2.数据库管理角色：dbAdmin、dbOwner、userAdmin；
3.集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4.备份恢复角色：backup、restore；
5. 所有数据库角色： readAnyDatabase 、readWriteAnyDatabase 、
userAdminAnyDatabase、dbAdminAnyDatabase
6.超级用户角色：root，__system（提供数据库所有对象任何操作的权限，不能分配给用户，
```

- mongodb常用的工具
```


```