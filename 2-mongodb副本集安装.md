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

- MongoDB副本集

```
MongoDB4.0之前一直使用master/salve主从架构
MongoDB4.0之后不支持master/salve这种方式了,故的复制集群用了新()replica Sets):副本集
生产环境一个集群成员最多12个(官方支持50个,仲裁节点只有7个成员拥有投票权),心跳每2秒发送一次

MongoDB复制集实现了冗余备份和故障转移两大功能,这样能保证数据库的高可用性,在生产环境,复制集至少
包括三个节点,其中一个必须为主节点,一个从节点,一个仲裁节点,其中每一个节点都是MongoDB进程对应的实例,节点间通过心跳检查对方的状态
	primary节点,负责数据库的读写操作
	secondary节点:备份primary节点上的数据,可以有多个
	arbiter节点:主节点故障时候,参与复制集剩下的节点中选举一个新的primary节点

生产一般建议:6节点-1主-4从-1仲裁
```

- 安装配置

```
10.10.10.11 master 主节点
10.10.10.12 repl 从节点
10.10.10.13 arb 仲裁节点

主从配置:
rs.initiate({_id:'rep1',members:[{_id:1,host:'10.10.10.11:27017'}]})
rs.add("10.10.10.12:27017")
rs.addArb("10.10.10.13:27017")
rs.conf()
rs.status()

查询主从信息:
use local
show collections
检查各个节点local库信息：
复制集工作原理:
复制集通过local库下的oplog.rs集合进行数据同步。
oplog会包含所有对数据有修改的的操作（查询操作不会记录）



测试数据:
use test
for(var i =0; i <11; i ++){db.test1.insert({userName:"kkk"+i,agex:i})}
show collections;
db.test1.find();

其它从节点查询:
rs.secondaryOk();
db.test1.find();


```


- mongodb副本集中增加减少主机
```
增加副本集成员
use admin
db.auth('admin','123456') //验证用户，即该用户需要有管理mongodb数据库中用户权限的权限
rs.add("10.10.10.14:27017") //增加主机
rs.conf() //查看是否添加成功
rs.status()


删除副本集成员
1、登录要移除的目标mongodb实例；
2、利用shutdownServer()命令关闭实例；即db.shutdownServer()
3、登录复制集的primary；
4、use admin
   rs.remove("del_node:port");
删除主机
use admin
rs.remove("10.10.10.14:27017") //删除主机

移除Arbiter
rs.remove("IP:port")

```



- 关于读写分离
```
常见的解决方案是读写分离，mongodb副本集的读写分离如何做呢？
常规写操作来说并没有读操作多，所以一台主节点负责写，两台副本节点负责读。
1 、设置读写分离需要先在副本节点SECONDARY 设置rs.secondaryOk();
( db.getMongo().rs.secondaryOk(); )

primary、primaryPreferred、secondary、secondaryPreferred、nearest。
primary:默认参数，只从主节点上进行读取操作；
primaryPreferred:大部分从主节点上读取数据,只有主节点不可用时从secondary节点读取数据。
secondary:只从secondary节点上进行读取操作，存在的问题是secondary节点的数据会比primary节点数据“旧”。
secondaryPreferred:优先从secondary节点进行读取操作，secondary节点不可用时从主节点读取数据；
nearest:不管是主节点、secondary节点，从网络延迟最低的节点上读取数据。

```

- 节点关闭顺序
```
先把从节点和仲裁节点关掉，再关主节点
先启动主节点，再启动从节点和仲裁节点
			
```