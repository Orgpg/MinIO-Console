# PhaLel MinIO + PostgreSQL + Nginx Setup (Ubuntu 24.04 + Docker Compose)

This document adds **Nginx Reverse Proxy** and **Let's Encrypt SSL** to
your existing MinIO + PostgreSQL deployment.

## Install Nginx

``` bash
sudo apt update
sudo apt install nginx -y
```

## Remove Default Site

``` bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo rm -f /etc/nginx/sites-available/default
```

## Create Nginx Configuration

``` bash
sudo nano /etc/nginx/sites-available/minio
```

``` nginx
server {
    listen 80;
    server_name storage.waiphyoaung.dev;

    client_max_body_size 5G;

    location / {
        proxy_pass http://127.0.0.1:9000;

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Connection "";
        chunked_transfer_encoding off;
    }
}

server {
    listen 80;
    server_name console.waiphyoaung.dev;

    client_max_body_size 5G;

    location / {
        proxy_pass http://127.0.0.1:9001;

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable configuration:

``` bash
sudo ln -s /etc/nginx/sites-available/minio /etc/nginx/sites-enabled/minio
sudo nginx -t
sudo systemctl restart nginx
```

## Install SSL

``` bash
sudo apt install certbot python3-certbot-nginx -y

sudo certbot --nginx -d storage.waiphyoaung.dev -d console.waiphyoaung.dev
```

## Docker Compose Update

``` yaml
environment:
  MINIO_SERVER_URL: "https://storage.waiphyoaung.dev"
  MINIO_BROWSER_REDIRECT_URL: "https://console.waiphyoaung.dev"
```

## URLs

-   API: https://storage.waiphyoaung.dev
-   Console: https://console.waiphyoaung.dev
