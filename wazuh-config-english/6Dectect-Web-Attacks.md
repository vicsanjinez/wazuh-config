##### Detect Log4shell Attack

[Detecting Log4shell Attack Documentation](https://wazuh.com/blog/detecting-log4shell-with-wazuh/)

As we already have a capture of the Apache Server logs, we will use it to test the detection of the Log4shell attack.

(If you want you can install a Tomcat Server to perform the detection test. [How to install Tomcat on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-10-on-ubuntu-20-04))

The Log4shell attack comes from a vulnerability in the Java logging library (CVE-2021-44228), to exploit the vulnerability a specific request must be created with the jndi, ldap, etc. values, so, it's possible to define a Rule that through regular expressions detects these values in a request.

Create the Rule in /var/ossec/etc/rules/local_rules.xml file on Wazuh Manager:
[Custom Rules and Decoders Documentation](https://documentation.wazuh.com/current/user-manual/ruleset/custom.html)

The "id" of Rule must be a value between 100000 and 120000 (preferably incremental as custom Rules are included) (for example, 110002)

The "level" of Rule must be a value between 1 and 16 to define the criticality of the event (for example, 7)
[Alert Documentation](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/alerts.html#reference-ossec-alerts)

```code
foo@bar:~$ sudo vim /var/ossec/etc/rules/local_rules.xml
```

```xml
<group name="log4j, attack,">
<rule id="110002" level="7">
<if_group>web|accesslog|attack</if_group>
<regex type="pcre2">(?i)(((\$|24)\S*)((\{|7B)\S*)((\S*j\S*n\S*d\S*i))|JHtqbmRp)</regex>
<description>Possible Log4j RCE attack attempt detected.</description>
<mitre>
<id>T1190</id>
<id>T1210</id>
<id>T1211</id>
</mitre>
</rule>
<rule id="110003" level="12">
<if_sid>110002</if_sid>
<regex type="pcre2">ldap[s]?|rmi|dns|nis|iiop|corba|nds|http|lower|upper|(\$\{\S*\w\}\S*)+</regex>
<description>Log4j RCE attack attempt detected.</description>
<mitre>
<id>T1190</id>
<id>T1210</id>
<id>T1211</id>
</mitre>
</rule>
</group>
```

Restart Wazuh Manager
```code
foo@bar:~$ systemctl restart wazuh-manager
```

Request for test Log4shell Attack (in a Browser)
```code
http://the_system_ip_address/?x=${jndi:ldap://${localhost}.{{test}}/a}
```
The detected events will be visible in the Dashboard


##### Detect Shellshock Attack

[Detect Shellshock Attack Documentation](https://documentation.wazuh.com/current/learning-wazuh/shellshock.html)

As we already have a capture of the Apache Server logs, we will use it to test the detection of the Shellshock attack.

The Shellshock attack allow inject malicious commands in a custom request that is sent to a Linux web server, so, it's possible to define a Rule that detects characters, words, etc. through regular expressions in a request.

Shellshock detection is set by default, in the following Rule:

```xml
<rule id="31168" level="15">
  <if_sid>31108</if_sid>
  <regex>"\(\)\s*{\s*\w*:;\s*}\s*;|"\(\)\s*{\s*\w*;\s*}\s*;</regex>
  <description>Shellshock attack detected</description>
  <mitre>
    <id>T1068</id>
    <id>T1190</id>
  </mitre>
  <info type="cve">CVE-2014-6271</info>
  <info type="link">https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271</info>
  <group>attack,pci_dss_11.4,gdpr_IV_35.7.d,nist_800_53_SI.4,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
</rule>
```

On a terminal, execute the request to test the Shellshock Attack
```code
foo@bar:~$ curl -H "User-Agent: () { :; }; /bin/cat /etc/passwd" your_system_ip_address
```

The detected events will be visible in the Dashboard


##### Detect SQL Injection Attack

[What is SQL Injection - Portswigger](https://portswigger.net/web-security/sql-injection)

[Rule for detect Web Attacks](https://github.com/wazuh/wazuh-ruleset/blob/master/rules/0245-web_rules.xml)

As we already have a capture of the Apache Server logs, we will use it to test the detection of the SQL Injection attack.

The SQL Injection attack allows get information from a database through personalized queries, so, it's possible to define a Rule that detects characters, words, etc. through regular expressions in a request.


SQL Injection detection is set by default, in the following Rule:

```xml
  <rule id="31103" level="7">
    <if_sid>31100,31108</if_sid>
    <url>=select%20|select+|insert%20|%20from%20|%20where%20|union%20|</url>
    <url>union+|where+|null,null|xp_cmdshell</url>
    <description>SQL injection attempt.</description>
    <mitre>
      <id>T1190</id>
    </mitre>
    <group>attack,sql_injection,pci_dss_6.5,pci_dss_11.4,pci_dss_6.5.1,gdpr_IV_35.7.d,nist_800_53_SA.11,nist_800_53_SI.4,tsc_CC6.6,tsc_CC7.1,tsc_CC8.1,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>
```

Request for test SQL Injection Attack (in a Browser)
```code
http://the_system_ip_address/x=select * from users
http://the_system_ip_address/x=select+*+from+users
```

The detected events will be visible in the Dashboard


##### Detect XSS Cross-site scripting Attack

[What is XSS Cross-site scripting - Portswigger](https://portswigger.net/web-security/cross-site-scripting)

[Rule for detect Web Attacks](https://github.com/wazuh/wazuh-ruleset/blob/master/rules/0245-web_rules.xml)


As we already have a capture of the Apache Server logs, we will use it to test the detection of the XSS Cross-site scripting attack.

XSS Cross-site scripting attack allows a cybercriminal to get users to perform actions without them noticing through modified requests that users access through shorted links, etc., so, it's possible to define a Rule that through regular expressions detects characters, words, etc. in a request.


XSS Cross-site scripting is set by default, in the following Rule:

```xml
  <rule id="31105" level="6">
    <if_sid>31100</if_sid>
    <url>%3Cscript|%3C%2Fscript|script>|script%3E|SRC=javascript|IMG%20|</url>
    <url>%20ONLOAD=|INPUT%20|iframe%20</url>
    <description>XSS (Cross Site Scripting) attempt.</description>
    <mitre>
      <id>T1064</id>
    </mitre>
    <group>attack,pci_dss_6.5,pci_dss_11.4,pci_dss_6.5.7,gdpr_IV_35.7.d,nist_800_53_SA.11,nist_800_53_SI.4,tsc_CC6.6,tsc_CC7.1,tsc_CC8.1,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>
```

Request for test XSS Cross-site scripting Attack (in a Browser)
```code
http://the_system_ip_address/x<script>alert(5)</script>
```

The detected events will be visible in the Dashboard