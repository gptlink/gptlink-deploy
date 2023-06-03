## PHP 环境部署

代码仓库地址：https://github.com/gptlink/gptlink

环境部署需要动手能力较强的同学进行。项目目录结构为

- gptserver  服务器项目目录
- gptweb 用户端项目目录
- gptadmin 管理端项目目录

环境要求

- Nginx
- MySql 5.7 +
- Redis 5.0 +
- PHP 8.0
    - ext-swoole
    - ext-openssl
    - ext-json
    - ext-pdo_mysql
    - ext-redis
    - ext-bcmath

项目提供的 `Nginx` 配置文件，位于 `conf/nginx-default.conf`，可以参考或借鉴。前端项目目前请求的接口地址是固定的 `/api/`

服务启动命令参考 `gptserver/start.sh`
