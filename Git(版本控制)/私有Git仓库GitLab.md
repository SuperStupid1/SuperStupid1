# 首先创建挂载目录

```shell
mkdir -p /docker/gitlab/{config,logs,data}
```

# 启动容器

然后启动容器,我这采用的汉化版的gitlab，如果需要其他版本请搜索下载并且修改

```shell
docker run --restart=always -d \
--name gitlab \
-h 111.67.196.127 \
-p 10443:443 \
-p 10080:80 \
-p 10022:22 \
-v /docker/gitlab/config:/etc/gitlab \
-v /docker/gitlab/logs:/var/log/gitlab \
-v /docker/gitlab/data:/var/opt/gitlab \
docker.io/twang2218/gitlab-ce-zh
```

然后访问10080，设置默认的root密码

# 问题

切记不要使用root用户去操作，否则会出现权限问题

## GItLab的URl和SSH问题

修改了之后我们会发现url和ssh上都没有端口号了

![](https://blog-kang.oss-cn-beijing.aliyuncs.com/UTOOLS1569571302989.png)

我们就需要去修改他的配置了

进入挂载目录并且编辑配置文件

```shell
vim /docker/gitlab/config/gitlab.rb

然后添加上
external_url 'http://111.67.196.127:10080'
nginx['listen_port'] = 10080
gitlab_rails['gitlab_shell_ssh_port'] = 10022
nginx['listen_https'] = false
我们把它复制出来修后需要删除docker容器并且重新启动，然后修改端口

docker stop gitlab
docker rm gitlab

docker run --restart=always -d \
--name gitlab \
-h 111.67.196.127 \
-p 10443:443 \
-p 10080:10080 \
-p 10022:10022 \
-v /docker/gitlab/config:/etc/gitlab \
-v /docker/gitlab/logs:/var/log/gitlab \
-v /docker/gitlab/data:/var/opt/gitlab \
docker.io/twang2218/gitlab-ce-zh

```





# Compose启动



```sh
# 创建挂载文件
cd ~ && mkdir -p deploy && cd deploy && mkdir -p gitlab-server && cd gitlab-server

# 创建Compose文件
cat > ./docker-compose-gitlab-server.yml << EOF
version: '3.4'
services:
  gitlab-server:
    container_name: gitlab-server       # 指定容器的名称
    image: docker.io/gitlab/gitlab-ce:latest        # 指定镜像和版本
    restart: always  # 自动重启
    hostname: gitlab-server					# 主机名
    ports:
      - 10443:443
      - 10080:80
      - 10022:22
    privileged: true
    volumes: # 挂载目录
     - ./config:/etc/gitlab
     - ./logs:/var/log/gitlab
     - ./data:/var/opt/gitlab
EOF


# 启动（先修改下方配置再进行启动）
docker-compose -f docker-compose-gitlab-server.yml up -d
```

