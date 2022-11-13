##### Detectar Log4shell Attack

[Documentacion Detectando Log4shell Attack](https://wazuh.com/blog/detecting-log4shell-with-wazuh/)

Como ya se tiene captura de los logs del Servidor Apache, utilizaremos el mismo para probar la deteccion del ataque Log4shell.

(Si desea puede instalar un Servidor Tomcat para realizar la prueba de deteccion. [Como instalar un Servidor Tomcat en Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-10-on-ubuntu-20-04))

El ataque Log4shell viene por una vulnerabilidad en la libreria de logging de Java (CVE-2021-44228), para explotar la vulnerabilidad se debe crear un request (consulta web) especifico con los valores jndi, ldap, etc., de manera que es posible definir una Regla (Rule) que mediante expresiones regulares detecte estos valores en una request.

Realizamos la configuracion de la Regla (Rule) en el archivo /var/ossec/etc/rules/local_rules.xml del Wazuh Manager:
[Documentacion Personalizar Rules y Decoders](https://documentation.wazuh.com/current/user-manual/ruleset/custom.html)

El "id" de la Rule debe ser un valor entre 100000 y 120000 (de preferencia incremental a medida que se incluya Rules personalizadas) (ejemplo 110002)

El "level" de la Rule corresponde a un valor entre 1 y 16 que defina la criticidad del evento (ejemplo 7)
[Documentacion Alertas](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/alerts.html#reference-ossec-alerts)

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

Reiniciar el Wazuh Manager
```code
foo@bar:~$ systemctl restart wazuh-manager
```

Solicitud Web Personalizada para probar la deteccion del ataque Log4shell (en un Navegador web)
```code
http://the_system_ip_address/?x=${jndi:ldap://${localhost}.{{test}}/a}
```
Los eventos detectados seran visibles en el Dashboard


##### Detectar el Ataque Shellshock

[Documentacion detectando Shellshock Attack](https://documentation.wazuh.com/current/learning-wazuh/shellshock.html)

Como ya se tiene captura de los logs del Servidor Apache, utilizaremos el mismo para probar la deteccion del ataque Shellshock.

El ataque Shellshock, permite injectar comandos maliciosos en un request personalizado que es enviado a un servidor web en Linux, de manera que es posible definir una Regla (Rule) que mediante expresiones regulares detecte caracteres, palabras, etc. en un request.

La deteccion de Shellshock esta definida predeterminadamente en la siguiente Rule:

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

Request en una terminal para probar el Ataque Shellshock
```code
foo@bar:~$ curl -H "User-Agent: () { :; }; /bin/cat /etc/passwd" your_system_ip_address
```

Los eventos detectados seran visibles en el Dashboard


##### Detectar el Ataque SQL Injection

[Que es SQL Injection - Portswigger](https://portswigger.net/web-security/sql-injection)

[Regla de Deteccion de Ataques Web](https://github.com/wazuh/wazuh-ruleset/blob/master/rules/0245-web_rules.xml)

Como ya se tiene captura de los logs del Servidor Apache, utilizaremos el mismo para probar la deteccion del ataque SQL Injection.

El ataque SQL Injection, permite obtener informacion de una base de datos a traves de consultas (query) personalizadas, de manera que es posible definir una Regla (Rule) que mediante expresiones regulares detecte caracteres, palabras, etc. en un request.


La deteccion de SQL Injection esta definida predeterminadamente en la siguiente Rule:

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

Request en un Navegador para probar el ataque SQL Injection
```code
http://the_system_ip_address/x=select * from users
http://the_system_ip_address/x=select+*+from+users
```


Los eventos detectados seran visibles en el Dashboard



##### Detectar el Ataque XSS Cross-site scripting

[Que es XSS Cross-site scripting - Portswigger](https://portswigger.net/web-security/cross-site-scripting)

[Regla de Deteccion de Ataques Web](https://github.com/wazuh/wazuh-ruleset/blob/master/rules/0245-web_rules.xml)


Como ya se tiene captura de los logs del Servidor Apache, utilizaremos el mismo para probar la deteccion del ataque XSS Cross-site scripting.

El ataque XSS Cross-site scripting, permite a un ciberdelincuente lograr que los usuarios realicen acciones sin que ellos se percaten a traves de request modificadas a las que los usuarios acceden a traves de links acortados, etc., de manera que es posible definir una Regla (Rule) que mediante expresiones regulares detecte caracteres, palabras, etc. en una request.


La deteccion de XSS Cross-site scripting esta definida predeterminadamente en la siguiente Rule:

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

Request en un Navegador para probar el ataque XSS Cross-site scripting
```code
http://the_system_ip_address/x<script>alert(5)</script>
```


Los eventos detectados seran visibles en el Dashboard