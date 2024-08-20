---
layout: post
title:  "Installing OpenVPN Server"
date:   2015-10-04 16:45:20 +0100
tag: [sysadmin]
image: "/installing-openvpn-server/image.webp"
---

Today I’ve been installing **OpenVPN server** on my home server to being able to securely connect to my local network from anywhere. This can allow you to:

- Having access to local services on your local network , like a (in my case) a [Time Machine docker][time-machine] service.
- Access services only available from your country IP when travelling to other places.
- Being able to use open hotspots while encrypting your whole network traffic.

In order to get it, my first thought was to install the server on the bare metal machine (not on a _Docker_ container) because it may require some network privileges that can make it unable to work behind the _Docker_ network layer.

But searching a bit over the Internet I’ve found [this repository][repo] that includes a functional Docker image. It doesn't just works, but it includes a full documentation file with instructions to set the server up in two different containers (the one that has the daemon running and the one that stores the secure data).

**So, in less that 5 minutes my OpenVPN server was up and running!** Also I have generated the CA file and the client certificates, so I can connect from anywhere like being at home.

## Clients

In Mac OSX (10.11 — El Capitán) I installed [Tunnelblick][tunnelblick], a free and open source alternative to OpenVPN client or Viscoso.app. It has an easy and lightweight GUI, perfect to avoid resource consumption and annoyance while using it.

In Windows 10 I used the official OpenVPN client, but it doesn’t work _out-of-the-box_. To make it work, **its executables should be marked to be run with administrator privileges**, so they can add the networking routes needed to route the traffic through the VPN.

[time-machine]: https://github.com/odarriba/docker-timemachine
[repo]: https://github.com/kylemanna/docker-openvpn
[tunnelblick]: https://tunnelblick.net/
