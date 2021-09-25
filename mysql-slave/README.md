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
        server-id              = 2
        log_bin                = mysql-bin
    systemctl restart mysqld
    mysql -uroot -p
    -> STOP SLAVE;
    -> CHANGE MASTER TO MASTER_HOST='10.30.0.75', MASTER_USER='repl', MASTER_PASSWORD='Otus23@!', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=707, GET_MASTER_PUBLIC_KEY = 1;
    -> START SLAVE;
# на slave проверяем появление новой базы
    -> SHOW DATABASES;

# backups базы данных и восстановление
    mysqldump -u root -p -all-databases --no-create-info > dump-data.sql
 #
    mysqldump -u root -p -all-databases --no-create-info > dump-data.sql
    mysqld  database_name < file.sql
    mysql --one-database database_name < all_databases.sql
