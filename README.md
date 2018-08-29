# Nginx 筆記

**[index](#index)**
* [install module]](#install module)
* [$document_root](#$document_root)
* [$uri](#$uri)
* [$args](#$args)
* [try_files](#try_files)

## install module
假設想要安裝nginx第三方模組則需要重新編譯nginx執行檔，本篇設定mac電腦上已有安裝nginx且是透過brew install nginx安裝

以nginx echo-nginx-module為例子

首先看看本地的nginx版本

    $ nginx -v
      nginx version: nginx/1.15.2

到nginx官網下載相對應的nginx

    $ wget 'http://nginx.org/download/nginx-1.15.2.tar.gz'
    $ tar -xzvf nginx-1.15.2.tar.gz
    $ cd nginx-1.15.2

下載 echo-nginx-module

    $ wget 'https://github.com/openresty/echo-nginx-module/archive/v0.61.tar.gz'
    $ tar -xzvf echo-nginx-module-0.61.tar.gz

要重新編譯nginx並替換掉本地的nginx需要先知道目前nginx install的編譯資訊
    $ nginx -V
    nginx version: nginx/1.15.2
    built by clang 9.1.0 (clang-902.0.39.2)
    built with OpenSSL 1.0.2p  14 Aug 2018
    TLS SNI support enabled
    configure arguments: --prefix=/usr/local/Cellar/nginx/1.15.3 --sbin-path=/usr/local/Cellar/nginx/1.15.3/bin/nginx --with-cc-opt='-I/usr/local/opt/pcre/include -I/usr/local/opt/openssl/include' --with-ld-opt='-L/usr/local/opt/pcre/lib -L/usr/local/opt/openssl/lib' --conf-path=/usr/local/etc/nginx/nginx.conf --pid-path=/usr/local/var/run/nginx.pid --lock-path=/usr/local/var/run/nginx.lock --http-client-body-temp-path=/usr/local/var/run/nginx/client_body_temp --http-proxy-temp-path=/usr/local/var/run/nginx/proxy_temp --http-fastcgi-temp-path=/usr/local/var/run/nginx/fastcgi_temp --http-uwsgi-temp-path=/usr/local/var/run/nginx/uwsgi_temp --http-scgi-temp-path=/usr/local/var/run/nginx/scgi_temp --http-log-path=/usr/local/var/log/nginx/access.log --error-log-path=/usr/local/var/log/nginx/error.log --with-debug --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-ipv6 --with-mail --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --add-module=/Users/ne0002/Downloads/echo-nginx-module-0.61

從--prefix到最後就是編譯本地nginx的資訊，在最後面加上--add-module=/path/to/echo-nginx-module，要安裝的module

    $ ./configure --prefix=/usr/local/Cellar/nginx/1.15.3 --sbin-path=/usr/local/Cellar/nginx/1.15.3/bin/nginx --with-cc-opt='-I/usr/local/opt/pcre/include -I/usr/local/opt/openssl/include' --with-ld-opt='-L/usr/local/opt/pcre/lib -L/usr/local/opt/openssl/lib' --conf-path=/usr/local/etc/nginx/nginx.conf --pid-path=/usr/local/var/run/nginx.pid --lock-path=/usr/local/var/run/nginx.lock --http-client-body-temp-path=/usr/local/var/run/nginx/client_body_temp --http-proxy-temp-path=/usr/local/var/run/nginx/proxy_temp --http-fastcgi-temp-path=/usr/local/var/run/nginx/fastcgi_temp --http-uwsgi-temp-path=/usr/local/var/run/nginx/uwsgi_temp --http-scgi-temp-path=/usr/local/var/run/nginx/scgi_temp --http-log-path=/usr/local/var/log/nginx/access.log --error-log-path=/usr/local/var/log/nginx/error.log --with-debug --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-ipv6 --with-mail --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --add-module=/Users/ne0002/Downloads/echo-nginx-module-0.61 --add-module=/path/to/echo-nginx-module

    $make

然後在將編譯完成的nginx覆蓋到本地的nginx，重啟nginx就完成

    $ cp nginx-1.15.2/objs/nginx /usr/local/Cellar/nginx/1.15.3/bin

> echo-nginx-module會echo 資料在瀏覽器

測試

    server {
        listen       80;
        
        server_name  nginx.web.test;

        root    /var/www;

        index  index.php;

        location / {
            echo $document_root; # 利用echo輸出$document_root做測試
        }
    }

## $document_root

由root配置，當root是`/var/www`，同理$document_root就是`/var/www`

## $uri

假設請求為http://www.api.test/nginx/index.php，$uri就是/nginx/index.php

## $args
假設請求為http://www.api.test/nginx?q=1，$args就是q=1

## try_files 