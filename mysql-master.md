- в контейнере уже установлен openssh-server и выключен SeLinux
- выходим на хостовую машину и подключаемся по sftp ``` sftp root@10.30.0.69 ``` копируем файл ```put mysql-master.run```
- выполняем скрипт ``` ./mysql-master.run```
```
yum -y install  wget nano	#устанавливаем доп программы
wget http://repo.mysql.com/mysql-community-release-el7-7.noarch.rpm # скачивание пакета настроек репозитария
yum -y install mysql-community-release-el7-7.noarch.rpm # установка пакета настроек репозитария
yum -y install mysql-server	# установка mysql сервера
systemctl start mysqld.service	# запуск mysql сервера
systemctl enable mysqld.service	# автозапуск демона mysql
mysql_secure_installation<<EOF

y
12345678
12345678
y
y
y
y
EOF
```
- 
