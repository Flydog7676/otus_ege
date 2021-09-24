# Установка Elastiksearch
yum -y install nano wget java-1.8.0
# java -version
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
nano /etc/yum.repos.d/elasticsearch.repo
```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```
yum install --enablerepo=elasticsearch elasticsearch
nano /etc/elasticsearch/elasticsearch.yml
# там приводим к виду
```
bootstrap.memory_lock: true
network.host: localhost
http.port: 9200
```
nano /etc/sysconfig/elasticsearch
# приводим к виду MAX_LOCKED_MEMORY=unlimited
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
# Установка Kibana
nano /etc/yum.repos.d/kibana.repo
```
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
yum -y install kibana
nano /etc/kibana/kibana.yml
# привести к виду
```
server.port: 5601
server.host: "10.30.0.70"
elasticsearch.hosts: "http://localhost:9200"
```
systemctl enable kibana
systemctl start kibana
# установка 
yum install logstash
nano /etc/logstash/conf.d/logstash.conf
```
input {
beats {
port => 5044
}
}
output {
elasticsearch {
hosts => [ "10.30.0.67:9200" ]
        manage_template => false
        index => "nginx-%{+YYYY.MM.dd}"
    }
}
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

```
systemctl start logstash
# ss -tunlp | grep 5044
# Ставим клиента
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
nano /etc/yum.repos.d/elasticsearch.repo
```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```
yum install -y --enablerepo=elasticsearch filebeat
nano /etc/filebeat/filebeat.yml
# раскоментируем Elasticsearch output
filebeat modules enable system
filebeat setup
systemctl start filebeat
systemctl enable filebeat
 
