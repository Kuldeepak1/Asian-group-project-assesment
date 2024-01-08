Install all necessary packages and compiler:
```
apt install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev
wget http://nginx.org/download/nginx-1.23.1.tar.gz
tar xfz nginx-1.23.1.tar.gz
cd nginx-1.23.1
```

add the nginx system user (with /etc/nginx/ as his home directory and with no valid shell).
```
useradd -d /etc/nginx/ -s /sbin/nologin nginx
```
Now start to compile Nginx by issuing the following command

```
./configure --user=nginx --group=nginx --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --with-http_ssl_module --with-pcre
```

Now Build Nginx from source file.

```
make
make install
```
Create web document root path as below
```
mkdir -p /var/www/html
/usr/sbin/nginx
```

Configure Nginx File.
```
vi /etc/nginx/nginx.conf
```
Replace ```root html; ``` with ```root /var/www/html;``` in the ```nginx.conf``` so update file will look like as below

```
user nginx;
location / {
                root /var/www/html;
                autoindex on;
                index index.html index.htm;
```

 check if Nginx is running using netstate command to verify TCP connection

 ```
netstat -tulpn | grep nginx
```
You will get output as below:
```
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      6119/nginx: master  
```
Create ```/var/www/html/index.html``` file with below content:

```
<!DOCTYPE html>
<html>
<body>

<h1>We are testing Nginx Load balancing...</h1>
<h2>My name is Kuldeepak</h2>
<p>Nginx test successful...</p>

</body>
</html>
```

Restart the Nginx:
```
/usr/sbin/nginx -s reload
```

Add host entry in ```/etc/hosts``` as below
```
127.0.0.1 backend1.kuldeepak.com
127.0.0.1 backend2.kuldeepak.com
```

Configure Load Balancing by updating ```vi /etc/nginx/nginx.conf``` as below
```
http {
    upstream backend {
        server backend1.kuldeepak.com;
        server backend2.kuldeepak.com;
        # Add more backend servers as needed
    }

    server {
        listen 80;
        server_name kuldeepak.com;

        location / {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }
}
```

Restart the Nginx
```
/usr/sbin/nginx -s reload
```
Test the loadbalancing with curl
```
curl backend1.kuldeepak.com
# Or
curl backend2.kuldeepak.com
```
You will get same output as below:
```
<!DOCTYPE html>
<html>
<body>

<h1>We are testing Nginx Load balencing...</h1>
<h2>My name is Kuldeepak</h2>
<p>Nginx test successful...</p>

</body>
</html>
```

Open Nginx on Firewalld
```
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
systemctl restart firewalld
```
