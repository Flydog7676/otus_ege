- nginx сервер
# добавляем репозитории для php 7.4 
    yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum -y install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
    yum -y install yum-utils
    yum-config-manager --enable remi-php74
# устанавливаем Apache
    yum -y install httpd
    systemctl start httpd
    systemctl enable httpd
    # установка mysql
    wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
    yum install -y mysql80-community-release-el7-3.noarch.rpm
    yum install -y mysql-server
    systemctl start mysqld
    systemctl enable mysql
    # смотрим временный пароль и делаем начальную настройку
    grep password /var/log/mysqld.log
    mysql_secure_installation
    # установка php 7.4
    yum -y install php php-cli
    yum install php-fpm php-mysqlnd php-zip php-devel php-gd  php-intl php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json
    systemctl restart httpd
    # проверка php
    echo '<?php phpinfo(); ?>' > /var/www/html/info.php
# репликация mysql
    #в файл конфигурации /etc/my.cnf в конец добавляем:
        bind-address           = 10.30.0.75
        server-id              = 1
        log_bin                = mysql-bin
    systemctl restart mysqld
    mysql -uroot -p
    -> CREATE USER repl@10.30.0.71 IDENTIFIED WITH caching_sha2_password BY 'Otus23@!';
    -> GRANT REPLICATION SLAVE ON *.* TO repl@10.30.0.71;
    -> SHOW MASTER STATUS;
# на mysql-slave установить Mysql
# установка mysql
    yum -y install wget nano
    wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
    yum install -y mysql80-community-release-el7-3.noarch.rpm
    yum install -y mysql-server
    systemctl start mysqld
    systemctl enable mysqld
    # смотрим временный пароль и делаем начальную настройку
    grep password /var/log/mysqld.log
    mysql_secure_installation
    #в файл конфигурации /etc/my.cnf в конец добавляем:
        server-id              = 2
        log_bin                = mysql-bin
    systemctl restart mysqld
    mysql -uroot -p
    -> STOP SLAVE;
    -> CHANGE MASTER TO MASTER_HOST='10.30.0.75', MASTER_USER='repl', MASTER_PASSWORD='Otus23@!', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=707, GET_MASTER_PUBLIC_KEY = 1;
    -> START SLAVE;
# проверка работы репликации
    # на master делаем
    -> CREATE DATABASE replicatest;
    -> SHOW DATABASES;
    # на slave проверяем
    -> SHOW DATABASES;
# установка MEDIAWIKI
    # настраиваем mysql для wiki на mysql-master
    mysql -uroot -p
    -> CREATE DATABASE my_wiki;
    -> CREATE USER 'jake'@'localhost' IDENTIFIED BY 'Otus23@!';
    -> GRANT ALL PRIVILEGES ON my_wiki.* TO 'jake'@'localhost';
    -> 
    # создание 2 папок для сайтов
    mkdir /var/www/html/8{1..2}
    wget https://releases.wikimedia.org/mediawiki/1.36/mediawiki-1.36.1.tar.gz
    tar xvzf mediawiki-1.36.1.tar.gz
    # распакованную папку в папки 
    cp -r mediawiki-1.36.1/* /var/www/html/81/
    cp -r mediawiki-1.36.1/* /var/www/html/82/
    # заходим через броузер в */81 папку и производим настройку
    #после настройки копируем файл в паку
    sftp root@10.30.0.75
    -> put LocalSettings.php
    exit
    ssh root@10.30.0.75
    cp  LocalSettings.php /var/www/html/81/
    cp  LocalSettings.php /var/www/html/82/
    # меняем в LocalSettings.php имена wiki и путь расположения
    # запускаем в броузере 10.30.0.75/81 и 10.30.0.75/82
    nano /etc/httpd/conf/httpd.conf
    # заменяем Listen80 на
    Listen 8081
    Listen 8082
    # создаем файлы конфигурации
    nano /etc/httpd/conf.d/server1.conf
    # заполняем файл
    <VirtualHost *:8081>
   ServerAdmin alex@ulnanotech.com
   DocumentRoot /var/www/html/81
   </VirtualHost>
   # создаем файлы конфигурации
    nano /etc/httpd/conf.d/server2.conf
    # заполняем файл
    <VirtualHost *:8082>
   ServerAdmin alex@ulnanotech.com
   DocumentRoot /var/www/html/82
   </VirtualHost>
# Установка и настройка NGINX
    # настройка репозитария
    nano /etc/yum.repo.d/ngix.repo
```
    [nginx-stable]
    name=nginx stable repo
    baseurl=http://nginx.org/packages/centos/$releasever/$basearch
    gpgcheck=1
    enabled=1
    gpgkey=https://nginx.org/keys/nginx_signing.key
    module_hotfixes=true
```
    yum -y install nginx
    # настройка серверов upstream
    nano  /etc/nginx/conf.d/upstream.conf
```
    upstream test-upstream {
    server 10.30.0.75:8081;
    server 10.30.0.75:8082;
    }
```
    nano /etc/nginx/nginx.conf
    # добавляем секцию location в server
```
       location / {
                proxy_pass http://test-upstream;
        }

```
    # проверяем конфигурацию nginx
    nginx -t
    # запускаем nginx
    systemctl start nginx
    systemctl enable nginx


# Установка мониторинга
# установка node_exporter на 10.30.0.75
    useradd --no-create-home --shell /bin/false node_exporter
    wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
    tar xvzf node_exporter-1.2.2.linux-amd64.tar.gz
    cd node_exporter-1.2.2.linux-amd64
    cp node_exporter /usr/local/bin/
    chown node_exporter: /usr/local/bin/node_exporter
    nano /etc/systemd/system/node_exporter.service
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter
# Установка Prometheus
    yum -y nano wget
    useradd --no-create-home --shell /usr/sbin/nologin prometheus
    mkdir /etc/prometheus /var/lib/prometheus
    chown -Rv prometheus: /etc/prometheus/ /var/lib/prometheus/
    wget https://dl.grafana.com/oss/release/grafana-8.1.1-1.x86_64.rpm
    wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
    tar xvzf prometheus-2.29.1.linux-amd64.tar.gz
    cd prometheus-2.29.1.linux-amd64
    cp -r console_libraries consoles prometheus.yml /etc/prometheus
    cp prometheus promtool /usr/local/bin/
    chown -Rv prometheus: /etc/prometheus 
    chown prometheus: /usr/local/bin/{prometheus,promtool}
    nano /etc/systemd/system/prometheus.service
```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```
    nano /etc/prometheus/prometheus.yml
```
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```
# Установка grafana
    yum -y install grafana-8.1.1-1.x86_64.rpm
    systemctl start grafana-server
    systemctl enable grafana-server
    # http://10.30.0.67:3000    #admin admin
    # add Data Sources -> http://localhost:9090
    

