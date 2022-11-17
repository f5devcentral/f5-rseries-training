=========================
Securing F5OS on rSeries
=========================

F5OS Platform Layer Isolation
=============================

Management of the new F5OS platform layer is completely isolated from in-band traffic networking and VLANs. It is purposely isolated so that it is only accessible via the out-of-band management network. In fact, there are no in-band IP addresses assigned to the F5OS layer, only tenants will have in-band management IP addresses and access. Tenants also have out-of-band connectivity.

This allows customers to run a secure/locked-down out-of-band management network where access is tightly restricted. The diagram below shows the out-of-band management access entering the rSeries appliance through **MGMT** port. The external MGMT port is bridged to an internal out-of-band network that connects to all tenants within the rSeries appliance. 

.. image:: images/rseries_security/image1.png
  :align: center

Allow List for F5OS Management
===============================

F5OS only allows management access via a single out-of-band management interface. Access to that single management IP address may be restricted to specific IP addresses (both IPv4 and IPv6), subnets (via Prefix Length), as well as protocols - 443 (HTTPS), 80 (HTTP), 8888 (RESTCONF), 161 (SNMP), 7001 (VCONSOLE), and 22 (SSH). An administrator can add one or more Allow List entries via the CLI, webUI or API to lock down access to specific endpoints.

By default, all ports except for 161 (SNMP) are enabled for access, meaning ports 80, 443, 8888, 7001, and 22 are allowed access. Port 80 is only open to allow a redirect to port 443 in case someone tries to access the webUI over port 80. The webUI itself is not accessible over port 80. Port 161 is typically viewed as un-secure, and is therefore not accessible until an allow list entry is created for the endpoint trying to access F5OS using SNMP queries. Ideally SNMPv3 should be utilized to provide additional layers of security on an otherwise un-secure protocol. VCONSOLE access also has to be explicitly configured before access to the tenants is possible over port 7001. 

To further lock down access you may add an Allow List entry including an IP address and optional prefix for each of the protocols listed above. As an example, if you wanted to restrict API and webUI access to a particular IP address and/or subnet, you can add an Allow List entry for the desired IP or subnet (using the prefix length), specify port 443 and all access from other IP endpoints will be prevented.


Adding Allow List Entries via CLI
-----------------------------------

If you would like to lock down one of the protocols to either a single IP address or subnet, use the **system allowed-ip** command. Be sure to commit any changes.

.. code-block:: bash

    r10900-2(config)# system allowed-ips allowed-ip snmp config ipv4 address 10.255.0.0 prefix-length 24 port 161
    r10900-2(config-allowed-ip-snmp)# commit
    Commit complete.

Currently you can add one ip address/port pair per **allowed-ip** name with an optional prefix length to specify a CIDR block contaning multiple addresses. If you require more than one non-contiguous IP address you can add it under another name as seen below. 

.. code-block:: bash

    appliance-1(config)# system allowed-ips allowed-ip SNMP-144 config ipv4 address 10.255.0.144 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


    appliance-1(config)# system allowed-ips allowed-ip SNMP-145 config ipv4 address 10.255.2.145 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


Adding Allow List Entries via API
-----------------------------------

By default SNMP queries are not allowed into the F5OS layer. Before enabling SNMP you'll need to open up the out-of-band management port on F5OS-A to allow SNMP queries. Below is an example of allowing an multiple SNMP endpoints at to access SNMP on the system on port 161.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

Within the body of the API call, specific IP address/port combinations can be added under a given name. In the current release, you are limited to one IP address/port per name. 

.. code-block:: json

    {
        "allowed-ip": [
            {
                "name": "SNMP-142",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.142",
                        "port": 161
                    }
                }
            },
            {
                "name": "SNMP-143",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.143",
                        "port": 161
                    }
                }
            },
            {
                "name": "SNMP-144",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.144",
                        "port": 161
                    }
                }
            }
        ]
    }



To view the allowed IP's in the API, use the following call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

The output will show the previously configured allowed-ip's.


.. code-block:: json

    {
        "f5-allowed-ips:allowed-ips": {
            "allowed-ip": [
                {
                    "name": "SNMP-142",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.142",
                            "port": 161
                        }
                    }
                },
                {
                    "name": "SNMP-143",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.143",
                            "port": 161
                        }
                    }
                },
                {
                    "name": "SNMP-144",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.144",
                            "port": 161
                        }
                    }
                }
            ]
        }
    }

Adding Allow List Entries via webUI
-----------------------------------

By default, SNMP queries are not allowed into the F5OS platform layer. Before enabling SNMP, you'll need to open up the out-of-band management port on F5OS-A to allow SNMP queries from particular SNMP management endpoints. Below is an example of allowing any SNMP endpoint at 10.255.0.0 (prefix length of 24) to query the F5OS layer on port 161.

.. image:: images/rseries_monitoring_snmp/image1.png
  :align: center
  :scale: 70%




Certificates
=============

Appliance Mode
=================

Disabling Basic Authentication
==============================

F5OS utilizes basic authentication (username / password) as well as token based authentication for both the API and the webUI. Generally, username / password is issued by the client in order to obtain a token from F5OS, which is then used to make further inquiries or changes. Tokens have a relatively short lifetime for security reasons, and the user is allowed to refresh that token a certain number of times before they are forced to re-authenticate again. Although token based authentication is supported, basic authntication can still be utilized to access the devices and make changes. A new option was added in F5OS-A 1.3.0 to allow basic authentication to be disabled, excpet for the means of obtaining a token. Once the token is issued it is the only way to access both the webUI and the API. 

Token lifetime details

Disabling Basic Auth via the CLI
--------------------------------

The default setting for basic auth is enabled, and the current state can be seen by utilizing the **show system aaa** command.

.. code-block:: bash

    r10900# show system aaa
    system aaa restconf-token state lifetime 15
    system aaa primary-key state hash gK/F47uQfi7JWYFirStCVhIaGcuoctpbGpx63MNy/korwigBW6piKx9TldiRazHmE8Y+qylGY4MOcs9IZ+KG4Q==
    system aaa primary-key state status NONE
    system aaa authentication state basic enabled
            LAST        TALLY  EXPIRY                  
    USERNAME  CHANGE      COUNT  DATE    ROLE            
    -----------------------------------------------------
    admin     2022-06-02  0      -1      admin           
    jim-test  2022-09-02  10     -1      admin           
    operator  2022-10-11  0      -1      operator        
    root      2022-06-02  0      -1      root            
    tenant1   0           0      1       tenant-console  
    tenant2   0           0      1       tenant-console  

    ROLENAME        GID   USERS  
    -----------------------------
    admin           9000  -      
    operator        9001  -      
    root            0     -      
    tenant-console  9100  -      

    NAME    NAME    TYPE    
    ------------------------
    tacacs  tacacs  TACACS  

    r10900# 

You may disable basic auth by issuing the cli command **system aaa authenitcation config basic disabled**, and then committing the change.

.. code-block:: bash

    r10900(config)# system aaa authentication config basic disbaled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#

To re-enable basic auth change the state to enabled and commit.

.. code-block:: bash

    r10900(config)# system aaa authentication config basic enabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#
  

Disabling Basic Auth via the API
--------------------------------

Disabling Basic Auth via the webUI
----------------------------------

Disabling of basic authentication via the webUI is a new feature that has been added in F5OS-A 1.4.0.


Remote Authentication
=====================

Audit Logs / Audit Logs to Remote Server
========================================


Session Timeouts
================

Idle timeouts were configurable in previous releases, but the configuration only applied to the current sesssion and was not persistent. F5OS-A 1.3.0 added the ability to configure persistent idle timeouts for both the CLI and webUI. The CLI timeout is configured under system settings, and is controlled via the idle-timeout option. For the webUI a toekn based timeout is now configurable under the system aaa settings. a restconf-token config lifetime option has been added. Once a client to the webUI has a token they are allowed to refresh it up to five times. If the toekn lifetime is set to 1 minute, then a timeout won't occur until five times that value or 5 minutes later. This is because of the token refresh has to fail 5 times before disconnecting the client.  

Configuring SSH and HTTPS Timeouts via CLI
------------------------------------------

To configure the CLI timeout via the CLI, use the command **system settings config idle-timeout <value-in-seconds>**. Be sure to issue a commit to save the changes. In the case below, the CLI session should disconnect after 300 seonds of inactivity.


.. code-block:: bash

    r10900(config)# system settings config idle-timeout 300
    r10900(config)# commit
    Commit complete.     
 
 
As mentioned in the introduction, the webUI uses tokens and the timeout is based on 5 token refreshes failing, so the value is essentiallyu 5 times the configured tken lifetime. Use the command **

.. code-block:: bash

    5900-2(config)# system aaa restconf-token config lifetime 1
    r5900-2(config)# commit
    Commit complete.
    r5900-2(config)# 
 
Configuring SSH and HTTPS Timeouts via API
------------------------------------------
 


Configuring SSH and HTTPS Timeouts via webUI
------------------------------------------


Login Banner / MOTD
===================


Console Logins
==============


SNMPv3
=======


NTP Auth
========


Disable IPv6
============

Encrypt TLS Private Key
=======================

Configurable Management Ciphers
===============================

Client Certificate Based Auth
=============================

iHealth Proxy Server
====================

