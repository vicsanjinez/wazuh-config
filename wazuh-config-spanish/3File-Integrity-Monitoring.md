##### Monitoreo de la Integridad de Archivos en Windows

[Documentacion File Integrity Monitoring](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/fim-configuration.html)

[Documentacion Windows Syscheck](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/syscheck.html#reference-ossec-syscheck-windows-registry)

###### En el Agente en Windows:

Modificar el archivo ossec.conf para incluir la siguiente configuración y monitorizar los cambios en la carpeta c:/temporal (por ejemplo)

```code
C:\> notepad C:\Program Files (x86)\ossec-agent/ossec.conf
```

* "frecuency" tiene que tener un valor entero que define cada cuentos segundos se realizar la verificacion de la integridad 
* "directories check_all="yes" " define que se verificara la integridad de todos los archivos que existan en el directorio
* "recursion_level="3" " define que se verificara la integridad de los archivos y carpetas que se encuentren dentro de otras carpetas hasta 3 niveles adentro, 
Ej. carpeta1->carpeta2->carpeta3->archivo3monitorizado.txt 
Ej. carpeta1->carpeta2->carpeta3->archivo3monitorizado.txt->carpeta4->archivo4NOmonitorizado.txt

```xml
<!-- File integrity monitoring -->
<syscheck>
<disabled>no</disabled>
<!-- Frequency that syscheck is executed default every 12 hours -->
<frequency>5</frequency>
<alert_new_files>yes</alert_new_files>
<directories check_all="yes" realtime="yes" report_changes="yes" recursion_level="3">c:/temporal</directories>
<windows_registry arch="both" check_all="yes">HKEY_LOCAL_MACHINE\SOFTWARE</windows_registry>
<windows_registry arch="32bit" check_all="yes" check_mtime="yes">HKEY_LOCAL_MACHINE\SYSTEM\Setup</windows_registry>
```

Si se realiza algun cambio en la carpeta c:/temporal se generan los eventos respectivos y se visualizaran en el Dashboard



##### Monitoreo de la Integridad de Archivos en Linux

###### En el Agente en Linux:

Modificar el archivo ossec.conf para incluir la siguiente configuración y monitorizar los cambios en la carpeta /home/administrador (por ejemplo)

```code
foo@bar:~$ sudo vim /var/ossec/etc/ossec.conf
```


```xml
<frequency>5</frequency>
<alert_new_files>yes</alert_new_files>
<directories check_all="yes" realtime="yes" report_changes="yes" recursion_level="3">/home/administrador</directories>
```

Reiniciar el servicio de Wazuh Agent
```code
foo@bar:~$ sudo systemctl restart wazuh-agent
```

Si se realiza algun cambio en la carpeta /home/administrador se generan los eventos respectivos y se visualizaran en el Dashboard