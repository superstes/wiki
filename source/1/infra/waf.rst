.. _infra_waf_analyze:

.. include:: ../../_inc/head.rst

***************************
Web Application Firewalling
***************************

First we'll go through some basic knowledge and after that we'll look into **practical** log analysis.


Level of attacks
################

There are some different levels of attacks you will encounter:


1. **Bad crawlers**

   Gathering information from your site for some unknown use-case using GET/HEAD.

   You might be able to block them by implementing :ref:`bot-detection <infra_waf_analyze_bot_detect>` and :ref:`crawler verification <infra_waf_analyze_crawler_verify>`.


2. **Probes**

   Checking your site for known exploits that are far-spread or common data-/information-leaks.

   They only use GET/HEAD.

   You might be able to block them by creating a simple ruleset from the information you could gather by analyzing the request logs.


3. **Dumb scripts**

   This one might try to exploit single functions behind an URL or submit unprotected forms.

   Attackers will not change the user-agent of their http-client or set it to none.

   They might already use a pool of IP-addresses.

   You can easily block them by creating a list of bad user-agents.


4. **Better script**

   Also might try to exploit single functions behind an URL or submit unprotected forms.

   Attackers change their user-agent and use a pool of IP-addresses.

   Sometimes they still use a static user-agent or show the same major browser version but with different minor versions and operating system versions. (p.e. Chrome 103.x.x from Windows NT 6.x-10.x)

   The TLS-fingerprint will always be the same as the actual http-client does not change.

   They might also forget to set a referer or some *accept* headers that are set by all mainstream browsers nowadays.

   Client-side fingerprinting will catch them.


5. **Advanced bot tools**

   There are some bot-tools out there that can mimic a default browser in nearly every aspect.

   It's a continuous battle between them and client-side fingerprinting to fight for/against detection.

   These kind of attacks might be detectable by abnormal traffic patterns related to GeoIP country/ASN.

   Most unofficial botnets use hijacked servers to proxy their requests. If you are able to find out that user-actions are performed by clients that originate from hosting-/datacenter-IPs you might be onto something.

   You can also just make it harder to perform important actions by using `captcha <https://www.hcaptcha.com/>`_ or some other logic. Of course - that is a fight against AI tools.


6. **Human bots**

   As a last resort - some might even leverage humans as 'bots'.

   Some countries have low price-points for human work-hours. An example is India.

   There are whole companies that provide such a service. You tell them what to do and they have thousands of smartphones with proxies/VPNs that can target a website.

   As the devices are actual 'users' and can handle some anti-bot logic they might be hard to fight.

   It helps if you create some alerts for web-actions (*p.e. POST on some form*) that will be triggered if a threshold is exceeded. That way you can manually check if there is some unusual traffic going on.


.. _infra_waf_analyze_fp:
Fingerprinting
##############

Fingerprinting is used to:

* identify clients across IP addresses

* give us more information to identify bad traffic patterns that might indicate an attack


Usually systems use both client-side and server-side fingerprinting.

See also: `information provided by niespodd <https://github.com/niespodd/browser-fingerprinting>`_


Server-Side Fingerprint
***********************

This is most of the times implemented on your proxy/WAF or the application itself.

It is pretty straight-forward to implement, but has some major limitations as we are limited by the information we get from the client.

Therefor it alone cannot be used to get an unique fingerprint per client.

GeoIP information like country and ASN can be very useful to limit the matching-scope of such a fingerprint - if you want to lower the risk of blocking/flagging a single one. (more uniqueness, less global matching)

**How can it be assembled?**

* boolean values (1/0)

  * existence of headers (*referer, accept, accept-encoding, accept-language*)

  * existence of header values

  * settings inside headers

  * Is the domain inside the request or 'just' set as host-header?

  * sorting of URL-parameters or values inside headers or cookies

  * upper-/lower-/mixed-case of values

  * unusual special characters inside values

  * LF or CRLF line breaks used

  * usage of whitespace

* hashes

  * hash of some header-/cookie-value

  * of the TLS-Fingerprint


* information

  * limiting the match-scope by using the first 8-16 bits of the IP-address, GeoIP country and/or GeoIP ASN



TLS Fingerprint
***************

The SSL/TLS fingerprint can be useful as it differs between http-clients.

Nearly each http-client software and version of it uses a specific set of TLS settings.

The common settings used to build such a fingerprint are:

.. code-block:: bash

    SSLVersion,Cipher,SSLExtension,EllipticCurve,EllipticCurvePointFormat


Such a fingerprint enables us to compare it against the user-agent a client supplied. If there is an abnormality we can investigate it.

You could also create a mapping of known-good TLS fingerprints to user-agents and block/flag requests that don't match.

But for now there seems to be no public JA3 fingerprint database to compare your findings against. But I have a project like that in my backlog.. (;

See also: `JA3 SSL-fingerprint <https://github.com/salesforce/ja3>`_

----

.. _infra_waf_analyze_bot_detect:
Bot detection
#############

You might want to flag requests that might be bots so your application can handle them differently.

You can use a boolean flag or a bot-score.

Either way you might want to block 'dumb script' bots first or flag them as such.

Script bots
***********

How might one detect them?

* Search the user-agent for common http-clients used by scripting languages:

  * Headless browsers (\*headless\*)

  * Python3 libraries (\*python\*, scrapy, aiohttp, httpx, ...)

  * Golang (go-http-client, ...)

  * Javascript packages (axios, ...)

  * Shell tools (curl, wget, ...)

  * Powershell

  * C++ (cpp-httplib)

  * C#

  * Java (\*java\*)

  * Ruby (\*ruby\*)

  * Perl (\*perl\*)

  * PHP (guzzlehttp, phpcrawl, ...)

  * Darknet crawlers (test-bot, \*tiny-bot, fidget-spinner-bot, ...)

  * No user-agent at all

This list is only scratching the surface of tools/libraries that are used in the wild. You will have to check your logs and extend the list if needed.

Actual bots
***********

Note: This bot-check will not differentiate between 'good crawler bots' and others.

You might want to **flag a request as possible bot** if:

* They use known good-crawler user-agents

* They have 'bot' or 'spider' in their user-agent

* They have headers like 'accept', 'accept-language', 'accept-encoding' or 'referer' unset

   Note: You might encounter false-positives on the users 'entry page' if it is the first site the user opens (*no referer set*). But this is uncommon.

   Note: Old browsers (<2015) might not set the \*accept\* headers. But this is uncommon nowadays.

* They originate from

   * a datacenter (*GeoIP database needed*)

   * a country that is not expected to request your site (*GeoIP database needed*)

   * an ASN that is known to be used by bots: `ASN spamlist <https://cleantalk.org/blacklists/asn>`_, `Spamhaus ASN-DROP <https://www.spamhaus.org/news/article/820/the-return-of-the-asn-drop>`_ (*GeoIP database needed*)

   * IPs that are known to be used by bots: `Tor exit node IPList <https://check.torproject.org/torbulkexitlist>`_, `Spamhaus DROP <https://www.spamhaus.org/drop/>`_

Going further - you might want to **flag them as 'bad'** if:

* They use a common good-crawler user-agent but failed the :ref:`crawler verification <infra_waf_analyze_crawler_verify>`

* You can use the ASN-/IP-Lists mentioned above

----

.. _infra_waf_analyze_crawler_verify:
Crawler verification
####################

The large organizations that use crawlers to supply their services with information will provide you with a way to verify if a crawler, that uses their user-agent, is a legitimate one.

Most will either:

* configure their bot source-IPs with a specific reverse-DNS (PTR) record

* supply you with a way to check their source-IP with a list of IP-ranges


**Examples**:

* `Google Bots via reverse-DNS <https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot>`_

* `Bing Bots via reverse-DNS <https://www.bing.com/webmasters/help/how-to-verify-bingbot-3905dc26>`_

* `Yandex Bots via reverse-DNS <https://yandex.com/support/webmaster/robot-workings/check-yandex-robots.html>`_

* `Facebook Bots via IP-List <https://developers.facebook.com/docs/sharing/bot>`_

----

GeoIP information
#################

GeoIP databases can help you to identify attacks.

The most **interesting data** in my opinion is:

* Country

* ASN (*internet-/hosting-provider*)

* `Hosting detection <https://ipapi.is/hosting-detection.html>`_

* `VPN/Proxy/Tor/Relay detection <https://ipinfo.io/developers/privacy-detection-database>`_


**You can check-out some databases**:

* Maxmind Free GeoLite2-databases: `GeoLite2 database download <https://dev.maxmind.com/geoip/geolite2-free-geolocation-data>`_, `GeoLite2 database information <https://dev.maxmind.com/static/pdf/GeoLite2-IP-MetaData-Databases-Comparison-Chart.pdf>`_, `Maxmind Docs <https://dev.maxmind.com/geoip/docs/databases/city-and-country#binary-databases>`_

* Maxmind Paid databases: `Maxmind API <https://dev.maxmind.com/geoip/docs/web-services/requests>`_, `Maxmind databases <https://www.maxmind.com/en/geoip-databases>`_

* IP-Info: `ipinfo.io API <https://ipinfo.io/pricing#data>`_, `ipinfo.io Docs <https://ipinfo.io/developers/ip-to-geolocation-database>`_

* IP-API: `ipapi.is Docs <https://ipapi.is/geolocation.html>`_, `FREE databases <https://github.com/ipapi-is/ipapi>`_


----

Analyzing Request Logs
######################

If you want to protect a web-application from threats you will have to analyze the requests targeting it.

When analyzing request/access logs the right way, you might be able to detect 'hidden' attacks targeting the application.


