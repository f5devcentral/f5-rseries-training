====================================
Securing / Hardening F5OS on rSeries
====================================

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

If you would like to lock down one of the protocols to either a single IP address or subnet, use the **system allowed-ips** command. Be sure to commit any changes. The **prefix-length** parameter is optional. If you omit it, then you will lock down access to a specific IP endpoint, if you add it you can lock down access to a specific subnet.

.. code-block:: bash

    r10900-2(config)# system allowed-ips allowed-ip snmp config ipv4 address 10.255.0.0 prefix-length 24 port 161
    r10900-2(config-allowed-ip-snmp)# commit
    Commit complete.

Currently you can add one ip address/port pair per **allowed-ip** name with an optional prefix length to specify a CIDR block contaning multiple addresses. If you require more than one non-contiguous IP address or subnets you can add it under another name as seen below. 

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

Below is an example of allowing multiple SNMP endpoints (port 161) to query SNMP on the F5OS platfrom layer.

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

You can configure the **Allow List** in the webUI under the **System Settings** section. 

.. image:: images/rseries_security/image2.png
  :align: center
  :scale: 70%

Below is an example of allowing any SNMP endpoint at 10.255.0.0 (prefix length of 24) to query the F5OS layer on port 161.

.. image:: images/rseries_security/image3.png
  :align: center
  :scale: 70%


Certificates
=============

Appliance Mode for F5OS
=======================

If you would like to prevent root / bash level access to the F5OS layer, you can enable **Appliance Mode**, which operates in a similar manner as TMOS appliance mode. Enabling Appliance mode will disable the root account, and access to the underlying bash shell is disabled. The admin account to the F5OS CLI is still enabled. This is viewed as a more secure setting as many vulberabilites can be avodied by not allowing access to the bash shell. In some heavily audited environments, this setting may be mandatory, but it may prevent lower level debugging from occuring directly in the bash shell.

Enabling Appliance Mode via the CLI
-----------------------------------

Appliance mode can be enabled or disabled via the CLI using the command **system appliance-mode config** and entering either **enabled** or **disabled**. The command **show system appliance-mode** will display the current status. Be sure to commit any changes. 

.. code-block:: bash

    r10900(config)# system appliance-mode config enabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

To display the current status.

.. code-block:: bash

    r10900(config)# do show system appliance-mode       
    system appliance-mode state enabled
    r10900(config)# 

If you then try to login as root, you will get a permission denied error. You can still login as admin to gain access to the F5OS CLI.

To disable appliance mode.

.. code-block:: bash

    r10900(config)# system appliance-mode config disabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#

Enabling Appliance Mode via the webUI
------------------------------------- 

Appliance mode can be enabled or disabled via the webUI under the **System Settings -> General** page.

.. image:: images/rseries_security/image4.png
  :align: center
  :scale: 70%


Enabling Appliance Mode via the API
-----------------------------------

Appliance mode can be enabled or disabled via the API. To view the current status of appliance mode use the following API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-security-appliance-mode:appliance-mode


You will see output similar to the response below showing the config and state of appliance mode for F5OS.

.. code-block:: json

    {
        "f5-security-appliance-mode:appliance-mode": {
            "config": {
                "enabled": false
            },
            "state": {
                "enabled": false
            }
        }
    }

To change the mode from disabled to enabled, use the following API call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-security-appliance-mode:appliance-mode/f5-security-appliance-mode:config

In the body of the API call add the following:

.. code-block:: json

    {
        "f5-security-appliance-mode:config": {
            "f5-security-appliance-mode:enabled": "true"
        }
    }

Disabling Basic Authentication
==============================

F5OS utilizes basic authentication (username/password) as well as token based authentication for both the API and the webUI. Generally, username/password is issued by the client in order to obtain a token from F5OS, which is then used to make further inquiries or changes. Tokens have a relatively short lifetime for security reasons, and the user is allowed to refresh that token a certain number of times before they are forced to re-authenticate again. Although token based authentication is supported, basic authentication can still be utilized to access the devices and make changes. A new option was added in F5OS-A 1.3.0 to allow basic authentication to be disabled, except for the means of obtaining a token. Once the token is issued it is the only way to access both the webUI and the API. 


Disabling Basic Auth via the CLI
--------------------------------

The default setting for basic auth is enabled, and the current state can be seen by entering the **show system aaa** command. 

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

You may enable or disable basic authentication via the API. The default setting for basic auth is enabled, and the current state can be seen by entering the **show system aaa** command. 

Use the following API PATCH call to set the restconf-token:basic setting to **true** or **false**, or any other password policy parameter.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa

In the body of the API call adjust the restconf-token:basic setting to to **true** or **false**.

.. code-block:: json

    {
        "openconfig-system:aaa": {
            "authentication": {
                "config": {
                    "f5-aaa-confd-restconf-token:basic": {
                        "enabled": true
                    }
                }
            },
            "f5-aaa-confd-restconf-token:restconf-token": {
                "config": {
                    "lifetime": 10
                }
            },
            "f5-openconfig-aaa-password-policy:password-policy": {
                "config": {
                    "min-length": 6,
                    "required-numeric": 0,
                    "required-uppercase": 0,
                    "required-lowercase": 0,
                    "required-special": 0,
                    "required-differences": 8,
                    "reject-username": false,
                    "apply-to-root": true,
                    "retries": 3,
                    "max-login-failures": 10,
                    "unlock-time": 60,
                    "root-lockout": true,
                    "root-unlock-time": 60,
                    "max-age": 0
                }
            }
        }
    }


Disabling Basic Auth via the webUI
----------------------------------

Disabling basic authentication via the webUI is a new feature that has been added in F5OS-A 1.4.0. In the webUI got to **User Management -> Authentication Settings** and you'll see a drop down box to enable or disable **Basic Authentication**.

.. image:: images/rseries_security/image5.png
  :align: center
  :scale: 70%

Token Lifetime via CLI
----------------------

You may configure the restconf-token lifetime via the CLI. The value is in minutes, and the client is able to refresh the token five times before it expires. As an example, if the restconf-token lifeftime is set to 1 minute, an inactive webUI session will have a token expire after one minute, but it can be refreshed a maximum of five times. This will result in the webUI session timing out after 5 minutes.

.. code-block:: bash

    r10900(config)# system aaa restconf-token config lifetime 1 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

To display the current restconf-token lifetime setting, use the command **show system aaa***.

.. code-block:: bash

    r10900(config)# do show system aaa
    system aaa restconf-token state lifetime 1
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
    tenant-console  9100  -      

    NAME    NAME    TYPE    
    ------------------------
    tacacs  tacacs  TACACS  

    system aaa tls state verify-client false
    system aaa tls state verify-client-depth 1

Token Lifetime via webUI
------------------------

You may configure the restconf-token lifetime via the webUI (new feature added in F5OS-A 1.4.0). The value is in minutes, and the client is able to refresh the token five times before it expires. As an example, if the token lifeftime is set to 1 minute, an inactive webUI session will have a token expire after one minute, but it can be refreshed a maximum of five times. This will result in the webUI session timing out after 5 minutes.

.. image:: images/rseries_security/image6.png
  :align: center
  :scale: 70%

Token Lifetime via API
----------------------

You may configure the restconf-token lifetime via the API. The value is in minutes, and the client is able to refresh the token five times before it expires. As an example, if the token lifeftime is set to 1 minute, an inactive webUI session or API session will have a token expire after one minute, but it can be refreshed a maximum of five times. This will result in the webUI session timing out after 5 minutes.

Use the following API PATCH call to set the restconf-token lifetime, or any other password policy parameter.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa

In the body of the API call adjust the restconf-token lifetime setting to the desired timeout in minutes. The example below is 10 minutes, and the session will timeout at five times the value of the lifetime setting due to tken refresh.

.. code-block:: json

    {
        "openconfig-system:aaa": {
            "authentication": {
                "config": {
                    "f5-aaa-confd-restconf-token:basic": {
                        "enabled": true
                    }
                }
            },
            "f5-aaa-confd-restconf-token:restconf-token": {
                "config": {
                    "lifetime": 10
                }
            },
            "f5-openconfig-aaa-password-policy:password-policy": {
                "config": {
                    "min-length": 6,
                    "required-numeric": 0,
                    "required-uppercase": 0,
                    "required-lowercase": 0,
                    "required-special": 0,
                    "required-differences": 8,
                    "reject-username": false,
                    "apply-to-root": true,
                    "retries": 3,
                    "max-login-failures": 10,
                    "unlock-time": 60,
                    "root-lockout": true,
                    "root-unlock-time": 60,
                    "max-age": 0
                }
            }
        }
    }


Remote Authentication
=====================

The F5OS platform layer supports both local and remote authentication. By default there are local users enabled for both admin and root access. You will be forced to change passwords for both of these accounts on intial login. Many customers will prefer to configure the F5OS layer to use remote authentication via LDAP, RADIUS, or TACACS+.



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


Login Banner / Message of the Day
===================

Some environments require warning or acceptance messages to be displayed to clients connecting to the F5OS layer at intial connection time and/or upon successful login. The F5OS layer supports configurable Message of the Day (MoTD) and Login Banners that are displayed to clients connecting to the F5OS layer via both CLI and the webUI. The MoTD and Login Banner can be configured via CLI, webUI, or API. The Login Banner is displayed at initial connect time and is commonly used to notify users they are connecting to a specific resource, and that they should not connect if they are not authorized. The MoTD is displayed after successful login, and may also display some information about the resource the user is connecting to.

Configuring Login Banner / MoTD via CLI
---------------------------------------

.. code-block:: bash

    r10900(config)# system config login-banner "This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized."                                                 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

.. code-block:: bash

    r10900(config)# system config motd-banner "Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket." 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

.. code-block:: bash

    r10900# show system state 
    system state hostname r10900.f5demo.net
    system state login-banner This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized.
    system state motd-banner Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket.
    system state current-datetime "2022-11-29 11:12:27-05:00"
    system state base-mac 00:94:a1:69:59:00
    system state mac-pool-size 256
    r10900# 



Configuring Login Banner / MoTD via webUI
-----------------------------------------

.. image:: images/rseries_security/image7.png
  :align: center
  :scale: 70%



Configuring Login Banner / MoTD via API
---------------------------------------

Display of Login Banner and MoTD
--------------------------------

Below is an example of the Login Banner being displayed before the user is prompted for a password during an SSH connection to the F5OS platform layer. After a successfull user login, the MoTD is then displayed. Both are highlighted in bold below. 

.. code-block:: bash

    FLD-ML-00054045:~ jmccarron$ ssh -l admin 10.255.0.132
    **This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized.**
    admin@10.255.0.132's password: 
    Last login: Tue Nov 29 10:41:06 2022 from 10.10.10.16
    **Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket.**
    System Time: 2022-11-29 11:17:00 EST
    Welcome to the Management CLI
    User admin last logged in 2022-11-29T16:17:00.008317+00:00, to appliance-1, from 10.10.10.16 using cli-ssh
    admin connected from 10.10.10.16 using ssh on r10900.f5demo.net
    r10900# 

Below is an example of the Login Banner being displayed before the user is prompted for a password during a webUI connection to the F5OS platform layer. After a successfull user login, the MoTD is then displayed.


.. image:: images/rseries_security/image8.png
  :align: center
  :scale: 70%


.. image:: images/rseries_security/image9.png
  :align: center
  :scale: 70%  

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

