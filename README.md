![](https://www.oxyops.live/wp-content/uploads/2022/10/LOGO-HONEYPOT.png)

Red de Honeypot para la detección de eventos del ciberespacio del Paraguay.
# Despliegue de sensores 
Los sensores honeypot albergarán sistemas vulnerables y transmitirán los datos de registro al servidor honeypot a través de Filebeat. Estos sensores pueden ser desplegados en cualquiera de las distribuciones Linux disponibles.

## Requisitos del sistema
La instalación puede ser realizada en hardware real, en una máquina virtual o en la nube. 
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
sudo mv oxy-ops/filebeat_conf/filebeat.yml /etc/filebeat/
```
#### Mover el certificado logstash-remote.crt a /etc/filebeat
```
sudo mv oxy-ops/filebeat_conf/logstash-remote.crt /etc/filebeat/logstash-remote.crt
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
### Creación y configuración de usuario docker 
```
sudo addgroup --gid 2000 oxyops
sudo adduser --system --no-create-home --uid 2000 --disabled-password --disabled-login --gid 2000 oxyops
sudo mkdir -vp /data/
sudo chmod 770 -R /data
sudo chown oxyops:oxyops -R /data
```
### Cambiar el puerto SSH del sistema host (Descomentar y cambiar el puerto predeterminado. Ej: Port 2224)
```
sudo nano /etc/ssh/sshd_config
sudo service ssh reload
```
## Instalación de honeypot
### Instalación de todos los honeypot
```
cd /oxy-ops/docker
sudo docker compose up --build
```
Una vez terminada la construcción del contenedor se deberá detener la misma con Ctrl + C y actualizar los permisos: 
```
sudo chmod 770 -R /data
sudo chown oxyops:oxyops -R /data
```
Nuevamente se deberá ejecutar el comando docker compose up - d, la opción -d indica que se ejecutará como demonio:
```
sudo docker compose up -d
```
### Instalación de honeypot (si no desea instalar todos los sensores)
Se deberá acceder al directorio del honeypot, por ejemplo:
```
cd /oxy-ops/docker/cowrie
sudo docker compose up --build
```
Una vez terminada la construcción del contenedor se deberá detener la misma con Ctrl + C y actualizar los permisos: 
```
sudo chmod 770 -R /data
sudo chown oxyops:oxyops -R /data
```
Nuevamente se deberá ejecutar el comando docker compose up - d, la opción -d indica que se ejecutará como demonio:
```
sudo docker compose up -d
```
### Configuración del firewall o router
Por el despliegue de cada honeypot se deberán habilitar los puertos correspondientes a cada servicio, esta configuración dependerá de su firewall o router:
- Cowrie: habilitar puerto 22 y 23 y redirigir a la ip del sensor puerto 2222 y 2323.
- Elasticpot: habilitar puerto 9200 y redirigir a la ip del sensor puerto 9200.
- Mailoney: habilitar puerto 25 y redirigir a la ip del sensor puerto 2525.
