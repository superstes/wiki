.. _net_tor:

.. include:: ../../_inc/head.rst


***
Tor
***


.. warning::

  Tor SHOULD NOT be used to masquerade the source of malicious network requests.

  That can be illegal => you are warned.


Introduction
############

Tor is used to anonymize network requests and host/allow access to hidden (*.onion*) services.

.. warning::

  Traffic you send over tor should **ALWAYS be encrypted**!

  Else malicious tor nodes might be able to read and even modify your requests.

Just using the tor network won't be enough to stay anonymous!

You should also always follow the basic `guidelines on how to stay anonymous <https://support.torproject.org/faq/staying-anonymous/>`_.

If you are serious about your anonymity you might even go a set further and:

* Set-up a virtual machine to use for accessing tor.

  Per example: `whonix <https://www.whonix.org/wiki/VirtualBox>`_

* Connect that vm to a network that has no access to internal resources. (*only internet access, maybe a guest-network or 'dmz'*)


Route traffic over tor
######################

Tor uses the protocol `SOCKS5 <https://en.wikipedia.org/wiki/SOCKS>`_.

A `technical limitation of the tor implementation <https://gitlab.torproject.org/legacy/trac/-/issues/7830>`_ is that only `TCP-traffic <https://en.wikipedia.org/wiki/Transmission_Control_Protocol>`_ can be sent over those connections.

DNS
***

DNS-requests can be tunneled specially to `prevent dns-leaking <https://gitlab.torproject.org/legacy/trac/-/wikis/doc/Preventing_Tor_DNS_Leaks>`_. (*your provider [and so on..] will know what you are accessing*)

Implementation examples:

* `Cloudflare hidden resolver <https://blog.cloudflare.com/welcome-hidden-resolver/>`_

* `local tor dns-port <https://gitlab.torproject.org/legacy/trac/-/wikis/doc/TransparentProxy#local-redirection-through-tor/>`_

  Basically:

  .. code-block:: bash

    # example on ubuntu/debian

    sudo -i
    #   installing
    apt update
    apt install tor -y
    #     if you want to start it on system boot
    systemctl enable tor

    #   configuration
    echo 'DNSPort 5353' > /etc/tor/torrc
    systemctl start tor
    echo 'nameserver 127.0.0.1' > /etc/resolv.conf  # will not be persistent

* You might want to enable `DNS-over-HTTPS or DNS-over-TLS <https://www.cloudflare.com/learning/dns/dns-over-tls/>`_ to keep your requests secure.

Web access
**********

To access websites anonymously you can use the `tor browser <https://www.torproject.org/download/>`_.

Basics:

* Use DuckDuckGo as **search engine**!

  Other engines like Google will compromise your anonymity!

* Don't log-in with your **personal accounts** => it will compromise your anonymity.

* Don't enable **JavaScript** (*disabled via NoScript by default*)

  Some websites won't work correctly => but it keeps you safe(r).

* You might see **websites blocking your requests** as the most providers are blocking or limiting requests from tor exit-nodes.

  Maybe pressing 'Ctrl+Shift+L', to use a new 'route' for accessing the current page, might help.

* Using **hidden services**

  The 'default' websites you use daily using your normal browser are in a domain of the internet called 'clear-net'.

  You can browse them without worrying too much - they might 'just' compromise your anonymity.

  Hidden services are in the 'deep-net' => those are hidden for the usual user and only reachable using tor.

  *For clarification: tor is only ONE network that hosts hidden services - there are more out there*

  Hidden services have their own search engines but they have not listed many of the existing services!

  For the most** part you need to know the unique **.onion** address of a service to access it.

  .. warning::

      **Be aware**:

        * You might see/find disturbing and/or illegal content on those hidden services.

        * You need to have a basic technical understanding on how to interact with those services securely - else you might even get hacked.

----

Specific program
****************

Whenever you want a program to use tor as 'gateway' for its connection you need to 'proxy' it.

That proxy needs a tor 'SOCKS' socket to connect to.

SOCKS socket:

* The easiest way of starting such a socket is by opening the tor browser

  It starts such a socket in the background!

  .. code-block:: bash

    socks5 127.0.0.1 9150

* Another way is to install & start tor as service

  .. code-block:: bash

    socks5 127.0.0.1 9050

Linux
=====

On linux I would recommend using the application 'proxychains4' to achieve that.

You just need to set the SOCKS target to use.

.. code-block:: bash

  # example: tor browser SOCKS
  sudo -i
  echo 'socks5  127.0.0.1 9150' > /etc/proxychains4.conf

After that you can just start the application that should connect over tor by prepending 'proxychains4' to its command:

.. code-block:: bash

  # without tor
  curl https://ipinfo.io/city
  # using tor
  proxychains4 curl https://ipinfo.io/city

SSH
---

You can also set a proxy for ssh-connections.

Another program called 'netcat' is needed to archive that.

You will need to install the variant 'netcat-openbsd' as the 'default' one does not implement the needed options.

.. code-block:: bash

  # example on ubuntu/debian using tor browser SOCKS
  #   install dependencies
  sudo apt update
  sudo apt install openssh netcat-openbsd -y

  #   use
  ssh -p PORT -o ProxyCommand="nc -X5 -x127.0.0.1:9150 %h %p" USER@SERVER

----

All Traffic
***********

There are options to send all **TCP-Traffic and DNS-Requests** over the tor network.

.. warning::

  This should only be used if you really know what you are doing - as there are many ways you might compromise your anonymity!


Linux
=====

Here is the `official guide to proxying <https://gitlab.torproject.org/legacy/trac/-/wikis/doc/TransparentProxy>`_.

I won't go into the details on how to set this up - as I have not got experience with it.

It is done something like this: (*copied from the guide*)


.. code-block:: bash

    # example for 'middlebox' on ubuntu/debian
    sudo -i
    #   installing
    apt update
    apt install tor -y
    #     if you want to start it on system boot
    systemctl enable tor

    #   writing config
    echo 'VirtualAddrNetworkIPv4 10.192.0.0/10' > /etc/tor/torrc
    echo 'AutomapHostsOnResolve 1' >> /etc/tor/torrc
    echo 'TransPort 192.168.1.1:9040' >> /etc/tor/torrc
    echo 'DNSPort 192.168.1.1:5353' >> /etc/tor/torrc
    systemctl restart tor

    #   adding traffic redirection
    _trans_port="9040"
    _inc_if="eth1"  # you need to update the interface
    iptables -t nat -F  # WARNING: will remove all existing NAT-rules
    iptables -t nat -A PREROUTING -i $_inc_if -p udp --dport 53 -j REDIRECT --to-ports 5353
    iptables -t nat -A PREROUTING -i $_inc_if -p udp --dport 5353 -j REDIRECT --to-ports 5353
    iptables -t nat -A PREROUTING -i $_inc_if -p tcp --syn -j REDIRECT --to-ports $_trans_port

Windows
=======

You can use a tool like `OnionFruit <https://dragonfruit.network/onionfruit>`_.
