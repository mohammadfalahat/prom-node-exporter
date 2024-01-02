# install Prometheus Node Exporter with Authentication on Ubuntu

1. Check for latest version here: https://github.com/prometheus/node_exporter/tags

2. Define your own username and password.
```
password='YOURPASSWORD'
username='admin'
node_exporter_url='https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz'
node_exporter_name='node_exporter-1.7.0.linux-amd64'
```

3. Get and extract node_exporter.
```
mkdir /etc/node_exporter
cd /etc/node_exporter
wget ${node_exporter_url}
tar xvf ${node_exporter_name}.tar.gz
sudo cp ${node_exporter_name}/node_exporter /usr/local/bin
```

4. Add user and credential information
```
sudo useradd --no-create-home --shell /bin/false node_exporter

apt install -y apache2-utils
passwordHashed=`echo ${password} | htpasswd -inBC 10 "" | tr -d ':\n'`
sudo cat << EOF > /etc/node_exporter/web.yml
basic_auth_users:
  ${username}: ${passwordHashed}
EOF
```

5. Add node exporter to services and start it!
```
sudo cat << EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/node_exporter --web.config.file=/etc/node_exporter/web.yml
Restart=always
RestartSec=3
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

netstat -tulpn | grep 9100
```
