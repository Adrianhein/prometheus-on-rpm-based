
### Prometheus:
### https://prometheus.io/download/#prometheus
    
    cd ~
    
    echo $(pwd)
    
    groupadd --system prometheus
    
    useradd -s /sbin/nologin --system -g prometheus prometheus
    
    mkdir /etc/prometheus /var/lib/prometheus
    
    wget https://github.com/prometheus/prometheus/releases/download/v2.53.5/prometheus-2.53.5.linux-amd64.tar.gz
    
    tar -xvf prometheus-2.53.5.linux-amd64.tar.gz
    
    mv prometheus-2.53.5.linux-amd64 prometheus
    
    chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
    
    cp $(pwd)/prometheus/prometheus /usr/local/bin/
    
    cp $(pwd)/prometheus/promtool /usr/local/bin/
    
    chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
    
    cp -r $(pwd)/prometheus/consoles $(pwd)/prometheus/console_libraries /etc/prometheus/
    
    chown -R prometheus:prometheus /etc/prometheus/consoles /etc/prometheus/console_libraries
#    
    cat <<EOF > /etc/prometheus/prometheus.yml
    
    global:
      scrape_interval: 10s
    
    scrape_configs:
      - job_name: 'prometheus_master'
        scrape_interval: 5s
        static_configs:
          - targets: ['localhost:9091']
    
    EOF
#    
    #
    systemctl daemon-reload
    #
    
### port to listen at 9091 for prometheus.service
    cat <<EOF > /etc/systemd/system/prometheus.service
    
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml --web.listen-address=":9091" \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
    
    [Install]
    WantedBy=multi-user.target
    
    EOF
 #   
    firewall-cmd --add-port=9091/tcp --permanent
    
    firewall-cmd --reload
    
    semanage fcontext -a -t systemd_unit_file_t  /etc/systemd/system/prometheus.service
    
    systemctl daemon-reload
    systemctl enable --now prometheus
    systemctl start prometheus
    systemctl status prometheus
    
    ## http://<localhost>:9091
    ## http://<ip-address>:9091



