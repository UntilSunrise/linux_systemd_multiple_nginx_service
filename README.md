# linux_systemd_multiple_nginx_service
## Task conditions
Copy and Modify nginx services to run multiple nginx on multiple ports with separate log, tmp and config files.

* additional you can provide separated default virtualhost to illustrate which service run current process
* create single script (startup script) for service creation and activation

### The first option
* install nginx
  ```
  sudo yum install nginx
  ```
* copy and modify systemd nginx service
  ```
  sudo cp /etc/systemd/system/nginx.service /etc/systemd/system/nginx3.service
  ```
  and modify it:
  ```
  [Unit]
Description=Nginx nginx3
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx/nginx3.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx3/nginx3.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx3/nginx3.conf
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s stop
PrivateTmp=true
ExecStartPre=/bin/mkdir -p /var/log/nginx/nginx3
ExecStartPre=/bin/mkdir -p /tmp/nginx/nginx3
ExecStartPre=/bin/chown -R nginx:nginx /var/log/nginx/nginx3
ExecStartPre=/bin/chown -R nginx:nginx /tmp/nginx/nginx3

[Install]
WantedBy=multi-user.target
  ```
* copy and modify nginx conf file
```
events {
# something else
}

http {
    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://localhost:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        
            index index.html;
        }

        location /var/www/projects/project3/ {
            alias /var/www/projects/project3/;
            index index.html;
        }
    }
}

```
* restart/reload daemon and service
```
sudo systemctl daemon-reload
sudo systemctl enable nginx3
sudo systemctl start nginx3
```
