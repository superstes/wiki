.. _net_gost:

.. include:: ../../_inc/head.rst


*****************
Proxy Tool - GOST
*****************

.. warning::

  This tool can be used to hide/forward malicious network traffic.

  That can be illegal => you are warned.

----

Introduction
############

`GOST <https://latest.gost.run/en/>`_ is a tool for proxying pretty much anything and anyhow you want/need to.

Check out the `nice documentation <https://latest.gost.run/en/tutorials>`_!

It can act as proxy server and client/forwarder.

If you need to be able to route some traffic through some kind of proxy - this is the tool for you!

**It can proxy:**

* `TLS Tunnel <https://latest.gost.run/en/tutorials/tls/>`_
* `HTTP/HTTPS Tunnel <https://latest.gost.run/en/tutorials/http-tunnel/>`_
* `PROXY Protocol <https://latest.gost.run/en/tutorials/proxy-protocol/>`_
* `DNAT/REDIRECT Traffic <https://latest.gost.run/en/tutorials/redirect/#redirect>`_ (*originated from the same host*))
* `TRPOXY integration <https://latest.gost.run/en/tutorials/redirect/#tproxy>`_
* `ICMP tunnel <https://latest.gost.run/en/tutorials/icmp/>`_
* And has many more hacky features

----

Forwarding to HTTP Proxy
########################

.. code-block:: text

    # NFTables =TCP=> TPROXY (forwarder @ 127.0.0.1) =HTTP[TCP]=> PROXY (squid http_port)


The current implementation of HTTP-forwarding in gost does not work correctly.

**Problems**:

* HTTP not working (*always wants to tunnel over HTTP-CONNECT*)
* HTTPS over IPv6 not working

**Solution**:

I've created a patched version of gost for exactly this purpose: `proxy-forwarder <https://github.com/superstes/proxy-forwarder>`_

DNAT
****

You can use it to catch DNAT traffic and forward it to a remote proxy-server like squid:

.. code-block:: bash

    proxy-forwarder -P 4128 -F http://192.168.10.20:3128
    # creates tcp & udp listeners for IPv4 & IPv6 on localhost:4128

    # NAT non-internal targets to the proxy
    ## nftables
    nft add rule nat output ip daddr != { 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12 } tcp dport { 80, 443 } dnat to 127.0.0.1:3128

    ## iptables
    iptables -t nat -A OUTPUT -p tcp ! -d 172.22.0.0/12 --dport 443 -j DNAT --to-destination 127.0.0.1:3128
    iptables -t nat -A OUTPUT -p tcp ! -d 172.22.0.0/12 --dport 80 -j DNAT --to-destination 127.0.0.1:3128


TPROXY
******

If you want to also proxy UDP traffic - you might want to `use the TPROXY integration <https://latest.gost.run/en/tutorials/redirect/#tproxy>`_:

.. code-block:: bash

    proxy-forwarder -P 4128 -F http://192.168.10.20:3128 -T

    # to also set a fw-mark on processed traffic
    proxy-forwarder -P 4128 -F http://192.168.10.20:3128 -T -M 100


Config Examples:

* `IPTables TPROXY <https://gist.github.com/superstes/c4fefbf403f61812abf89165d7bc4000>`_
* `NFTables TPROXY <https://gist.github.com/superstes/6b7ed764482e4a8a75334f269493ac2e>`_


Privileges
==========

You can run GOST TPROXY-mode with non-root users if you `add a capability <https://man7.org/linux/man-pages/man7/capabilities.7.html>`_ to the binary:

.. code-block:: bash

    sudo setcap cap_net_raw+ep /usr/local/bin/gost
    sudo chown root:gost /usr/local/bin/gost
    chmod 750 /usr/local/bin/gost

