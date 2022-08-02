source: [laelbe/my-image-cdn](https://github.com/laelbe/my-image-cdn)


### Nginx Conf
```conf
server {
    listen       80;
    server_name  example.com;
   
    return       301 https://$server_name$request_uri;
}
   
   
server {
    listen       443 ssl http2;
    server_name  example.com;
    root   /home/exampleuser/img2;
  
    server_tokens off;
  
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
  
    charset utf-8;
  
    #set same size as post_max_size(php.ini or php_admin_value).
    client_max_body_size 10M;
  
    ssl_certificate "/etc/letsencrypt/live/example.com/fullchain.pem";
    ssl_certificate_key "/etc/letsencrypt/live/example.com/privkey.pem";
    ssl_dhparam "/etc/ssl/certs/dhparam.pem";
       
    # Enable HSTS. This forces SSL on clients that respect it, most modern browsers. The includeSubDomains flag is optional.
    add_header Strict-Transport-Security "max-age=31536000";
       
    # Set caches, protocols, and accepted ciphers. This config will merit an A+ SSL Labs score.
    ssl_session_cache shared:SSL:20m;
    ssl_session_timeout 10m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5';
       
    access_log /var/log/nginx/example.com.access.log main;
    error_log  /var/log/nginx/example.com.error.log warn;
   
    location / {
        index  index.php index.html;
    }
   
    # Allow Lets Encrypt Domain Validation Program
    location ^~ /.well-known/acme-challenge/ {
        allow all;
    }
 
    # Image headers
    location ~* \.(png|jpg|jpeg|gif)$ {
        valid_referers none blocked blog.example.com *data.example-site.com ;
        if ($invalid_referer) {
            return   403;
        }
 
        try_files $uri /index.php?$args;
        add_header  my-ray  "KR1";
        add_header  my-cache-status  "HIT";
        add_header  cache-control  "public, max-age=2678400";
    }
 
    # Block dot file (.htaccess .htpasswd .svn .git .env and so on.)
    location ~ /\. {
        deny all;
    }
   
    # Block (log file, binary, certificate, shell script, sql dump file) access.
    location ~* \.(log|binary|pem|enc|crt|conf|cnf|sql|sh|yml|lock)$ {
        deny all;
    }
   
    # Block access
    location ~* (composer\.json|composer\.lock|composer\.phar|contributing\.md|license\.txt|readme\.rst|readme\.md|readme\.txt|copyright|artisan|gulpfile\.js|package\.json|phpunit\.xml|access_log|error_log|gruntfile\.js)$ {
        deny all;
    }
   
    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }
   
    location = /robots.txt {
        log_not_found off;
        access_log off;
    }
   
    # Block .php file inside upload folder. uploads(wp), files(drupal), data(gnuboard).
    location ~* /(?:uploads|default/files|data)/.*\.php$ {
        deny all;
    }
   
    # Add PHP handler
    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
   
        fastcgi_pass unix:/run/php/exampleuser.sock;
        fastcgi_index index.php;
        fastcgi_buffers 64 16k; # default 8 4k
  
        include fastcgi_params;
         
        add_header  my-ray  "KR1";
        add_header  cache-control  "public, max-age=2678400";
    }
}
```