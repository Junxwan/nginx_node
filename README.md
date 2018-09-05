# Nginx 筆記
- [Nginx 筆記](#nginx-%E7%AD%86%E8%A8%98)
    - [install module](#install-module)
    - [$document_root](#documentroot)
    - [$fastcgi_script_name](#fastcgiscriptname)
    - [$uri](#uri)
    - [$args](#args)
    - [$request_uri](#requesturi)
    - [set](#set)
    - [try_files](#try_files)
    - [rewrite](#rewrite)
        - [配對到某的path之後全改寫](#配對到某的path之後全改寫)
    - [基本範例](#%E5%9F%BA%E6%9C%AC%E7%AF%84%E4%BE%8B)

## install module
假設想要安裝nginx第三方模組則需要重新編譯nginx執行檔，本篇設定mac電腦上已有安裝nginx且是透過brew install nginx安裝，以nginx echo-nginx-module為例子

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

從`--prefix`到最後就是編譯本地nginx的資訊，在最後面加上`--add-module=/path/to/echo-nginx-module`，要安裝的module

    $ ./configure --prefix=/usr/local/Cellar/nginx/1.15.3 --sbin-path=/usr/local/Cellar/nginx/1.15.3/bin/nginx --with-cc-opt='-I/usr/local/opt/pcre/include -I/usr/local/opt/openssl/include' --with-ld-opt='-L/usr/local/opt/pcre/lib -L/usr/local/opt/openssl/lib' --conf-path=/usr/local/etc/nginx/nginx.conf --pid-path=/usr/local/var/run/nginx.pid --lock-path=/usr/local/var/run/nginx.lock --http-client-body-temp-path=/usr/local/var/run/nginx/client_body_temp --http-proxy-temp-path=/usr/local/var/run/nginx/proxy_temp --http-fastcgi-temp-path=/usr/local/var/run/nginx/fastcgi_temp --http-uwsgi-temp-path=/usr/local/var/run/nginx/uwsgi_temp --http-scgi-temp-path=/usr/local/var/run/nginx/scgi_temp --http-log-path=/usr/local/var/log/nginx/access.log --error-log-path=/usr/local/var/log/nginx/error.log --with-debug --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-ipv6 --with-mail --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --add-module=/Users/ne0002/Downloads/echo-nginx-module-0.61 --add-module=/path/to/echo-nginx-module

    $ make

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

## $fastcgi_script_name

假設請求為`http://www.api.test/nginx`，$fastcgi_script_name為`/nginx`

> 此值會傳到PHP $_SERVER['SCRIPT_NAME']

## $uri

假設請求為`http://www.api.test/nginx/index.php`，$uri就是`/nginx/index.php`

## $args
假設請求為`http://www.api.test/nginx?q=1`，$args就是`q=1`

## $request_uri

假設請求為`http://www.api.test/nginx`，$request_uri為`/nginx`

> 此值會傳到PHP $_SERVER['REQUEST_URI']

## set

設定變數如下將`$document_root`設定為`/var/www/public`

    server {
        listen       80;

        server_name  nginx.web.test;

        root    /var/www;

        set $document_root /var/www/public;

        index  index.php;

        location / {
            echo $document_root; # 利用echo輸出$document_root做測試
        }
    }

設定一個變數

## try_files

假設發送一個請求為`http://web.api.test`，符合`location /`而`try_files $uri $uri/ /index.php`，代表著嘗試存取該規則檔案或是目錄

    server {
        listen       80;

        server_name  web.api.test;

        root    /var/www/web/public

        index  index.php;

        location / {
            try_files $uri $uri/ /index.php;
        }

        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }

1. `$uri`為` `會嘗試讀取/var/www/web/public檔案

2. `$uri/`為`/`會嘗試讀取/var/www/web/public/目錄

3. 當前面嘗試存取的檔案都不存在時，最後一個規則一定要能存取成功
`/index.php`代表讀取/var/www/web/public/index.php

4. 由於是`.php`檔案所以會藉由fastcgi將index.php傳送至php-fpm做解析

# rewrite 

改寫url

## 配對到某的path之後全改寫

    rewrite ^(/dashbord)/(.*)$ /login/ last;

## 基本範例

    server {
        listen       80;                    # 監聽80 prot                                            
        
        server_name  web.fontend.test;      # host
        
        root    /var/www/web/public;        # 專案根目錄

        index  index.php;                   # 預設執行檔案

        location / {                        # 當url匹配到/
            try_files $uri $uri/ /index.php$is_args$args;   # 嘗試找這些檔案
        }

        location ~ \.php$ {                                 # url檔案為.php就利用fastcgi機制送到php-fpm解析php檔
            root           /var/www/web/public;;            
            fastcgi_pass   127.0.0.1:9000;                  # php-fpm默認執行在9000 prot，將php送到此prot做解析 
            fastcgi_index  index.php;                   
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; # 要解析的php檔案與url path info
            include        fastcgi_params;                                      # 載入nginx/fastcgi_params做一些設定環境變數                       
        }

        error_log /www/log/nginx/web_fontend_error.log;
        access_log /www/log/nginx/web_fontend_access.log;
    }