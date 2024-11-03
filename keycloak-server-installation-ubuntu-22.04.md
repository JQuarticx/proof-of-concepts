keycloak-server-installation-ubuntu-22.04.md
---

you need to install the dependencies for keycloak service

```console
sudo apt-get update
sudo apt install curl wget vim -y
sudo apt-get install default-jdk -y
sudo apt install openjdk-21-jre -y
# make sure jave is installed
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

Run keycloak as non root user

```console
sudo groupadd keycloak
sudo useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
sudo chown -R keycloak:keycloak /opt/keycloak
sudo chmod 774 /opt/keycloak/bin
```

Run keycloak as systemd service


```console
# :vim /etc/systemd/system/keycloak.service
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
StandardOutput=null

[Install]
WantedBy=multi-user.target
```

```console
systemctl daemon-reload
systemctl start keycloak
systemctl status keycloak
```
