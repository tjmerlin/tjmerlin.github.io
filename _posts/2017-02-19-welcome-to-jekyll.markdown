---
layout: post
title:  openresty
date:   2017-02-19 09:12:08 +0800
categories: jekyll update
---

# openresty 笔记

## openresty 解决了哪些问题

`Linux kernel` 发布了 **2.6** 后引入了新的非阻塞 **I/O** 复用机制 (`epoll`), 在高性能服务器领域得到了广泛的应用. 其中 `nginx` 就是使用该机制开的.


后续有人基于 `nginx` 开发了 `openresty`(实际上就是用 `nginx` 最核心的部分, 加上了 `lua`, 并且提供了很多 `lua` 的库, 让第三方服务可以很好对接, 让 `nginx` 可以不只是一个 `web server`, 而是一个 `web framework`.

openresty ≈ nginx + lua


## openresty 安装 (centos)

### 依赖安装

```
$ yum install readline-devel pcre-devel openssl-devel perl gcc lua luajit perl-Digest-MD5 # 其中 perl-Digest-MD5 为 opm 使用
```

### 编译 (安装)

```
$ wget https://openresty.org/download/openresty-1.11.2.2.tar.gz
$ tar -zxvf openresty-1.11.2.2.tar.gz
$ ./configure --with-luajit
$ gmake
$ gmake install
```
默认 `openresty` 安装到 `/usr/local/openresty`, 需手动配置 **PATH**


## 使用 openresty

### 1. 初始化项目目录


```
$ mkdir -p ~/openresty-test/conf ~/openresty-test/logs
```

### 2. 创建项目配置文件

```
$ touch ~/openresty-test/conf/nginx.conf
```

写入如下内容

```nginx
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
    worker_connections 1024;
}

http {
    server {
        #监听端口，若你的6699端口已经被占用，则需要修改
        listen 6699;
        location / {
            default_type text/html;

            content_by_lua_block {
                ngx.say('Hello World')
            }
        }
    }
}
```

### 3. 运行 openresty 服务

```
$ nginx -p ~/openresty-test
```

可使用如下命令检查是否正常服务启用

* `netstat -anp | grep 6699`
* `ps aux | grep nginx`
* `curl localhost:6699 -i`


### 4. 进阶 （使用 `opm` 进行依赖管理)

#### opm 简介

`opm` : `openresty packager manager`

`opm` 网站: `http://opm.openresty.org`


#### opm 简单使用

`opm install openresty/lua-resty-redis` 安装

安装完成后, 会默认安装到 `/usr/local/openresty/site/lualib` 目录中.

#### ngixn 配置中使用

```nginx
 content_by_lua_block {
	local args = ngx.req.get_uri_args()
	local redis = require "resty.redis"
	local red = redis:new()
	red:set_timeout(1000) -- 1 sec
	local ok, err = red:connect('127.0.0.1', 6379)
	if not ok then
	    ngx.say('fail to connect:', err)
	    return
	end
	ok, err = red:set(args.a, args.b)
	if not ok then
	    ngx.say('set error: ', err)
	end

	ngx.say('success!')
}
```
