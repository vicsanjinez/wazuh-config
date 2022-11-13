##### Download Ubuntu Server 20.04.4 LTS

[Ubuntu Server Donwload Site](https://ubuntu.com/download/server)

##### Recomendations

For a Wazuh installation in a production environment, it is recommended that the operating system has a Hard Disk space of 1Tb or more) (for 15 servers, with data for 12 months)

Review the following link to calculate the space you may need depending on the number of devices, type and time.

[Wazuh - Calculate the hot storage you need](https://wazuh.com/cloud/#pricing)

Download and install your Ubuntu Server 20.04.4 LTS (you can install by default and/or follow the [CIS Benchmarks](https://edu.anarcho-copy.org/GNU%20Linux%20-%20Unix-Like/Debian/CIS_Ubuntu_Linux_20.04_LTS_Benchmark_v1.1.0.pdf) security guidelines

Execute the next commands:

##### Update the Ubuntu Server

```code
foo@bar:~$ sudo apt update
foo@bar:~$ sudo apt upgrade
```

##### Enable SSH on the Firewall

```code
foo@bar:~$ sudo ufw allow 22
```

##### Enable helper tools (htop to see server performance, net-tools to have ifconfig)

```code
foo@bar:~$ sudo apt install htop
foo@bar:~$ sudo apt install net-tools
```

##### Change the date and time on Ubuntu Server

```code
foo@bar:~$ sudo timedatectl
foo@bar:~$ sudo timedatectl set-ntp no
foo@bar:~$ sudo timedatectl set-timezone America/La_Paz
foo@bar:~$ sudo timedatectl set-time hh:mm:ss
foo@bar:~$ sudo timedatectl set-time YYYY-MM-DD
foo@bar:~$ sudo timedatectl
```

##### Change the language of the keyboard

```code
foo@bar:~$ sudo localectl list-keymaps
foo@bar:~$ sudo localectl set-keymap latam
```

##### Change the Static IP on Ubuntu Server

In the example, the IP is changed to 192.168.121.221/24, 192.168.121.1 is used as a gateway and the DNS is defined to have access to the Internet.

[Reference link to change the IP](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/)

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

Execute the command to apply the changes in the network configuration
```code
foo@bar:~$ sudo netplan apply
```

##### Change the DNS to have access to the internet

Edit the resolv.conf file
```code
foo@bar:~$ sudo vim /etc/resolv.conf
```

Add to the file
```code
nameserver 8.8.8.8
```

Restart the DNS service
```code
foo@bar:~$ sudo systemctl restart systemd-resolved.service
```

##### Install Wazuh 4.3
Wazuh has 3 components, Indexer, Server (also known as Manager) and Dashboard, the commands correspond to an installation of all 3 components on the same server (that the reason why the IP is 127.0.0.1)

```code
# requiere usuario root
foo@bar:~$ sudo su
```

##### Install Indexer

[Indexer Documentation](https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html)

Download the install script wazuh-install.sh
```code
foo@bar:~$ curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
foo@bar:~$ curl -sO https://packages.wazuh.com/4.3/config.yml
```

In the downloaded config.yml file, the configuration of the 3 components is defined,
For example:
* There will only be one Indexer that will have the name "node-1" (it is possible to install the Indexer as a Cluster)
* There will only be one Server that will have the name "wazuh-1" (it is possible to install the Server as a Cluster)
* There is only one Dashboard instance that will have the name "dashboard"


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

Perform a test to verify that the service is installed and running

```code
foo@bar:~$ telnet 127.0.0.1 9200
```

##### Install Server

[Server Documentation](https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html)


```code
foo@bar:~$ bash wazuh-install.sh --wazuh-server wazuh-1
```

##### Install Dashboard

[Dashboard Documentation](https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/installation-assistant.html)


```code
foo@bar:~$ bash wazuh-install.sh --wazuh-dashboard dashboard
```

##### Installation finished.

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

The keys generated in the installation are found in the file wazuh-passwords.txt inside the archive wazuh-install-files.tar

```code
You now have installed and configured Wazuh. All passwords generated by the Wazuh installation assistant can be found in the wazuh-passwords.txt file inside the wazuh-install-files.tar archive. To print them, run the following command:

tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Content of wazuh-install-files/wazuh-passwords.txt file (as an example)

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