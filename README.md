# fluentd-jboss-eap-tail-example

This was tested using `td-agent` - Implementation should be identical for fluentd, besides different configuration files.

This setup tails JBoss Java logs and forwards to configured ES. For example:

Given the config:


```
<source>
  @type tail
  path /opt/rh/eap7/root/usr/share/wildfly/domain/servers/server-one/log/server.log
  tag my.node.eap.logs
  <parse>
    @type multiline
    format_firstline /\d{4}-\d{1,2}-\d{1,2}/
    format1 /^(?<time>\d{4}-\d{1,2}-\d{1,2} \d{1,2}:\d{1,2}:\d{1,2},\d{1,3})\s*(?<level>[a-zA-Z]*)\s*\[(?<thread>.+?)\] (?<message>.*)/
  </parse>
</source>
```


And the logs:

```
2019-08-06 10:18:27,742 INFO  [org.jboss.as] (MSC service thread 1-1) WFLYSRV0000: JBoss EAP 7.1.6.GA (WildFly Core 3.0.21.Final-redhat-00001) starting
2019-08-06 10:18:27,743 DEBUG [org.jboss.as.config] (MSC service thread 1-1) Configured system properties:
    file.encoding = UTF-8
    file.encoding.pkg = sun.io
    file.separator = /
    java.awt.graphicsenv = sun.awt.X11GraphicsEnvironment
    java.awt.headless = true
    java.awt.printerjob = sun.print.PSPrinterJob
    java.class.version = 52.0
    java.io.tmpdir = /tmp
    [etc]
2019-08-06 [..etc]
```

We produce the JSON:


```
  "_source": {
    "level": "INFO",
    "thread": "org.jboss.as",
    "message": "(MSC service thread 1-1) WFLYSRV0000: JBoss EAP 7.1.6.GA (WildFly Core 3.0.21.Final-redhat-00001) starting",
    "@timestamp": "2019-08-06T10:18:27.000000000-04:00",
    "@log_name": "my.node.eap.logs"
  }
 ```
and
```
  "_source": {
    "level": "DEBUG",
    "thread": "org.jboss.as.config",
    "message": "This\n is really\n huge\n redacted\n [etc] \n",
    "@timestamp": "2019-08-06T10:18:27.000000000-04:00",
    "@log_name": "my.node.eap.logs"
  }
```
