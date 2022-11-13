##### Detectar Logs Personalizados

[Documentacion Decoders](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#regex-decoders)

[Documentacion Reglas Personalizadas](https://documentation.wazuh.com/current/user-manual/ruleset/custom.html#ruleset-custom)

La deteccion de Logs Personalizados en Wazuh funciona de la siguiente manera:
El log que llega a Wazuh, primero es analizado por un Decoder, con el objetivo de extraer los valores que existen en el log.

Luego el mismo log, es analizado por las Reglas (Rules), para poder establecer una identificacion, clasificacion, generar una alerta, etc.


###### Incluir un nuevo Decoder y Rule

Se configura los archivos /var/ossec/etc/decoders/local_decoder.xml y /var/ossec/etc/rules/local_rules.xml los cuales tienen el proposito de realizar detecciones simples. En caso de que requiera realizar detecciones complejas, lo recomendable es crear nuevos archivos Decoders y Rules.

El valor del "id" de la Rule personalizada debe estar entre 100000 y 120000.

El log personalizado a detectar es el siguiente:

```code
Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'
```

Como se menciono, lo primero es definir el Decoder, para ello se configura el archivo local_decoder.xml:

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

Ahora, se incluye la siguiente Rule en el archivo local_rules.xml:

```code
foo@bar:~$ sudo vim /var/ossec/etc/rules/local_rules.xml:
```

(Como el valor de "level" es 0, la regla sera aplicada pero no generara una Alerta)
```xml
<group name="example">

<rule id="100010" level="0">
  <program_name>example</program_name>
  <description>User logged</description>
</rule>

</group>
```

Utilizando el aplicativo "/var/ossec/bin/wazuh-logtest" podemos probar si el Decoder como la Rule creada estan funcionando correctamente, ya que una vez ejecutado el programa insertamos el log para que el mismo sea analizado por los Decoders y Rules existentes

```code
foo@bar:~$ cd /var/ossec/bin/
foo@bar:~$ ./wazuh-logtest
Starting wazuh-logtest v4.3.8
Type one log per line
```

Log a probar e insertar en wazuh-logtest:
```code
Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'
```

Respuesta de wazuh-logtest:
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
[Documentation Wazuh Ruleset Regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html)
[Test Regex101](https://regex101.com/)
[Test Regexr](https://regexr.com/)

Como segundo ejemplo, se creara un Log Personalizado con la siguiente estructura:
```code
SISTEMA,STATUS,IP,FECHA,HORA,REQUEST,STATUS WEB,DETALLE ERROR
```

Algunos ejemplos del Log Personalizado son los siguientes:
```code
sistemainterno,OK,192.168.147.1,11-10-2022,19:55:23,GET /parametro=hola HTTP/1.1,200,NULL

sistemainterno,ERROR,192.168.147.2,11-10-2022,19:56:49,GET /parametro=#$%$## HTTP/1.1,500,CARACTERES NO VALIDOS

sistemainterno,OK,192.168.147.1,11-10-2022,19:55:31,GET /parametro=hello HTTP/1.1,200,NULL

sistemainterno,ERROR,192.168.147.1,11-10-2022,19:57:28,GET /asdfasdf=saludo HTTP/1.1,404,PATH NO ENCONTRADO
```

###### En el Wazuh Manager:

Editar el archivo local_decoder.xml para definir un Decoder y extraer los valores que existen en el Log Personalizado
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
  <order>estado,ip,fecha,hora,request,status_web</order>
</decoder>
```

Ahora, incluir la siguiente Rule en el archivo local_rules.xml, para que genere una alerta cuando el valor de "estado" sea "ERROR".

```code
foo@bar:~$ sudo vim /var/ossec/etc/rules/local_rules.xml
```

```xml
<group name="sistemainterno">

<rule id="100012" level="10">
  <decoded_as>sistemainterno</decoded_as>
   <field name="estado">ERROR</field>
   <description>Error en la aplicacion</description>
</rule>

</group>
```

###### Primera prueba:

Utilizando el aplicativo "/var/ossec/bin/wazuh-logtest" podemos probar si el Decoder como la Rule creada estan funcionando correctamente, ya que una vez ejecutado el programa insertamos el log para que el mismo sea analizado por los Decoders y Rules existentes

```code
foo@bar:~$ cd /var/ossec/bin/
foo@bar:~$ ./wazuh-logtest
Starting wazuh-logtest v4.3.8
Type one log per line
```

Log a probar e insertar en wazuh-logtest:
```code
sistemainterno,ERROR,192.168.147.1,11-10-2022,19:57:28,GET /asdfasdf=saludo HTTP/1.1,404,PATH NO ENCONTRADO
```

Respuesta de wazuh-logtest:
```code
OUTPUT

**Phase 1: Completed pre-decoding.
full event: 'sistemainterno,ERROR,192.168.147.1,11-10-2022,19:57:28,GET /asdfasdf=saludo HTTP/1.1,404,PATH NO ENCONTRADO'

**Phase 2: Completed decoding.
name: 'sistemainterno'
estado: 'ERROR'
fecha: '11-10-2022'
hora: '19:57:28'
ip: '192.168.147.1'
request: 'GET /asdfasdf=saludo HTTP/1.1'
status_web: '404,PATH NO ENCONTRADO'

**Phase 3: Completed filtering (rules).
id: '100012'
level: '10'
description: 'Error en la aplicacion'
groups: '['sistemainterno']'
firedtimes: '1'
mail: 'False'
**Alert to be generated.
```

Se puede observar que genera la alerta ya que el valor de "estado" es "ERROR"


###### Segunda prueba:

Utilizando el aplicativo "/var/ossec/bin/wazuh-logtest" podemos probar si el Decoder como la Rule creada estan funcionando correctamente, ya que una vez ejecutado el programa insertamos el log para que el mismo sea analizado por los Decoders y Rules existentes

```code
foo@bar:~$ cd /var/ossec/bin/
foo@bar:~$ ./wazuh-logtest
Starting wazuh-logtest v4.3.8
Type one log per line
```

Log a probar e insertar en wazuh-logtest:
```code
sistemainterno,OK,192.168.147.1,11-10-2022,19:55:31,GET /parametro=hello HTTP/1.1,200,NULL
```

Respuesta de wazuh-logtest:
```code
OUTPUT

**Phase 1: Completed pre-decoding.
full event: 'sistemainterno,OK,192.168.147.1,11-10-2022,19:55:31,GET /parametro=hello HTTP/1.1,200,NULL'

**Phase 2: Completed decoding.
name: 'sistemainterno'
estado: 'OK'
fecha: '11-10-2022'
hora: '19:55:31'
ip: '192.168.147.1'
request: 'GET /parametro=hello HTTP/1.1'
status_web: '200,NULL'
```

Se puede observar que no genera la alerta ya que el valor de "estado" es "OK"