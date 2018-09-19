[root@nginx02 conf]# cat nginx.conf

worker_processes  40;
error_log   logs/error.log  crit;
worker_rlimit_nofile 204800;
events {
    use epoll;
    worker_connections  204800;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    server_names_hash_bucket_size 128;
    client_header_buffer_size 2k;
    large_client_header_buffers 4 4k;
    client_max_body_size 8m;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  60;
    open_file_cache max=204800 inactive=20s;
    open_file_cache_min_uses 1;
    open_file_cache_valid 30s;
    tcp_nodelay on;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;
    upstream  server_pools{
        server 10.10.10.11:8080;
        server 10.10.10.10:8080;
    }
    upstream  manager_pools{
        server 10.10.10.50:8080;
    }
    server {
        listen       80;
        server_name  106.3.149.18  106.3.149.19;		
		location /eidservice/ {
		 proxy_pass http://101.230.3.115:8033$document_uri;
          proxy_set_header Host $http_host;
          proxy_set_header X-Forwarded-Host $http_host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		}

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }  
    server {
        listen       8080;
        server_name  106.3.149.18  106.3.149.19;
        location / {
            proxy_pass http://manager_pools;
            proxy_redirect default;
        }
     
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

