# NGINX
### nginx proxy_pass to localhost not working
Comment out a line in nginx.conf file:
````
# include /etc/nginx/conf.d/*.conf
````
### Nginx config reload without downtime
This will send HUP signal:
````
/usr/sbin/nginx -s reload
````
