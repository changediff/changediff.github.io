---
title: 压测中使用openresty作为upstream来业务耗时
date: 2023-08-14 22:00:59
tags: ["load test", "openresty"]
---
最近在压测Apisix，需要按照一定的概率分布模拟请求耗时，以便达到模拟真实业务请求来压测的目的。

第一版后端程序使用Go语言，性能不达标，没法压到Apisix的极限；
第二版使用Nginx作为后端程序，性能达标，压测4C8G的Apisix，极限TPS约1.4W；
第三版使用openresty配合lua来模拟耗时，性能达标，4C8G的实例TPS约1.2W。

```
# conf.d/default.conf
server {
    listen       80;
    server_name  _;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
    access_log off;

    location /test {
        content_by_lua_block {
            ngx.say("hello world");
        } 
    }

    location /sleep {
        content_by_lua_block {
            local resty_random = require "resty.random"

            local function random_wait_time(min_wait_time, max_wait_time)
                local random_bytes = resty_random.bytes(4, true)
                local random_float = 1 + (0xffffffff % tonumber(0xffffffff)) * 2^-32
                local wait_time = min_wait_time + random_bytes:byte(1) / 0xff * (max_wait_time - min_wait_time)
                return wait_time
            end

            local wait_time_seed = random_wait_time(0, 10000)
            local wait_time = 0
            if wait_time_seed > 9990 then
                wait_time = random_wait_time(2, 5)
            elseif wait_time_seed > 9900 then
                wait_time = random_wait_time(1, 2)
            elseif wait_time_seed > 9000 then
                wait_time = random_wait_time(0.5, 1)
            elseif wait_time_seed > 8000 then
                wait_time = random_wait_time(0.05, 0.5)
            else
                wait_time = random_wait_time(0, 0.05)
            end
            ngx.sleep(wait_time)
            ngx.header["Content-type"] = 'text/plain'
            ngx.say("Hello, OpenResty! I waited for " .. wait_time .. " seconds. Seed is " .. wait_time_seed .. ".\n")
        }
    }

    location / {
        root   /usr/local/openresty/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/local/openresty/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           /usr/local/openresty/nginx/html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```