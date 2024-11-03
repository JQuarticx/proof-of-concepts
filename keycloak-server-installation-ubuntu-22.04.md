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
wget -O keycloak.tar.gz https://github.com/keycloak/keycloak/releases/download/26.0.5/keycloak-26.0.5.tar.gz
mkdir -p /opt/keycloak
tar -xzvf keycloak.tar.gz keycloak --strip-components=1
rm keycloak.tar.gz
```

Run keycloak as non root user

```console
groupadd keycloak
useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
chown -R keycloak:keycloak /opt/keycloak
chmod 774 /opt/keycloak/bin
```
