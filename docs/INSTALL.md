XBDD
====

Pre-requisites
--------------

* MongoDB 4. See https://www.mongodb.com/download-center/v2/community
* Tomcat 9. See http://tomcat.apache.org/download-90.cgi
* Maven 3. See http://maven.apache.org/download.cgi

Optional requirements
---------------------

A driver for running the automated tests is required. By default, the automated tests run against the Firefox Web Driver.
See https://github.com/mozilla/geckodriver for details.

This can be overridden with the `selenium-profile` property and the CI server runs with `phantom-js`.
Other supported values are `selenium-grid` and `chrome`. These have their own requirements.
See [XbddDriver.java](https://github.com/orionhealth/XBDD/blob/master/src/test/java/xbdd/XbddDriver.java) for more details.

Configuration
-------------

In the instructions that follow, `$CATALINE_BASE` refers to the Tomcat installation directory.

### SSL/TLS
The XBDD application requires a secure connection. This can be achieved through the Tomcat SSL connector.

You must first have configured a keystore. You can [create one](http://java.dzone.com/articles/setting-ssl-tomcat-5-minutes) or skip ahead if you have an existing one.

Open `$CATALINA_BASE/conf/server.xml` and uncomment the 8443 connector block. Add the `keystoreFile` and `keystorePass` attributes, e.g.:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol" sslImplementationName="org.apache.tomcat.util.net.jsse.JSSEImplementation"
    maxThreads="150" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="FILE_LOCATION" certificateKeystorePass="PASSWORD_HERE" type="RSA"/>
    </SSLHostConfig>
</Connector>
```

Replace `FILE_LOCATION` with the location of your security certificate e.g. `${user.home}/.keystore` and `PASSWORD_HERE` with the password.

### Authentication

#### Local Authentication
To get started quickly without configuring enterprise authentication, it is possible to use Tomcat's default local UserDatabaseRealm with XBDD.

Configure a user by editing `$CATALINA_BASE/conf/tomcat-users.xml` and adding a `user` element within the `tomcat-users` element, e.g.:

```xml
<user username="xbdd" password="xbdd"/>
```

#### LDAP
If you want to use LDAP, configure the realm in `$CATALINA_BASE/conf/server.xml` or `$CATALINA_BASE/conf/context.xml` with the JNDIRealm. See the [documentation](https://tomcat.apache.org/tomcat-7.0-doc/config/realm.html#JNDI_Directory_Realm_-_org.apache.catalina.realm.JNDIRealm) for details on the required fields.

For example:
```	xml
<Realm className="org.apache.catalina.realm.LockOutRealm">
    <Realm allRolesMode="authOnly" className="org.apache.catalina.realm.JNDIRealm"
    connectionName="USERNAME" connectionPassword="PASSWORD" connectionURL="ldap://LDAP_HOST:389"
    referrals="follow" roleBase="ou=Groups,ou=Employees,dc=Example,dc=Internal" roleName="cn"
    roleSearch="(member={0})" roleSubtree="true" userBase="ou=Employees,dc=Example,dc=Internal"
    userRoleName="memberOf" userSearch="(sAMAccountName={0})" userSubtree="true"/>
</Realm>
```

### Configure Mongo Server Connection

By default XBDD will connect to MongoDB at its default address of `localhost:27017`.

To configure an alternative server or to add authentication, add the following parameters to `$CATALINA_BASE/conf/context.xml`

```xml
    <Parameter name="xbdd.mongo.hostname" value="<hostname>"/>
    <Parameter name="xbdd.mongo.port" value="<port>"/>
    <Parameter name="xbdd.mongo.username" value="<username>"/>
    <Parameter name="xbdd.mongo.password" value="<password>"/>
```

### A word on securing the connection to MongoDB
MongoDB provides user access on a per-DB basis. XBDD uses two databases, `bdd` and `grid`. The user needs read/write permissions for both.

### Running Mongo in a docker container.
1. Install docker on your system. You can download it [here](https://docs.docker.com/engine/installation/).
2. Pull the docker container in using `docker pull mongo`
3. Start the docker container with the command
`docker run -p=27017:27017 --name mongo -d mongo`

This will give you a docker container named mongo which is accessible at
[localhost:27017](http://localhost:27017)

Installation
============

XBDD can be run as a standalone webapp in Tomcat (recommended) or via Eclipse.
It can also be run with an embedded Tomcat instance however the above configuration will not be applied if using this mode. This may be useful for development purposes though.

### Standalone mode

1. From the top level directory run `mvn clean package`.
2. Copy `target/xbdd.war` into `$CATALINA_BASE/webapps`.
3. Start Tomcat with `$CATALINA_BASE/bin/startup.sh`.
4. Open https://localhost:8443/xbdd

### Embedded mode

1. From the top level directory (or within an IDE) run `mvn tomcat7:run`
2. Open <https://localhost:28443/xbdd>

Running the tests
=================

You need to have the Gecko Web Driver installed and the system property `webdriver.gecko.driver=/path/to/your/gecko/webdriver` set, in order for the tests to pass. The easiest way to do this is via your `~/.m2/settings.xml` file. Add the following snippet to an active profile:

```
<properties>
    <webdriver.gecko.driver>/usr/local/Cellar/geckodriver/0.18.0</webdriver.gecko.driver>
</properties>

```

If you don't have an active profile, or a settings.xml file, add this:
```
<?xml version="1.0" encoding="UTF-8"?>
<settings>
	<profiles>
		<profile>
			<id>global</id>
			<properties>
   				<webdriver.gecko.driver>/usr/local/Cellar/geckodriver/0.18.0</webdriver.gecko.driver>
   			</properties>
   		</profile>
	</profiles>
	<activeProfiles>
		<activeProfile>global</activeProfile>
	</activeProfiles>
</settings>
```
