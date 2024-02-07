### Descritpion
:v: Magento 2 STACK dockeriszed with Nginx self-signed ssl setting.

### Installation
Clone this repository then clone Magento 2 source code in **./src** folder.

For example:

    git clone -b 2.4.5-p1 git@github.com:magento/magento2.git src
#### Set the permissions to execute the index.php file
    chmod -R 777 .


#### Magento 2 installation (fresh install only)
    docker exec -it m2phpfpm bin/magento setup:install --base-url=http://localhost --db-host=m2percona --db-name=magento --db-user=magento --db-password=magento --admin-firstname=admin --admin-lastname=admin --admin-email=admin@admin.com --admin-user=admin --admin-password=admin123 --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1 --elasticsearch-host=m2elasticsearch --search-engine=elasticsearch7

### How to:
#### Access to Magento 2
https://localhost
#### Access to Adminer
http://localhost:8080
#### Start the stack
    docker compose up -d
    
#### Stop the stack
    docker compose down
    
#### Follow Nginx logs
    tail -f ./nginx/logs/*
    
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

