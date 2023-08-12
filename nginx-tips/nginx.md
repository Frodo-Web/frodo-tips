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
## Guide
````
/var/www/html              // Default Server Block
/var/www/your_domain/html
chown -R $USER:$USER /var/www/your_domain/html
chmod -R 755 /var/www/your_domain
vim /var/www/your_domain/html/index.html

sudo vim /etc/nginx/sites-available/your-domain
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}

sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/

sudo nano /etc/nginx/nginx.conf
uncomment
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...

test to check for errors
sudo nginx -t
restart to enable your changes
sudo systemctl restart nginx

Server Configuration
/etc/nginx: The Nginx configuration directory. All of the Nginx configuration files reside here.
/etc/nginx/nginx.conf: The main Nginx configuration file. This can be modified to make changes to the Nginx global configuration.
/etc/nginx/sites-available/: The directory where per-site server blocks can be stored. Nginx will not use the configuration files found in this directory unless they are linked to the sites-enabled directory. Typically, all server block configuration is done in this directory, and then enabled by linking to the other directory.
/etc/nginx/sites-enabled/: The directory where enabled per-site server blocks are stored. Typically, these are created by linking to configuration files found in the sites-available directory.
/etc/nginx/snippets: This directory contains configuration fragments that can be included elsewhere in the Nginx configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

Server Logs
/var/log/nginx/access.log: Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.
/var/log/nginx/error.log: Any Nginx errors will be recorded in this log.

nginx config file:
/etc/nginx/nginx.conf
Опции также называются директивами, они группированы в штуки называемы блоки или контексты
Точка с запятой обязательно.
То что ниже относится к главному, общему контексту
Другие директивы находятся в блоках events, http и также существуют внутри главного контекста
user nginx;

worker_processes 1;

error_log /var/log/nginx/error.log warn;

pid /var/run/nginx.pid;

events {

. . .

}

http {

. . .

}

http блок содержит директивы для обработки веб трафика, которые обычно называются универсальными. Это потому что они будут переданы в конфигурацию каждого вебсайта под nginx

http {

include /etc/nginx/mime.types;

default_type application/octet-stream;

log_format main '$remote_addr - $remote_user [$time_local] "$request" '

'$status $body_bytes_sent "$http_referer" '

'"$http_user_agent" "$http_x_forwarded_for"';

access_log /var/log/nginx/access.log main;

sendfile on;

#tcp_nopush on;

keepalive_timeout 65;

#gzip on;

include /etc/nginx/conf.d/*.conf;

}
Блок выше inlucde информирует nginx где находятся конфигурационные файлы индивидуально
для сайтов. Имя должно быть форматировано как /etc/nginx/conf.d/example.com.conf
Если его нужно вырубить надо example.com.conf.disabled
Ну видишь эта конфигурация если с nginxa устанавливать, у дебиана другие пути прописаны

Server configuration files feature one or more server blocks for a site
/etc/nginx/conf.d/example.com.conf:
server {

listen 80 default_server;

listen [::]:80 default_server;

server_name example.com www.example.com;

root /var/www/example.com;

index index.html;

try_files $uri /index.html;

}

Nginx Location Blocks
Помогают настроить путь в котором nginx отвечает на реквесты для ресурсов внутри сервера
Как директива server_name информирует nginx как поступить с реквестом на домен,
location directives применяют на реквесты определенные директории и файлы, e.g. example.com/blog/

/etc/nginx/sites-available/example.com
location / { }

location /images/ { }

location /blog/ { }

location /planet/ { }

location /planet/blog/ { }

req: example.com
returns: nginx находит server_name entry for example.com, затем location / директивы определяют что произойдёт с реквестом дальше

Nginx пофиг на порядок, он идёт по наиболее специфичным путям
req: example.com/planet/blog or /planet/blog/about
returns: возвращает /planet/blog потому что более специфик путь, несмотря на то что
/planet тоже совпадает

/etc/nginx/sites-available/example.com:
location ~ IndexPage\.php$ { }
location ~ ^/BlogPlanet(/|/index\.php)$ { }
С тильдой nginx начинает поиск по регуляркам, case-sensetive

For example, IndexPage.php would be a match with the first of the above examples, while indexpage.php wouldn’t.

In the second example, the regular expression ^/BlogPlanet(/|/index\.php)$ { } would match requests for /BlogPlanet/ and /BlogPlanet/index.php but not /BlogPlanet, /blogplanet/, or /blogplanet/index.php. NGINX utilizes Perl Compatible Regular Expressions (PCRE).

~* для case-insensitive
^~ как тока nginx находит строку, прекратить искать более специфичные и использовать эту
location = / { } - = exact match

Итого, sequence flow таков:
1. = полностью совпадающие идут первыми, nginx перестанет искать если нашёл
2. ^= идут следующие, если nginx не найдёт, то continue processing of location directives
3. ~, ~*   следующие
4. Если ни одна из предыдущих не найдена, то ищет наиболее специфичный путь

Как только nginx найдёт location директиву которая лучше подходит реквесту, респонсбудет зависеть ассоциированного с ней блока
location / {

root html;    <- указывает на /etc/nginx/html

index index.html index.html;

}

req: /blog/includes/style.css
returns: попытается найти этот файл в /etc/nginx/html/blog/includes/style.css
переменная index указывает что за файл отсылать если ни одного не найдёено
req: /
returns:  /etc/nginx/html/index.html
Если файлов несколько, nginx пройдёт по порядку и возьмёт первый существующий

nginx отправит 404 ивент если ничего не найдено

/etc/nginx/sites-available/example.com:

location / {

root /srv/www/example.com/public_html;

index index.html index.htm;

}

location ~ \.pl$ {

gzip off;

include /etc/nginx/fastcgi_params;

fastcgi_pass unix:/var/run/fcgiwrap.socket;

fastcgi_index index.pl;

fastcgi_param SCRIPT_FILENAME /srv/www/example.com/public_html$fastcgi_script_name;

}

Первым пытается найти этот путь .pl, если не находит пытается отдать index рута, если 
не получается то даёт 404
````
