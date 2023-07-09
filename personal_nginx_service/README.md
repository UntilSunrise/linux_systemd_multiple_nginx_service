# HOW TO SET UP AND RUN OWN NGINX SERVICE
## Task conditions
Copy and Modify nginx services to run multiple nginx on multiple ports with separate log, tmp and config files.

### The first option
* disable SELinux
  ```
  sudo vi /etc/selinux/config
  SELINUX=enforcing -> SELINUX=disabled
  sudo reboot
  sestatus
  ```
* install nginx
  ```
  sudo yum install nginx
  ```
* create folder at /run -> nginx
  ```
  sudo mkdir /run/nginx
  ```
  Need for autocreate .pid file
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
  ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx3/nginx3.conf -g "pid /run/nginx/nginx3.pid; error_log /var/log/nginx/nginx3/error.log;"
  ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx3/nginx3.conf -g "pid /run/nginx/nginx3.pid; error_log /var/log/nginx/nginx3/error.log;"
  ExecReload=/usr/sbin/nginx -s reload
  ExecStop=/usr/sbin/nginx -s stop
  PrivateTmp=true
  ExecStartPre=/bin/mkdir -p /run/nginx
  ExecStartPre=/bin/chown -R nginx:nginx /run/nginx/
  ExecStartPre=/bin/mkdir -p /var/log/nginx/nginx3
  ExecStartPre=/bin/mkdir -p /tmp/nginx/nginx3
  ExecStartPre=/bin/chown -R nginx:nginx /var/log/nginx/nginx3
  ExecStartPre=/bin/chown -R nginx:nginx /tmp/nginx/nginx3
  ExecStartPre=/bin/chown -R nginx:nginx /run/nginx
  
  [Install]
  WantedBy=multi-user.target
  ```
* copy and modify nginx conf file:
```
events {
# something else
}

http {

    access_log /var/log/nginx/nginx3/access.log;
    error_log /var/log/nginx/nginx3/error.log;

    server {
        listen 8083;
        
        root /var/www/projects/project3;
        
        index index.html;

        location / {
            try_files $uri $uri/ =404;
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
* DO IT TWICE
  Please don't forget change ports and number of service
