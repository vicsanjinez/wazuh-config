##### File Integrity Monitoring in Windows

[File Integrity Monitoring Documentation](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/fim-configuration.html)

[Windows Syscheck Documentation](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/syscheck.html#reference-ossec-syscheck-windows-registry)

###### On the Windows Agent:

Modify the ossec.conf file to include the following settings and monitor changes to the c:/temporal folder (for example)

```code
C:\> notepad C:\Program Files (x86)\ossec-agent/ossec.conf
```
* "frequency" must have an integer value that defines how many seconds the integrity check will be performed
* "directories check_all="yes" " defines that the integrity of all files that exist in the directory will be checked
* "recursion_level="3" " defines that the integrity of files and folders inside other folders up to 3 levels inside will be verified,

Ej. folder1->folder2->folder3->file3monitored.txt 
Ej. folder1->folder2->folder3->file3monitored.txt->folder4->file4NOmonitored.txt

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
If any change is made in the folder c:/temporal, the respective events are generated and displayed in the Dashboard



##### File Integrity Monitoring in Linux

###### On the Linux Agent:
Modify the ossec.conf file to include the following settings and monitor changes in the /home/administrador folder (for example)

```code
foo@bar:~$ sudo vim /var/ossec/etc/ossec.conf
```


```xml
<frequency>5</frequency>
<alert_new_files>yes</alert_new_files>
<directories check_all="yes" realtime="yes" report_changes="yes" recursion_level="3">/home/administrador</directories>
```

Restart the Wazuh Agent service
```code
foo@bar:~$ sudo systemctl restart wazuh-agent
```

If any change is made in the folder /home/administrador, the respective events are generated and displayed in the Dashboard
