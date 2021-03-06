# 注意版本问题

如果采用Boot2.1.X以及Cloud [Greenwich](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies/Greenwich.SR1)版请安装1.0.0以上的版本（包括1.0.0）

如果采用Boot2.0.X以及Cloud  [Finchley](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies/Finchley.SR2) 版请安装1.0.0以下的版本（不包括1.0.0）

如果采用Boot1.5以及以下请安装更低版本

# 如何安装Nacos

​	首先我们要先去下载nacos

​	https://github.com/alibaba/nacos/releases

​	这里是nacos的下载路径

​	下载好之后我们解压到目录下

# 运行

## 在windows的环境下

​	找到目录的bin目录，里面有个startup.cmd这个文件，双击执行，然后出现cmd控制台窗口此时不要关闭

​	![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566807216644.png)

然后我们可以看得到他的console路径，复制下来放到浏览器上，进入浏览器，默认账户密码都为nacos，

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566807244938.png)

然后登陆

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566807280640.png)

这样我们就运行好了

## Linux环境

找到当前目录，进入bin目录下执行sh startup.sh -m standalone

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566807333597.png)

然后等待启动

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566807405595.png)

然后也能直接访问了

后台启动使用nohup sh startup.sh -m standalone &

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566807429271.png)



# Docker版

## 单机

### 快速启动

docker一键运行

```sh
docker run -d --name nacos -e MODE=standalone -p 8848:8848 docker.io/nacos/nacos-server:1.0.0
```

### 生产启动

！！！！！！！！！！！！！(注意现在全部采用Nacos1.0.0为兼容SpringClou以及Boot2.1)

首先下载mysql配置mysql的地址，先安装mysql（MYSQL注意使用，5.7）

首先创建文件夹以及挂载目录。将配置文件以及数据文件持久化出来

```sh
mkdir -p /docker/nacos/conf
mkdir -p /docker/nacos/data
```

然后进入mysql配置文件加入

```sh
vim /docker/nacos/conf/my.cnf

加入下面内容
[mysqld]
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```

然后启动mysql容器

```bash
docker run -p 3306:3306 \
--name nacos-mysql \
-e MYSQL_ROOT_PASSWORD=root \
--privileged=true \
-v /docker/nacos/data:/var/lib/mysql \
-v /docker/nacos/conf/my.cnf:/etc/mysql/conf.d/mysql.cnf \
-d docker.io/mysql:5.7
```

然后我们进入容器，创建数据库

```sql
docker exec -it nacos-mysql bash

mysql -u root -p
输入上面的密码root
然后创建数据库
create database nacos;
```

然后点击下方标题MySQL数据初始化



这样MySQL就已经搭建完毕了，我们来搭建Nacos,这里注意一定要用自己的MySQL的ip地址

```properties
mkdir -p /docker/nacos/conf
首先创建配置文件
vim /docker/nacos/conf/application.properties
将下面内容粘贴进去，注意这里用户和密码以及mysql的ip地址都要修改

# spring
server.servlet.contextPath=${SERVER_SERVLET_CONTEXTPATH:/nacos}
server.contextPath=/nacos
server.port=${NACOS_SERVER_PORT:8848}
spring.datasource.platform=${SPRING_DATASOURCE_PLATFORM:""}
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.loadDataAtStart=false


spring.datasource.platform=mysql
db.num=1
# 注意修改自己IP重要的话重复说
db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&serverTimezone=Asia/Shanghai
db.user=root
db.password=bigkang


server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D
# default current work dir
server.tomcat.basedir=
## spring security config
### turn off security

nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
# metrics for elastic search
management.metrics.export.elastic.enabled=fasle
management.metrics.export.influx.enabled=false

nacos.naming.distro.taskDispatchThreadCount=10
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
```

然后我们启动容器,

```bash
#单配置文件，指定mysql地址启动
docker run -p 8848:8848 \
--name nacos-server \
-v /docker/nacos/conf/application.properties:/home/nacos/conf/application.properties \
-e MODE=standalone \
--privileged=true \
-d docker.io/nacos/nacos-server:1.1.3

#如测试服务器内存不够请修改sh文件，nacos官方镜像jvm参数无效
docker run -p 8848:8848 \
--name nacos-server \
-e MODE=standalone \
-v /docker/nacos/conf/application.properties:/home/nacos/conf/application.properties \
-v /docker/nacos/conf/docker-startup.sh:/home/nacos/bin/docker-startup.sh \
--privileged=true \
-d docker.io/nacos/nacos-server:1.1.0
```

然后访问ip:8848/nacos/index.html

### Compose

​		单安装Nacos

```sh
# 创建部署文件目录
cd && mkdir -p deploys && cd deploys && mkdir nacos && cd nacos

# 写入Compose配置文件
cat > ./docker-compose-nacos.yaml << EOF
version: '3.4' 
services:   
  nacos-server:
    image: nacos/nacos-server:1.1.4
    container_name: nacos-server
    hostname: nacos-server
    restart: always
    ports:
      - 8848:8848
    environment:
    	MODE: standalone
    	NACOS_APPLICATION_PORT: 8848
      SPRING_DATASOURCE_PLATFORM: mysql
      MYSQL_SERVICE_HOST: 192.168.1.12
      MYSQL_SERVICE_PORT: 3306
      MYSQL_SERVICE_DB_NAME: nacos
      MYSQL_SERVICE_USER: root
      MYSQL_SERVICE_PASSWORD: bigkang
      MYSQL_DATABASE_NUM: 1
      MYSQL_SERVICE_DB_PARAM: characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
      JVM_XMS: 1g
      JVM_XMX: 1g
      JVM_XMN: 512m
      JVM_MS: 128m
      JVM_MMS: 220m
      #NACOS_DEBUG: n
      TOMCAT_ACCESSLOG_ENABLED: 'true'
    privileged: true
EOF


# 启动Nacos
docker-compose -f docker-compose-nacos.yaml up -d
```

# K8s安装Nacos

​		官网地址：[点击进入](https://nacos.io/zh-cn/docs/use-nacos-with-kubernetes.html)

## 快速启动

​		首先下载部署脚本

```
git clone https://github.com/nacos-group/nacos-k8s.git
```

​		然后进入目录文件执行权限，启动		

```sh
cd nacos-k8s
chmod +x quick-startup.sh
./quick-startup.sh
```

​		然后我们使用ingress-nginx暴露出去，修改文件,我们直接往原nacos文件中写入

```yaml
echo "

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nacos-quick
  annotations:
    kubernets.io/ingress.class: \"nginx\"
spec:
  rules:
  - host: yunyaonacos.bigkang.club
    http:
      paths:
      - path:
        backend:
          serviceName: nacos-headless
          servicePort: 8848" >> test.yaml
```




# 集群版本搭建

## Docker-Compose搭建（推荐）

注意此处没有搭建mysql主从所以从库也是主库，！！！（真实环境不推荐）

### 搭建nacos1

创建目录

```properties
mkdir -p /docker/nacos-clust/nacos1/{log,conf}
mkdir -p /root/nacos-clust/nacos1/
cd /root/nacos-clust/nacos1
touch /docker/nacos-clust/nacos1/conf/custom.properties

echo "#spring.security.enabled=false
#management.security=false
#security.basic.enabled=false
#nacos.security.ignore.urls=/**
#management.metrics.export.elastic.host=http://localhost:9200
# metrics for prometheus
management.endpoints.web.exposure.include=*
server.port=8841
# metrics for elastic search
#management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200

# metrics for influx
#management.metrics.export.influx.enabled=false
#management.metrics.export.influx.db=springboot
#management.metrics.export.influx.uri=http://localhost:8086
#management.metrics.export.influx.auto-create-db=true
#management.metrics.export.influx.consistency=one
#management.metrics.export.influx.compressed=true" > /docker/nacos-clust/nacos1/conf/custom.properties

echo "version: '3' 
services:   
  nacos1:
    image: nacos/nacos-server:1.1.4
    container_name: nacos1
    ports:
      - "8841:8841"
    restart: always
    network_mode: host
    privileged: true
    environment:
      SPRING_DATASOURCE_PLATFORM: mysql
      NACOS_SERVER_IP: 192.168.0.3
      NACOS_SERVERS: 192.168.0.3:8841 192.168.0.3:8842 192.168.0.3:8843
      MYSQL_MASTER_SERVICE_HOST: 192.168.0.3
      MYSQL_MASTER_SERVICE_PORT: 3301
      MYSQL_MASTER_SERVICE_DB_NAME: nacos
      MYSQL_MASTER_SERVICE_USER: root
      MYSQL_MASTER_SERVICE_PASSWORD: 123456
      MYSQL_SLAVE_SERVICE_HOST: 192.168.0.3
      MYSQL_SLAVE_SERVICE_PORT: 3302
      JVM_XMS: 1g
      JVM_XMX: 1g
      JVM_XMN: 512m
      JVM_MS: 128m
      JVM_MMS: 220m
      #NACOS_DEBUG: n
      TOMCAT_ACCESSLOG_ENABLED: 'true'
    volumes:
      - /docker/nacos-clust/nacos1/log:/home/nacos/logs
      - /docker/nacos-clust/nacos1/conf/custom.properties:/home/nacos/init.d/custom.properties" > /root/nacos-clust/nacos1/docker-compose.yaml
docker-compose up -d
```



### 搭建nacos2

创建目录

```
mkdir -p /docker/nacos-clust/nacos2/{log,conf}
mkdir -p /root/nacos-clust/nacos2/
cd /root/nacos-clust/nacos2
touch /docker/nacos-clust/nacos2/conf/custom.properties

echo "#spring.security.enabled=false
#management.security=false
#security.basic.enabled=false
#nacos.security.ignore.urls=/**
#management.metrics.export.elastic.host=http://localhost:9200
# metrics for prometheus
management.endpoints.web.exposure.include=*
server.port=8842
# metrics for elastic search
#management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200

# metrics for influx
#management.metrics.export.influx.enabled=false
#management.metrics.export.influx.db=springboot
#management.metrics.export.influx.uri=http://localhost:8086
#management.metrics.export.influx.auto-create-db=true
#management.metrics.export.influx.consistency=one
#management.metrics.export.influx.compressed=true" > /docker/nacos-clust/nacos2/conf/custom.properties

echo "version: '3' 
services:   
  nacos2:
    image: nacos/nacos-server:1.1.4
    container_name: nacos2
    ports:
      - "8842:8842"
    restart: always
    network_mode: host
    privileged: true
    environment:
      SPRING_DATASOURCE_PLATFORM: mysql
      NACOS_SERVER_IP: 192.168.0.3
      NACOS_SERVERS: 192.168.0.3:8841 192.168.0.3:8842 192.168.0.3:8843
      MYSQL_MASTER_SERVICE_HOST: 192.168.0.3
      MYSQL_MASTER_SERVICE_PORT: 3301
      MYSQL_MASTER_SERVICE_DB_NAME: nacos
      MYSQL_MASTER_SERVICE_USER: root
      MYSQL_MASTER_SERVICE_PASSWORD: 123456
      MYSQL_SLAVE_SERVICE_HOST: 192.168.0.3
      MYSQL_SLAVE_SERVICE_PORT: 3302
      JVM_XMS: 1g
      JVM_XMX: 1g
      JVM_XMN: 512m
      JVM_MS: 128m
      JVM_MMS: 220m
      #NACOS_DEBUG: n
      TOMCAT_ACCESSLOG_ENABLED: 'true'
    volumes:
      - /docker/nacos-clust/nacos2/log:/home/nacos/logs
      - /docker/nacos-clust/nacos2/conf/custom.properties:/home/nacos/init.d/custom.properties" > /root/nacos-clust/nacos2/docker-compose.yaml
docker-compose up -d
```

### 搭建nacos3

创建目录

```
mkdir -p /docker/nacos-clust/nacos3/{log,conf}
mkdir -p /root/nacos-clust/nacos3/
cd /root/nacos-clust/nacos3
touch /docker/nacos-clust/nacos3/conf/custom.properties

echo "#spring.security.enabled=false
#management.security=false
#security.basic.enabled=false
#nacos.security.ignore.urls=/**
#management.metrics.export.elastic.host=http://localhost:9200
# metrics for prometheus
management.endpoints.web.exposure.include=*
server.port=8843
# metrics for elastic search
#management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200

# metrics for influx
#management.metrics.export.influx.enabled=false
#management.metrics.export.influx.db=springboot
#management.metrics.export.influx.uri=http://localhost:8086
#management.metrics.export.influx.auto-create-db=true
#management.metrics.export.influx.consistency=one
#management.metrics.export.influx.compressed=true" > /docker/nacos-clust/nacos3/conf/custom.properties

echo "version: '3' 
services:   
  nacos3:
    image: nacos/nacos-server:1.1.4
    container_name: nacos3
    ports:
      - "8843:8843"
    restart: always
    network_mode: host
    privileged: true
    environment:
      SPRING_DATASOURCE_PLATFORM: mysql
      NACOS_SERVER_IP: 192.168.0.3
      NACOS_SERVERS: 192.168.0.3:8841 192.168.0.3:8842 192.168.0.3:8843
      MYSQL_MASTER_SERVICE_HOST: 192.168.0.3
      MYSQL_MASTER_SERVICE_PORT: 3301
      MYSQL_MASTER_SERVICE_DB_NAME: nacos
      MYSQL_MASTER_SERVICE_USER: root
      MYSQL_MASTER_SERVICE_PASSWORD: 123456
      MYSQL_SLAVE_SERVICE_HOST: 192.168.0.3
      MYSQL_SLAVE_SERVICE_PORT: 3302
      JVM_XMS: 1g
      JVM_XMX: 1g
      JVM_XMN: 512m
      JVM_MS: 128m
      JVM_MMS: 220m
      #NACOS_DEBUG: n
      TOMCAT_ACCESSLOG_ENABLED: 'true'
    volumes:
      - /docker/nacos-clust/nacos3/log:/home/nacos/logs
      - /docker/nacos-clust/nacos3/conf/custom.properties:/home/nacos/init.d/custom.properties" > /root/nacos-clust/nacos3/docker-compose.yaml
docker-compose up -d
```

### Nginx代理

创建挂载文件

```sh
mkdir -p /docker/nacos-nginx/{conf,logs}
```

创建初始配置文件



添加权限

```sh
touch  /docker/nacos-nginx/conf/nginx.conf
chmod 777 /docker/nacos-nginx/
```

文件添加如下

```nginx
vim 

echo "server {
       listen       8848;
       server_name  localhost;

       location / {
           proxy_pass http://nacos-cluster; 
       }

}

upstream nacos-cluster{
       server 192.168.0.3:8841 weight=5;
       server 192.168.0.3:8842 weight=5;
       server 192.168.0.3:8843 weight=5;
}" > /docker/nacos-nginx/conf/nginx.conf

```

启动容器

```sh
docker run -d \
-p 8848:8848 \
--name nginx-nacos-server \
-v /docker/nacos-nginx/conf/:/etc/nginx/conf.d/ \
-v /docker/nacos-nginx/logs:/var/log/nginx nginx
```

然后直接访问8848端口即可



# MySQL数据初始化

​		参考目录

​		https://github.com/alibaba/nacos/blob/develop/distribution/conf/nacos-mysql.sql

## 小于2.0版本

密码默认为nacos(默认加密的算法)

```sql
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `src_user` text,
  `src_ip` varchar(20) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE users (
	username varchar(50) NOT NULL PRIMARY KEY,
	password varchar(500) NOT NULL,
	enabled boolean NOT NULL
);

CREATE TABLE roles (
	username varchar(50) NOT NULL,
	role varchar(50) NOT NULL,
	constraint uk_username_role UNIQUE (username,role)
);

CREATE TABLE permissions (
    role varchar(50) NOT NULL,
    resource varchar(512) NOT NULL,
    action varchar(8) NOT NULL,
    constraint uk_role_permission UNIQUE (role,resource,action)
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

数据库修改之后修改配置文件

进入他的conf目录下找到

 cluster.conf.example这里只是一个模板我们把它复制一下然后改个名字,将他改为cluster.conf

```sh
cp cluster.conf.example cluster.conf
```

然后进入编辑修改集群地址，（最少三个）(这里配成我们的ip和端口号)

```sh
vim cluster.conf
```

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566808665597.png)

然后去修改他的mysql路径

首先编辑配置文件application.properties

```sh
vim application.properties
```

然后在里面加上mysql的配置信息（路径和用户名密码写自己的还有注意数据库名一致）(三个都要改还有端口)

```sh
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&serverTimezone=Asia/Shanghai
db.user=root
db.password=root
```

然后修改他的端口号（因为我们在一台机器中进行集群所以需要修改端口号）

分别修改

```properties
nacos1的配置文件
server.port=8801

nacos2的配置文件
server.port=8802

nacos3的配置文件
server.port=8803
```

这样我们就编写好了集群然后我们来启动吧

直接进入bin目录下找到启动脚本直接启动

```sh
 sh startup.sh 
```

我们一个一个地启动就完成了（注：可能会出现很多问题）

## 2.0.3

```sh
/*
 * Copyright 1999-2018 Alibaba Group Holding Ltd.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(50) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');

```



# 集群问题

​	 首先是秒退那么肯定就是Jvm的内存不够了，由于集群模式默认的启动大小是2个g所以我们需要去进行堆内存的调整具体如下

​	![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566808642815.png)

我们可以看到单机版启动512m，集群的将他已经修改默认是2g，我们把它修改为

```sh
-server -Xms256m -Xmx512m  -XX:PermSize=128m -XX:MaxPermSize=256m
```

然后再启动就行了



​	如果还是有问题那么我们就先去查看他的日志文件

​	进入logs然后查看文件

```sh
vim nacos.log
```

这里我们看到了

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1566808620519.png)

这个原因可能是因为数据源没写对，还有就是使用了8.0的数据库作为nacos的数据存储，只要将他改成5.6或者修改mysql8的一些配置文件就能实现

如果出现了各种错误首先我们先排查两个日志文件

```sh
vim nacos.log 

vim start.out 
```

通过这两个日志文件我们就能定位他的错误了



# 环境搭建完毕一键清理

## 清理单机版生产级Docker

停止以及删除容器

```sh
docker stop nacos-server
docker stop nacos-mysql

docker rm nacos-server
docker rm nacos-mysql
```

删除镜像（可以不用删除留着以后用）

```sh
docker rmi docker.io/nacos/nacos-server
docker rmi docker.io/mysql:5.7
```

删除挂载目录（删除后不可恢复）

```sh
rm -rf /docker/nacos
```

重新快速部署

```sh
docker run -p 3306:3306 \
--name nacos-mysql \
-e MYSQL_ROOT_PASSWORD=bigkang \
--privileged=true \
-v /docker/nacos/data:/var/lib/mysql \
-v /docker/nacos/conf/my.cnf:/etc/mysql/conf.d/mysql.cnf \
-d docker.io/mysql:5.7


--------------------------------------------------

docker run -p 8848:8848 \
--name nacos-server \
-e JAVA_TS='-Xmx256m -Xms256m -XX:+UseG1' \
-e MODE=standalone \
--privileged=true \
-v /docker/nacos/conf/application.properties:/home/nacos/conf/application.properties \
-d docker.io/nacos/nacos-server
```

## 清理集群版生产级Docker

停止容器

```sh
docker stop  nacos-node1
docker stop  nacos-node2
docker stop  nacos-node3
```

删除容器

```sh
docker rm  nacos-node1
docker rm  nacos-node2
docker rm  nacos-node3
```

