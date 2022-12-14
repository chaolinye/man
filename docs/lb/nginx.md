# Nginx

## 功能

- web 服务器

- 反向代理

- 负载均衡

## 请求转发规则

![](../images/nginx-request.png)

[How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html)

[location 文档](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)

示例定义

```nginx
  server {
      listen      80;
      server_name example.org www.example.org;
      ...
  }

  server {
      listen      80 default_server;
      server_name example.net www.example.net;
      ...
      # 精准匹配
      location = / {
          root /data/www/index.html
      }
      # 前缀匹配
      location / {
          root /data/www;
      }
      # 高优先级前缀匹配
      location ^~ /images/ {
          proxy_pass http://127.0.0.1:8989;
      }
      # 大小写不敏感的正则匹配
      location ~* \.(gif|jpg|png)$ {
          expires 30d;
      }
      # 正则匹配
      location ~ \.php$ {
          fastcgi_pass  localhost:9000;
          fastcgi_param SCRIPT_FILENAME
                        $document_root$fastcgi_script_name;
          include       fastcgi_params;
      }
  }

  server {
      listen      80;
      server_name example.com www.example.com;
      ...
  }
  
```

## 常用命令

```bash
# 启动 nginx
nginx
# 验证配置文件的正确性，每次修改配置文件最好都验证下
nginx -t
# 重新载入配置文件，不停机
nginx -s reload
```

## 创建

## 配置

```nginx
user       www www;  ## Default: nobody
worker_processes  5;  ## Default: 1
error_log  logs/error.log;
pid        logs/nginx.pid;
worker_rlimit_nofile 8192;

events {
  worker_connections  4096;  ## Default: 1024
}

http {
  include    conf/mime.types;
  include    /etc/nginx/proxy.conf;
  include    /etc/nginx/fastcgi.conf;
  index    index.html index.htm index.php;

  default_type application/octet-stream;
  log_format   main '$remote_addr - $remote_user [$time_local]  $status '
    '"$request" $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log   logs/access.log  main;
  sendfile     on;
  tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required for some vhosts

  server { # php/fastcgi
    listen       80;
    server_name  domain1.com www.domain1.com;
    access_log   logs/domain1.access.log  main;
    root         html;

    location ~ \.php$ {
      fastcgi_pass   127.0.0.1:1025;
    }
  }

  server { # simple reverse-proxy
    listen       80;
    server_name  domain2.com www.domain2.com;
    access_log   logs/domain2.access.log  main;

    # serve static files
    location ~ ^/(images|javascript|js|css|flash|media|static)/  {
      root    /var/www/virtual/big.server.com/htdocs;
      expires 30d;
    }

    # pass requests for dynamic content to rails/turbogears/zope, et al
    location / {
      proxy_pass      http://127.0.0.1:8080;
    }
  }

  upstream big_server_com {
    server 127.0.0.3:8000 weight=5;
    server 127.0.0.3:8001 weight=5;
    server 192.168.0.1:8000;
    server 192.168.0.1:8001;
  }

  server { # simple load balancing
    listen          80;
    server_name     big.server.com;
    access_log      logs/big.server.access.log main;

    location / {
      proxy_pass      http://big_server_com;
    }
  }
}
```

## References

- [Nginx 官方文档](https://nginx.org/en/docs/)
- [Nginx 配置 Example]()
- [Nginx vs HAProxy](https://www.keycdn.com/support/haproxy-vs-nginx)