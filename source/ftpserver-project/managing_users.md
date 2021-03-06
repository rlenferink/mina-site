---
type: ftpserver
title: FtpServer Managing users
---

# Managing users

Since FtpServer user manager might encrypt user passwords before saving them, manually administering the user database can be hard. There is basically three ways:

## Manually

Using either the PropertiesUserManager or DbUserManager you can access the data store (file or database) directly to edit users. For clear text passwords you can simply edit them. For MD5 hashed passwords, you can you use any of the available MD5 tools, for example <http://www.iwebtool.com/md5>, to hash the password before editing it. For salted passwords, use one of the methods described below.

## Using the API

Using the FtpServer API, you can create a user manager, configure it as your real user manager and use that to edit users. This is a simple example:

```java
PropertiesUserManagerFactory userManagerFactory = new PropertiesUserManagerFactory();
userManagerFactory.setFile(new File("myusers.properties"));
userManagerFactory.setPasswordEncryptor(new SaltedPasswordEncryptor());
UserManager um = userManagerFactory.createUserManager();
BaseUser user = new BaseUser();
user.setName("myNewUser");
user.setPassword("secret");
user.setHomeDirectory("ftproot");
um.save(user);
```

## Using command line tool

If you're using the XML configuration, there is a command line tool available for adding new users to your user manager.

In the examples below, make sure you update the versions to reflect the correct versions for your release

Windows:

```bash
java -cp ftpserver-core-1.0.0-M4.jar;ftplet-api-1.0.0-M4.jar;mina-core-2.0.0-M3.jar; 
    [slf4j-api-1.5.2.jar;<br>slf4j-simple-1.5.2.jar 
    [ org.apache.ftpserver.main.AddUser path/to/your/config.xml
```

MacOS/Linux/Unix

```bash
java -cp ftpserver-core-1.0.0-M4.jar:ftplet-api-1.0.0-M4.jar:mina-core-2.0.0-M3.jar:\
    slf4j-api-1.5.2.jar:<br>slf4j-simple-1.5.2.jar \
    org.apache.ftpserver.main.AddUser path/to/your/config.xml
```

The program will ask you for the required data.
