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


Certificates for Device Management
==================================

F5OS supports TLS device certificates and keys to secure connections to the management interface. You can either create a self-signed certificate, or load you own into the system.

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

F5OS utilizes basic authentication (username/password) as well as token based authentication for both the API and the webUI. Generally, username/password is issued by the client in order to obtain a token from F5OS, which is then used to make further inquiries or changes. Tokens have a relatively short lifetime for security reasons, and the user is allowed to refresh that token a certain number of times before they are forced to re-authenticate again. Although token based authentication is supported, basic authentication can still be utilized to access F5OS and make changes. A new option was added in F5OS-A 1.3.0 to allow basic authentication to be disabled, except for the means of obtaining a token. Once a token is issued, it will be the only way to make changes via the webUI or the API. 


Disabling Basic Auth via the CLI
--------------------------------

The default setting for basic auth is enabled, and the current state can be seen by entering the **show system aaa** command. The line **system aaa authentication state basic enabled** indicates that basic authentication is still enabled. 

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

You may disable basic authentication by issuing the cli command **system aaa authenitcation config basic disabled**, and then committing the change.

.. code-block:: bash

    r10900(config)# system aaa authentication config basic disbaled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#

To re-enable basic authentication, change the state to enabled and commit.

.. code-block:: bash

    r10900(config)# system aaa authentication config basic enabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#



Disabling Basic Auth via the API
--------------------------------

You may enable or disable basic authentication via the API. The default setting for basic autentication is enabled, and the current state can be seen by entering the following API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa/authentication/config

You should see the returned output below with the basic authentication state set to either **true** or **false**.

.. code-block:: json

    {
        "openconfig-system:config": {
            "f5-aaa-confd-restconf-token:basic": {
                "enabled": true
            }
        }
    }

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


Session Timeouts
================

Idle timeouts were configurable in previous releases, but the configuration only applied to the current session and was not persistent. F5OS-A 1.3.0 added the ability to configure persistent idle timeouts for both the CLI and webUI. The CLI timeout is configured under system settings, and is controlled via the idle-timeout option. For the webUI a token based timeout is now configurable under the system aaa settings. a restconf-token config lifetime option has been added. Once a client to the webUI has a token they are allowed to refresh it up to five times. If the token lifetime is set to 1 minute, then a timeout won't occur until five times that value or 5 minutes later. This is because of the token refresh has to fail 5 times before disconnecting the client.  

Configuring SSH and HTTPS Timeouts via CLI
------------------------------------------

To configure the CLI timeout via the CLI, use the command **system settings config idle-timeout <value-in-seconds>**. Be sure to issue a commit to save the changes. In the case below, the CLI session should disconnect after 300 seconds of inactivity.


.. code-block:: bash

    r10900(config)# system settings config idle-timeout 300
    r10900(config)# commit
    Commit complete.     
 
 
As mentioned in the introduction, the webUI uses tokens and the timeout is based on 5 token refreshes failing, so the value is essentiallyu 5 times the configured tken lifetime. Use the command **system aaa restconf-token config lifetime <value-in-minutes>**.

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

Enter config mode and use the command **system config login-banner** to configure the login banner via the CLI. You must commit the change afterwards.

.. code-block:: bash

    r10900(config)# system config login-banner "This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized."                                                 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

Enter config mode and use the command **system config motd-banner** to configure the Message of the Day banner via the CLI. You must commit the change afterwards.

.. code-block:: bash

    r10900(config)# system config motd-banner "Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket." 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

To display both settings, use the **show system state** command.

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

You may configure both the Login Banner and the Message of the Day Banner via the webUI on the **System Settings -> General** page.

.. image:: images/rseries_security/image7.png
  :align: center
  :scale: 70%



Configuring Login Banner / MoTD via API
---------------------------------------

You may configure both the Login Banner and the Message of the Day Banner via the API using the following API calls.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system

In the body of the API call configure the desired message of the day and login banner settings.

.. code-block:: json

    {
        "openconfig-system:system": {
            "config": {
                "hostname": "r10900-1.f5demo.net",
                "login-banner": "This is the Global Solution Architect's rSeries r10900 unit-1 in the Boston Lab. Unauthorized use is prohibited. Please reach out to Jim McCarron with any questions.",
                "motd-banner": "Welcome to the GSA r10900 Unit 1 in Boston"
            }
        }
    }


.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/config

.. code-block:: json

    {
        "openconfig-system:config": {
            "hostname": "r10900.f5demo.net",
            "login-banner": "This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized.",
            "motd-banner": "This is a test"
        }
    }


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


NTP Authentication
==================

NTP Authentication can be enabled to provide a secure communication channel for Network Time Protocol queries form the F5OS platform layer.

Disable IPv6
============

Encrypt TLS Private Key
=======================

It looks like we already encrypt the certificate key using AES256-GCM(AKA $8$) even before 1.3.0.
So the user can see on the GUI is the encrypted key, not the real/plain key.


Configurable Management Ciphers
===============================

Client Certificate Based Auth
=============================

iHealth Proxy Server
====================

F5OS supports the ability to capture detialed logs and configuration using the qkView utility. To speed up support case resolution the qkView can be uploaded directly to F5's iHealth service, which will give F5 support personnel access to the detailed information to aid problem resolution. In some environments, F5 devices may not have the ability to access the Internet without going through a proxy. The F5OS-A 1.3.0 release added the ability to upload qkViews through a proxy device.


Adding a Proxy Server via CLI
------------------------------


.. code-block:: bash

    r10900(config)# system diagnostics proxy config proxy-username myusername proxy-server https://myproxy.com:3128 proxy-password 
    (<AES encrypted string>): **************
    r10900(config)# 

Adding a Proxy Server via webUI
-------------------------------

.. image:: images/rseries_security/imageproxy1.png
  :align: center
  :scale: 70%  

Adding a Proxy Server via API
------------------------------

Audit Logging
=============

F5OS will log all access and configuration changes in two seprate **audit.log** files, which reside in the in one of the following paths; **log/system/audit.log** and **log/host/audit.log**. In versions prior to F5OS-A ???? the audit.log file may only be viewed locally within the F5OS layer, the audit log cannot be sent to a remote location. A new enhancement has been added in F5OS-A ?????? to allow audit.log entires to be redirected to a remote location. The formatting of audit logs provide the date/time in UTC, the account and ID who performed the action, the type of event, the asset affected, the type of access, and success or failure of the request. Separate log entries provide details on user access (login/login failures) information such as IP address and port and wether access was granted or not.


Viewing Audit Logs via F5OS CLI
-------------------------------

Most audit events go to the **log/system/audit.log** location, while a few others such as CLI login failures are logged to **log/host/audit.log** in the current F5OS releases. In the F5OS CLI, the paths are simplified so that you don’t have to know the underlying directory structure. You can use the **file list path** command to see the files inside the **log/system/** directory; use the tab complete to see the options. You may choose either the **log/system** directory or the **log/host** directory. Note the **audit.log** file. 

.. code-block:: bash

    appliance-1# file list path log/
    Possible completions:
    confd/  host/  system/
    appliance-1# file list path log/system/
    Possible completions:
    audit.log                      confd.log          devel.log     devel.log.1    lcd.log           lcd.log.1           lcd.log.2.gz       
    lcd.log.3.gz                   lcd.log.4.gz       lcd.log.5.gz  logrotate.log  logrotate.log.1   logrotate.log.2.gz  platform.log       
    reprogram_chassis_network.log  rsyslogd_init.log  snmp.log      startup.log    startup.log.prev  trace/              vconsole_auth.log  
    vconsole_startup.log           velos.log          webUI/        
    appliance-1# file list path log/system/

To view the contents of the **audit.log** file, use the command **file show path /log/system/audit.log**. This will show the entire log file from the beginning, but may not be the best way to troubleshoot a recent event:

.. code-block:: bash

    r10900# file show log/system/audit.log
    <INFO> 9-Dec-2021::17:13:57.506 appliance-1 confd[106]: audit user: admin/20518 assigned to groups: admin
    <INFO> 9-Dec-2021::17:13:57.506 appliance-1 confd[106]: audit user: admin/20518 created new session via cli from 172.27.196.47:52582 with ssh
    <INFO> 9-Dec-2021::17:13:57.589 appliance-1 confd[106]: audit user: admin/20518 terminated session (reason: normal)
    <INFO> 9-Dec-2021::17:13:57.633 appliance-1 confd[106]: audit user: admin/20519 assigned to groups: admin
    <INFO> 9-Dec-2021::17:13:57.633 appliance-1 confd[106]: audit user: admin/20519 created new session via cli from 172.27.196.47:52582 with ssh
    <INFO> 9-Dec-2021::18:14:14.380 appliance-1 confd[106]: audit user: admin/20519 terminated session (reason: timeout)
    <INFO> 9-Dec-2021::18:19:38.135 appliance-1 confd[106]: audit user: admin/0 external authentication succeeded via rest from 172.18.3.162:0 with http, member of groups: admin
    <INFO> 9-Dec-2021::18:19:38.135 appliance-1 confd[106]: audit user: admin/0 logged in via rest from 172.18.3.162:0 with http using external authentication
    <INFO> 9-Dec-2021::18:19:38.136 appliance-1 confd[106]: audit user: admin/21353 assigned to groups: admin
    <INFO> 9-Dec-2021::18:19:38.136 appliance-1 confd[106]: audit user: admin/21353 created new session via rest from 172.18.3.162:0 with http
    <INFO> 9-Dec-2021::18:19:38.136 appliance-1 confd[106]: audit user: admin/21353 RESTCONF: request with http: GET /restconf/ HTTP/1.1
    <INFO> 9-Dec-2021::18:19:38.137 appliance-1 confd[106]: audit user: admin/21353 terminated session (reason: normal)
    <INFO> 9-Dec-2021::18:19:38.137 appliance-1 confd[106]: audit user: admin/21353 RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 62361 ms


There are options to manipulate the output of the file. Add **| ?** to the command to see the options available to manipulate the file output.

.. code-block:: bash

    r10900# file show log/system/audit.log | ?
    Possible completions:
    append    Append output text to a file
    begin     Begin with the line that matches
    count     Count the number of lines in the output
    exclude   Exclude lines that match
    include   Include lines that match
    linnum    Enumerate lines in the output
    more      Paginate output
    nomore    Suppress pagination
    save      Save output text to a file
    until     End with the line that matches
    r10900# file show log/system/audit.log | 

There are other file options that allow the user to tail the log file using **file tail -f** for a live tail,  or **file tail -n <number of lines>** to view a specific number of the most recent lines.

.. code-block:: bash

    r10900# file tail -f log/system/audit.log
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:02.007 appliance-1 confd[125]: audit user: admin/13692368 CLI 'show system state hostname'
    <INFO> 7-Dec-2022::15:05:02.008 appliance-1 confd[125]: audit user: admin/13692368 CLI done
    <INFO> 7-Dec-2022::15:05:02.009 appliance-1 confd[125]: audit user: admin/13692368 terminated session (reason: normal)
    <INFO> 7-Dec-2022::15:05:02.052 appliance-1 confd[125]: audit user: admin/13692371 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:02.053 appliance-1 confd[125]: audit user: admin/13692371 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:19.428 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file show log/system/audit.log'
    <INFO> 7-Dec-2022::15:05:21.784 appliance-1 confd[125]: audit user: admin/13692371 CLI done
    <INFO> 7-Dec-2022::15:08:59.462 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file tail -f log/system/audit.log'



    r10900# file tail -n 20 log/system/audit.log
    <INFO> 7-Dec-2022::14:46:50.546 appliance-1 confd[125]: audit user: admin/13672920 RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 37668 ms
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/0 external token authentication succeeded via rest from 172.18.104.73:0 with http, member of groups: admin session-id:admin1670421700
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/0 logged in via rest from 172.18.104.73:0 with http using externalvalidation authentication
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/13673201 assigned to groups: admin
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/13673201 created new session via rest from 172.18.104.73:0 with http
    <INFO> 7-Dec-2022::14:47:05.977 appliance-1 confd[125]: audit user: admin/13673201 RESTCONF: request with http: GET /restconf/ HTTP/1.1
    <INFO> 7-Dec-2022::14:47:05.980 appliance-1 confd[125]: audit user: admin/13673201 terminated session (reason: normal)
    <INFO> 7-Dec-2022::14:47:05.981 appliance-1 confd[125]: audit user: admin/13673201 RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 35923 ms
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:02.007 appliance-1 confd[125]: audit user: admin/13692368 CLI 'show system state hostname'
    <INFO> 7-Dec-2022::15:05:02.008 appliance-1 confd[125]: audit user: admin/13692368 CLI done
    <INFO> 7-Dec-2022::15:05:02.009 appliance-1 confd[125]: audit user: admin/13692368 terminated session (reason: normal)
    <INFO> 7-Dec-2022::15:05:02.052 appliance-1 confd[125]: audit user: admin/13692371 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:02.053 appliance-1 confd[125]: audit user: admin/13692371 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:19.428 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file show log/system/audit.log'
    <INFO> 7-Dec-2022::15:05:21.784 appliance-1 confd[125]: audit user: admin/13692371 CLI done
    <INFO> 7-Dec-2022::15:08:59.462 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file tail -f log/system/audit.log'
    <INFO> 7-Dec-2022::15:09:22.907 appliance-1 confd[125]: audit user: admin/13692371 CLI done
    <INFO> 7-Dec-2022::15:09:31.142 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file tail -n 20 log/system/audit.log' 

Within the bash shell if you are logged in as root, the path for the logging is different; **/var/F5/system/log**. Note that older audit.log files are gzipped and rotated.

.. code-block:: bash

    [root@appliance-1(r10900.f5demo.net) ~]# ls -al /var/F5/system/log/
    total 2541432
    drwxr-xr-x.  4 root root       4096 Dec  6 20:14 .
    drwxr-xr-x. 26 root root       4096 Nov 28 12:38 ..
    -rw-r--r--.  1 root root   71290161 Dec  7 10:10 audit.log
    -rw-r--r--.  1 root root    1743543 Dec  9  2021 audit.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 audit.log.2.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 audit.log.3.gz
    -rw-r--r--.  1 root root    1847232 Dec  7  2021 audit.log.4.gz
    -rw-r--r--.  1 root root    9848782 Nov 28 12:35 confd.log
    -rw-r--r--.  1 root root      29979 Dec  9  2021 confd.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 confd.log.2.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 confd.log.3.gz
    -rw-r--r--.  1 root root      33306 Dec  7  2021 confd.log.4.gz
    -rw-r--r--.  1 root root   81663088 Dec  7 10:10 devel.log
    -rw-r--r--.  1 root root  104858977 Nov 13 15:11 devel.log.1
    -rw-r--r--.  1 root root    4541548 Oct 14 02:37 devel.log.2.gz
    -rw-r--r--.  1 root root    4838903 Aug 23 01:14 devel.log.3.gz
    -rw-r--r--.  1 root root    4747221 Jun 22 18:45 devel.log.4.gz
    -rw-r--r--.  1 root root    4788922 Apr 13  2022 devel.log.5.gz
    -rw-r--r--.  1 root root   24263778 Nov 28 13:40 k3s_events.log
    -rw-r--r--.  1 root root  105344182 Nov 28 12:54 k3s_events.log.1
    -rw-r--r--.  1 root root    8073081 Sep 19 11:30 k3s_events.log.2.gz
    -rw-r--r--.  1 root root   68972233 Jan 23  2022 lacp_out_132
    -rw-r--r--.  1 root root   50821845 Dec  7 10:10 lcd.log
    -rw-r--r--.  1 root root  104858247 Oct  6 23:13 lcd.log.1
    -rw-r--r--.  1 root root    6501076 Jun 27 10:24 lcd.log.2.gz
    -rw-r--r--.  1 root root    6518411 Jun  8 00:41 lcd.log.3.gz
    -rw-r--r--.  1 root root    6541114 May 19  2022 lcd.log.4.gz
    -rw-r--r--.  1 root root    6561702 Apr 22  2022 lcd.log.5.gz
    -rw-r--r--.  1 root root    1909130 Dec  7 10:10 logrotate.log
    -rw-r--r--.  1 root root    5244641 Dec  6 20:14 logrotate.log.1
    -rw-r--r--.  1 root root      31197 Dec  5 05:57 logrotate.log.2.gz
    -rw-r--r--.  1 root root  607087556 Dec  7 10:09 platform.log
    -rw-r--r--.  1 root root 1073833624 Jan 12  2022 platform.log.1
    -rw-r--r--.  1 root root   60136728 Jan  4  2022 platform.log.2.gz
    -rw-r--r--.  1 root root     454400 Dec  8  2021 platform.log.3.gz
    -rw-r--r--.  1 root root        621 Dec  7  2021 platform.log.4.gz
    -rw-r--r--.  1 root root       7841 Dec  7  2021 platform.log.5.gz
    -rw-r--r--.  1 root root       7734 Dec  7  2021 platform.log.6.gz
    -rw-r--r--.  1 root root  152724547 Dec  7  2021 platform.log.7.gz
    -rw-r--r--.  1 root root          0 Sep 30  2021 reprogram_chassis_network.log
    -rw-r--r--.  1 root root      41122 Nov 28 12:34 rsyslogd_init.log
    -rw-r--r--.  1 root root   16070999 Dec  5 23:48 snmp.log
    -rw-r--r--.  1 root root          0 Dec  9  2021 snmp.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 snmp.log.2.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 snmp.log.3.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 snmp.log.4.gz
    -rw-r--r--.  1 root root        435 Nov 28 12:34 startup.log
    -rw-r--r--.  1 root root        190 Nov 28 12:27 startup.log.prev
    drwxr-xr-x.  2 root root       4096 Sep 28  2021 trace
    -rw-r--r--.  1 root root       8424 Nov 28 12:34 vconsole_auth.log
    -rw-r--r--.  1 root root      31966 Nov 28 12:34 vconsole_startup.log
    -rw-r--r--.  1 root root          0 Dec  9  2021 velos.log
    -rw-r--r--.  1 root root          0 Dec  7  2021 velos.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 velos.log.2.gz
    -rw-r--r--.  1 root root    5960344 Oct 18  2021 velos.log.3.gz
    -rw-r--r--.  1 root root       4096 Oct 15  2021 .velos.log.swp
    drwxr-xr-x.  2 root root       4096 Nov 28 12:34 webui
    [root@appliance-1(r10900.f5demo.net) ~]# 
  
Viewing Logs from the webUI
--------------------------

In the current F5OS releases, you cannot view the F5OS audit.log file directly from the webUI, although you can download it from the webUI. To view the audit.log, you can use the CLI or API, or download the files and then view. To download log files from the webUI, go to the **System Settings -> File Utilities** page. Here there are various logs directories you can download files from. You have the option to **Export** files to a remote HTTPS server, or **Download** the files directly to your client machine through the browser.

.. image:: images/rseries_security/image10.png
  :align: center
  :scale: 70%

If you want to download the main **audit.log**, select the directory **/log/system**.


.. image:: images/rseries_security/image11.png
  :align: center
  :scale: 70%


Viewing Audit Logs via F5OS API
-------------------------------

Example Audit Logging of CLI Changes
------------------------------------


Example Audit Logging of API Changes
------------------------------------

In F5OS release prior to F5OS-A 1.4.0 API audit logs captured configuration changes, but did not log the full configuration payload. 

Example Audit Logging of webUI Changes
--------------------------------------




Downloading Audit Logs via CLI
------------------------------

Downloading Audit Logs via API
------------------------------

Downloading Audit Logs via webUI
-------------------------------

