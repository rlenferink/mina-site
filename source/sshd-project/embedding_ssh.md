---
type: sshd
title: Embedding SSHD in 5 minutes
---

# Embedding SSHD in 5 minutes

SSHD is designed to be easily embedded in your application as an SSH server. SSH Server needs to be configured before it can be started. Essentially, there are three steps for creating the Server

* Create an instance of SshServer class
* Configure the Server
* Start the Server

Lets look at all these steps. Refer to this class for details [SshServer.java](http://svn.apache.org/viewvc/mina/sshd/trunk/sshd-core/src/main/java/org/apache/sshd/SshServer.java?view=markup)

## Creating an instance of SshServer

This is as simple as creating a new object

```java
SshServer sshd = SshServer.setUpDefaultServer();
```

It will configure the server with sensible defaults for ciphers, macs, key exchange algorithm, etc...
If you want a different behavior, you can look at the code of the setUpDefaultServer method and configure the SSH server the way you need.

## Configuring the Server

There are a few things that needs to be configured on the server before being able to actually use it:

### Port

```java
sshd.setPort(22);
```

### KeyPairProvider

```java
sshd.setKeyPairProvider(new SimpleGeneratorHostKeyProvider("hostkey.ser"));
```

It's usually a good idea to give the host key generator a path, so that if you restart the SSHD server, the same key will be used to authenticate the server.

### ShellFactory

That's the part you will usually have to write to customize the SSHD server. The shell factory will be used to create a new shell each time a user logs in. SSHD provides a single implementation that you can use if you want. This implementation will create a process and delegate everything to it, so it's mostly useful to launch the OS native shell.

```java
sshd.setShellFactory(new ProcessShellFactory(new String[] { "/bin/sh", "-i", "-l" }));
```

Note that the ShellFactory is not required. If none is configured, any request for a shell will be denied to users.

### CommandFactory

The CommandFactory can be used in addition to the ShellFactory (it can also be used instead of the ShellFactory). The CommandFactory is used when direct commands are sent to the SSH server, as this is the case when running __ssh localhost shutdown__ or __scp xxx__

SSHD provides a CommandFactory to support SCP that can be configure in the following way:

```java
sshd.setCommandFactory(new ScpCommandFactory());
```

You can also use the ScpCommandFactory on top of your own CommandFactory:

```java
sshd.setCommandFactory(new ScpCommandFactory(myCommandFactory));
```

Note that the CommandFactory is optional. If none is configured, any direct command sent by users will be rejected.

## Start the Server

Once we have configured the Server, we need to call start(), to start the Server

```java
sshd.start();
```
