- в контейнере уже установлен openssh-server и выключен SeLinux
- выходим на хостовую машину и подключаемся по sftp к mysql-master ``` sftp root@10.30.0.66 ``` копируем файл ```put mysql.run```
- выполняем скрипт ``` ./mysql.run```
```
#!/bin/bash
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
yum -y install mysql80-community-release-el7-3.noarch.rpm
yum -y install mysql-server
systemctl start mysqld.service
systemctl enable mysqld.service


yum -y install  wget nano	#устанавливаем доп программы
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm # скачивание пакета настроек репозитария
yum -y install mysql80-community-release-el7-3.noarch.rpm # установка пакета настроек репозитария
yum -y install mysql-server	# установка mysql сервера
systemctl start mysqld	# запуск mysql сервера
systemctl enable mysqld	# автозапуск демона mysql
```
- смотрим временный пароль ```grep password /var/log/mysqld.log```
- запускаем скрипт начальной настройки ```mysql_secure_installation``` и меняем пароль на Otus23@!
- проделываем это и на mysql-slave 10.30.0.65

На mysql-master
- в файл конфигурации /etc/my.cnf в конец добавляем:
```
bind-address           = 10.30.0.66
server-id              = 1
log_bin                = mysql-bin
```
- перезагружаем mysql ```systemctl restart mysqld```
- заходим в Mysql под root ```mysql -uroot -p``` и вносим нового пользователя:
```
CREATE USER repl@10.30.0.65 IDENTIFIED WITH caching_sha2_password BY 'Otus23@!';
GRANT REPLICATION SLAVE ON *.* TO repl@10.30.0.65;
```
- проверяем файл бинлога и текущую позицию записи ```SHOW MASTER STATUS;```
На mysql-slave
- в файл конфигурации /etc/my.cnf в конец добавляем:
```
server-id              = 2
log_bin                = mysql-bin
```
- перезагружаем mysql ```systemctl restart mysqld```
- заходим в Mysql под root ```mysql -uroot -p``` и подключаемся к master репликации:
```
STOP SLAVE;
CHANGE MASTER TO MASTER_HOST='10.30.0.66', MASTER_USER='repl', MASTER_PASSWORD='Otus23@!', MASTER_LOG_FILE='mysql-log.00001', MASTER_LOG_POS=1250, GET_MASTER_PUBLIC_KEY = 1;
START SLAVE;
```
- на mysql-master создаем БД ```CREATE DATABASE replicatest;```
- на mysql-slave проверяем БД ```SHOW DATABASES;```

