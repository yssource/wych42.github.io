+++
title = "socat"
date = 2019-12-20T18:07:00+08:00
lastmod = 2020-05-17T17:24:13+08:00
tags = ["socat"]
categories = ["Verse"]
draft = false
toc = true
+++

snippets:

-   simple udp echo server: echo payload with client ip:port

```bash
socat -d -d udp-recvfrom:1926,fork system:"echo \"\$SOCAT_PEERADDR:\$SOCAT_PEERPORT\"; cat"

client > echo "0817" | socat - udp-sendto:server-ip:1926
```

-   simple tcp echo server: echo payload with client ip:port

```bash
socat -d -d tcp-l:1926,fork system:"echo \"\$SOCAT_PEERADDR:\$SOCAT_PEERPORT\"; cat"
client > echo "0817" | socat  tcp:server-ip:1926 -
```