# Docker 部署

`Docker` 镜像中不包含数据库与 `Redis`，关于 `Docker` 的安装，可以自行百度或问 GPT。

连接地址不要填 localhost，127.0.0.1，等指向本机 ip 的地址，可以填写网关ip 或宿主机的内网 ip 

## 运行项目

```shell
docker run -d -p 80:80 \
   --name=gptlink \
   -e DB_HOST="数据库连接地址" \
   -e DB_DATABASE="数据库名称" \
   -e DB_USERNAME="数据库用户名" \
   -e DB_PASSWORD="数据库密码" \
   -e REDIS_HOST="Redis 链接地址" \
   -e REDIS_PORT="Redis 端口号" \
   overnick/gptlink

# 如果你需要指定其他环境变量，请自行在上述命令中增加 `-e 环境变量=环境变量值` 来指定。

# 测试配置项是否正常
docker run -it --rm gptlink /app/gptserver/test.sh

```
ARM 架构芯片设备部署时需增加参数 `--platform=linux/x86_64` , 示例如下

```shell
docker run -d -p 80:80 \
   --platform=linux/x86_64
   --name=gptlink \
   -e DB_HOST="数据库连接地址" \
   -e DB_DATABASE="数据库名称" \
   -e DB_USERNAME="数据库用户名" \
   -e DB_PASSWORD="数据库密码" \
   -e REDIS_HOST="Redis 链接地址" \
   -e REDIS_PORT="Redis 端口号" \
   overnick/gptlink
```

### 更新版本/更新配置

删除运行的容器，并且使用安装命令重新运行指定版本镜像或修改配置即可
