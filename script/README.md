# Automatization script for nginx services
## Task conditions
Create single script (startup script) for service creation and activation

### Commands for start
* create multiple_nginx_service.sh or download file from git project
```
sudo touch multiple_nginx_service.sh
```
and fill it out
```
#!/bin/bash

# Set the desired number of services, projects and ports
num_nginxes=2
num_ports=2
num_proj=2

# Set up paths
nginx_dir="/etc/nginx/"
log_dir="/var/log/nginx"
www_dir="/var/www/projects"
systemd_dir="/etc/systemd/system"

# Generate massives for services and ports
nginxes=()
ports=()
projects=()

# Populate the arrays with services and port names
for ((i=1; i<=num_nginxes; i++)); do
    nginxes+=("nginx$i")
done

for ((i=1; i<=num_ports; i++)); do
    ports+=("808$i")
done

for ((i=1; i<=num_proj; i++)); do
    projects+=("project$i")
done

# Iterate over the services and ports
for i in "${!nginxes[@]}"; do
    nginx="${nginxes[$i]}"
    port="${ports[$i]}"
    project="${projects[$i]}"

    # Create directories for the service
    sudo mkdir -p /run/nginx
    sudo mkdir -p "${nginx_dir}/${nginx}"
    sudo mkdir -p "${log_dir}/${nginx}"
    sudo mkdir -p "${www_dir}/${project}"
    sudo chmod -R 755 "${nginx_dir}/${nginx}" "${log_dir}/${nginx}" "${www_dir}/${project}"

    # Clear and write custom values to nginx.conf for the service
    sudo tee "${nginx_dir}/${nginx}/${nginx}.conf" > /dev/null <<EOF
events {
# something else
}

http {

    access_log ${log_dir}/${nginx}/access.log;
    error_log ${log_dir}/${nginx}/error.log;

    server {
        listen ${port};
        
        root ${www_dir}/${project};
        
        index index.html;

        location / {
            try_files \$uri \$uri/ =404;
        }
    }
}
EOF

    # Create an HTML file with some text and the desired port
    echo "<html><body><h1>Welcome to ${nginx} on port ${port}</h1></body></html>" | sudo tee "${www_dir}/${project}/index.html" > /dev/null

    # Create a systemd service unit file for the service
    sudo tee "${systemd_dir}/${nginx}.service" > /dev/null <<EOF
[Unit]
Description=Nginx ${nginx}
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx/${nginx}.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/${nginx}/${nginx}.conf -g "pid /run/nginx/${nginx}.pid; error_log /var/log/nginx/${nginx}/error.log;"
ExecStart=/usr/sbin/nginx -c /etc/nginx/${nginx}/${nginx}.conf -g "pid /run/nginx/${nginx}.pid; error_log /var/log/nginx/${nginx}/error.log;"
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s stop
PrivateTmp=true
ExecStartPre=/bin/mkdir -p /run/nginx
ExecStartPre=/bin/chown -R nginx:nginx /run/nginx/
ExecStartPre=/bin/mkdir -p /var/log/nginx/${nginx}
ExecStartPre=/bin/mkdir -p /tmp/nginx/${nginx}
ExecStartPre=/bin/chown -R nginx:nginx /var/log/nginx/${nginx}
ExecStartPre=/bin/chown -R nginx:nginx /tmp/nginx/${nginx}
ExecStartPre=/bin/chown -R nginx:nginx /run/nginx

[Install]
WantedBy=multi-user.target
EOF
    sudo chown -R nginx:nginx /run/nginx
    sudo chmod 755 /run/nginx
    
    # reload daemon and nginx
    sudo systemctl daemon-reload
    sudo systemctl restart nginx

    # Enable and start the instance's systemd service
    sudo systemctl enable "${nginx}.service"
    sudo systemctl start "${nginx}.service"
done
```
* give rigths for file and run
```
sudo chown +755 multiple_nginx_service.sh
sudo ./multiple_nginx_service.sh
```
