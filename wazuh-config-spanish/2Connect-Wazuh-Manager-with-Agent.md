##### Conectar Wazuh Manager y el Agente con Password

[Documentación Connect Wazuh Manager and Agent with Password](https://documentation.wazuh.com/current/user-manual/agent-enrollment/security-options/using-password-authentication.html)


###### En Wazuh Manager:

Habilita la autenticacion por password, incluyendo la configuración "auth" en el archivo /var/ossec/etc/ossec.conf

```code
foo@bar:~$ sudo vim /var/ossec/etc/ossec.conf
```

```xml
<auth>
<use_password>yes</use_password>
</auth>
```

Establece una password que será usada en el proceso de registro de los agentes,
La password debe ser creada y almacenada en el nuevo archivo authd.pass el cual debe ser guardado en el path /var/ossec/etc/authd.pass

Reemplace CUSTOM_PASSWORD con la password que usted eligio para el registro de los agentes y ejecute el siguiente comando:

```code
foo@bar:~$ echo "CUSTOM_PASSWORD" > /var/ossec/etc/authd.pass
```

Cambie los permisos y propiedad del archivo creado authd.pass

```code
foo@bar:~$ chmod 640 /var/ossec/etc/authd.pass
foo@bar:~$ chown root:wazuh /var/ossec/etc/authd.pass
```

Despues de esto, reinicie el servicio de Wazuh Manager

```code
foo@bar:~$ systemctl restart wazuh-manager
```

###### En el Agente Windows:

Para instalar el Agente con password en Windows existen los siguientes requerimientos:
* Se requiere privilegios de administrador en la terminal de Powershell para realizar la instalacion.
* Se requiere PowerShell 3.0 o superior.

Comando para verificar la version de Powershell
```powershell
PS C:\> $PSVersionTable
```

Instalacion del Agente en Windows
```powershell
PS C:\> Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.3.8-1.msi -OutFile ${env:tmp}\wazuh-agent-4.3.8.msi; msiexec.exe /i ${env:tmp}\wazuh-agent-4.3.8.msi /q WAZUH_MANAGER='192.168.147.128' WAZUH_REGISTRATION_SERVER='192.168.147.128' WAZUH_AGENT_GROUP='default' WAZUH_REGISTRATION_PASSWORD='CUSTOM_PASSWORD'
```

Iniciar el servicio del Agente
```powershell
PS C:\> NET START WazuhSvc
```

###### En el Agente Linux:

Para instalar el Agente con password en Ubuntu Server 20.04 existe los siguientes requerimientos:
* Se requiere permisos de Root

```code
curl -so wazuh-agent-4.3.8.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.3.8-1_amd64.deb && sudo WAZUH_MANAGER='192.168.147.128' WAZUH_AGENT_GROUP='default' WAZUH_REGISTRATION_PASSWORD='CUSTOM_PASSWORD' WAZUH_AGENT_NAME='ubuntulinux64' dpkg -i ./wazuh-agent-4.3.8.deb
```

Iniciar el Servicio del Agente
```code
foo@bar:~$ sudo systemctl daemon-reload
foo@bar:~$ sudo systemctl enable wazuh-agent
foo@bar:~$ sudo systemctl start wazuh-agent
```

###### Eliminar el Agente

###### Eliminar el Agente en Windows 
Simplemente desinstalar el programa en la opcion "Agregar o quitar programas"


###### Eliminar el Agente en Linux
Ejecutar los siguientes comandos:
```code
foo@bar:~$ sudo dpkg --purge wazuh-agent
foo@bar:~$ sudo systemctl disable wazuh-agent
foo@bar:~$ sudo systemctl daemon-reload
```
