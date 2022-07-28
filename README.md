# Nginx Lua 代理 Redis 哨兵节点
## 环境准备
### 1. 下载各个模块源码
+ Ngxin v1.20.2
 	> 官网: http://nginx.org/en/download.html  
	> 命令: `wget http://nginx.org/download/nginx-1.20.2.tar.gz`
+ Openresty LuaJIT v2.1.0-beta3 
 	> 官网: https://github.com/openresty/luajit2  
	> 命令: `git clone -b v2.1.0-beta3 https://github.com/openresty/luajit2.git`
+ NDK (ngx_devel_kit）module v0.3.1
 	> 官网: https://github.com/vision5/ngx_devel_kit  
	> 命令: `git clone -b v0.3.1 https://github.com/vision5/ngx_devel_kit.git`
+ ngx_lua module v0.10.21
 	> 官网: https://github.com/openresty/lua-nginx-module  
	> 命令: `git clone -b v0.10.21 https://github.com/openresty/lua-nginx-module.git`
+ stream-lua-nginx module v0.0.11
 	> 官网: https://github.com/openresty/stream-lua-nginx-module  
	> 命令: `git clone -b v0.0.11 https://github.com/openresty/stream-lua-nginx-module.git`
+ lua-resty-core v0.1.23
 	> 官网: https://github.com/openresty/lua-resty-core  
	> 命令: `git clone -b v0.1.23 https://github.com/openresty/lua-resty-core.git`
+ lua-resty-lrucache v0.13
 	> 官网: https://github.com/openresty/lua-resty-lrucache  
	> 命令: `git clone -b v0.13 https://github.com/openresty/lua-resty-lrucache.git`
+ lua-resty-redis v0.30
 	> 官网: https://github.com/openresty/lua-resty-redis  
	> 命令: `git clone -b v0.30 https://github.com/openresty/lua-resty-redis.git`
+ 

### 2. 源代码编译
```shell
# 安装依赖 make gcc
yum install -y make gcc pcre-devel openssl-devel

# 编译 luaJIT
cd LuaJit
make install PREFIX=/usr/local/LuaJIT
export LUAJIT_LIB=/usr/local/LuaJIT/lib
export LUAJIT_INC=/usr/local/LuaJIT/include/luajit-2.1



# 创建 nginx 用户和用户组
groupadd nginx
useradd -g nginx nginx -s /sbin/nologin

# 创建 nginx 目录
mkdir -p /var/cache/nginx

# 编译 nginx
tar -xzvf nginx-1.20.2.tar.gz
cd nginx-1.20.2

./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-rpath,/usr/local/LuaJIT/lib' --add-module=/root/nginxlua/ngx_devel_kit --add-module=/root/nginxlua/lua-nginx-module --add-module=/root/nginxlua/stream-lua-nginx-module-0.0.11

make -j2
make install 

# 编译核心模块
cd lua-resty-core
make install PREFIX=/etc/nginx
cd lua-resty-lrucache
make install PREFIX=/etc/nginx
cd lua-resty-redis
make install PREFIX=/etc/nginx
```
### 3. 修改 nginx 配置文件
vim /etc/nginx/nginx.conf

添加到 http 上下文括号中 `lua_package_path "/etc/nginx/lib/lua/?.lua;;";`
> 在 stream 上下文中使用也需要添加
### 4. 清理环境
rm -rf nginxlua