部署实例后要全部重新启动



# Mss
Matrix-synapse-server

创建管理员

docker exec -it synapse register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml -a

//////////////////////////////////////////////
创建用户

docker exec -it synapse register_new_matrix_user -c /data/homeserver.yaml http://localhost:8008

//////////////////////////////////////////////////

nginx 配置文件:


client_max_body_size 0;
client_body_buffer_size  128k;
sendfile        on;  # 启用高效的文件传输模式
tcp_nopush      on;  # 减少网络包的数量
tcp_nodelay     on;  # 禁用Nagle算法，加快小数据包的发送
keepalive_timeout  75;  # 保持连接的时间
keepalive_requests 1000;  # 单个keepalive连接允许的最大请求数
client_body_timeout   120;  # 客户端请求体的超时时间，单位秒
client_header_timeout 120;  # 客户端请求头的超时时间，单位秒
send_timeout          120;  #
    gzip  on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss;

location  / {
       proxy_pass http://localhost:8070;
       proxy_set_header X-Forwarded-For $remote_addr;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header Host $host;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-Host $host;
       proxy_buffering off;
       proxy_request_buffering off;
        }

location ~ ^(/_matrix|/_synapse/client|/_synapse/admin)  {
       proxy_pass http://localhost:8008;
       proxy_set_header X-Forwarded-For $remote_addr;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header Host $host;
       Support WebSocket
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-Host $host;
       proxy_buffering off;
       proxy_request_buffering off;
}
      
location /.well-known/matrix/server {
       default_type application/json;  # 设置返回类型为 JSON   联邦通讯
       add_header Access-Control-Allow-Origin *;  # 允许跨域访问
       return 200 '{"m.server": "ms1.aosn.de:443"}';  
}
location /.well-known/matrix/client {
      access_log off;
      add_header Access-Control-Allow-Origin *;
      default_type application/json;
      return 200 '{"m.homeserver": {"base_url": "https://ms1.aosn.de"}}';
}
location ~ ^/_matrix/client/(r0|v3)/sync$ {
      proxy_pass http://localhost:8081;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;
}

location ~ ^/_matrix/client/(api/v1|r0|v3)/events$ {
      proxy_pass http://localhost:8081;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;
}

location ~ ^/_matrix/client/(api/v1|r0|v3)/initialSync$ {
      proxy_pass http://localhost:8081;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;
}

location ~ ^/_matrix/client/(api/v1|r0|v3)/rooms/[^/]+/initialSync$ {
      proxy_pass http://localhost:8081;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;
}


