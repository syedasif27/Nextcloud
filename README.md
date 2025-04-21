# Nextcloud Setup with Apache, PHP-FPM, Redis, and MariaDB Galera Cluster

## Objective:
This guide outlines the steps to configure a **Nextcloud** setup with **Apache** and **PHP-FPM** for the web server, **MariaDB Galera Cluster** for the database, and **Redis** for caching and session management.

---

## Prerequisites:
- 1 Web Server with Apache and PHP-FPM
- 2 Database Servers with MariaDB Galera Cluster
- 1 Redis Server for caching and sessions
- Nextcloud installed on the Web Server

---

## Step 1: Setup Web Server with Apache and PHP-FPM

### 1.1. Install Apache and PHP-FPM
Install Apache and PHP-FPM on the Web Server:
```bash
sudo apt update
sudo apt install apache2 php-fpm libapache2-mod-fcgid
```

### 1.2. Install PHP Modules for Nextcloud
```bash
sudo apt install php-cli php-common php-curl php-gd php-mbstring php-xml php-zip php-mysql php-bz2 php-intl php-imagick
```

### 1.3. Configure Apache for PHP-FPM
Enable necessary Apache modules:
```bash
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php7.4-fpm  # Replace with your PHP version
```

Create or modify the Apache Virtual Host configuration for Nextcloud:
```bash
sudo nano /etc/apache2/sites-available/nextcloud.conf
```

Example configuration:
```apache
<VirtualHost *:80>
    ServerName nextcloud.example.com
    DocumentRoot /var/www/nextcloud

    # Enable PHP-FPM for Nextcloud
    <IfModule mod_php7.c>
        FcgidMaxRequestsPerProcess 1000
        FcgidMinProcessesPerServe 1
        FcgidProcessLifeTime 3600
        FcgidConnectTimeout 10
        FcgidIOTimeout 120
        FcgidBusyTimeout 120
    </IfModule>

    # FastCGI
    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/nextcloud/$1
</VirtualHost>
```

Enable the site:
```bash
sudo a2ensite nextcloud.conf
sudo systemctl reload apache2
```

### 1.4. Configure PHP for Nextcloud
Edit the `php.ini` file for PHP-FPM:
```bash
sudo nano /etc/php/7.4/fpm/php.ini  # Replace with your PHP version
```

Set these values:
```ini
upload_max_filesize = 512M
post_max_size = 512M
memory_limit = 512M
max_execution_time = 3600
max_input_time = 3600
```

Restart PHP-FPM:
```bash
sudo systemctl restart php7.4-fpm  # Replace with your PHP version
```

---

## Step 2: Setup Database Servers with MariaDB Galera Cluster

### 2.1. Install MariaDB on Database Servers
On each database server, install MariaDB:
```bash
sudo apt update
sudo apt install mariadb-server
```

### 2.2. Configure Galera Cluster on All Nodes

Edit `/etc/mysql/my.cnf`:
```bash
sudo nano /etc/mysql/my.cnf
```

Add the following configuration:
```ini
[mysqld]
wsrep_on=ON
wsrep_cluster_address="gcomm://DB1_IP,DB2_IP"
wsrep_cluster_name="nextcloud_cluster"
wsrep_node_address="DB1_IP"
wsrep_node_name="node1"  # Change for each node
wsrep_sst_method=rsync
```

Start the cluster on the first node:
```bash
sudo systemctl start mariadb
```

For the second node, start it as a joiner:
```bash
sudo systemctl start mariadb
```

---

## Step 3: Setup Redis Server for Caching and Session Management

### 3.1. Install Redis
On the Redis server, install Redis:
```bash
sudo apt update
sudo apt install redis-server
```

### 3.2. Configure Redis for Nextcloud

Edit `/etc/redis/redis.conf`:
```bash
sudo nano /etc/redis/redis.conf
```

Make the following changes:
```ini
bind 0.0.0.0  # Or specify your server's IP
requirepass your_secure_redis_password
maxmemory 4gb
maxmemory-policy allkeys-lru
```

Restart Redis:
```bash
sudo systemctl restart redis-server
```

### 3.3. Install Redis PHP Extension on Web Server
On the web server, install the Redis PHP extension:
```bash
sudo apt install php-redis
```

Restart PHP-FPM:
```bash
sudo systemctl restart php7.4-fpm  # Replace with your PHP version
```

### 3.4. Configure Nextcloud to Use Redis

Edit `/var/www/nextcloud/config/config.php`:
```bash
sudo nano /var/www/nextcloud/config/config.php
```

Add the following configuration:
```php
'memcache.local' => '\OC\Memcache\Redis',
'memcache.distributed' => '\OC\Memcache\Redis',
'redis' => array(
    'host' => 'redis_server_ip',
    'port' => 6379,
    'password' => 'your_secure_redis_password',
    'timeout' => 0.0,
),
'session.handler' => 'redis',
'session.redis' => array(
    'host' => 'redis_server_ip',
    'port' => 6379,
    'password' => 'your_secure_redis_password',
    'timeout' => 0.0,
),
```

---

## Step 4: Final Configurations

### 4.1. Test the Setup

1. **Test Database Connection**:
   ```bash
   sudo mysql -u nextcloud_user -p -h DB1_IP
   ```

2. **Test Redis**:
   ```bash
   redis-cli -h redis_server_ip -p 6379 -a your_secure_redis_password ping
   ```

   You should get:
   ```bash
   PONG
   ```

3. **Check Nextcloud Logs**:
   ```bash
   tail -f /var/www/nextcloud/data/nextcloud.log
   ```

### 4.2. Enable SSL (Optional)
For SSL, use **Certbot**:
```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d nextcloud.example.com
```

### 4.3. Restart Services

Finally, restart all necessary services:

1. **Restart Apache**:
   ```bash
   sudo systemctl restart apache2
   ```

2. **Restart PHP-FPM**:
   ```bash
   sudo systemctl restart php7.4-fpm  # Replace with your PHP version
   ```

3. **Restart Redis**:
   ```bash
   sudo systemctl restart redis-server
   ```

---

## Conclusion:
You now have a fully configured **Nextcloud** environment with a **MariaDB Galera Cluster** for high availability, **Redis** for caching and session management, and **Apache with PHP-FPM** on the web server. This setup ensures optimized performance, reliability, and scalability for your Nextcloud instance.

---
