##### Notificaciones en Slack

[Documentacion Integracion con Slack](https://documentation.wazuh.com/current/proof-of-concept-guide/poc-integrate-slack.html)

Para que funcione el sistema de notificaciones, es necesario crear una "Slack App" y un Webhook, realizar esto es posible ingresar al siguiente enlace (antes de ingresar al enlace debe crear una cuenta en Slack y estar autenticado)

```code
https://api.slack.com/messaging/webhooks
```

Activar los "Incoming Webhooks"
```code
https://api.slack.com/apps/A04ANEZV5D1/incoming-webhooks?
```

Hacer click en el Boton "Add New Webhooks to the Workspace" 
Luego elegir el channel por el que iran los mensajes (elegir el channel recien creado)


Una vez que se tiene el webhook, por ejemplo:
```code
https://hooks.slack.com/services/T032B4KTT5W/B03EA0BLCH4/36Xn7h57btlf6Dciuxxxxxxx
```

Realizar la configuracion en el Wazuh Manager:

Editar el archivo ossec.conf, colocando el webhook y en la etiqueta "level" se define un valor entre 1 y 16 (1 significa evento poco critico, 16 significa evento critico), por ejemplo si el valor es 10, significa que llegaran alertas criticas (es decir con una valor entre 10 y 16) sobre ataques y/o eventos detectados y no tanto asi alertas de rendimiento del equipo, etc.

[Documentacion Alertas](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/alerts.html#reference-ossec-alerts)

```code
foo@bar:~$ sudo vim /var/ossec/etc/ossec.conf
```
Como ejemplo "level" tiene un valor de 3 para observar todas las posibles notificaciones
```xml
<integration>
	<name>slack</name> 
	<hook_url>https://hooks.slack.com/services/T032B4KTT5W/B03EA0BLCH4/36Xn7h57btlf6Dciuxxxxxxx 
	</hook_url>
	<level>3</level>
	<alert_format>json</alert_format>
</integration>
```

Reiniciar el servicio Wazuh Manager:
```code
foo@bar:~$ sudo systemctl restart wazuh-manager
```