# установка mysql
    yum -y install nano wget
    wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
    yum install -y mysql80-community-release-el7-3.noarch.rpm
    yum install -y mysql-server
    systemctl enable mysql
    systemctl start mysqld 
    # смотрим временный пароль и делаем начальную настройку
    grep password /var/log/mysqld.log
    mysql_secure_installation
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
 # проверка работы репликации
    # на master делаем
    -> CREATE DATABASE replicatest;
    -> SHOW DATABASES;
# устанавливаем Apache
    yum -y install httpd
    systemctl enable httpd
# установка php 7.4
 # добавляем репозитории для php 7.4 
    yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum -y install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
    yum -y install yum-utils
    yum-config-manager --enable remi-php74
 # установка php 
    yum -y install php php-cli
    yum install -y php-fpm php-mysqlnd php-zip php-devel php-gd  php-intl php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json
    systemctl start httpd
 # проверка php и зайти через браузер на 10.30.0.75
    echo '<?php phpinfo(); ?>' > /var/www/html/info.php
# установка MEDIAWIKI
 # настраиваем mysql для wiki на mysql-master
    mysql -uroot -p
    -> CREATE DATABASE my_wiki;
    -> CREATE USER 'jake'@'localhost' IDENTIFIED BY 'Otus23@!';
    -> GRANT ALL PRIVILEGES ON my_wiki.* TO 'jake'@'localhost';
 # создание 2 папок для сайтов
    mkdir /var/www/html/8{1..2}
    wget https://releases.wikimedia.org/mediawiki/1.36/mediawiki-1.36.1.tar.gz
    tar xvzf mediawiki-1.36.1.tar.gz
    # копируем в папки 
    cp -r mediawiki-1.36.1/* /var/www/html/81/
    # установку и настройку 10.30.0.75.81
    cp -r /var/www/html/81/* /var/www/html/82/
    cp LocalSettings.php /var/www/html/81/
    cp LocalSettings1.php /var/www/html/82/LocalSettings.php
# настраиваем на apache 2 виртуальных сервера на портах 8081 и 8082
    cp server{1..2} conf /etc/httpd/conf.d/
    nano /etc/httpd/conf/httpd.conf
    # заменяем Listen80 на
    Listen 8081
    Listen 8082
    #
    systemctl restart httpd
# устанавливаем nginx и делаем upstream
    cp nginx.repo /etc/yum.repos.d/
    yum -y install nginx
    cp upstream.conf /etc/nginx/conf.d/
    nano /etc/nginx/nginx.conf
 # в секции server
       location / {
                proxy_pass http://test-upstream;
        }
 #
    nginx -t
    systemctl start nginx
    systemctl enable nginx
 # проверяем в браузере 10.30.0.75 ( выключаем /etc/httpd/conf/httpd.conf и проверяем )
# установка node_exporter для Prometheus
    useradd --no-create-home --shell /bin/false node_exporter
    wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
    tar xvzf node_exporter-1.2.2.linux-amd64.tar.gz
    cd node_exporter-1.2.2.linux-amd64
    cp node_exporter /usr/local/bin/
    chown node_exporter: /usr/local/bin/node_exporter
 # создаем модуль для автозагрузки systemd
    cp ~/nginx/node_exporter.service /etc/systemd/system/
    systemctl daemon-reload
    systemctl start node_exporter
    systemctl enable node_exporter

    

