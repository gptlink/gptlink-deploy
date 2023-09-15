## 1. 安装php8.0版本：

```jsx
brew install php@8.0
```
安装完成后提示如下：
```jsx
To enable PHP in Apache add the following to httpd.conf and restart Apache:
    LoadModule php_module /opt/homebrew/opt/php@8.0/lib/httpd/modules/libphp.so

    <FilesMatch \.php$>
        SetHandler application/x-httpd-php
    </FilesMatch>

Finally, check DirectoryIndex includes index.php
    DirectoryIndex index.php index.html

The php.ini and php-fpm.ini file can be found in:
    /opt/homebrew/etc/php/8.0/

php@8.0 is keg-only, which means it was not symlinked into /opt/homebrew,
because this is an alternate version of another formula.

If you need to have php@8.0 first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/php@8.0/bin:$PATH"' >> ~/.zshrc
  echo 'export PATH="/opt/homebrew/opt/php@8.0/sbin:$PATH"' >> ~/.zshrc

For compilers to find php@8.0 you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/php@8.0/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/php@8.0/include"

To start php@8.0 now and restart at login:
  brew services start php@8.0
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/php@8.0/sbin/php-fpm --nodaemonize
==> scrcpy
At runtime, adb must be accessible from your PATH.

You can install adb from Homebrew Cask:
  brew install --cask android-platform-tools

zsh completions have been installed to:
  /opt/homebrew/share/zsh/site-functions
```

根据如上提示在 ～/.zshrc 文件中添加下面环境变量配置：

```jsx

export PATH="/opt/homebrew/opt/php@8.0/bin:$PATH"
export PATH="/opt/homebrew/opt/php@8.0/sbin:$PATH"
```

添加完成后重启命令行执行 `php -v` 打印如下说明 php 安装成功：

```jsx
➜ php -v
PHP 8.0.29 (cli) (built: Jun 15 2023 05:07:53) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.0.29, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.29, Copyright (c), by Zend Technologies
```

## 2. 安装 php 的包管理工具 composer:
```jsx
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer update
```
安装完成后执行 `composer -v` 查看是否安装成功
```jsx
➜ composer -v
Composer version 2.5.8 2023-06-09 17:13:21
```
此时进入项目内 gptserver 目录内执行 `composer update` 更新 composer.json 内的依赖
```jsx
cd gptlink/gptserver
composer update
```
执行 `composer update` 时报错如下：（没有安装 php 的 swoole 扩展）
```jsx
96qbhy/hyperf-auth[v2.0,...v2.7.1]require e ext-swoole >=4.4 -it is missing from your system
To enable extensions, verify that they are enabled in your .ini files:
    - /opt/homebrew/etc/php/8.0/php.ini
    - /opt/homebrew/etc/php/8.0/conf.d/ext-opcache.ini
```

## 3. 安装 swoole 扩展和 redis 扩展：

```jsx
pecl install swoole
pecl install redis
```

安装 swoole 扩展时会提示报错如下：（找不到 "pcre2.h" 文件）

```jsx
/opt/homebrew/Cellar/php@8.0/8.0.29_1/include/php/ext/pcre/php_pcre.h:23:10: fatal error: 'pcre2.h' file not found
#include "pcre2.h"
```

解决方法创建 pcre2.h 软链接到 /usr/local/include 文件夹下：

```jsx
// 1. 执行下面代码查看 pcre2 安装的路径找到 pcre2.h 文件的路径，我这里找到的是 /opt/homebrew/include/pcre2.h， 你可以看看有没有这个文件
brew info pcre2
// 2. 创建 /usr/local/include 文件夹
sudo mkdir /usr/local/include
// 3. 创建软链接
sudo ln -s /opt/homebrew/include/pcre2.h /usr/local/include/
// 4. 添加这个环境变量，安装 php swoole 扩展编译时候需要
export C_INCLUDE_PATH=/usr/local/include
// 5. 再次执行 pecl install swoole 一路回车就行
```

安装成功后 /opt/homebrew/etc/php/8.0/php.ini 增加的配置如下：

```jsx
extension="redis.so"
extension="swoole.so"
swoole.use_shortname=Off // 需要手动添加
```

## 4. 安装 gptserver 项目依赖：

```jsx
composer update
composer install
```

## 5. 启动 gptserver 项目：

启动项目前需启动 mysql 和 redis 服务，并在 gptserver 文件夹下创建 .env 文件内容如下：
```jsx
APP_NAME=chatgpt-link
APP_ENV=prod

# 数据库 通过 docker 启动命令：docker run -d -p 3306:3306 --name mysql57 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=gptlink mysql:5.7
DB_HOST=192.168.30.19 # 你本机电脑的 ip 地址：命令行输入 ifconfig 查看 en0
DB_PORT=3306
DB_DATABASE=gptlink
DB_USERNAME=root
DB_PASSWORD=123456

#redis 通过 docker 启动命令：docker run -d --name redis -p 6379:6379 redis
REDIS_HOST=192.168.30.19
REDIS_AUTH=(null)
REDIS_PORT=6379

# 管理员账号密码
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin888
ADMIN_TTL=7200
```

启动php后端服务：
```jsx
cd gptlink/gptserver
// 初始化数据库
php ./hyperf migrate
// 启动本地后端服务
php ./hyperf start
// 端口在 http://127.0.0.1:9503， 前后端设置 proxy 代理到该端口开发
```