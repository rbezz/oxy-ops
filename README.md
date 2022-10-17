# oxy-ops
Tesis Red Nacional de Honeypot OXYOPS
## Requisitos del sistema
La instalación pueder ser realizada en hardware real, en una máquina virtual o en la nube. 
Obs: Se recomienda desplegar el sensor aislado del entorno de producción.
### Requisitos mínimos
- RAM: 500MB
- Almacenamiento: 10GB
### Requisitos recomendados
- RAM: 2GB
- Almacenamiento: 30GB
## Configuración inicial
### Instalar Docker y Docker Compose
Para la instalación de Docker y Docker Compose siga la siguiente [guía](https://docs.docker.com/engine/install/) de acuerdo a su sistema operativo.
### Instalación de Docker y Docker Compose en Debian 11
#### Actualizar e instalar paquetes:
```sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    git \
    gnupg \
    lsb-release
```
#### Agregar la clave GPG oficial de Docker:
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
#### Configurar el repositorio de Docker:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### Instalar Docker Engine
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
### Clonar Repositorio
```
git clone https://github.com/rbezz/oxy-ops.git
```
### Instalación de Filebeat 
#### Linux DEB
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.4.2-amd64.deb
sudo dpkg -i filebeat-8.4.2-amd64.deb
```
#### Linux RPM
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.4.2-x86_64.rpm
sudo rpm -vi filebeat-8.4.2-x86_64.rpm
```
#### Reemplazar el archivo filebeat.yml
```
sudo cp oxy-ops/filebeat_conf/filebeat.yml /etc/filebeat/filebeat.yml
```
#### Copiar el certificado 
```
sudo cp oxy-ops/filebeat_conf/logstash-remote.crt /etc/filebeat/logstash-remote.crt
```
#### Test de configuración de Filebeat y conexión con Logstash
```
sudo filebeat test config
sudo filebeat test output
```
#### Iniciar y habilitar Filebeat
```
sudo systemctl enable filebeat
sudo service filebeat start
```
