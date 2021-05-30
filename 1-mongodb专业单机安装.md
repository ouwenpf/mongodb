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

mkdir -p /data/mongodb/mongodb27017/{data,logs,conf}
useradd mongodb /data/mongodb/mongodb27017/{data,logs,conf}

id mongodb
passwd
chown -R mongodb:mongodb /data/mongodb/mongodb27017
sed   -ri 's#local/bin#local/bin:/usr/local/mongodb/bin#g'     /home/mongodb/.bash_profile
source ~/.bash_profile


mongod  -f /data/mongodb/mongodb27017/conf/mongodb.conf


要启用自由监视，请运行以下命令：db.enableFreeMonitoring()
要永久禁用此提醒，请运行以下命令：db.disableFreeMonitoring()


use admin
db.createUser({user: "admin",pwd: "123456", roles: ["root"]})
db.dropUser("admin")
db.auth("admin","123456")