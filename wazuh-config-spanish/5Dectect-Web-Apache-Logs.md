##### Detectar Logs de Apache Web Server

###### En el Agente Linux:

Instalar Apache Server 
```code
foo@bar:~$ sudo apt install apache2
```

Configurar el Firewall para habilitar el puerto 80
```code
foo@bar:~$ sudo ufw allow 'Apache'
```

Testear el funcionamiento del Servidor Apache en un Navegador
```code
http://the_system_ip_address
```


###### En el Wazuh Manager:

Una opcion de Wazuh Manager, es la posibilidad de enviar configuraciones a los Agentes de manera remota,
Para lograr esto, se incluye el siguiente bloque de configuraciones en el archivo agent.conf


```code
foo@bar:~$ sudo vim /var/ossec/etc/shared/default/agent.conf
```


(Se especifica el path de los logs del Servidor Apache /var/log/apache2/access.log para que los mismos sean enviados a Wazuh Manager y puedan ser analizados)
```xml
<localfile>
<log_format>syslog</log_format>
<location>/var/log/apache2/access.log</location>
</localfile>
```

Reiniciar el Wazuh Manager

```code
foo@bar:~$ sudo systemctl restart wazuh-manager
```

Ahora las solicitudes web (request) que se realizan al Servidor Apache son analizados por Wazuh Manager a traves de los registros de los logs que llegan desde el Agente