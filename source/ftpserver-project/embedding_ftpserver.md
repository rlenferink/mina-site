---
type: ftpserver
title: Embedding FtpServer in 5 minutes
---

# Embedding FtpServer in 5 minutes

FtpServer is designed to be easily embedded into your application. Getting a basic server up and running is as simple as

```java
FtpServerFactory serverFactory = new FtpServerFactory();
FtpServer server = serverFactory.createServer();

// start the server
server.start();
```

To get this running, you need the following JAR files in your classpath:

* mina-core, 2.0-M3 or later
* slf4j-api
* A SLF4J implementation of your choice, for example slf4j-simple-1.5.3.jar
* ftplet-api
* ftpserver-core

Now, you will probably like to configure the server for your specific needs. For example, you might want to run on a non-privileged port to get around running as a root on Linux/Unix. To do that you need to configure a listener. Listeners are the part of FtpServer where network management is done. By default, a listener named "default" is created but you can add as many listeners as you like, for example to provide one for use outside of your firewall and one on the inside.

Now, let's configure the port on which the default listener waits for connections.

```java
FtpServerFactory serverFactory = new FtpServerFactory();
ListenerFactory factory = new ListenerFactory();

// set the port of the listener
factory.setPort(2221);

// replace the default listener
serverFactory.addListener("default", factory.createListener());

// start the server
FtpServer server = serverFactory.createServer();         
server.start();
```

Now, let's make it possible for a client to use FTPS (FTP over SSL) for the default listener.

```java
FtpServerFactory serverFactory = new FtpServerFactory();
ListenerFactory factory = new ListenerFactory();

// set the port of the listener
factory.setPort(2221);

// define SSL configuration
SslConfigurationFactory ssl = new SslConfigurationFactory();
ssl.setKeystoreFile(new File("src/test/resources/ftpserver.jks"));
ssl.setKeystorePassword("password");

// set the SSL configuration for the listener
factory.setSslConfiguration(ssl.createSslConfiguration());
factory.setImplicitSsl(true);

// replace the default listener
serverFactory.addListener("default", factory.createListener());
PropertiesUserManagerFactory userManagerFactory = new PropertiesUserManagerFactory();
userManagerFactory.setFile(new File("myusers.properties"));
serverFactory.setUserManager(userManagerFactory.createUserManager());

// start the server
FtpServer server = serverFactory.createServer(); 
server.start();
```

There you have it, that's the basics that you usually need. For more advanced features, have a look at our configuration documentation.
