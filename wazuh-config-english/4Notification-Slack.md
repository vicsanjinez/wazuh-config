##### Notifications in Slack

[Integration with Slack Documentation ](https://documentation.wazuh.com/current/proof-of-concept-guide/poc-integrate-slack.html)

For the integration with Slack, it is necessary to create a "Slack App" and a Webhook, this is possible to enter the following link (before entering the link you must create a Slack account and be authenticated)

```code
https://api.slack.com/messaging/webhooks
```

Enable the "Incoming Webhooks"
```code
https://api.slack.com/apps/A04ANEZV5D1/incoming-webhooks?
```

Click on the button "Add New Webhooks to the Workspace" 
Then choose the channel through which the messages will go (choose the newly created channel)

Once you have the webhook, for example:
```code
https://hooks.slack.com/services/T032B4KTT5W/B03EA0BLCH4/36Xn7h57btlf6Dciuxxxxxxx
```

Config the Wazuh Manager:

Edit the ossec.conf file, set the webhook, in the "level" tag, set a value between 1 and 16 (1 means non-critical event, 16 means critical event), for example if the value is 10, it means that alerts will be critical (that is, with a value between 10 and 16) and not will alerts about performance, etc.

[Alert Documentation](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/alerts.html#reference-ossec-alerts)

```code
foo@bar:~$ sudo vim /var/ossec/etc/ossec.conf
```

As an example "level" has a value of 3 to observe all possible notifications
```xml
<integration>
	<name>slack</name> 
	<hook_url>https://hooks.slack.com/services/T032B4KTT5W/B03EA0BLCH4/36Xn7h57btlf6Dciuxxxxxxx 
	</hook_url>
	<level>3</level>
	<alert_format>json</alert_format>
</integration>
```

Restart the Wazuh Manager:
```code
foo@bar:~$ sudo systemctl restart wazuh-manager
```