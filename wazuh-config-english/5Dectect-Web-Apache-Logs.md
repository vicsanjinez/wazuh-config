##### Detect Web Apache Logs

###### On the Linux Agent:

Install Apache Server 
```code
foo@bar:~$ sudo apt install apache2
```

Config the Firewall to enable 80 port
```code
foo@bar:~$ sudo ufw allow 'Apache'
```

Test the Apache Server on Browser
```code
http://the_system_ip_address
```

###### On Wazuh Manager:

An option of Wazuh Manager, is the possibility of sending configurations to Agents remotely,
To achieve this, add this configuration in the agent.conf file

```code
foo@bar:~$ sudo vim /var/ossec/etc/shared/default/agent.conf
```

(The path of the Apache Server logs /var/log/apache2/access.log is specified so that they are sent to Wazuh Manager and can be analyzed)
```xml
<localfile>
<log_format>syslog</log_format>
<location>/var/log/apache2/access.log</location>
</localfile>
```

Restart the Wazuh Manager

```code
foo@bar:~$ sudo systemctl restart wazuh-manager
```

Now the requests that are made to the Apache Server are analyzed by Wazuh Manager through the log records that arrive from the Agent