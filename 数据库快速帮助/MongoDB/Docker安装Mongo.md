# 下载镜像

```
docker pull docker.io/mongo:latest
```

# 运行容器

```
docker run --name mongo -p 27017:27017 -d docker.io/mongo:latest --auth
```

## 生产环境运行

```
首先创建文件夹用于挂载目录

mkdir -p /docker/mongo/{conf,data}
赋予权限
chmod 777 /docker/mongo/conf
chmod 777 /docker/mongo/data
然后直接启动容器
docker run --name mongo -d \
-p 27017:27017 \
--privileged=true \
-v /docker/mongo/conf:/data/configdb \
-v /docker/mongo/data:/data/db \
docker.io/mongo:latest \
--auth
```



# 创建用户

```
-----进入容器
docker exec -it mongo bash
-----进入mongo
mongo
-----选中admin数据库
use admin
-----创建用户，root用户
db.createUser({user:"root",pwd:"bigkang",roles:[{role:'root',db:'admin'}]})
-----退出mongo
exit
```

# 给数据库创建用户

首先创建数据库，如果有则选中如果没有则创建

```
use test
```

然后创建用户

user为用户名，pwd为用户密码，role为角色，db为数据库

```
db.createUser({user:"bigkang",pwd:"bigkang",roles:[{role:'dbOwner',db:'test'}]})
```

然后需要认证登录

```
use test
db.auth('admin','admin')
```

## 角色权限

```
1. 数据库用户角色：read、readWrite;  
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；       
3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root  
// 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
7. 内部角色：__system
```

```
read:允许用户读取指定数据库 
readWrite:允许用户读写指定数据库 
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile 
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户 
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。 
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限 
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限 
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限 
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。 
root：只在admin数据库中可用。超级账号，超级权限
```

# 创建集合

```
db.createCollection("testas");
```





# Mongodb集群搭建

mongodb 集群搭建的方式有三种：

1. 主从备份（Master - Slave）模式，或者叫主从复制模式。
2. 副本集（Replica Set）模式。
3. 分片（Sharding）模式。

> 其中，第一种方式基本没什么意义，官方也不推荐这种方式搭建。另外两种分别就是副本集和分片的方式。今天介绍副本集的方式搭建mongodb高可用集群

## 副本集集群版

副本集的方式也很容易理解，这里需要一个主节点，一个备节点，如果主节点发生故障，那么会启用备节点，当主节点修复之后，主节点再次恢复为主节点，备节点不再是主节点的角色。副本集的方式还需要一个角色，那就是仲裁节点，它不存储数据，他的作用就是当主节点出现故障，选举出备节点作为主节点，继续保证集群可用。客户端连接时只连接主节点或者备节点，不用连接仲裁节点。

> 集群节点为偶数时需要仲裁节点，如果集群节点为奇数是则不需要仲裁节点

副本集模式集群搭建，现采用四台机器进行mongodb replica set 模式的集群搭建。

### 集群搭建步骤

#### 1. 启动mongo

```
# 启动master
docker run -d --restart=always --name mongo-node1 \
-v /docker/mongo-cluster/mongo-node1/db:/data/db \
-v /docker/mongo-cluster/mongo-node1/conf:/data/configdb \
-p 20168:27017 \
docker.io/mongo mongod --dbpath /data/db --replSet mongoreplset --oplogSize 128

# 启动salve
docker run -d --restart=always --name mongo-node2 \
-v /docker/mongo-cluster/mongo-node2/db:/data/db \
-v /docker/mongo-cluster/mongo-node2/conf:/data/configdb \
-p 20167:27017 \
docker.io/mongo mongod --dbpath /data/db --replSet mongoreplset --oplogSize 128

# 启动arbiter
docker run -d --restart=always --name mongo-arbiter \
-v mongo-db:/data/db \
-v mongo-configdb:/data/configdb \
-p 27017:27017 mongo:3.4.9 mongod \
--dbpath /data/db --replSet mongoreplset --smallfiles --oplogSize 128
```

#### 2.集群配置

```
docker exec -it mongo mongo
config = {_id:"mongoreplset", version:1, members:[{_id:0, host:"10.18.81.18:20168", priority:5}, {_id:1, host:"10.18.81.19:20168", priority:4}, {_id:2, host:"10.18.81.20:20168", priority:3} ]}
rs.initiate(config)
```

#### 3.添加用户, 添加集群管理权限

```
use admin
db.createUser({ user: 'yjgl', pwd: 'yjgl12#$', roles: [ { role: "userAdminAnyDatabase", db: "admin" },  { role: "clusterAdmin", db: "admin" } ] });
db.createUser({ user: 'anjian', pwd: 'topcom123', roles: [ { role: "dbOwner", db: "anjian-db" } ] });
```

#### 4.生成keyFile

注意，这并没有结束，你需要启动mongo的时候，指定同一个一个key_file。生成一个key_file,并指定权限 在mongo_configdb的_data目录下运行如下命令：

```
openssl rand -base64 756 > key_file
chmod 400 key_file
chown 999:docker key_file
```

> 注意要修改 key_file 文件的权限和用户组，权限设置为只读权限，因为是 docker 部署，将用户组设置为 999：docker，文件在mongo的容器中会变成 mongodb:mongodb， 否则容器中的 key_file 文件用户组为 mongodb:root，这样可能会有问题。保证集群的key_file文件一致。

#### 5.重启mongo

先把所有的mongo服务的干掉，不要删除文件再运行下边的脚本

```
# 重启master
docker run -d --restart=always --name mongo-master \
-v mongo-db:/data/db \
-v mongo-configdb:/data/configdb \
-p 20168:27017 mongo:3.4.9 \
mongod --dbpath /data/db \
--replSet mongoreplset --oplogSize 128 --auth --keyFile=/data/configdb/key_file

# 重启salve
docker run -d --restart=always --name mongo-salve \
-v mongo-db:/data/db \
-v mongo-configdb:/data/configdb \
-p 20168:27017 mongo:3.4.9 \
mongod --dbpath /data/db \
--replSet mongoreplset --oplogSize 128 --auth --keyFile=/data/configdb/key_file

# 重启arbiter
docker run -d --restart=always --name mongo-arbiter \
-v mongo-db:/data/db \
-v mongo-configdb:/data/configdb \
-p 20167:27017 mongo:3.4.9 \
mongod --dbpath /data/db \
--replSet mongoreplset --smallfiles --oplogSize 128 --auth --keyFile=/data/configdb/key_file
```

> 注意，如果想看到日志，可以把-d去掉。

### 集群操作

#### 集群测试

```
# 进入mongo
docker exec -it mongo mongo

# 登录
use admin
db.auth("yjgl", "yjgl12#$")

# 查看状态
rs.status()
{
	"set" : "mongoreplset",
	"date" : ISODate("2019-07-04T08:44:45.161Z"),
	"myState" : 1,
	"term" : NumberLong(9),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1562229880, 1),
			"t" : NumberLong(9)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1562229880, 1),
			"t" : NumberLong(9)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1562229880, 1),
			"t" : NumberLong(9)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "10.18.81.18:20168",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 299,
			"optime" : {
				"ts" : Timestamp(1562229880, 1),
				"t" : NumberLong(9)
			},
			"optimeDate" : ISODate("2019-07-04T08:44:40Z"),
			"electionTime" : Timestamp(1562229629, 1),
			"electionDate" : ISODate("2019-07-04T08:40:29Z"),
			"configVersion" : 3,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "10.18.81.19:20168",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 273,
			"optime" : {
				"ts" : Timestamp(1562229880, 1),
				"t" : NumberLong(9)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1562229880, 1),
				"t" : NumberLong(9)
			},
			"optimeDate" : ISODate("2019-07-04T08:44:40Z"),
			"optimeDurableDate" : ISODate("2019-07-04T08:44:40Z"),
			"lastHeartbeat" : ISODate("2019-07-04T08:44:45.120Z"),
			"lastHeartbeatRecv" : ISODate("2019-07-04T08:44:44.409Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "10.18.81.18:20168",
			"configVersion" : 3
		},
		{
			"_id" : 2,
			"name" : "10.18.81.20:20168",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 258,
			"optime" : {
				"ts" : Timestamp(1562229880, 1),
				"t" : NumberLong(9)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1562229880, 1),
				"t" : NumberLong(9)
			},
			"optimeDate" : ISODate("2019-07-04T08:44:40Z"),
			"optimeDurableDate" : ISODate("2019-07-04T08:44:40Z"),
			"lastHeartbeat" : ISODate("2019-07-04T08:44:45.120Z"),
			"lastHeartbeatRecv" : ISODate("2019-07-04T08:44:43.951Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "10.18.81.19:20168",
			"configVersion" : 3
		},
		{
			"_id" : 3,
			"name" : "10.18.81.21:27017",
			"health" : 1,
			"state" : 7,
			"stateStr" : "ARBITER",
			"uptime" : 248,
			"lastHeartbeat" : ISODate("2019-07-04T08:44:45.134Z"),
			"lastHeartbeatRecv" : ISODate("2019-07-04T08:44:40.350Z"),
			"pingMs" : NumberLong(0),
			"configVersion" : 3
		}
	],
	"ok" : 1
}
```

#### 添加仲裁节点

```
rs.addArb("10.18.81.21:27017")
```

#### 集群操作基本命令

```
# 初始化集群配置
rs.initiate(config)

# 查看集群状态
rs.status()

# 查看配置
rs.conf()

# 修改原有的配置
rs.reconfig(config)

# 集群查询
rs.slaveOk() //副本集默认仅primary可读写, 查询secondary节点时要设置slaveOk
db.user.find()
```

### Spring Boot 配置Mongodb集群

```
# MongoDB URI配置 重要，添加了用户名和密码验证
spring.data.mongodb.uri=mongodb://anjian:topcom123@192.168.68.138:27017,192.168.68.137:27017,192.168.68.139:27017/anjian-db?slaveOk=true&replicaSet=mongoreplset&write=1&readPreference=secondaryPreferred&connectTimeoutMS=300000

#每个主机的连接数
spring.data.mongodb.connections-per-host=50
#线程队列数，它以上面connectionsPerHost值相乘的结果就是线程队列最大值
spring.data.mongodb.threads-allowed-to-block-for-connection-multiplier=50
spring.data.mongodb.connect-timeout=5000
spring.data.mongodb.socket-timeout=3000
spring.data.mongodb.max-wait-time=1500
#控制是否在一个连接时，系统会自动重试
spring.data.mongodb.auto-connect-retry=true
spring.data.mongodb.socket-keep-alive=true
```

## 分片集群版本

MongoDB分片群集包含以下组件：

​	分片：每个分片包含分片数据的子集。每个分片都可以部署为副本集。

​	mongos：`mongos`充当查询路由器，提供客户端应用程序和分片集群之间的接口。

​	config servers：配置服务器存储群集的元数据和配置设置。从MongoDB 3.4开始，必须将配置服务器部署为副本集（CSRS）。（也就是最少两台）

​	