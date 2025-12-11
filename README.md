### Description
:v: Magento 2 STACK dockerized with Nginx self-signed SSL setting.

### Installation
Clone this repository then clone Magento 2 source code in **./src** folder.

For example:

    git clone -b 2.4.5-p1 git@github.com:magento/magento2.git src

#### Magento 2 installation (fresh install only)

1. **Start the Docker stack:**
```bash
docker compose up -d
```

2. **Install Composer dependencies inside the phpfpm container:**
```bash
docker compose exec m2phpfpm composer install
```

3. **Install Magento 2:**
```bash
docker compose exec m2phpfpm bin/magento setup:install \
  --base-url=http://localhost:8000 \
  --db-host=m2mysql \
  --db-name=magento \
  --db-user=magento \
  --db-password=magento \
  --admin-firstname=admin \
  --admin-lastname=admin \
  --admin-email=admin@admin.com \
  --admin-user=admin \
  --admin-password=admin123 \
  --language=en_US \
  --currency=USD \
  --timezone=America/Chicago \
  --use-rewrites=1 \
  --search-engine=opensearch \
  --opensearch-host=m2opensearch
```

**Note:** Replace `http://localhost:8000` with your domain/port. OpenSearch is configured instead of Elasticsearch (recommended for Magento 2.4.8+).

#### 4. Permissions are automatically handled by Docker

Permissions are now optimized in `docker-compose.yml`:
- Containers run with user ID `1000:1000` (your host user)
- Volumes use the `:Z` flag for M2/SELinux compatibility
- Default UMASK is `0022` for proper file permissions

**No manual chmod needed!** Magento automatically gets the correct write permissions on `/var/www/html/var`, `/var/www/html/pub`, and `/var/www/html/generated`.

If you encounter permission issues, verify Docker is running and restart the stack:
```bash
docker compose down
docker compose up -d
```

---

### Host Nginx Configuration (Linux only)

If you're running Docker on a Linux machine and want to access Magento via a domain name instead of `localhost:8000`, configure your host's Nginx as a reverse proxy.

#### 1. Generate self-signed SSL certificate
### Linux
```bash
sudo mkdir -p /etc/nginx/ssl

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/magento.local.key \
  -out /etc/nginx/ssl/magento.local.crt \
  -subj "/C=FR/ST=State/L=City/O=Organization/CN=magento.local"

sudo chmod 600 /etc/nginx/ssl/magento.local.key
sudo chmod 644 /etc/nginx/ssl/magento.local.crt
```
### Mac OS
```bash
sudo mkdir -p /opt/homebrew/etc/nginx/ssl

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /opt/homebrew/etc/nginx/ssl/magento.local.key \
  -out /opt/homebrew/etc/nginx/ssl/magento.local.crt \
  -subj "/C=FR/ST=State/L=City/O=Organization/CN=magento.local"

sudo chmod 600 /opt/homebrew/etc/nginx/ssl/magento.local.key
sudo chmod 644 /opt/homebrew/etc/nginx/ssl/magento.local.crt
```

#### 2. Create Nginx reverse proxy configuration
### Linux
Create `/etc/nginx/sites-available/magento.local`:
```nginx
upstream magento_backend {
    server localhost:8000;
    keepalive 256;
}

server {
    listen 80;
    listen [::]:80;
    server_name magento.local www.magento.local;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name magento.local www.magento.local;

    ssl_certificate /etc/nginx/ssl/magento.local.crt;
    ssl_certificate_key /etc/nginx/ssl/magento.local.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    access_log /var/log/nginx/magento.local_access.log;
    error_log /var/log/nginx/magento.local_error.log;

    location / {
        proxy_pass http://magento_backend;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header Connection "";

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;
    }

    location ~ /\.env {
        deny all;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
### Mac OS
Create `/opt/homebrew/etc/nginx/sites-available/magento.local`:
```nginx
upstream magento_backend {
    server localhost:8000;
    keepalive 256;
}

server {
    listen 80;
    listen [::]:80;
    server_name magento.local www.magento.local;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name magento.local www.magento.local;

    ssl_certificate /opt/homebrew/etc/nginx/ssl/magento.local.crt;
    ssl_certificate_key /opt/homebrew/etc/nginx/ssl/magento.local.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    access_log /opt/homebrew/var/log/nginx/magento.local_access.log;
    error_log /opt/homebrew/var/log/nginx/magento.local_error.log;

    location / {
        proxy_pass http://magento_backend;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header Connection "";

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;
    }

    location ~ /\.env {
        deny all;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

#### 3. Enable the configuration
### Linux
```bash
sudo ln -s /etc/nginx/sites-available/magento.local /etc/nginx/sites-enabled/

sudo nginx -t

sudo systemctl reload nginx
```

#### 4. Add domain to /etc/hosts
```bash
echo "127.0.0.1 magento.local www.magento.local" | sudo tee -a /etc/hosts
```

#### 5. Access Magento
- **HTTPS:** https://magento.local
- **HTTP:** http://magento.local (auto-redirects to HTTPS)

**Note:** The SSL certificate is self-signed. Your browser will show a warning. This is normal for local development. You can ignore it or add an exception.

---

### How to:
#### Access to Magento 2 (Docker port)
https://localhost:8443 (or http://localhost:8000)

#### Access to Magento 2 (via Nginx reverse proxy)
https://magento.local

#### Access to Adminer
http://localhost:8080

#### Start the stack
```bash
docker compose up -d
```

#### Stop the stack
```bash
docker compose down
```

#### Follow Docker Nginx container logs
```bash
docker compose logs -f m2nginx
```

#### Follow Host Nginx logs
```bash
tail -f /var/log/nginx/magento.local_access.log
tail -f /var/log/nginx/magento.local_error.log
```
    
### Tree
    .
    ├── docker-compose.yml
    ├── mysql #folder mounted in m2mysql container /var/lib/mysql
    ├── nginx 
        ├── default.conf #nginx setting
        ├── logs #folder mounted in m2nginx container /var/log/nginx. Easy to follow nginx logs
        ├── Dockerfile #builder for custom image - ssl setting
    ├── phpfpm
        ├── Dockerfile #builder for custom image - php extensions and more
    ├── README.md
    └── src #folder which contents Magento 2 source code

