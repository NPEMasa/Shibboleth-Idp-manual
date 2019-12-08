# Shibboleth-Idp-manual

## ToC

- [OpenJDK 11 install](#OpenJDK-11-install)
- [Jetty 9.4 install](#Jetty-9.4-install)
- [Shibboleth Idp install](#Shibboleth-Idp-install)
- [Apache Web Server install](#Apache-Web-Server-install)
- [Other Software Configuration](#Other-Software-Configuration)


# OpenJDK 11 install


`# yum install java-11-openjdk`
```
# alternatives --config java

1 プログラムがあり 'java' を提供します。

  選択       コマンド
-----------------------------------------------
*+ 1           java-11-openjdk.x86_64 (/usr/lib/jvm/java-11-openjdk-11.0.5.10-0.el7_7.x86_64/bin/java)

Enter を押して現在の選択 [+] を保持するか、選択番号を入力します:1
```
copy path.

`/usr/lib/jvm/java-11-openjdk-11.0.5.10-0.el7_7.x86_64`

execute this command.

```
# echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.5.10-0.el7_7.x86_64" >> /etc/profile
# source /etc/profile
```


# Jetty 9.4 install


Jetty Download from eclipse site.


[Eclipse-Jetty](https://www.eclipse.org/jetty/download.html)

```shell
# useradd -s /sbin/nologin jetty
# cd /opt/
# wget -O jetty.tar.gz https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.4.24.v20191120/jetty-distribution-9.4.24.v20191120.tar.gz
# tar zxvf jetty.tar.gz
# mv jetty-distribution-9.4.24.v20191120 jetty-dist
# cd /opt/jetty-dist/
# cp bin/jetty.sh /etc/init.d/jetty
# echo JETTY_HOME=`pwd` > /etc/default/jetty
# source /etc/default/jetty
# service jetty start
```

[Details Link](https://www.eclipse.org/jetty/documentation/current/startup-unix-service.html)

modify start.ini
```shell
# cp start.ini start.ini.org
# vim ./start.ini
```

```shell:/opt/jetty-dist/start.ini
#===========================================================
# Jetty Startup
#
# Starting Jetty from this {jetty.home} is not recommended.
#
# A proper {jetty.base} directory should be configured, instead
# of making changes to this {jetty.home} directory.
#
# See documentation about {jetty.base} at
# http://www.eclipse.org/jetty/documentation/current/startup.html
#
# A demo-base directory has been provided as an example of
# this sort of setup.
#
# $ cd demo-base
# $ java -jar ../start.jar
#
#===========================================================

# To disable the warning message, comment the following line
--module=home-base-warning

# ---------------------------------------
# Module: ext
--module=ext

# ---------------------------------------
# Module: server
--module=server

### ThreadPool configuration
## Minimum number of threads
jetty.threadPool.minThreads=10

## Maximum number of threads
jetty.threadPool.maxThreads=200

## Thread idle timeout (in milliseconds)
jetty.threadPool.idleTimeout=60000

## Response content buffer size (in bytes)
jetty.httpConfig.outputBufferSize=32768

## Max request headers size (in bytes)
jetty.httpConfig.requestHeaderSize=8192

## Max response headers size (in bytes)
jetty.httpConfig.responseHeaderSize=8192

## Whether to send the Server: header
jetty.httpConfig.sendServerVersion=true

## Whether to send the Date: header
jetty.httpConfig.sendDateHeader=false

## Dump the state of the Jetty server, components, and webapps after startup
jetty.server.dumpAfterStart=false

## Dump the state of the Jetty server, components, and webapps before shutdown
jetty.server.dumpBeforeStop=false

# ---------------------------------------
# Module: jsp
--module=jsp

# ---------------------------------------
# Module: resources
--module=resources

# ---------------------------------------
# Module: deploy
--module=deploy

# ---------------------------------------
# Module: jstl
--module=jstl

# ---------------------------------------
# Module: websocket
--module=websocket

# ---------------------------------------
# Module: http
--module=http

### HTTP Connector Configuration

## Connector host/address to bind to
jetty.http.host=localhost

## Connector port to listen on
jetty.http.port=8080

## Connector idle timeout in milliseconds
jetty.http.idleTimeout=30000

# ---------------------------------------
# Module: annotations
--module=annotations

# Module: console-capture
--module=console-capture

jetty.console-capture.dir=/var/log/jetty

# ---------------------------------------
# Module: requestlog
--module=requestlog

# ---------------------------------------
# Module: servlets
--module=servlets

# ---------------------------------------
# Module: plus
--module=plus

# ---------------------------------------
# Mwdule: http-forwarded
--module=http-forwarded

# Allows setting Java system properties (-Dname=value)
# and JVM flags (-X, -XX) in this file
# NOTE: spawns child Java process
--exec

# Set the IdP home dir
-Didp.home=/opt/shibboleth-idp

# Newer garbage collector that reduces memory needed for larger metadata files
-XX:+UseG1GC

# Maximum amount of memory that Jetty may use, at least 1.5G is recommended
# for handling larger (> 25M) metadata files but you will need to test on
# your particular metadata configuration
-Xmx2000m

# Prevent blocking for entropy.
-Djava.security.egd=file:/dev/urandom

# Set Java tmp location
-Djava.io.tmpdir=tmp

# Java contains a lot of classes which assume that there is a some sort of display and a keyboard attached.
# A code that runs on a server does not have them and this is called Headless mode.
-Djava.awt.headless=true
```

Copy start program.

```shell
# cp bin/jetty.sh /etc/init.d/jetty

```

# Shibboleth Idp Install

Download Shibboleth Idp Package from [shibboleth.net](https://shibboleth.net/downloads/identity-provider/)
```shell
# cd /opt/
# wget -O shibboleth-idp.tar.gz https://shibboleth.net/downloads/identity-provider/3.4.6/shibboleth-identity-provider-3.4.6.tar.gz
# mkdir shibboleth && tar zxvf shibboleth-idp.tar.gz -C shibboleth --strip-components 1
# cd shibboleth/
# cd webapp/WEB-INF/lib
# wget https://build.shibboleth.net/nexus/service/local/repositories/thirdparty/content/javax/servlet/jstl/1.2/jstl-1.2.jar
# cd ../../../
```

```shell
# bin/install.sh
Source (Distribution) Directory (press <enter> to accept default): [/opt/shibboleth]

Installation Directory: [/opt/shibboleth-idp]

Hostname: [localhost.localdomain]
idp.example.com
SAML EntityID: [https://idp.example.com/idp/shibboleth]

Attribute Scope: [localdomain]
example.com
Backchannel PKCS12 Password:
Re-enter password:
Cookie Encryption Key Password:
Re-enter password:
Warning: /opt/shibboleth-idp/bin does not exist.
Warning: /opt/shibboleth-idp/edit-webapp does not exist.
Warning: /opt/shibboleth-idp/dist does not exist.
Warning: /opt/shibboleth-idp/doc does not exist.
Warning: /opt/shibboleth-idp/system does not exist.
Generating Signing Key, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth ...
...done
Creating Encryption Key, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth ...
...done
Creating Backchannel keystore, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth ...
...done
Creating cookie encryption key files...
...done
Rebuilding /opt/shibboleth-idp/war/idp.war ...
...done

BUILD SUCCESSFUL
Total time: 40 seconds
```

```shell
# cd ../shibboleth-idp/
# chown -R jetty logs/ metadata/ credentials/ conf/ system/ war/

```


# Apache Web Server install

`# yum install -y httpd`

```shell
# mkdir /var/www/html/idp.exmaple.com
# chown -R apache: /var/www/html/idp.exmaple.com

```

```shell
# vim /etc/httpd/conf.d/idp.example.org.conf
<VirtualHost <SERVER-IP-ADDRESS>:80>
  ServerName idp.example.org
  ServerAlias idp.example.org

  RedirectMatch ^/(.*)$ https://idp.example.org/$1
</VirtualHost>
<VirtualHost 127.0.0.1:80>
  ProxyPass /idp http://localhost:8080/idp retry=5
  ProxyPassReverse /idp http://localhost:8080/idp retry=5
  <Location /idp>
    Require all granted
  </Location>
</VirtualHost>
```

```# systemctl restart httpd```


# Other Software Configuration

```shell
# cd /opt/jetty-dist/
# vim webapps/idp.xml
<Configure class="org.eclipse.jetty.webapp.WebAppContext">
  <Set name="war"><SystemProperty name="idp.home"/>/war/idp.war</Set>
  <Set name="contextPath">/idp</Set>
  <Set name="extractWAR">false</Set>
  <Set name="copyWebDir">false</Set>
  <Set name="copyWebInf">true</Set>
  <Set name="persistTempDirectory">false</Set>
</Configure>
```



[github link](https://github.com/ConsortiumGARR/idem-tutorials/blob/master/idem-fedops/HOWTO-Shibboleth/Identity%20Provider/CentOS/HOWTO%20Install%20and%20Configure%20a%20Shibboleth%20IdP%20v3.4.x%20on%20CentOS%207%20with%20Apache2%20%2B%20Jetty9.md)
