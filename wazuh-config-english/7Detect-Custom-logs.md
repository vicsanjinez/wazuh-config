##### Detect Custom Logs

[Decoders Documentation](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#regex-decoders)

[Custom Rules Documentation](https://documentation.wazuh.com/current/user-manual/ruleset/custom.html#ruleset-custom)

The detection of Custom Logs in Wazuh works as follows:
The log that arrives at Wazuh, is first analyzed by a Decoder, in order to extract the values that exist in the log.

Then, the same log is analyzed by the Rules, in order to establish an identification, classification, generate an alert, etc.

###### Adding new Decoders and Rules

Config the /var/ossec/etc/decoders/local_decoder.xml and /var/ossec/etc/rules/local_rules.xml files, which have the purpose of performing simple detections. In case you need to perform complex detections, it's recommended to create new Decoders and Rules files.

The value of "id" of the Custom Rule must be a value between 100000 and 120000.

The custom log to detect is:

```code
Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'
```

As mentioned, the first thing is to define the Decoder, so, config the local_decoder.xml file:

```code
foo@bar:~$ sudo vim /var/ossec/etc/decoders/local_decoder.xml:
```

```xml
<decoder name="example">
  <program_name>^example</program_name>
</decoder>

<decoder name="example">
  <parent>example</parent>
  <regex>User '(\w+)' logged from '(\d+.\d+.\d+.\d+)'</regex>
  <order>user, srcip</order>
</decoder>
```

Now, we will add the following Rule to local_rules.xml:

```code
foo@bar:~$ sudo vim /var/ossec/etc/rules/local_rules.xml:
```

(Since the value of "level" is 0, the rule will be applied but will not generate an Alert)
```xml
<group name="example">

<rule id="100010" level="0">
  <program_name>example</program_name>
  <description>User logged</description>
</rule>

</group>
```

Using the application "/var/ossec/bin/wazuh-logtest" we can test if the Decoder and Rule are working correctly, when the program is executed, we type the custom log, so, that it can be analyzed by the existing Decoders and Rules

```code
foo@bar:~$ cd /var/ossec/bin/
foo@bar:~$ ./wazuh-logtest
Starting wazuh-logtest v4.3.8
Type one log per line
```

Log to test:
```code
Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'
```

Output of wazuh-logtest:
```code
OUTPUT

Type one log per line

Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'

**Phase 1: Completed pre-decoding.
full event: 'Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100''
timestamp: 'Dec 25 20:45:02'
hostname: 'MyHost'
program_name: 'example'

**Phase 2: Completed decoding.
name: 'example'
dstuser: 'admin'
srcip: '192.168.1.100'

**Phase 3: Completed filtering (rules).
id: '100010'
level: '0'
description: 'User logged'
groups: '['local', 'syslog', 'sshd']'
firedtimes: '1'
mail: 'False'
```

##### Detect Custom Logs 2

[Socfortress Decoders](https://socfortress.medium.com/understanding-wazuh-decoders-4093e8fc242c)
[Wazuh Ruleset Regex Documentation](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html)
[Test Regex101](https://regex101.com/)
[Test Regexr](https://regexr.com/)

As a second example, a Custom Log will be created with the following structure:
```code
SYSTEM,CUSTOM STATUS,IP,DATE,TIME,REQUEST,WEB STATUS,DETAIL ERROR
```

Some examples of the Custom Log are the following:
```code
sistemainterno,OK,192.168.147.1,11-10-2022,19:55:23,GET /param=hi HTTP/1.1,200,NULL

sistemainterno,ERROR,192.168.147.2,11-10-2022,19:56:49,GET /param=#$%$## HTTP/1.1,500,INVALID CHARACTERS

sistemainterno,OK,192.168.147.1,11-10-2022,19:55:31,GET /param=hello HTTP/1.1,200,NULL

sistemainterno,ERROR,192.168.147.1,11-10-2022,19:57:28,GET /asdfasdf=regards HTTP/1.1,404,NOT FOUND
```

###### On Wazuh Manager:

Edit the local_decoder.xml file to define a Decoder and extract the values that exist in the Custom Log

```code
foo@bar:~$ sudo vim /var/ossec/etc/decoders/local_decoder.xml
```

```xml
<decoder name="sistemainterno">
  <prematch>^sistemainterno,</prematch>
</decoder>

<decoder name="sistemainterno_child">
  <parent>sistemainterno</parent>
  <regex offset="after_parent">(\.+),(\.+),(\.+),(\.+),(\.+),(\.+)</regex> 
  <order>custom_status,ip,date,time,request,web_status</order>
</decoder>
```
Now, include the following Rule in the local_rules.xml file, so that it generates an alert when the "custom_status" value is "ERROR".

```code
foo@bar:~$ sudo vim /var/ossec/etc/rules/local_rules.xml
```

```xml
<group name="sistemainterno">

<rule id="100012" level="10">
  <decoded_as>sistemainterno</decoded_as>
   <field name="custom_status">ERROR</field>
   <description>Application Error</description>
</rule>

</group>
```

###### First Test:

Using the application "/var/ossec/bin/wazuh-logtest" we can test if the Decoder and Rule are working correctly, when the program is executed, we type the custom log, so, that it can be analyzed by the existing Decoders and Rules

```code
foo@bar:~$ cd /var/ossec/bin/
foo@bar:~$ ./wazuh-logtest
Starting wazuh-logtest v4.3.8
Type one log per line
```

Log to test:
```code
sistemainterno,ERROR,192.168.147.1,11-10-2022,19:57:28,GET /asdfasdf=regards HTTP/1.1,404,NOT FOUND
```

Output of wazuh-logtest:
```code
OUTPUT

**Phase 1: Completed pre-decoding.
full event: 'sistemainterno,ERROR,192.168.147.1,11-10-2022,19:57:28,GET /asdfasdf=regards HTTP/1.1,404,NOT FOUND'

**Phase 2: Completed decoding.
name: 'sistemainterno'
custom_status: 'ERROR'
date: '11-10-2022'
time: '19:57:28'
ip: '192.168.147.1'
request: 'GET /asdfasdf=regards HTTP/1.1'
web_status: '404,NOT FOUND'

**Phase 3: Completed filtering (rules).
id: '100012'
level: '10'
description: 'Application Error'
groups: '['sistemainterno']'
firedtimes: '1'
mail: 'False'
**Alert to be generated.
```

It can be seen that it generates the alert, because the value of "custom_status" is "ERROR"

###### Second test:

Using the application "/var/ossec/bin/wazuh-logtest" we can test if the Decoder and Rule are working correctly, when the program is executed, we type the custom log, so, that it can be analyzed by the existing Decoders and Rules

```code
foo@bar:~$ cd /var/ossec/bin/
foo@bar:~$ ./wazuh-logtest
Starting wazuh-logtest v4.3.8
Type one log per line
```

Log to test:
```code
sistemainterno,OK,192.168.147.1,11-10-2022,19:55:31,GET /param=hello HTTP/1.1,200,NULL
```

Output of wazuh-logtest:
```code
OUTPUT

**Phase 1: Completed pre-decoding.
full event: 'sistemainterno,OK,192.168.147.1,11-10-2022,19:55:31,GET /param=hello HTTP/1.1,200,NULL'

**Phase 2: Completed decoding.
name: 'sistemainterno'
custom_status: 'OK'
date: '11-10-2022'
time: '19:55:31'
ip: '192.168.147.1'
request: 'GET /param=hello HTTP/1.1'
web_status: '200,NULL'
```

It can be seen that is not generate the alert, because the value of "custom_status" is "OK"