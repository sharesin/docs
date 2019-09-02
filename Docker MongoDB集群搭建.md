### Docker MongoDB 4.2.0环境搭建


#### 1. 下载mongo镜像

```
docker pull mongo:4.2.0
```

官网下载地址: [https://hub.docker.com](https://hub.docker.com/)，可以使用国内镜像源

#### 2. 跨主机Docker容器网络互通

##### 1. 打开路由转发功能

```
当然前提是，docker0 网桥的网段改掉，参照下图的方式，同时需要提醒的是，需要把本机的路由转发打开。

ubuntu的话：
修改 /etc/sysctl.conf，把ip_forward = 1 的注释去掉即可
centos的话：
修改 /etc/sysctl.d/99-sysctl.conf
添加net.ipv4.ip_forward = 1，然后试试sysctl -p
很简单的方式，其实很像flannel网络的Host-Gateway的原理
```
参考地址：https://blog.csdn.net/xialingming/article/details/83093031

于docker 建立与主机同网段的zookeeper集群:
https://blog.csdn.net/tiezhong2004/article/details/80658794

##### 2. 修改Docker默认地址范围

```
/letc/docker/daemon.json

{
"bip": "172.17.2.1/24"
}
~
```

#### 3. 集群结构

<table style="width:100%;text-align:center;margin:0;padding:0;">
  <tr>
    <th>服务器IP</td>
    <th>端口</td>
    <th>mongo角色</td>
  </tr>
  <tr>
    <td rowspan="4">10.200.36.195</td>
    <td> 27017 </td>
    <td> mongos </td>
  </tr>
  <tr>
    <td>27019</td>
    <td>config-server</td>
  </tr>
  <tr>
    <td>27021</td>
    <td>shard-1</td>
  </tr>
  <tr>
    <td> 27022 </td>
    <td>shard-2</td>
  </tr>
  
  <tr>
    <td rowspan="4">10.200.36.196</td>
    <td> 27017 </td>
    <td> mongos </td>
  </tr>
  <tr>
    <td>27019</td>
    <td>config-server</td>
  </tr>
  <tr>
    <td>27022</td>
    <td>shard-2</td>
  </tr>
  <tr>
    <td> 27023 </td>
    <td>shard-3</td>
  </tr>
  
  <tr>
    <td rowspan="4">10.200.36.197</td>
    <td> 27017 </td>
    <td> mongos </td>
  </tr>
  <tr>
    <td>27019</td>
    <td>config-server</td>
  </tr>
  <tr>
    <td>27021</td>
    <td>shard-1</td>
  </tr>
  <tr>
    <td> 27023 </td>
    <td>shard-3</td>
  </tr>
  
</table>


参考文档：[https://blog.csdn.net/guan0005/article/details/86995019](https://blog.csdn.net/guan0005/article/details/86995019)

修改docker端口：https://blog.csdn.net/qq_37467907/article/details/79537801


nohup consul agent -server -bootstrap -syslog -ui \
	-node=consul-server01 \
	-client='0.0.0.0' \
	-advertise=10.200.36.195 \
	-bind=0.0.0.0 \
	-pid-file=/home/consul/run/consul.pid \
	-data-dir=/home/consul/data \
	-config-dir=/home/consul/conf >> /home/consul/log/output.log 2>&1 &
	


consul agent -server -bootstrap -syslog \
    -data-dir=/opt/consul/data \
    -config-dir=/opt/consul/conf \
    -pid-file=/opt/consul/run/consul.pid \
    -client=0.0.0.0 \
    -node=consul-server01 \
    -disable-host-node-id


```
docker network create --subnet 172.100.100.0/24 mongodb

docker network create -d overlay mongodb-overlay-net

docker network create -d overlay ov_net1

docker network create -d overlay mongodb

docker  network  create  -d  overlay  --subnet  10.2.0.0/24   mongodb
docker  network  create  -d  overlay  --subnet  10.2.0.0/24   mongodb



docker network ls


centos7 firewall指定IP与端口访问（常用）

https://blog.csdn.net/cn_yaojin/article/details/86351392

添加指定需要开放的端口：
firewall-cmd --add-port=123/tcp --permanent
重载入添加的端口：
firewall-cmd --reload
查询指定端口是否开启成功：
firewall-cmd --query-port=123/tcp

移除指定端口：
firewall-cmd --permanent --remove-port=123/tcp

```

```
docker run -d --name=mongodb-cluster-configsvr-01 \
--user=1001 \
--network=mongodb \
--restart=always \
-v /etc/localtime:/etc/localtime \
-p 27019:27019 \
-v /home/mongodb/security/:/mongodb/security/ \
-v /home/mongodb/configsvr/conf/:/etc/mongodb/ \
-v /home/mongodb/configsvr/data/:/data/db/ \
-v /home/mongodb/configsvr/logs/:/var/log/mongodb/ \
mongo:4.2.0 \
--config /etc/mongodb/configsvr.conf


docker run -d --name=mongodb-cluster-configsvr-02 \
--user=1001 \
--network=mongodb \
--restart=always \
-v /etc/localtime:/etc/localtime \
-p 27019:27019 \
-v /home/mongodb/security/:/mongodb/security/ \
-v /home/mongodb/configsvr/conf/:/etc/mongodb/ \
-v /home/mongodb/configsvr/data/:/data/db/ \
-v /home/mongodb/configsvr/logs/:/var/log/mongodb/ \
mongo:4.2.0 \
--config /etc/mongodb/configsvr.conf

docker run -d --name=mongodb-cluster-configsvr-03 \
--user=1001 \
--network=mongodb \
--restart=always \
-v /etc/localtime:/etc/localtime \
-p 27019:27019 \
-v /home/mongodb/security/:/mongodb/security/ \
-v /home/mongodb/configsvr/conf/:/etc/mongodb/ \
-v /home/mongodb/configsvr/data/:/data/db/ \
-v /home/mongodb/configsvr/logs/:/var/log/mongodb/ \
mongo:4.2.0 \
--config /etc/mongodb/configsvr.conf


在mongo中执行以下命令初始化config-server副本集

docker exec -it mongodb-cluster-configsvr-01 /bin/bash

mongo --port 27019

config = {
    _id: "configsvr",
    members: [
      { _id : 0, host : "10.200.36.195:27019" },
      { _id : 1, host : "10.200.36.196:27019" },
      { _id : 2, host : "10.200.36.197:27019" }
    ]
  }
  
  config = {
    _id: "configsvr",
    members: [
      { _id : 0, host : "10.2.0.2:27019" },
      { _id : 1, host : "10.2.0.3:27019" },
      { _id : 2, host : "10.2.0.4:27019" }
    ]
  }
  
  config = {
    _id: "configsvr",
    members: [
      { _id : 0, host : "10.200.36.195:27019" }
    ]
  }
  
 rs.initiate(config);


rs.initiate(
  {
    _id: "configsvr",
    members: [
      { _id : 0, host : "10.200.36.195:27019" },
      { _id : 1, host : "10.200.36.196:27019" },
      { _id : 2, host : "10.200.36.197:27019" }
    ]
  }
)

> rs.initiate(
...   {
...     _id: "configsvr",
...     members: [
...       { _id : 1, host : "10.200.36.195:27019" },
...       { _id : 2, host : "10.200.36.196:27019" },
...       { _id : 3, host : "10.200.36.197:27019" }
...     ]
...   }
... )
{
        "operationTime" : Timestamp(0, 0),
        "ok" : 0,
        "errmsg" : "No host described in new configuration 1 for replica set configsvr maps to this node",
        "code" : 93,
        "codeName" : "InvalidReplicaSetConfig",
        "$gleStats" : {
                "lastOpTime" : Timestamp(0, 0),
                "electionId" : ObjectId("000000000000000000000000")
        },
        "lastCommittedOpTime" : Timestamp(0, 0),
        "$clusterTime" : {
                "clusterTime" : Timestamp(0, 0),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}

```


##### MongoDB安装

https://blog.csdn.net/ASAS1314/article/details/100133205



```
开放端口号

firewall-cmd --add-port=27000-27030/tcp --permanent

firewall-cmd --add-port=27017/tcp --permanent

firewall-cmd --reload

firewall-cmd --list-ports

启动config

mongod -f /home/mongodb/configsvr/conf/configsvr.conf 

mongo --port 27019

rs.initiate(
  {
    _id: "configsvr",
    members: [
      { _id : 0, host : "10.200.36.195:27019" },
      { _id : 1, host : "10.200.36.196:27019" },
      { _id : 2, host : "10.200.36.197:27019" }
    ]
  }
)

mongo --port 27021

rs.initiate(
	{
	_id : "shard1",
	members: [
		{ _id : 0, host : "10.200.36.195:27021" },
		{ _id : 1, host : "10.200.36.197:27021" }
	]
	}
)

mongo --port 27022
rs.initiate(
	{
	_id : "shard2",
	members: [
		{ _id : 0, host : "10.200.36.195:27022" },
		{ _id : 1, host : "10.200.36.196:27022" }
	]
	}
)

mongo --port 27023
rs.initiate(
	{
	_id : "shard3",
	members: [
		{ _id : 0, host : "10.200.36.196:27023" },
		{ _id : 1, host : "10.200.36.197:27023" }
	]
	}
)


mongo --port 27017

sh.addShard("shard1/10.200.36.195:27021,10.200.36.197:27021")
sh.addShard("shard2/10.200.36.195:27022,10.200.36.196:27022")
sh.addShard("shard3/10.200.36.196:27023,10.200.36.197:27023")

db和collection开启分片功能

sh.enableSharding("NSTL")

sh.shardCollection("NSTL.result_original_article", { article_id : "hashed"})




```
