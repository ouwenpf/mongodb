# 专业环mongodb境安装

## 下载

mongodb下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[下载地址](https://www.mongodb.com/try/download/community/)  



## 安装前准备

- 安装依赖
```
yum install -y cyrus-sasl cyrus-sasl-plain cyrus-sasl-gssapi krb5-libs rpm-libs tcp_wrappers-libs lm_sensors-libs net-snmp-agent-libs net-snmp openssl  openssl-devel 

相关参数
sysctl -w vm.zone_reclaim_mode=0
echo never > /sys/kernel/mm/transparent_hugepage/enabled 
echo never > /sys/kernel/mm/transparent_hugepage/defrag

cat >> /etc/security/limits.conf << EOF
mongodb         hard    nofile        25600
mongodb         soft    nofile        25600
mongodb         hard    nproc         25600
mongodb         soft    nproc         25600

EOF
```

- 

- MongoDB分布式集规群服务器规划

```
10.10.10.100: mongos(27017) mongos(27018) mongos(27019)
10.10.10.101:config(27017) config(27018) config(27019)
shard1:10.10.10.11:27017(主节点),10.10.10.12:27017(副节点),10.10.10.13:27017(仲裁节点)
shard2:10.10.10.14:27017(主节点),10.10.10.15:27017(副节点),10.10.10.16:27017(仲裁节点)
shard3:10.10.10.17:27017(主节点),10.10.10.18:27017(副节点),10.10.10.19:27017(仲裁节点)


```

- 部署config配置服务器

```
mongo 10.10.10.100:27017

1.创建复制集
config={_id:"config",members:[
{_id:0,host:"10.10.10.100:27017"},
{_id:1,host:"10.10.10.100:27018"},
{_id:2,host:"10.10.10.100:27019"}
]}
2.初始化复制集
rs.initiate(config)
3.检查状态
rs.status()

```

- 部署分片shard服务器
```
shard1:10.10.10.11:27017(主节点),10.10.10.12:27017(副节点),10.10.10.13:27017(仲裁节点)
shard2:10.10.10.14:27017(主节点),10.10.10.15:27017(副节点),10.10.10.16:27017(仲裁节点)
shard3:10.10.10.17:27017(主节点),10.10.10.18:27017(副节点),10.10.10.19:27017(仲裁节点)


创建shard1复制集
use admin
config={_id:"shard1",members:[
{_id:0,host:"10.10.10.11:27017",priority:2},
{_id:1,host:"10.10.10.12:27017",priority:1},
{_id:2,host:"10.10.10.13:27017",arbiterOnly:true}
]}
初始化复制集
rs.initiate(config)
{ "ok" : 1 }表示成功的标准
检查状态
rs.status()


创建shard2复制集
config={_id:"shard2",members:[
{_id:0,host:"10.10.10.14:27017",priority:1},
{_id:1,host:"10.10.10.15:27017",priority:2},
{_id:2,host:"10.10.10.16:27017",arbiterOnly:true}
]}
初始化复制集
rs.initiate(config)
{ "ok" : 1 }表示成功的标准
检查状态
rs.status()


创建shard3复制集
config={_id:"shard3",members:[
{_id:0,host:"10.10.10.17:27017",priority:1},
{_id:1,host:"10.10.10.18:27017",priority:2},
{_id:2,host:"10.10.10.19:27017",arbiterOnly:true}
]}
初始化复制集
rs.initiate(config)
{ "ok" : 1 }表示成功的标准
检查状态
rs.status()

注意vim复制粘贴导致多行出现#号解决办法
:set paste
```

- 部署路由服务器
```
参数中的"config"需要对应config集群中的参数replSet=config这个值
configdb =config/10.10.10.100:27017,10.10.10.100:27018,10.10.10.100:27019

mongos  -f  /data/mongodb/mongodb27017/conf/mongodb-route.conf 
mongos  -f  /data/mongodb/mongodb27018/conf/mongodb-route.conf 
mongos  -f  /data/mongodb/mongodb27019/conf/mongodb-route.conf  
三台服务器操作一致,*注意*这里是"mongos"而非"mongod""
```

- 启用分片功能
```
登录路由节点:mongo 10.10.10.101:27017
use admin
sh.addShard("shard1/10.10.10.11:27017,10.10.10.12:27017,10.10.10.13:27017")
sh.addShard("shard2/10.10.10.14:27017,10.10.10.15:27017,10.10.10.16:27017")
sh.addShard("shard3/10.10.10.17:27017,10.10.10.18:27017,10.10.10.19:27017")
查看状态
sh.status()

测试分片数据


use testdb

use config
db.settings.save({"_id":"chunksize","value":1})

sh.enableSharding('testdb')
sh.shardCollection('testdb.users',{uid:1})
for(i=1;i<=300000;i++) db.users.insert({uid:i,name:'hukey',age:23})


```




- 常用操作命令
```
主从状态查询
>rs.status()
>rs.conf()
>use local;
>rs.isMaster();
>db.system.replset.find()

查看当前主库：
db.$cmd.findOne({ismaster:1});
rs.isMaster()

#查看各Collection状态
db.printCollectionStats();输出信太多息,不适应查看

#查看主从复制状态
db.printReplicationInfo();

#查看主从同步信息：
db.printSecondaryReplicationInfo();
show dbs;

主节点降为secondary
mongo>use admin
mongo>rs.stepDown(60)#单位为秒
```