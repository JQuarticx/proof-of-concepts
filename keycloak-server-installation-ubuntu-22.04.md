keycloak-server-installation-ubuntu-22.04.md
---

you need to install the dependencies for keycloak service

```console
sudo apt-get update
sudo apt install curl wget vim -y
sudo apt-get install default-jdk -y
sudo apt install openjdk-21-jre -y
java -version
```

Download the keycloak package from the official site

```console
sudo wget -O keycloak.tar.gz https://github.com/keycloak/keycloak/releases/download/26.0.5/keycloak-26.0.5.tar.gz
sudo mkdir -p /opt/keycloak
cd /opt
sudo tar -xzvf keycloak.tar.gz keycloak --strip-components=1
sudo rm keycloak.tar.gz
```

Prepare the keycloak configuration file

```console
# sudo vim keycloak.conf
# Basic settings for running in production. Change accordingly before deploying the server.

# Database
# The database vendor.
db=postgres
db-username=keycloak
db-password=Keycloak@!123456788910
db-url=jdbc:postgresql://localhost/keycloak

# Observability: If the server should expose healthcheck endpoints and metrics endpoints.
health-enabled=true
metrics-enabled=true

# HTTP
http-enabled=true
http-host=0.0.0.0
http-port=8080

# HTTPS
# The file path to a server certificate or certificate chain in PEM format.
# https-certificate-file=${kc.home.dir}conf/server.crt.pem
# The file path to a private key in PEM format.
# https-certificate-key-file=${kc.home.dir}conf/server.key.pem

# HOSTNAME INFO
hostname-strict=true
hostname-backchannel-dynamic=true
hostname=https://sso.radianterp.in
hostname-admin=https://admin-sso.radianterp.in

# PROXY: The proxy address forwarding mode if the server is behind a reverse proxy.
# proxy=reencrypt
proxy-headers=xforwarded

# LOGGING
log=file
log-file=/var/log/keycloak/events.log
spi-events-listener-jboss-logging-success-level=info
spi-events-listener-jboss-logging-error-level=info
# Do not attach route to cookies and rely on the session affinity capabilities from reverse proxy
# spi-sticky-session-encoder-infinispan-should-attach-route=false
```

Run keycloak as non root user

```console
sudo groupadd keycloak
sudo useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
sudo cp /opt/keycloak/conf/keycloak.conf,/opt/keycloak/conf/keycloak.conf.backup
sudo cp /opt/keycloak/conf/cache-ispn.xml /opt/keycloak/conf/cache-ispn.xml.backup
sudo mv keycloak.conf /opt/keycloak/conf/keycloak.conf
sudo chown -R keycloak:keycloak /opt/keycloak
sudo chmod 774 /opt/keycloak/bin
sudo mkdir -p /var/log/keycloak
sudo chown -R keycloak:keycloak /var/log/keycloak
```

Create the temp admin username and password to access the keycloak web console.

```console
/opt/keycloak/bin/kc.sh  bootstrap-admin user
# Enter username [temp-admin]: xxx
# Enter password [temp-admin]: xxx
```

Run keycloak as systemd service

```console
# sudo vim /etc/systemd/system/keycloak.service
[Unit]
Description=The Keycloak Server
After=syslog.target network.target
# Before=nginx.service

[Service]
User=keycloak
Group=keycloak
LimitNOFILE=102642
PIDFile=/run/keycloak/keycloak.pid
ExecStart=/opt/keycloak/bin/kc.sh start --optimized
StandardOutput=file:/var/log/keycloak/standard.log
StandardError=file:/var/log/keycloak/error.log

[Install]
WantedBy=multi-user.target
```

```console
sudo systemctl daemon-reload
sudo systemctl start keycloak
sudo systemctl status keycloak
```
