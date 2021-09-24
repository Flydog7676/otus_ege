# установка Elasticsearch
    yum -y install nano wget java-1.8.0
 # java -version
    rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
    cp elasticsearch.repo /etc/yum.repos.d/
    yum install -y --enablerepo=elasticsearch elasticsearch
    nano /etc/elasticsearch/elasticsearch.yml
 # приводим к виду
    bootstrap.memory_lock: true
    network.host: localhost
    http.port: 9200
 #
    nano /etc/sysconfig/elasticsearch
 # приводим к виду MAX_LOCKED_MEMORY=unlimited
    systemctl daemon-reload
    systemctl enable elasticsearch
    systemctl start elasticsearch
# Установка Kibana
    yum install -y --enablerepo=elasticsearch kibana
    nano /etc/kibana/kibana.yml
 # привести к виду
    server.port: 5601
    server.host: "10.30.0.70"
    elasticsearch.hosts: "http://localhost:9200"
 #
    systemctl enable kibana
    systemctl start kibana
# установка logstash
    yum install -y --enablerepo=elasticsearch logstash
    cp logstash.conf /etc/logstash/conf.d/logstash.conf
    systemctl enable logstash
    systemctl start logstash
# установка filebeat
    yum install -y --enablerepo=elasticsearch filebeat
    nano /etc/filebeat/filebeat.yml
 # раскоментируем Elasticsearch output
    filebeat modules enable system
    filebeat setup
    systemctl start filebeat
    systemctl enable filebeat
    

    
