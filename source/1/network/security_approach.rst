.. _net_security_approach:

.. include:: ../../_inc/head.rst

.. include:: ../../_inc/in_progress.rst

*******************
Security Approaches
*******************

----

Introduction
############

When it comes to IT- and Network security there are some main approaches/stances you may want to know.


.. tip::

    Feel free to `start Discussions <https://github.com/superstes/wiki/discussions>`_ regarding this topic!

    If you too have gained experiences in this area I would be interested if you are missing something.


----

What to secure
##############

Network components
==================

* Switches
* Routers
* Network firewalls
* VPN endpoints

Network clients
===============

* User devices like laptops, desktops, thin-clients

  * Different operating systems like Windows, MacOS, Linux, ChromeOS, Android, ...

* Servers

  * Different operating systems like Windows Server, Linux, ...

* Smartphones, tables
* IOT, automation devices
* Printers

Cloud resources
===============

Nearly all IT infrastructures have at least some components hosted on some cloud platform.

The main security of those services needs to ensured by the cloud providers - but consumers also need to ensure secure configuration on their side.

Virtualization infrastructure
=============================

Many companies use virtualization to share server-hardware between many virtual servers.

* Hypervisors
* Storage servers
* Control nodes

Some companies also use containerization like plain docker or even kubernetes to run their services.


Information
===========

Information is essential - it also needs to be secured.

* Databases
* Files
* Backup systems
* Applications

----

Overview
########

I will try to keep this overview of existing stances/mindsets as objective as possible.

These are very theoretically - but I will go into how they can translate into practical usage in the `personal experience <net_security_approach_personal>` section!

----

Perimeter-based
===============

Classic network designs were built around the concept of an enterprise LAN consisting of switches, routers and Wi-Fi connectivity.

The LAN contained one or more data centers, which housed applications and data.

This LAN formed the security network perimeter.

Accessing apps and services via the internet, VPNs and remote sites across WAN connections is considered external to the organization with perimeter-based security.

Everything connected to the LAN is considered "trusted," and devices coming from outside the perimeter are "untrusted".

This means external users must prove who they are through various security and identification tools.

----

Zero Trust
==========

Reference: `NIST SP 800-207 - Zero Trust Architecture <https://csrc.nist.gov/pubs/sp/800/207/final>`_

TLDR: 'Implicit trust is always a vulnerability, and therefore security must be designed with the strategy of “Never trust, always verify”'

Quotes
******

I'll quote some companies that provide solutions in the Zero-Trust area:

* **NIST SP 800-207**

  Source: `NIST SP 800-207 PDF <https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207.pdf>`_

  ..

    A typical enterprise’s infrastructure has grown increasingly complex. A single enterprise may operate several internal networks, remote offices with their own local infrastructure, remote and/or mobile individuals, and cloud services. This complexity has outstripped legacy methods of perimeter-based network security as there is no single, easily identified perimeter for the enterprise. Perimeter-based network security has also been shown to be insufficient since once attackers breach the perimeter, further lateral movement is unhindered.

    This complex enterprise has led to the development of a new model for cybersecurity known as “zero trust” (ZT). A ZT approach is primarily focused on data and service protection but can and should be expanded to include all enterprise assets (devices, infrastructure components, applications, virtual and cloud components) and subjects (end users, applications and other non-human entities that request information from resources). Throughout this document, “subject” will be used unless the section relates directly to a human end user in which “user” will be specifically used instead of the more generic “subject.” Zero trust security models assume that an attacker is present in the environment and that an enterprise-owned environment is no different—or no more trustworthy—than any nonenterprise-owned environment. In this new paradigm, an enterprise must assume no implicit trust and continually analyze and evaluate the risks to its assets and business functions and then enact protections to mitigate these risks. In zero trust, these protections usually involve minimizing access to resources (such as data and compute resources and applications/services) to only those subjects and assets identified as needing access as well as continually authenticating and authorizing the identity and security posture of each access request.

* **VMWare**

  Source: `VMWare - Zero Trust <https://www.vmware.com/topics/glossary/content/zero-trust-security.html>`_

  ..

      Zero Trust Security is a concept created on the belief that implicit trust is always a vulnerability, and therefore security must be designed with the strategy of “Never trust, always verify”. In its simplest form, Zero Trust restricts access to IT resources using strictly enforced identity and device verification processes.

  ..

      Zero Trust enforces the use of stringent security controls for users and devices before they can gain access to protected resources. Zero Trust identity authentication and authorization use the principle of least privilege (PoLP), which grants the absolute minimum rights required for a given function – before a single packet is transferred.

      This has become necessary because of the changes in how network resources are accessed. Gone are the days of a network perimeter or VPN-only access; today’s increasingly mobile workforce and growth in the work-at-home movement demand new security methods be considered for users, while the increasingly distributed nature of computing with containers and micro-services means that device-to-device connections are increasing as well.

      Thus, Zero Trust requires mutual authentication to confirm the identity and integrity of devices regardless of location to grant access based on the confidence of device identity, device health, and user authentication combined.

* **Microsoft**

  Source: `Transform to zero-trust-model <https://www.microsoft.com/en-us/security/blog/2019/10/23/perimeter-based-network-defense-transform-zero-trust-model/>`_

  ..

      The traditional firewall (VPN security model) assumed you could establish a strong perimeter, and then trust that activities within that perimeter were “safe.” The problem is today’s digital estates typically consist of services and endpoints managed by public cloud providers, devices owned by employees, partners, and customers, and web-enabled smart devices that the traditional perimeter-based model was never built to protect. We’ve learned from both our own experience, and the customers we’ve supported in their own journeys, that this model is too cumbersome, too expensive, and too vulnerable to keep going.

      We can’t assume there are “threat free” environments. As we digitally transform our companies, we need to transform our security model to one which assumes breach, and as a result, explicitly verifies activities and automatically enforces security controls using all available signal and employs the principle of least privilege access. This model is commonly referred to as “Zero Trust.”

Basically
*********

**Benefits:**

* tbc..

**Drawbacks:**

* Many different systems and lots of configuration is needed to ensure practical Zero-Trust - this brings some problems with it:

  * It may be very costly to operate those systems
  * Administrating those systems may consume more time
  * Keeping an overview of the configuration may get harder
  * How these systems need to be operated differs greatly from

* The users experience might be negatively impacted (*false-positive blocks, increased latency, more authentication*)

  * This essentially can bring down the company's productivity/efficiency by a little

* tbc..


To keep an overview over the configuration of all those systems one might also need to implement `infrastructure-as-code <https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac>`_ to centralize it.

----

.. _net_security_approach_personal:

Personal experience
###################

Here I will go into what these theoretical approaches can look like in practice.

Take this information with a grain of salt as it is spawned from the IT-environments I've interacted with and
