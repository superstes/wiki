.. _net_squid

.. include:: ../../_inc/head.rst


***********
Squid Proxy
***********

----

Introduction
############

Whenever we are referring to a 'client' - it will be a server, workstation or network device in most cases.


----

Source
******



----

References
**********

* `Config examples <https://wiki.squid-cache.org/ConfigExamples/>`_ (*WARNING: some examples are deprecated and will not work on current versions*)


----

Installation
############

SSL
***

If you are only 'peaking' at SSL connections - this should be enough:

.. code-block:: bash

    sudo apt install squid-openssl  # the package needs to have ssl-support enabled at compile-time

    openssl dhparam -outform PEM -out /etc/squid/bump.dh.pem 2048

    # openssl create self-signed cert
    openssl req -x509 -newkey rsa:4096 -keyout /etc/squid/bump.key -out /etc/squid/bump.crt -sha256 -days 3650 -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=Forward Proxy"

    # create ssl cache DB
    mkdir -p /var/lib/squid
    /usr/lib/squid/security_file_certgen -c -s /var/lib/squid/ssl_db -M 20MB

If you want to intercept SSL connections (*Man-in-the-middle like*) - you will have to go through some more steps: `squid docs - ssl interception <https://wiki.squid-cache.org/ConfigExamples/Intercept/SslBumpExplicit>`_

----

Modes
#####

----

HTTP_PORT
*********

The 'http_port' mode can be used as target proxy in applications like browsers.

Usual port 3128 is used for this mode.

The application creates a HTTP-CONNECT tunnel to the proxy and wraps its requests in it.

DNS resolution is done by the proxy.

HTTPS_PORT
**********

Like mode 'http_mode' but the HTTP-CONNECT tunnel is wrapped in TLS.

Usual port 3129 is used for this mode.

For the proxy to be able to handle the DNS resolution - **ssl-bump** must be configured. Else the proxy will not be able to read the **S**erver-**N**ame-**I**dentifier used in the TLS handshake.

INTERCEPT
*********

In this mode the proxy will expect the plain traffic to arrive.

You will have to create a dedicated listener with **ssl-bump** enabled if you want to handle TLS traffic.

SSL-BUMP
********

SSL-BUMP allows us to:
* read TLS handshake information
* intercept (*read/modify*) TLS traffic
* check server certificates for issues (*expired, untrusted*)

PEAK
====

By *peaking* at TLS handshake information in ssl-bump step-1 we are able to gain some important information:

* target DNS/hostname from SNI

Benefits:

* less performance needed than full ssl-interception
* faster than full ssl-interception
* less problems with applications that check certificates on their end (*p.e. banking*)
* no need to create/manage an internal Sub-CA to dynamically create and sign certificates for ssl-intercepted targets

**Drawbacks**:

* less options to filter the traffic on
* connections to *trustable* targets could carry dangerous payloads

In some cases a basic DNS 'allow-list' will be enough to ensure good security. Many automated attacks can be blocked using this approach.


INTERCEPT
=========

This one will be used in **zero-trust** environments.

**Benefits**:

* ssl-interception gives us much information that can be used to run IPS/IDS checks on
* possible dangerous payloads like downloads can be checked by anti-virus
* more restrictions make even interactive attacks harder to

**Drawbacks**:

* complex ruleset if you go with an *implicit-deny* approach
* much more performance needed
* increasing latency
* with a bad ruleset you will still have security-leaks but also have worse performance (*lose-lose*)

----

TPROXY
******

TProxy is a functionality built into current kernels.

It allows us to redirect traffic without modifying it. This solves the issue with overwritten destination-IPs by using Destination-NAT.

The major two integrations of TPROXY we will focus on are the ones in IPTables and NFTables.

In both implementations this is how we will need to handle the three main traffic types:

* **INPUT** traffic: can be redirected to TPROXY directly
* **FORWARD** traffic: can be redirected to TPROXY directly
* **OUTPUT** traffic: needs to be **routed to loopback** to be redirected to TPROXY

Why do we need to send 'output' traffic to loopback? Because TPROXY is only available in the 'prerouting-filter' chain and 'output' traffic does not hit that one by default.

NFTables
========

See: :ref:`NFTables TProxy <net_nftables_tproxy>`


IPTables
========

See: `IPTables TPROXY <https://gist.github.com/superstes/c4fefbf403f61812abf89165d7bc4000>`_

----

Config
######


----

Service
#######

To keep invalid configuration from stopping/failing your `squid.service` - you can add a config-validation in it:

.. code-block:: text

    # /etc/systemd/system/squid.service.d/override.conf

    [Service]
    ExecStartPre=
    ExecStartPre=/usr/sbin/squid -k parse
    ExecStartPre=/usr/sbin/squid --foreground -z

    ExecReload=
    ExecReload=/usr/sbin/squid -k parse
    ExecReload=/bin/kill -HUP $MAINPID

    Restart=on-failure
    RestartSec=5s

This will catch and log config-errors before doing a reload/restart.

When doing a system-reboot it will still fail if your config is bad.

----

Examples
########


----

Transparent Proxy
*****************

Destination NAT
===============

In some older tutorials and write-ups you will see that people DNAT traffic from a 'client' system to a remote proxy server.

This **IS NOT SUPPORTED** by squid.

It will lead to an error like this: 'Forwarding loop detected'

Why is that?

Squid's transparent operation modes DO NOT handle DNS resolution! They instead use the actual destination IP from the IP-headers and send the outgoing traffic to it. That is because of `some vulnerability <http://www.squid-cache.org/Advisories/SQUID-2011_1.txt>`_

When using DNAT the destination IP is set to the proxy's IP. Therefore => loop.

Routed Traffic
==============

You can use this option if the proxy server shares a Layer 2 network with the system that sends or routes the traffic.

Practical examples of this:

* Network gateway (*router*) sends traffic to proxy for interception
* 'Client' devices use the proxy as gateway instead of the actual router

In this case we will need to set-up Squid listeners in **intercept** mode to process the traffic.

You could also use the **tproxy** mode - but that might be more complicated to set-up when you want to check the traffic that enters at the 'forwarding' chain.

Forwarded Traffic
=================

In some situations you will not be able to use the option to route the traffic to the proxy.

This might be because:

* you are not controlling the gateway/router
* the 'client' device is isolated (*only connected to WAN*)
* client and/or network restrictions don't allow for re-routing the traffic

Practical examples of this:

* A Cloud VPS or Root Server that is only connected to WAN
* Distributed Systems using a central proxy (*p.e. on-site at customers*)

In this case we might need other tools like :ref:`GOST <net_gost>` to act as forwarder:


.. code-block:: text

    gost -L red://127.0.0.1:${TPROXY_PORT} -F http://${PROXY_SERVER}:${PROXY_PORT}


----

Troubleshooting
###############

What does not work?
*******************

One might want to try some other ways of sending/redirecting traffic to a squid proxy.

Here are some examples that **DO NOT WORK**

* Destination NAT to remote Squid server in transparent mode

  .. code-block:: bash

      # journalctl -u squid.service -n 50
      ...
      WARNING: Forwarding loop detected for
      ...
      TCP_MISS/403 ORIGINAL_DST/<proxy-ip>
      ...

* DNAT 80/443 to squid in non-transparent mode

  .. code-block:: bash

      # journalctl -u squid.service -n 50
      ...
      Missing or incorrect access protocol
      ...
      NONE/400
      ...

* IPTables/NFTables TPROXY to `socat forwarder <https://manpages.debian.org/unstable/socat/socat.1.en.html>`_

  SOCat is actually correctly receiving and forwarding the traffic - BUT it seems to perform a DNAT operation in the background

  .. code-block:: bash

      # 'client'
      socat tcp-listen:3129,reuseaddr,fork,bind=127.0.0.1,ip-transparent tcp:<proxy-ip>:3129

      # journalctl -u squid.service -n 50
      ...
      WARNING: Forwarding loop detected for
      ...
      TCP_MISS/403 ORIGINAL_DST/<proxy-ip>
      ...


Known problems
**************


* **Clients have many timeouts**

  It may be that your cache size is too small.

  This can happen when many requests hit the proxy in a short time period.

  **Possible Solution:**

    * Increase your main cache:

      cache_mem => 1024 MB (see `docs - cache_mem <http://www.squid-cache.org/Versions/v4/cfgman/cache_mem.html>`_)

    * Increase your session cache:

      sslproxy_session_cache_size => 512 MB

    * Increase your ssl cache (*only if you intercept ssl*)

      ssl_db => sslcrtd_program /usr/lib/squid/security_file_certgen -s /var/lib/squid/ssl_db -M 256MB

    * Increase your ssl session timeout

      sslproxy_session_ttl 600
