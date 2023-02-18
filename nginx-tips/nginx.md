# NGINX
### nginx proxy_pass to localhost not working
Comment out a line in nginx.conf file:
````
# include /etc/nginx/conf.d/*.conf
````
#### Ways to enable round robin on localhost
1. Each server on different port:
````
http {
  upstream app {
    server localhost:3000;
    server localhost:3001;
    server localhost:3002;
     }
  server {
    listen 80;
    server_name localhost;
      location / {
        proxy_pass  http://app;
      }

}
````
2. Multiple docker containers running on the same port
````
http {
  server {
    listen 80;
    location / {
      proxy_pass  http://app:3000;
    }
  }
}
````
### Nginx config reload without downtime
This will send HUP signal:
````
/usr/sbin/nginx -s reload
````
