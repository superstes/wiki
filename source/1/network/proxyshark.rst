.. _net_proxyshark:

.. include:: ../../_inc/head.rst


**********
ProxyShark
**********


.. warning::

  This application let's you intercept and modify network traffic.

  That can be illegal => you are warned.

Introduction
############

I came across a challenge that needed me to modify tcp packages as man-in-the-middle.

I found a nice tool to do that; but without much information on how to set it up or use it.

Therefore I will provide those info's to you:

The tool I'm talking about is called 'proxyshark'. It's a python2 program/script written by the company '*CONIX Cybersecurity*' from france.

Just so I acknowledged it:

1. I know you could code this from scratch by only using the netfilterqueue and scapy modules
2. There are cleaner, faster and more up-to-date projects out there.. (p.e. pypacker)

Source
******

**Original**

* `GitHub <https://github.com/conix-security/audit-proxyshark>`_

**My fork** containing the fixes as described below:

* `ProxyShark Fork <https://github.com/superstes/audit-proxyshark>`_


----

Dependencies
############

At first I had some problems getting started with this script since the dependencies are 'a little' outdated.

**Disclaimer**: You might want to use this on a vm - since the dependencies will foul your system a little.

.. code-block:: bash

    # install pip2 dependencies
    apt install build-essential python-dev libnetfilter-queue-dev tshark git python2

    # install pip2
    wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
    sudo python2 get-pip.py

    # install pip packages
    python2 -m pip install dnspython pyparsing

    # scapy workaround
    python3 -m pip install scapy
    sudo mkdir /usr/lib/python2.7/dist-packages/scapy
    cd /usr/lib/python3/dist-packages/
    sudo cp -avr scapy/* /usr/lib/python2.7/dist-packages/scapy

----

Bugs & Fixes
############

Code
****

.. note::

  The fixes are included in my fork of this project => link can be found above.

The capture bugged out on me.

Therefore I fixed some 'bugs' that broke it:

.. code-block:: python2

    # Line 1159 => comment out the existing regex and replace it with this one:

    # regex = r'^.*(\d+\.\d+) +([^ ]+) +-> +([^ ]+) +([^ ]+) +[^ ]+ +(.*)$'
    regex = r'.*?(\d+\.\d+)\s*((?:[0-9]{1,3}\.){3}[0-9]{1,3}).*?((?:[0-9]{1,3}\.){3}[0-91,3})\s*([A-Z]{3,20})\s*(.*)'  # might be problematic in edge-cases

    # Line 1674 => add those lines before the 'if item_showname' matching:
    if type(item_show) == 'unicode':
        item_show = item_show.encode('utf8')
    if type(item_showname) == 'unicode':
        item_showname = item_showname.encode('utf8')


Execution
*********

Tshark
======

Run it with the '-t /usr/bin' argument to use the newer apt-version of **tshark** => the one from their repo bugged out on me..

Packet modification
===================

Somehow the 'direct' packet modification did not work.

I found the list object '_items_to_commit' and modified the value in it => that worked

.. code-block:: python2

    bpkt._items_to_commit[22]['value'] = 'aa02'

----

Usage
#####

Some basic commands can be found in the `ReadMe of the repository <https://github.com/conix-security/audit-proxyshark/blob/master/README.md>`_!

.. code-block:: bash

    python2 ps1/proxyshark.py -v --capture-filter 'dst host 8.8.8.8' --packet-filter 'udp' -t /usr/bin/



* Start the capture from the **interactive mode** by typing 'run'
* You should now see packages matching the filter by **typing 'packet'** (*last one*) or **'queue'** (*all*)
* Those packages can be 'caught' by defining a **breakpoint**.
* Captured packets can be **interacted** with in the default **python2 syntax**.
* **All possible attributes** etc. can be displayed by entering **'bpkt.__dict__'** (*after capturing some packet with a breakpoint*)
* Breakpoints can either pause the capture, so the packet can be edited manually, or modify it automatically with defined action

At first you might want to play around with the attributes of captures packets.

After that you can write automated actions to become a fully-grown m.i.t.m.

Examples
********

* show tcp data attributes

    .. code-block:: python2

        bpkt['tcp.data']

* show tcp flags

    .. code-block:: python2

        bpkt['tcp.flags']

* show ip destination

    .. code-block:: python2

        bpkt['ip.dst']



* rewrite tcp-reserved bits in auto-mode:

    .. code-block:: python2

        a add ta1 to tb1 "_flags = bpkt['tcp.flags'][0]['unmaskedvalue'][:1] + 'a' + bpkt['tcp.flags'][0]['unmaskedvalue'][2:]" "bpkt._items_to_commit[22]['value'] = _flags" "print _flags" "bpkt.accept()"
