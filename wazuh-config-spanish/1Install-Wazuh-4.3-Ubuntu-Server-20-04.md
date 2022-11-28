##### Descargar Ubuntu Server 20.04.4 LTS

[Ubuntu Server Donwload Site](https://ubuntu.com/download/server)

##### Recomendaciones

Para una instalación de Wazuh en un entorno de producción, se recomienda que el sistema operativo cuente con un espacio en Disco Duro de 1Tb o superior) (para unos 15 servidores, con datos para 12 meses)

Revisar el siguiente link para calcular el espacio que podria necesitar segun el número de dispositivos, el tipo y el tiempo.

[Wazuh - Calculate the hot storage you need](https://wazuh.com/cloud/#pricing)

Descargar e instalar su sistema operativo Ubuntu Server 20.04.4 LTS (puede instalar de manera predeterminada y/o siguiendo los lineamientos de seguridad del [CIS Benchmarks](https://edu.anarcho-copy.org/GNU%20Linux%20-%20Unix-Like/Debian/CIS_Ubuntu_Linux_20.04_LTS_Benchmark_v1.1.0.pdf)

Ejecute los siguientes comandos:

##### Actualizar el Sistema Operativo

```code
foo@bar:~$ sudo apt update
foo@bar:~$ sudo apt upgrade
```

##### Habilitar SSH en el Firewall

```code
foo@bar:~$ sudo ufw allow 22
```

##### Habilitar herramientas auxiliares (htop para ver el rendimiento del servidor, net-tools para tener ifconfig)

```code
foo@bar:~$ sudo apt install htop
foo@bar:~$ sudo apt install net-tools
```

##### Cambiar la hora del Sistema Operativo

```code
foo@bar:~$ sudo timedatectl
foo@bar:~$ sudo timedatectl set-ntp no
foo@bar:~$ sudo timedatectl set-timezone America/La_Paz
foo@bar:~$ sudo timedatectl set-time hh:mm:ss
foo@bar:~$ sudo timedatectl set-time YYYY-MM-DD
foo@bar:~$ sudo timedatectl
```

##### Cambiar de idioma el teclado

```code
foo@bar:~$ sudo localectl list-keymaps
foo@bar:~$ sudo localectl set-keymap latam
```

##### Cambiar la IP Estatica del Sistema Operativo

En el ejemplo se cambia la IP a 192.168.121.221/24, se utiliza como gateway 192.168.121.1 y se define los DNS para tener salida a internet.

[Link de referencia para cambiar la IP](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/)

Netplan configuration files are stored in the /etc/netplan directory. You’ll probably find one or more YAML files in this directory. The name of the file may differ from setup to setup. Usually, the file is named either 01-netcfg.yaml, 50-cloud-init.yaml, or NN_interfaceName.yaml, but in your system it may be different.

```code
foo@bar:~$ cd /etc/netplan
foo@bar:~$ sudo vim 01-netcfg.yaml
```

```code
#cambiar el contenido del archivo

network:
version: 2
renderer: networkd
ethernets:
ens33:

dhcp4: no

addresses:

- 192.168.121.221/24

gateway4: 192.168.121.1

nameservers:

addresses: [8.8.8.8, 1.1.1.1]
```

Ejecutar el comando para aplicar los cambios en la configuracion de la red
```code
foo@bar:~$ sudo netplan apply
```

##### Cambiar el DNS para tener salida a internet

Editar el archivo resolv.conf
```code
foo@bar:~$ sudo vim /etc/resolv.conf
```

Incluir al contenido del archivo
```code
nameserver 8.8.8.8
```

Reiniciar el servicio de DNS del servidor Ubuntu Server
```code
foo@bar:~$ sudo systemctl restart systemd-resolved.service
```

##### Instalación de Wazuh 4.3

Wazuh tiene 3 componentes, Indexer, Server (tambien conocido como Manager) y Dashboard, los comandos corresponden a una instalación de los 3 componentes en el mismo servidor (de ahi que la IP es 127.0.0.1)

```code
# requiere usuario root
foo@bar:~$ sudo su
```

##### Instalación del componente Indexer

[Documentación Indexer](https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html)

Descargar el script de instalacion wazuh-install.sh
```code
foo@bar:~$ curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
foo@bar:~$ curl -sO https://packages.wazuh.com/4.3/config.yml
```

En el archivo descargado config.yml se define la configuración de los 3 componentes,
Por ejemplo:
* Solo existirá un Indexer que tendra el nombre de "node-1" (es posible instalar el modulo Indexer como Cluster)
* Solo existirá un Server que tendra el nombre de "wazuh-1" (es posible instalar el modulo Server como Cluster)
* Solo existe una instancia del Dashboard que tendra el nombre de "dashboard"



```code

nodes:

# Wazuh indexer nodes

indexer:

- name: node-1
  ip: 127.0.0.1

# - name: node-2

# ip: <indexer-node-ip>

# - name: node-3

# ip: <indexer-node-ip>

# Wazuh server nodes

# Use node_type only with more than one Wazuh manager

server:

- name: wazuh-1
  ip: 127.0.0.1

# node_type: master

# - name: wazuh-2

# ip: <wazuh-manager-ip>

# node_type: worker

# Wazuh dashboard node

dashboard:

- name: dashboard
  ip: 127.0.0.1
```

```code
foo@bar:~$ bash wazuh-install.sh --generate-config-files
foo@bar:~$ bash wazuh-install.sh --wazuh-indexer node-1
foo@bar:~$ bash wazuh-install.sh --start-cluster
```

Realizar un test para verificar que el servicio esta instalado y corriendo

```code
foo@bar:~$ telnet 127.0.0.1 9200
```

##### Instalación del componente Server

[Documentación Server](https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html)


```code
foo@bar:~$ bash wazuh-install.sh --wazuh-server wazuh-1
```

##### Instalación del componente Dashboard

[Documentación Dashboard](https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/installation-assistant.html)


```code
foo@bar:~$ bash wazuh-install.sh --wazuh-dashboard dashboard
```

##### Instalación finalizada.

```code
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
User: admin
Password: <ADMIN_PASSWORD>
INFO: Installation finished.

04/10/2022 12:56:05 INFO: --- Summary ---
04/10/2022 12:56:05 INFO: You can access the web interface https://<wazuh-dashboard-ip>
User: admin
Password: BRPt?FMJAts.acMrldqqVp03OE9QRpA9
04/10/2022 12:56:05 INFO: Installation finished.
```
Las claves generadas en la instalacion se encuentran en el archivo wazuh-passwords.txt dentro del archvivo wazuh-install-files.tar
```code
You now have installed and configured Wazuh. All passwords generated by the Wazuh installation assistant can be found in the wazuh-passwords.txt file inside the wazuh-install-files.tar archive. To print them, run the following command:

tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Ejemplo del contenido del archivo wazuh-install-files/wazuh-passwords.txt

```code
cat wazuh-install-files/wazuh-passwords.txt

# Admin user for the web user interface and Wazuh indexer. Use this user to log in to Wazuh dashboard

indexer_username: 'admin'
indexer_password: 'BRPt?FMJAts.acMrldqqVp03OE9QRpA9'

# Wazuh dashboard user for establishing the connection with Wazuh indexer

indexer_username: 'kibanaserver'
indexer_password: 'ph+e02ii*HiNNHoZ9IG9k?LfdjXaDaW6'

# Regular Dashboard user, only has read permissions to all indices and all permissions on the .kibana index

indexer_username: 'kibanaro'
indexer_password: '9sm7G1ZiHb7Ib4iti2fc+qHi12NDSOxA'

# Filebeat user for CRUD operations on Wazuh indices

indexer_username: 'logstash'
indexer_password: 'CwO7pwHzvy0AvZIj4qFBdUAxjih2u+h*'

# User with READ access to all indices

indexer_username: 'readall'
indexer_password: 'WF4QSoSS.ilAxOf.?E2Cy2Y60IjqMNoA'

# User with permissions to perform snapshot and restore operations

indexer_username: 'snapshotrestore'
indexer_password: 'uNzbU0*EL4ly?2rES70eT81j+QXjrOfT'

# Password for wazuh API user

api_username: 'wazuh'
api_password: 'lO*BV+sYFPaerXbU*vEc*7.tge1rzgPF'

# Password for wazuh-wui API user

api_username: 'wazuh-wui'
api_password: 'S.f7K4nFCA4GEIZgPq3gxaWWV+9Ly9JS'
```