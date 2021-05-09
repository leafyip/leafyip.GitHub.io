---
title: nginx配置load-balancer
date: 2020-12-06 17:49:59
tags: 
	- nginx
	- minio
---

需求 :
使用 `http://www.minio.muklee.com` 
访问minio集群
`http://192.168.26.131:9000,http://192.168.26.132:9000,http://192.168.26.133:9000,http://192.168.26.133:9000`

修改windows本地host文件
编辑 C:\Windows\System32\drivers\etc\hosts
```
192.168.26.131 www.minio.muklee.com
```

注 : 192.168.26.131 为nginx所在机器的ip

<!--more-->

nginx.conf 配置如下

```


worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;


    server {
        listen       80;
        server_name  localhost;


        location / {
            root   html;
            index  index.html index.htm;
        }

        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }


	
	server {
        listen       80;
        server_name  www.minio.muklee.com;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
            proxy_pass   http://minio_cluster;
        }
    }
	
	upstream minio_cluster {
		server 192.168.26.131:9000;
		server 192.168.26.132:9000;
		server 192.168.26.133:9000;
		server 192.168.26.134:9000;
	}

}

```

windows打开浏览器,输入`http://www.minio.muklee.com/`

![Alt text](/images/minio_disturibe_login.png) 