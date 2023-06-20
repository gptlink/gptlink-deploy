# Docker Compose 部署

需要先自行安装 `Docker` 与 `Docker Compose` ，可以自行百度或问 GPT 安装。 此环境包含 `MySQL` 与 `Redis` 等组件，开箱即用

配置文件路径 `docker-compose/.env`

## 运行项目

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

ARM 架构芯片设备部署时需关闭容器 `platform: linux/x86_64` 注释
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

### 保持更新/切换指定版本
```shell
# 进入 docker compose 目录
cd docker-compose

# 视情况修改 .env 中 GPTLINK_VERSION 参数

# 获取最新版镜像
docker pull overnick/gptlink

# 重建镜像
docker-compose up -d --build gptlink
```

## 开启 SSL

将证书放于以下目录并重命名，证书文件位于

`docker-compose/gptlink/ssl/website.key`
`docker-compose/gptlink/ssl/website.pem`

给站点开启 SSl

```shell
# 将 docker-compose/gptlink/conf/nginx-default.conf.back 重命名为 nginx-default.conf
cp docker-compose/gptlink/conf/nginx-default.conf.back docker-compose/gptlink/conf/nginx-default.conf

# 查看并修改相关信息，默认不开启强制跳转至https
vim docker-compose/gptlink/conf/nginx-default.conf

# 修改 `docker-compose/gptlink/Dockerfile` 文件，解除相关注释
vim docker-compose/gptlink/Dockerfile

```

## 修改 Redis配置

如需给 Redis 添加密码，修改文件 `redis/redis.conf` 中的配置，修改内容自行参考 Redis 配置。

重新构建并运行 Redis 

```shell
docker-compose up -d --build redis
```


## 从 gptlink 迁移（gptlink < 1.0 版本迁移至本项目使用）

1. 暂停 mysql 与 redis 服务

```shell
docker-compose stop mysql redis
```

2. 复制数据到当前项目中，以下路径中 gptlink 代表 [gitlink](https://github.com/gptlink/gptlink) 项目目录， gitlink-deply 代表本项目目录

示例:

```shell
cp -r gptlink/docker-compose/data/mysql gptlink-deploy/docker-compose/data/mysql
cp -r gptlink/docker-compose/data/redis gptlink-deploy/docker-compose/data/redis
```

3. 复制 .env 文件

```shell
cp gptlink/docker-compose/.env gptlink-deploy/docker-compose/.env
```

4. 重新运行服务

```shell
docker-compose up -d mysql redis gptlink
```
