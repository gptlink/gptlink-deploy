# Docker Compose 部署

需要先自行安装 `Docker` 与 `Docker Compose` ，可以自行百度或问 GPT 安装。 此环境包含 `MySQL` 与 `Redis` 等组件，开箱即用

配置文件路径 `docker-compose/.env`

### 运行项目

```shell
# clone代码
git clone https://github.com/gptlink/gptlink-deploy.git

# 进入 docker compose 目录
cd gptlink-deploy/docker-compose

# 复制配置项文件，具体配置内容可以参考文件内注释
# 如无其他需求可不修改此文件内容
cp .env.example .env

# 运行 Mysql 与 Redis 服务，如已有相关服务，可不进行启动
# 如遇端口冲突，可尝试一下方案
# 1. 可关闭机器中的 MySQL 与 Redis
# 2. 修改 docker-compose/.env 中的 MYSQL_PORT , REDIS_PORT 配置重新运行
docker-compose up -d mysql redis

# 运行 Web 服务
docker-compose up -d gptlink
```

Mac M1，M2芯片设备本地部署调试需关闭容器 `platform: linux/x86_64` 注释
```yaml
# ... 
services:
  redis:
    build: ./redis
    platform: linux/x86_64
    volumes:
      - ${DATA_PATH}/redis:/data

# ... 
```

### 更新版本/更新配置

如果本地已存在旧镜像，建议先删除 `docker rmi overnick/gptlink`

```shell
# 更新 gptlink 代码
git pull origin master

# 进入 docker compose 目录
cd docker-compose

docker-compose up -d gptlink
```

### 开启 SSL

修改 `docker-compose/gptlink/Dockerfile` 文件，根据内容提示进行修改。

证书文件放于 

- docker-compose/gptlink/ssl/website.key
- docker-compose/gptlink/ssl/website.pem

如要开启强制跳转至 https ， 需修改 `docker-compose/gptlink/conf/nginx-default.conf` 中的配置
