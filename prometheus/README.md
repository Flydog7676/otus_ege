# установка Prometheus и Grafana
    yum -y install nano wget
    useradd --no-create-home --shell /usr/sbin/nologin prometheus
    mkdir /etc/prometheus /var/lib/prometheus
    chown -Rv prometheus: /etc/prometheus/ /var/lib/prometheus/
    wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
    tar xvzf prometheus-2.29.1.linux-amd64.tar.gz
    cd prometheus-2.29.1.linux-amd64
    cp -r console_libraries consoles prometheus.yml /etc/prometheus
    cp prometheus promtool /usr/local/bin/
    chown -Rv prometheus: /etc/prometheus 
    chown prometheus: /usr/local/bin/{prometheus,promtool}
 # создаем модуль запуска systemd
    cp prometheus.service /etc/systemd/system/
    nano /etc/prometheus/prometheus.yml
 # добавляем
      - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['10.30.0.75:9100']
 #
    systemctl daemon-reload
    systemctl start prometheus
    systemctl enable prometheus
# Установка grafana 
    wget https://dl.grafana.com/oss/release/grafana-8.1.1-1.x86_64.rpm
    yum -y install grafana-8.1.1-1.x86_64.rpm
    systemctl start grafana-server
    systemctl enable grafana-server
    # http://10.30.0.67:3000    #admin admin
    # add Data Sources -> http://localhost:9090 
    # add dashboard -> 11074

