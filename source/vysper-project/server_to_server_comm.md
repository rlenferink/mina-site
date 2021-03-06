---
type: vysper
title: Vysper Server-to-server
---

# Server-to-server

<div class="info" markdown="1">
    Available since version 0.7.
</div>

XMPP servers like Apache Vysper work in federated networks much like SMTP servers. Users connect to their own server, and when communicating with users on a different domain, the two servers will connect to each other and exchange XMPP stanzas. Server-to-server (S2S) connections differ from client-to-server (C2) connections in that stanzas for multiple users might be sent over the same connection. S2S connections are similar to C2S connections from the point that one of the servers serves as an initiator, like a client. To exchanges stanzas in both directions between servers, two connections are established.

To set up server-to-server functionality in Vysper, two configurations are needed:

## Allow server-to-server federation

Server-to-server federation is by default disabled in Vysper. To enable sending stanzas to other servers, federations must be enabled:

```java
XMPPServer server = new XMPPServer("vysper.org");

// other initialization
server.start();

// must be done after the server has been started
server.getServerRuntimeContext().getServerFeatures().setRelayingToFederationServers(true);
```

## Server-to-server endpoint

Next, an endpoint for incoming S2S connections must be added:

```java
XMPPServer server = new XMPPServer("vysper.org");

server.addEndpoint(new S2SEndpoint());

// other initialization
server.start();

// must be done after the server has been started
server.getServerRuntimeContext().getServerFeatures().setRelayingToFederationServers(true);
```

That’s all that needs to be done. If SSL/TLS should be enabled between servers (will be negotiated during the S2S connection handshake), a keystore and keystore password must be configured:

```java
server.setTLSCertificateInfo(new File("keystore.jks"), "sekrit");
```
