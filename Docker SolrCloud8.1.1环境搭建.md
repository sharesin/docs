### Docker SolrCloud8.1.1环境搭建


#### 1. 下载zookeeper，solr镜像

```
docker pull zookeeper
docker pull solr
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


#### 3. 启动zookeeper服务，建议使用奇数个

```
caas-zookeeper-1
docker run -d \
    --name=caas-zookeeper-1 \
    --restart=always \
    --net=host \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/zookeeper_1/conf:/conf \
    -v /home/solrcloud/zookeeper_1/data:/data \
    -v /home/solrcloud/zookeeper_1/datalog:/datalog \
    -e ZOO_MY_ID=1 \
    -e ZOO_SERVERS="server.1=10.200.36.90:2888:3888 server.2=10.200.36.91:2888:3888 server.3=10.200.36.25:2888:3888" \
    --privileged \
    zookeeper:3.4.14
    
caas-zookeeper-2
docker run -d \
    --name=caas-zookeeper-2 \
    --restart=always \
    --net=host \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/zookeeper_2/conf:/conf \
    -v /home/solrcloud/zookeeper_2/data:/data \
    -v /home/solrcloud/zookeeper_2/datalog:/datalog \
    -e ZOO_MY_ID=2 \
    -e ZOO_SERVERS="server.1=10.200.36.90:2888:3888 server.2=10.200.36.91:2888:3888 server.3=10.200.36.25:2888:3888" \
    --privileged \
    zookeeper:3.4.14
    
caas-zookeeper-3
docker run -d \
    --name=caas-zookeeper-3 \
    --restart=always \
    --net=host \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/zookeeper_3/conf:/conf \
    -v /home/solrcloud/zookeeper_3/data:/data \
    -v /home/solrcloud/zookeeper_3/datalog:/datalog \
    -e ZOO_MY_ID=3 \
    -e ZOO_SERVERS="server.1=10.200.36.90:2888:3888 server.2=10.200.36.91:2888:3888 server.3=10.200.36.25:2888:3888" \
    --privileged \
    zookeeper:3.4.14

查看状态命令
echo stat|nc localhost 2181
```




#### 3. 启动Solr镜像

##### 1. 为了避免docker容器权限访问问题，需对目录进行：

```
chown -R 8983:8983 /home/solrcloud/
```

##### 2. 启动镜像服务
```

docker run -d \
    --name caas-solrcloud_1 \
    --restart=always \
    --log-opt max-size=10m --log-opt max-file=3 \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/solrhome_1:/opt/solrhome -e SOLR_HOME=/opt/solrhome \
    -v /home/solrcloud/solrhome_1/logs:/var/solr/logs \
    -v /home/solrcloud/config:/opt/solrcloud/config \
    -v /home/solrcloud/config/ik/classes:/opt/solr/server/solr-webapp/webapp/WEB-INF/classes \
    --privileged=true \
    -p 8983:8983 \
    solr bash -c '/opt/solr/bin/solr start -m 28g -f -z 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181'


docker run -d \
    --name caas-solrcloud_2 \
    --restart=always \
    --log-opt max-size=10m --log-opt max-file=3 \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/solrhome_2:/opt/solrhome -e SOLR_HOME=/opt/solrhome \
    -v /home/solrcloud/solrhome_2/logs:/var/solr/logs \
    -v /home/solrcloud/config:/opt/solrcloud/config \
    -v /home/solrcloud/config/ik/classes:/opt/solr/server/solr-webapp/webapp/WEB-INF/classes \
    --privileged=true \
    -p 8984:8983 \
    solr bash -c '/opt/solr/bin/solr start -m 28g -f -z 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181'

docker run -d\
    --name caas-solrcloud_3 \
    --restart=always \
    --log-opt max-size=10m --log-opt max-file=3 \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/solrhome_3:/opt/solrhome -e SOLR_HOME=/opt/solrhome \
    -v /home/solrcloud/solrhome_3/logs:/var/solr/logs \
    -v /home/solrcloud/config:/opt/solrcloud/config \
    -v /home/solrcloud/config/ik/classes:/opt/solr/server/solr-webapp/webapp/WEB-INF/classes \
    --privileged=true \
    -p 8983:8983 \
    solr bash -c '/opt/solr/bin/solr start -m 28g -f -z 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181'

docker run -d \
    --name caas-solrcloud_4 \
    --restart=always \
    --log-opt max-size=10m --log-opt max-file=3 \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/solrhome_4:/opt/solrhome -e SOLR_HOME=/opt/solrhome \
    -v /home/solrcloud/solrhome_4/logs:/var/solr/logs \
    -v /home/solrcloud/config:/opt/solrcloud/config \
    -v /home/solrcloud/config/ik/classes:/opt/solr/server/solr-webapp/webapp/WEB-INF/classes \
    --privileged=true \
    -p 8984:8983 \
    solr bash -c '/opt/solr/bin/solr start -m 28g -f -z 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181'
    
    
docker run -d\
    --name caas-solrcloud_5 \
    --restart=always \
    --log-opt max-size=10m --log-opt max-file=3 \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/solrhome_5:/opt/solrhome -e SOLR_HOME=/opt/solrhome \
    -v /home/solrcloud/solrhome_5/logs:/var/solr/logs \
    -v /home/solrcloud/config:/opt/solrcloud/config \
    -v /home/solrcloud/config/ik/classes:/opt/solr/server/solr-webapp/webapp/WEB-INF/classes \
    --privileged=true \
    -p 8983:8983 \
    solr bash -c '/opt/solr/bin/solr start -m 28g -f -z 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181'

docker run -d \
    --name caas-solrcloud_6 \
    --restart=always \
    --log-opt max-size=10m --log-opt max-file=3 \
    -v /etc/localtime:/etc/localtime \
    -v /home/solrcloud/solrhome_6:/opt/solrhome -e SOLR_HOME=/opt/solrhome \
    -v /home/solrcloud/solrhome_6/logs:/var/solr/logs \
    -v /home/solrcloud/config:/opt/solrcloud/config \
    -v /home/solrcloud/config/ik/classes:/opt/solr/server/solr-webapp/webapp/WEB-INF/classes \
    --privileged=true \
    -p 8984:8983 \
    solr bash -c '/opt/solr/bin/solr start -m 28g -f -z 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181'

```
修改solr内存配置

[https://blog.csdn.net/qq_41665356/article/details/80374884](https://blog.csdn.net/qq_41665356/article/details/80374884)

参考地址:

[http://www.dczou.com/viemall/824.html](http://www.dczou.com/viemall/824.html)

[https://blog.csdn.net/bskfnvjtlyzmv867/article/details/81623416](https://blog.csdn.net/bskfnvjtlyzmv867/article/details/81623416)

##### 3. 新建配置内核文件

```
docker run -it \
-v /home/solrcloud/solrhome_1:/opt/solrhome \
-e SOLR_HOME=/opt/solrhome solr bash -c "cp -R /opt/solr/server/solr/* /opt/solrhome"
```

##### 4. 上传内核配置文件至zookeeper配置中心

```
docker 容器命令

拷贝配置文件至容器内部
docker cp casdd/ caas-solrcloud_1:/opt/solr/server/solr/configsets/

docker exec -u root -it caas-solrcloud_1 /bin/bash '/opt/solr/server/scripts/cloud-scripts/zkcli.sh -zkhost 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181 -cmd upconfig -confdir /opt/solrhome/configsets/casdd/conf -confname casdd'

shell命令

/opt/solr/server/scripts/cloud-scripts/zkcli.sh \
-zkhost 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181 -cmd upconfig \
-confdir /opt/solrhome/configsets/casdd/conf \
-confname casdd

/opt/solr/server/scripts/cloud-scripts/zkcli.sh -zkhost 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181 -cmd upconfig -confdir /opt/solrhome/configsets/agrinstl/conf -confname agrinstl

```

##### 4. 清除内核配置文件zookeeper配置中心

```
/opt/solr/server/scripts/cloud-scripts/zkcli.sh \
-zkhost 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181 -cmd clear /configs/casdd

/opt/solr/server/scripts/cloud-scripts/zkcli.sh \
-zkhost 10.200.36.90:2181,10.200.36.91:2181,10.200.36.25:2181 -cmd clear /configs/agrinstl
```
