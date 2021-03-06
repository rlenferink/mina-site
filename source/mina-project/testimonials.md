---
type: mina
title: Testimonials
---

[__Marko Asplund__](http://practicingtechie.wordpress.com/2012/08/06/asynchronous-event-driven-servers-with-apache-mina/) says:

> I found that Apache MINA really did fulfill its promise and implementing a high-performance, scalable and extensible network server was easy using it. MINA also helps very cleanly separate network communication and application level message processing logic. Supporting multiple different protocols in in the same server is well supported in MINA. As a downside the documentation for v2.0 is a bit lacking, but fortunately there are quite a few code samples that you can check out.

__Maarten Bosteels__ says:

> EURid used MINA during the landrush for .eu domain names on the 7th of april 2006. More than 700.000 domain names were registered during the first 4 hours. After one hour MINA had handled more than 0.5 million SSL connections.

> We found the speed and stability of MINA to be excellent. And although we are still using MINA 0.8.1, we found the API very elegant and easy.

__Fr{{< html "&eacute;" >}}d{{< html "&eacute;" >}}ric Br{{< html "&eacute;" >}}gier__ says:

> MINA helped us to get the network layout of OpenLSD done in about 2 months, saving us about 9 months to 1 year of development and fine-grained testing, so we were able to focus on our problem; Open Legacy Storage Document, a framework for document archiving in a huge storage. OpenLSD brings security, network layout, JDBC, and good performance, and allows at least 2 petabytes of documents (2000 terabytes, the limit is virtually 2^192 bytes).

> Our benchmark test went well with massive import capacity (through network with multiple processes) of 1400 documents per second, network (web) retrieves of 1000 documents per second with small latency. Network (MINA) was not the bottleneck so we were able to focus mainly on database optimization. For the web interface, our Tomcat web application connected to OpenLSD Server using pool of MINA connections almost like a JDBC pool.

> Any MINA problems were resolved very quickly either by a quick fix in MINA source code (a few) or by code fix with the support of the MINA mailing list. This was one reason of this success. Therefore we will continue to use MINA on other related project (email archiving and mimic of a professional file transfer monitor). Again, MINA is helping us to
focus on what the applications need to do and not too much on network layout.

__Alex Burmester__ says:

> We are using MINA at a telco to route low level protocol packets to a third party. We already had a SOAP and also a CORBA interface but for speed purposes we are trying out a lower level protocol and we needed a gateway of sorts to route messages between our cluster of servers and the third party's servers.

> I had been planning on using NIO and some aspects of SEDA but finding MINA was a real treat as it saved a lot of time, is well written and gets more testing than our in house QA would be able to cover. The speed and stability of our app on top of MINA has been excellent.

__Nicholas Clare__ says:

> We use MINA as a networking library to handle concurrent connections to our text based communication server. MINA has worked like a charm. It makes writing server applications simple and is much easier to use than Java's NIO libraries. Because of MINA's stability and ease of use, we plan on using MINA more in our future projects.

__Jean-Fran{{< html "&ccedil;" >}}ois Daune__ says:

> We use MINA to communicate with [Banksys](http://www.banksys.com/) 'point of sale' terminals (Visa, Mastercard...) for technical management operations. (software upgrade, remote monitoring, log transfer...)

> So far, MINA has worked really well for us. We used Netty2, and clearly saw the improvements in MINA. I like the MINA API more. MINA really makes it easier to write applications using NIO.

__Luke Hubbard__ says:

> We are using it for the network layer of [Red5](http://www.osflash.org/red5), an open source flash server. At the moment we have RTMP and AMF working and hope to add more protocols in the future. MINA's design and ease of use has helped us get a prototype up and running quickly.

__Thomas Muller__ says:

> What a fantastic API! Definitely the best I've seen since [Doug Lea's Concurrency API](http://gee.cs.oswego.edu/dl/classes/EDU/oswego/cs/dl/util/concurrent/intro.html).

__Paolo Perrucci__ says:

> We are using MINA to build the network layer of our multiplayer game server at [Leonardo.it](http://ludonet.leonardo.it/). Using MINA, we implemented different protocols in a few days; Game and HTTP tunneling. In the past, we used NIO, and the advantage of using MINA is evident; the MINA API is elegant and very simple to use. Last, but not least, MINA have a really responsive support.

__Fr{{< html "&eacute;" >}}d{{< html "&eacute;" >}}ric Soulier__ says:

> In 3 days, starting from scratch (knowing nothing about MINA) and with help from this list, I've re-implemented something that took us 2+ months to develop! I've thrown 4000 concurrent connections at it without a problem. The only problem I faced was to increase the limit for open files on my linux box (default was 1024).

__Niklas Therning__ says:

> [SpamDrain](http://www.spamdrain.net/), our online anti-spam service, has been using MINA since late 2005. So far we have developed custom proxies for the POP3, IMAP and SMTP protocols on top of the MINA API. Before MINA we used our own Java NIO abstraction layer which had some serious stability problems. With MINA we haven't experienced any stability issues and the MINA API has really helped us write cleaner code. I just love the way MINA helps you separate the decoding and encoding of protocol messages from the implementation of the protocol's state machine. Other features, like SSLFilter which gives you SSL support virtually without any effort, are also very much appreciated.

__Julien Vermillard__ says:

> I'm using MINA for supervisory control and data acquisition (SCADA) embedded application. It's used for several tasks; connecting supervision clients to the server, interaction of the server with different hardware (other SCADA systems, media stream matrix, programmable automaton, remote data acquisition systems), custom replication protocols for fail-over service. I found MINA when I started implementation using NIO and it was a great time saver. You can switch from RS232 to TCP/IP and add SSL connectivity easily. The stability and the support is really great. The code and the design are simple and efficient, so you can easily implement high quality protocol logic without bothering with all the NIO quirks. I didn't really tested the maximum performance you can get out of MINA, but all I can say is that MINA is running 24/7 with an amazing stability and I'm not afraid of using it in harsh environment.

[__Ashish Paliwal__](http://www.ashishpaliwal.com/blog) says:

> Used Apache MINA for Building a Trap Receiver

> see [White Paper](http://www.hsc.com/HSFiles/Wpos/WhitePaper_Trap_Receiver_using_Apache_MINA.pdf)

__Emmanuel L{{< html "&eacute;" >}}charny__ says:

> MINA handles the following protocols in ADS :

> * LDAP
> * DNS
> * NTP
> * DHCP
> * Kerberos
> We also use it to manage replication (using a specific protocol to communicate between two LDAP server). A LDAP client is being drafted atm, using MINA 2.0 too.
> We are using 1.1.7 currently, but a migration to 2.0 is ready (and we will switch as soon as 2.0.0-M4 will be released)

__Dan Creswell__ says:

> I've used it to build:

> 1. A framework for Paxos consensus
> 2. A remote transport for a JavaSpace
> 3. A transport for various gossip-based protocols

__Matthew Estes__ says:

> I'm using it for an asynchronous messaging framework (think RPC/RMI), which
> is mostly done, but needs documenting and clean up. I also plan to use it
> to for the HTTP part of a web framework/container. Most of my work is/will
> be Apache 2.0 licensed open source

__Kevin Williams__ says:

> We used Mina to build an internal distributed coherent cache system

__Matthew Phillips__ says:

> The Avis event notification router and client library uses MINA.

> [Avis](http://avis.sourceforge.net/)
