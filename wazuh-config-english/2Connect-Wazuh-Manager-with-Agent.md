##### Connect Wazuh Manager and Agent with Password

[Connect Wazuh Manager and Agent with Password Documentation](https://documentation.wazuh.com/current/user-manual/agent-enrollment/security-options/using-password-authentication.html)


###### On the Wazuh Manager:

Enable the password authentication option by adding the configuration highlighted below to the "auth" section of the manager configuration file /var/ossec/etc/ossec.conf

```code
foo@bar:~$ sudo vim /var/ossec/etc/ossec.conf
```

```xml
<auth>
<use_password>yes</use_password>
</auth>
```

Set a password to be used with agent enrollment,
The password must be created and stored in the new file authd.pass which must be saved in the path /var/ossec/etc/authd.pass

Replace CUSTOM_PASSWORD with your chosen agent enrollment password and run the following command:

```code
foo@bar:~$ echo "CUSTOM_PASSWORD" > /var/ossec/etc/authd.pass
```

Change the authd.pass file permissions and ownership

```code
foo@bar:~$ chmod 640 /var/ossec/etc/authd.pass
foo@bar:~$ chown root:wazuh /var/ossec/etc/authd.pass
```

After this, restart the Wazuh Manager service for the changes to take effect

```code
foo@bar:~$ systemctl restart wazuh-manager
```

###### On the Windows Agent:

To install the Agent with password in Windows there are the following requirements:
* Administrator privileges are required in the Powershell terminal to perform the installation.
* PowerShell 3.0 or higher is required.

Command to check the version of Powershell
```powershell
PS C:\> $PSVersionTable
```

Install de Agent in Windows
```powershell
PS C:\> Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.3.8-1.msi -OutFile ${env:tmp}\wazuh-agent-4.3.8.msi; msiexec.exe /i ${env:tmp}\wazuh-agent-4.3.8.msi /q WAZUH_MANAGER='192.168.147.128' WAZUH_REGISTRATION_SERVER='192.168.147.128' WAZUH_AGENT_GROUP='default' WAZUH_REGISTRATION_PASSWORD='CUSTOM_PASSWORD'
```

Start the Agent
```powershell
PS C:\> NET START WazuhSvc
```

###### En el Agente Linux:

To install the Agent with password in Ubuntu Server 20.04 there are the following requirements:
* Root permissions required

```code
curl -so wazuh-agent-4.3.8.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.3.8-1_amd64.deb && sudo WAZUH_MANAGER='192.168.147.128' WAZUH_AGENT_GROUP='default' WAZUH_REGISTRATION_PASSWORD='CUSTOM_PASSWORD' WAZUH_AGENT_NAME='ubuntulinux64' dpkg -i ./wazuh-agent-4.3.8.deb
```

Start the Agent
```code
foo@bar:~$ sudo systemctl daemon-reload
foo@bar:~$ sudo systemctl enable wazuh-agent
foo@bar:~$ sudo systemctl start wazuh-agent
```

###### Delete Agent

###### Remove Agent on Windows
Simply uninstall the program in the option "Add or remove programs"

###### Remove Agent on Linux
Run the following commands:

```code
foo@bar:~$ sudo dpkg --purge wazuh-agent
foo@bar:~$ sudo systemctl disable wazuh-agent
foo@bar:~$ sudo systemctl daemon-reload
```
