===========================================
rSeries F5OS-A SNMP Monitoring and Alerting
===========================================


Within rSeries tenants, SNMP support remains unchanged from existing BIG-IPs. SNMP monitoring and SNMP traps are supported in a similar manner as they are within a vCMP guest. You can continue to query the tenant via SNMP and receive SNMP traps. The F5OS-A platform layer handles the lower-level networking, and F5OS SNMP MIBs and traps are supported at this layer. The F5OS-A platform layer supported SNMP v1 and v2c versions initially, with SNMPv3 support added in F5OS-A 1.2.0.

Below are the latest SNMP MIBs as of the F5OS-A 1.6.0 release.

As of F5OS-A 1.6.0, the following NetSNMP MIBs are available:

- HOST-RESOURCES-MIB
- RFC1213-MIB
- EtherLike-MIB
- IANAifType-MIB
- IF-MIB
- IPV6-TC
- SNMP-COMMUNITY-MIB
- SNMP-FRAMEWORK-MIB
- SNMP-MPD-MIB
- SNMP-NOTIFICATION-MIB
- SNMP-TARGET-MIB
- SNMP-USER-BASED-SM-MIB
- SNMP-VIEW-BASED-ACM-MIB
- SNMPv2-CONF 
- SNMPv2-MIB
- SNMPv2 SMI
- SNMPv2-TC
- TRANSPORT-ADDRESS-MIB

As of F5OS-A 1.6.0.the following F5OS Appliance MIBs are available:

- F5-ALERT-DEF-MIB
- F5-COMMON-SMI-MIB
- F5-OS-LLDP-MIB
- F5-OS-PLATFORM-SMI-MIB
- F5-OS-SYSTEM-MIB
- F5-PLATFORM-STATS-MIB
- F5OS-APPLIANCE-ALERT-NOTIF-MIB

Downloading MIBs
================

MIBs can be downloaded directly from the F5OS layer starting in F5OS-A v1.2.0.


Downloading MIBs via webUI
--------------------------

From the webUI, you can go to the **System Settings > File Utility** page. Then, from the **Base Directory** drop down, select the **mibs** directory to download the MIB files. There are two separate MIB files: NetSNMP and F5OS MIBs for the appliance. Download both archives and extract them to see the individual MIB files.

.. image:: images/rseries_monitoring_snmp/image8.png
  :align: center
  :scale: 70%

Uploading MIBs to a Remote Server via CLI
-----------------------------------------

From the CLI, use the **file export** command to transfer the MIB files to a remote server. First, list the MIB files using the **file list** command as seen below.

.. code-block:: bash

    r10900-1# file list path mibs/
    entries {
        name mibs_f5os_appliance.tar.gz
        date Thu Nov 30 20:52:26 UTC 2023
        size 9.3KB
    }
    entries {
        name mibs_netsnmp.tar.gz
        date Thu Nov 30 20:52:26 UTC 2023
        size 110KB
    }
    r10900-1# 

To upload each of the files to a remote HTTPS server use the following command. You can also upload using SCP or SFTP by using the proper protocol option.

.. code-block:: bash

    appliance-1# file export local-file mibs/mibs_f5os_appliance.tar.gz remote-host 10.255.0.142 remote-file /upload/upload.php username corpuser insecure
    Value for 'password' (<string>): ********
    result File transfer is initiated.(mibs/mibs_f5os_appliance.tar.gz)
    appliance-1#

Repeat the same API call but change the filename to the **mibs_netsnmp.tar.gz** file.

Downloading MIBs via API
--------------------------

You can utilize the F5OS API to download the MIB files directly to a client machine, or to upload to a remote server over HTTPS, SCP, or SFTP. First, list the contents of the **mibs/** directory on the rSeries appliance using the following API call to get the filenames.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/f5-utils-file-transfer:file/list

In the body of the API call add the following:

.. code-block:: json

    {
    "f5-utils-file-transfer:path": "mibs/"
    }

This will list the contents of the mibs directory as seen below.

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "entries": [
                {
                    "name": "mibs_f5os_appliance.tar.gz",
                    "date": "Thu Nov 30 20:52:26 UTC 2023",
                    "size": "9.3KB"
                },
                {
                    "name": "mibs_netsnmp.tar.gz",
                    "date": "Thu Nov 30 20:52:26 UTC 2023",
                    "size": "110KB"
                }
            ]
        }
    }

You'll notice there are two separate MIB files, one is for Enterprise MIBs, while the other is for F5 specific MIBs. You'll need to download both files and add them to your SNMP manager. Below are example API calls to download each of the SNMP MIB files.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/f5-utils-file-transfer:file/f5-file-download:download-file/f5-file-download:start-download

For the **Headers** secion of the Postman request, be sure to add the following headers:

.. image:: images/rseries_monitoring_snmp/snmpheaders.png
  :align: center
  :scale: 70%

If you are using Postman, in the body of the API call select **Body**, then select **form-data**. Then enter the **file-name**, **path**, and **token** as seen below. 

.. image:: images/rseries_monitoring_snmp/downloadmibsapi1.png
  :align: center
  :scale: 70%

Repeat the same process for the other MIB file.

.. image:: images/rseries_monitoring_snmp/downloadmibsapi2.png
  :align: center
  :scale: 70%  

If you are using Postman, instead of clicking **Send**, click on the arrow next to Send, and then select **Send and Download**. You will then be prompted to save the file to your local file system.

.. image:: images/rseries_monitoring_snmp/sendanddownload.png
  :align: center
  :scale: 70%

Exporting MIBs to a Remote Server via the API
---------------------------------------------


To copy the SNMP MIB files from the appliance to a remote https server use the following API call:

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/f5-utils-file-transfer:file/export

In the body of the API call, add the remote server info and local file you want to export.

.. code-block:: json

    {
        "f5-utils-file-transfer:insecure": "",
        "f5-utils-file-transfer:protocol": "https",
        "f5-utils-file-transfer:username": "corpuser",
        "f5-utils-file-transfer:password": "password",
        "f5-utils-file-transfer:remote-host": "10.255.0.142",
        "f5-utils-file-transfer:remote-file": "/upload/upload.php",
        "f5-utils-file-transfer:local-file": "mibs/mibs/mibs_f5os_appliance.tar.gz"
    }
    
You can then check on the status of the export via the following API call:

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/api/data/f5-utils-file-transfer:file/transfer-status

The output will show the status of the file export.

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "result": "\nS.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                \n1    |Export file|HTTPS   |mibs/mibs_f5os_appliance.tar.gz                               |10.255.0.142        |/upload/upload.php                                          |         Completed|Thu Jan 20 05:11:44 2022"
        }
    }

Repeat the same steps for the other MIB file.


Adding Allowed IPs for SNMP
===========================

Adding Allowed IPs for SNMP via CLI
-----------------------------------

By default, SNMP queries are not allowed into the F5OS platform layer. Before enabling SNMP, you'll need to open the out-of-band management port on F5OS-A to allow SNMP queries from particular SNMP management endpoints. Below is an example of allowing any SNMP endpoint at 10.255.0.0 (prefix length of 24) to query the F5OS layer on port 161.


.. code-block:: bash

    r10900-2(config)# system allowed-ips allowed-ip snmp config ipv4 address 10.255.0.0 prefix-length 24 port 161
    r10900-2(config-allowed-ip-snmp)# commit
    Commit complete.

Currently you can add one IP address/port pair per **allowed-ip** name with an optional prefix length to specify a CIDR block containing multiple addresses. If you require more than one non-contiguous IP address, you can add it under another name as seen below. 

.. code-block:: bash

    appliance-1(config)# system allowed-ips allowed-ip SNMP-144 config ipv4 address 10.255.0.144 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


    appliance-1(config)# system allowed-ips allowed-ip SNMP-145 config ipv4 address 10.255.2.145 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


Adding Allowed IPs for SNMP via API
-----------------------------------

By default, SNMP queries are not allowed into the F5OS layer. Before enabling SNMP, you'll need to open up the out-of-band management port on F5OS-A to allow SNMP queries. Below is an example of allowing an multiple SNMP endpoints at to access SNMP on the system on port 161.

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



To view the allowed IPs in the API, use the following call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

The output will show the previously configured allowed-ips.


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

Adding Allowed IPs for SNMP via webUI
-----------------------------------

By default, SNMP queries are not allowed into the F5OS platform layer. Before enabling SNMP, you'll need to open up the out-of-band management port on F5OS-A to allow SNMP queries from particular SNMP management endpoints. Below is an example of allowing any SNMP endpoint at 10.255.0.0 (prefix length of 24) to query the F5OS layer on port 161.

.. image:: images/rseries_monitoring_snmp/image1.png
  :align: center
  :scale: 70%

In newer releases, the allowed IP functionality has been moved to the **System Settings -> Security** page as seen below.

.. image:: images/rseries_monitoring_snmp/image1a.png
  :align: center
  :scale: 70%

Adding Interface and LAG descriptions
=====================================


It is highly recommended that you put interface descriptions in your configuration, so that they will show up in the description field when using SNMP polling.

Adding Interface and LAG descriptions via CLI
---------------------------------------------

To add descriptions for both the in-band, and out-of-band management ports in the CLI, follow the examples below.

.. code-block:: bash

    appliance-1(config)# interfaces interface 1.0 config description "Interface 1.0"
    appliance-1(config-interface-1.0)# exit
    appliance-1(config)# interfaces interface 2.0 config description "Interface 2.0"               
    appliance-1(config-interface-2.0)# exit
    appliance-1(config)# interfaces interface 3.0 config description "Interface 3.0"
    appliance-1(config-interface-3.0)# interfaces interface 4.0 config description "Interface 4.0"
    appliance-1(config-interface-4.0)# interfaces interface 5.0 config description "Interface 5.0"
    appliance-1(config-interface-5.0)# interfaces interface 6.0 config description "Interface 6.0"
    appliance-1(config-interface-6.0)# interfaces interface 7.0 config description "Interface 7.0"
    appliance-1(config-interface-7.0)# interfaces interface 8.0 config description "Interface 8.0"
    appliance-1(config-interface-8.0)# interfaces interface 9.0 config description "Interface 9.0"
    appliance-1(config-interface-9.0)# interfaces interface 10.0 config description "Interface 10.0"
    appliance-1(config-interface-10.0)# interfaces interface 11.0 config description "Interface 11.0"
    appliance-1(config-interface-11.0)# interfaces interface 12.0 config description "Interface 12.0"
    appliance-1(config-interface-12.0)# interfaces interface 13.0 config description "Interface 13.0"
    appliance-1(config-interface-13.0)# interfaces interface 14.0 config description "Interface 14.0"
    appliance-1(config-interface-14.0)# interfaces interface 15.0 config description "Interface 15.0"
    appliance-1(config-interface-15.0)# interfaces interface 16.0 config description "Interface 16.0"
    appliance-1(config-interface-16.0)# interfaces interface 17.0 config description "Interface 17.0"
    appliance-1(config-interface-17.0)# interfaces interface 18.0 config description "Interface 18.0"
    appliance-1(config-interface-18.0)# interfaces interface 19.0 config description "Interface 19.0"
    appliance-1(config-interface-19.0)# interfaces interface 20.0 config description "Interface 20.0"
    appliance-1(config-interface-20.0)# exit
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 



    appliance-1(config)# interfaces interface mgmt  config description "Interface mgmt"
    appliance-1(config-interface-mgmt)# commit


If Link Aggregation Groups (LAGs) are configured, descriptions should be added to the LAG interfaces as well.

.. code-block:: bash

    appliance-1(config)# interfaces interface Arista config description "Arista LAG"
    appliance-1(config-interface-Arista)# exit
    appliance-1(config)# interfaces interface HA-Interconnect  config description "HA-Interconnect LAG"
    appliance-1(config-interface-HA-Interconnect)# exit
    appliance-1(config)# commit 
    Commit complete.
    appliance-1(config)# 


Adding Interface and LAG descriptions via API
---------------------------------------------

To add descriptions for both the in-band, and out-of-band management ports in the CLI, follow the examples below. The API example below is for the r10000 models, which have 20 interfaces and one management port. For the r5000 series models you should adjust for 10 interfaces and one management port.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/

.. code-block:: json

    {
        "openconfig-interfaces:interfaces": {
            "interface": [
                {
                    "name": "1.0",
                    "config": {
                        "description": "r10900 Interface 1.0"
                    }
                },
                {
                    "name": "2.0",
                    "config": {
                        "description": "r10900 Interface 2.0"
                    }
                },
                {
                    "name": "3.0",
                    "config": {
                        "description": "r10900 Interface 3.0"
                    }
                },
                {
                    "name": "4.0",
                    "config": {
                        "description": "r10900 Interface 4.0"
                    }
                },
                {
                    "name": "5.0",
                    "config": {
                        "description": "r10900 Interface 5.0"
                    }
                },
                {
                    "name": "6.0",
                    "config": {
                        "description": "r10900 Interface 6.0"
                    }
                },
                {
                    "name": "7.0",
                    "config": {
                        "description": "r10900 Interface 7.0"
                    }
                },
                {
                    "name": "8.0",
                    "config": {
                        "description": "r10900 Interface 8.0"
                    }
                },
                {
                    "name": "9.0",
                    "config": {
                        "description": "r10900 Interface 9.0"
                    }
                },
                {
                    "name": "10.0",
                    "config": {
                        "description": "r10900 Interface 10.0"
                    }
                },
                {
                    "name": "11.0",
                    "config": {
                        "description": "r10900 Interface 11.0"
                    }
                },
                {
                    "name": "12.0",
                    "config": {
                        "description": "r10900 Interface 12.0"
                    }
                },
                {
                    "name": "13.0",
                    "config": {
                        "description": "r10900 Interface 13.0"
                    }
                },
                {
                    "name": "14.0",
                    "config": {
                        "description": "r10900 Interface 14.0"
                    }
                },
                {
                    "name": "15.0",
                    "config": {
                        "description": "r10900 Interface 15.0"
                    }
                },
                {
                    "name": "16.0",
                    "config": {
                        "description": "r10900 Interface 16.0"
                    }
                },
                {
                    "name": "17.0",
                    "config": {
                        "description": "r10900 Interface 17.0"
                    }
                },
                {
                    "name": "18.0",
                    "config": {
                        "description": "r10900 Interface 18.0"
                    }
                },
                {
                    "name": "19.0",
                    "config": {
                        "description": "r10900 Interface 19.0"
                    }
                },
                {
                    "name": "20.0",
                    "config": {
                        "description": "r10900 Interface 20.0"
                    }
                },
                {
                    "name": "mgmt",
                    "config": {
                        "description": "r10900 Interface mgmt"
                    }
                }
            ]
        }
    }


If Link Aggregation Groups (LAGs) are configured, descriptions should be added to the LAG interfaces as well.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/

The body of the API call should contain JSON data that includes the descriptions for each LAG.

.. code-block:: json

    {
        "openconfig-interfaces:interfaces": {
            "interface": [
                {
                    "name": "Arista",
                    "config": {
                        "description": "LAG to Arista"
                    }
                },
                {
                    "name": "HA-Interconnect",
                    "config": {
                        "description": "LAG to other r10900"
                    }
                }

            ]
        }
    }


Configuring SNMP Access
=======================

To enable SNMP, you'll need to configure basic SNMP parameters like **system contact**, **location** and **name**. Then configure access for specific SNMP communities and versions. Currently SNMP can be setup via CLI and API, with configuration via webUI added in F5OS-A 1.3.0. 

Configuring SNMP Access via CLI F5OS-A 1.2.0 or Later
-----------------------------------------------------

You can configure the SNMP System parameters including the **System Contact**, **System Location**, and **System Name** as seen below:

.. code-block:: bash

    appliance-1(config)# SNMPv2-MIB system sysContact jim@f5.com sysLocation Boston sysName r5900-2
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 

SNMP configuration was only available in the CLI and API prior to F5OS-A 1.3.0, and the CLI configuration was not intuitive. F5OS-A 1.2.0 has improved and streamlined SNMP configuration in the CLI and then configuration via the webUI was also added in F5OS-A 1.3.0. The example below is utilizing the new and improved SNMP CLI configuration for rSeries systems running F5OS-A 1.2.0 or later. 

Enabling SNMP can be done from the CLI by configuring the **public** SNMP community, and then configuring a **security-model**. The command below sets up an SNMP community of **public** with v1 and v2c security models. You may choose to enable both security models or only one.

.. code-block:: bash

    r5900-2(config)# system snmp communities community public config security-model [ v1 v2c ]
    r5900-2(config-community-public)# exit
    r5900-2(config)# commit


You can then display the SNMP community configuration using the **show system snmp** command.

.. code-block:: bash

    r5900-2(config)# do show system snmp 
    system snmp engine-id state engine-id 80:00:2f:f4:03:00:94:a1:69:35:02
    system snmp engine-id state type mac
                    SECURITY    
    NAME    NAME    MODEL       
    ----------------------------
    public  public  [ v1 v2c ]  

    r5900-2(config)# 

You may also configure SNMP users for SNMPv3 support, since SNMPv3 is a user-based security model. This provides additional support for authentication and privacy protocols. Authentication protocols of **md5**, **sha**, or **none** are supported. For privacy protocols **aes**, **des**, or **none** are supported. You'll then be prompted to enter the privacy-password.

.. code-block:: bash

    r5900-2(config)# system snmp users user snmpv3user config authentication-protocol md5 privacy-protocol aes privacy-password 
    (<string, min: 8 chars, max: 32 chars>): **************
    r5900-2(config-user-snmpv3user)# commit
    Commit complete.

You may display the SNMP user configuration by entering the command **show system snmp users**.

.. code-block:: bash

    r5900-2(config)# do show system snmp users
                            AUTHENTICATION  PRIVACY   
    NAME        NAME        PROTOCOL        PROTOCOL  
    --------------------------------------------------
    snmpv3user  snmpv3user  md5             aes       

    r5900-2(config)# 

Configuring SNMP Access via CLI Prior to F5OS-A 1.2.0
-----------------------------------------------------

Below is the SNMP CLI configuration for systems running a version prior to F5OS-A 1.2.0. You can configure the SNMP System parameters including the **System Contact**, **System Location**, and **System Name** as seen below:

.. code-block:: bash

    appliance-1(config)# SNMPv2-MIB system sysContact jim@f5.com sysLocation Boston sysName r5900-2
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 

Enabling SNMP can be done from the CLI by configuring the public SNMP community, and then configuring a Security Access Group. Below is an example of enabling SNMP monitoring at the F5OS layer. F5OS only supports read-only access for SNMP monitoring.

.. code-block:: bash


    appliance-1# config
    Entering configuration mode terminal
    appliance-1(config)# SNMP-COMMUNITY-MIB snmpCommunityTable snmpCommunityEntry public snmpCommunityName public snmpCommunitySecurityName public
    appliance-1(config-snmpCommunityEntry-public)# exit
  

To configure a Security Group for both SNMPv1 and SNMPv2c.

.. code-block:: bash

    appliance-1(config)# SNMP-VIEW-BASED-ACM-MIB vacmSecurityToGroupTable vacmSecurityToGroupEntry 2 public vacmGroupName read-access
    appliance-1(config-vacmSecurityToGroupEntry-2/public)# exit
    appliance-1(config)# SNMP-VIEW-BASED-ACM-MIB vacmSecurityToGroupTable vacmSecurityToGroupEntry 1 public vacmGroupName read-access
    appliance-1(config-vacmSecurityToGroupEntry-1/public)# exit
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 


Configuring SNMP Access via API
-------------------------------

SNMP Communities, Users, and Targets can be setup via the API. An admin can enable access for SNMP monitoring of the system through either a community for SNMPv1/v2c, or through users for SNMPv3. In addition, remote SNMP Trap receiver locations can be enabled for alerting.

To configure the SNMP system parameters via API use the following API call:

.. code-block:: bash

    PATCH https://{{velos_chassis1_system_controller_ip}}:8888/restconf/data/SNMPv2-MIB:SNMPv2-MIB/system

In the body of the API add the SNMP sysContact, sysName, and sysLocation.

.. code-block:: json

    {
    "SNMPv2-MIB:system": {
        "sysContact": "jim@f5.com",
        "sysName": "r10900-1.f5demo.net",
        "sysLocation": "Boston"
        }
    }

To view the SNMP system parameters use the following API call:

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/SNMPv2-MIB:SNMPv2-MIB/system

A response similar to the one below will be displayed.

.. code-block:: json

    {
        "SNMPv2-MIB:system": {
            "sysDescr": "F5 rSeries-r10900 : Linux 3.10.0-1160.71.1.F5.1.el7_8.x86_64 : Appliance services version 1.8.0-8478",
            "sysObjectID": "1.3.6.1.4.1.12276.1.3.1.2",
            "sysUpTime": 61877485,
            "sysContact": "jim@f5.com",
            "sysName": "r10900-1.f5demo2.net",
            "sysLocation": "Boston",
            "sysServices": 72,
            "sysORLastChange": 9
        }
    }



To create an SNMPv3 user use the following API call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-system-snmp:snmp

Within the body of the API call, add the following JSON to add a user.

.. code-block:: json

    {
        "f5-system-snmp:snmp": {
            "users": {
                "user": [
                    {
                        "name": "snmpv3-user3",
                        "config": {
                            "name": "snmpv3-user3",
                            "authentication-protocol": "md5",
                            "f5-system-snmp:authentication-password": "{{rseries_password}}",
                            "privacy-protocol": "aes",
                            "f5-system-snmp:privacy-password": "{{rseries_password}}"
                        }
                    }
                ]
            }
        }
    }

If you are using SNMPv1/v2c then communities are the means of access. You can create an SNMP community via the API with the following API call:

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-system-snmp:snmp


In the body of the API call, add the community name you want to use to allow access to SNMP on the rSeries system. In this case a community called public2 is being used to enable access.

.. code-block:: json

    {
        "f5-system-snmp:snmp": {
            "communities": {
                "community": [
                    {
                        "name": "public2",
                        "config": {
                            "name": "public2",
                            "security-model": [
                                "v1",
                                "v2c"
                            ]
                        }
                    }
                ]
            }
        }
    }

To view the current SNMP configuration, issue the following API call:

.. code-block:: bash

    GET https://{{rseries_appliance_ip}}:8888/restconf/data/openconfig-system:system/f5-system-snmp:snmp

The output should appear similar to the example below.

.. code-block:: json

    {
        "f5-system-snmp:snmp": {
            "users": {
                "user": [
                    {
                        "name": "jim",
                        "config": {
                            "name": "jim",
                            "authentication-protocol": "md5",
                            "privacy-protocol": "aes"
                        },
                        "state": {
                            "name": "jim",
                            "authentication-protocol": "md5",
                            "privacy-protocol": "aes"
                        }
                    },
                    {
                        "name": "snmpv3-user3",
                        "config": {
                            "name": "snmpv3-user3",
                            "authentication-protocol": "md5",
                            "privacy-protocol": "aes"
                        },
                        "state": {
                            "name": "snmpv3-user3",
                            "authentication-protocol": "md5",
                            "privacy-protocol": "aes"
                        }
                    },
                    {
                        "name": "snmpv3user",
                        "config": {
                            "name": "snmpv3user",
                            "authentication-protocol": "md5",
                            "privacy-protocol": "aes"
                        },
                        "state": {
                            "name": "snmpv3user",
                            "authentication-protocol": "md5",
                            "privacy-protocol": "aes"
                        }
                    }
                ]
            },
            "communities": {
                "community": [
                    {
                        "name": "public",
                        "config": {
                            "name": "public",
                            "security-model": [
                                "v1",
                                "v2c"
                            ]
                        },
                        "state": {
                            "name": "public",
                            "security-model": [
                                "v1",
                                "v2c"
                            ]
                        }
                    },
                    {
                        "name": "public2",
                        "config": {
                            "name": "public2",
                            "security-model": [
                                "v1",
                                "v2c"
                            ]
                        },
                        "state": {
                            "name": "public2",
                            "security-model": [
                                "v1",
                                "v2c"
                            ]
                        }
                    }
                ]
            },
            "engine-id": {
                "config": {
                    "value": "mac"
                },
                "state": {
                    "engine-id": "80:00:2f:f4:03:00:94:a1:69:59:02",
                    "type": "mac"
                }
            },
            "config": {
                "port": 161
            },
            "state": {
                "port": 161
            }
        }


Configuring SNMP Access via webUI
---------------------------------

SNMP configuration via the webUI was added in the F5OS-A 1.3.0 release. You may configure SNMP Communities, SNMP Users, and SNMP Targets. SNMP is configured under **System Settings -> SNMP Configuration**.

.. image:: images/rseries_monitoring_snmp/image2.png
  :align: center
  :scale: 70%

An SNMP Community may be added for v1, v2c, or both v1 and v2c.

.. image:: images/rseries_monitoring_snmp/image3.png
  :align: center
  :scale: 100%

SNMP users can be added for environments which utilize SNMPv3.

.. image:: images/rseries_monitoring_snmp/image4.png
  :align: center
  :scale: 100%

SNMP Trap receivers may be added and a community or a user is added depending on the security model.

.. image:: images/rseries_monitoring_snmp/image5.png
  :align: center
  :scale: 100%

SNMP Trap Support in F5OS-A
===========================

You can enable SNMP traps for the F5OS-A platform layer. The **F5OS-APPLIANCE-ALERT-NOTIF-MIB** provides details about supported rSeries appliance SNMP traps. Below is the current full list of traps supported as of F5OS-A 1.6.0. NOTE: the file will contain alerts for both F5OS-A (rSeries appliances) and F5OS-C (VELOS chassis). You only need to rely on one file if you are using both platforms. Some traps may be specific to one platform or the other. 

SNMP Trap events that note a fault should also trigger an alert that can be viewed in the show alerts output in the CLI, WebUI, and API. They are also logged in the snmp.log file. Once a clear SNMP Trap is sent, it should clear the event from the **show events** output.

+----------------------------+----------------------------------+
| **Alert**                  | **OID**                          |
+============================+==================================+
| hardware-device-fault      | .1.3.6.1.4.1.12276.1.1.1.65536   |
+----------------------------+----------------------------------+
| firmware-fault             | .1.3.6.1.4.1.12276.1.1.1.65537   |
+----------------------------+----------------------------------+
| unknown-alarm              | .1.3.6.1.4.1.12276.1.1.1.65538   |
+----------------------------+----------------------------------+
| memory-fault               | .1.3.6.1.4.1.12276.1.1.1.65539   |
+----------------------------+----------------------------------+
| drive-fault                | .1.3.6.1.4.1.12276.1.1.1.65540   |
+----------------------------+----------------------------------+
| cpu-fault                  | .1.3.6.1.4.1.12276.1.1.1.65541   |
+----------------------------+----------------------------------+
| pcie-fault                 | .1.3.6.1.4.1.12276.1.1.1.65542   |
+----------------------------+----------------------------------+
| aom-fault                  | .1.3.6.1.4.1.12276.1.1.1.65543   |
+----------------------------+----------------------------------+
| drive-capacity-fault       | .1.3.6.1.4.1.12276.1.1.1.65544   |
+----------------------------+----------------------------------+
| power-fault                | .1.3.6.1.4.1.12276.1.1.1.65545   |
+----------------------------+----------------------------------+
| thermal-fault              | .1.3.6.1.4.1.12276.1.1.1.65546   |
+----------------------------+----------------------------------+
| drive-thermal-throttle     | .1.3.6.1.4.1.12276.1.1.1.65547   |
+----------------------------+----------------------------------+
| blade-thermal-fault        | .1.3.6.1.4.1.12276.1.1.1.65548   |
+----------------------------+----------------------------------+
| blade-hardware-fault       | .1.3.6.1.4.1.12276.1.1.1.65549   |
+----------------------------+----------------------------------+
| firmware-update-status     | .1.3.6.1.4.1.12276.1.1.1.65550   |
+----------------------------+----------------------------------+
| drive-utilization          | .1.3.6.1.4.1.12276.1.1.1.65551   |
+----------------------------+----------------------------------+
| sensor-fault               | .1.3.6.1.4.1.12276.1.1.1.65577   |
+----------------------------+----------------------------------+
| module-present             | .1.3.6.1.4.1.12276.1.1.1.66304   |
+----------------------------+----------------------------------+
| psu-fault                  | .1.3.6.1.4.1.12276.1.1.1.66305   |
+----------------------------+----------------------------------+
| lcd-fault                  | .1.3.6.1.4.1.12276.1.1.1.66306   |
+----------------------------+----------------------------------+
| module-communication-error | .1.3.6.1.4.1.12276.1.1.1.66307   |
+----------------------------+----------------------------------+
| fips-fault                 | .1.3.6.1.4.1.12276.1.1.1.196308  |
+----------------------------+----------------------------------+
| fipsError                  | .1.3.6.1.4.1.12276.1.1.1.196608  |
+----------------------------+----------------------------------+
| core-dump                  | .1.3.6.1.4.1.12276.1.1.1.327680  |
+----------------------------+----------------------------------+
| reboot                     | .1.3.6.1.4.1.12276.1.1.1.327681  |
+----------------------------+----------------------------------+
| raid-event                 | .1.3.6.1.4.1.12276.1.1.1.393216  |
+----------------------------+----------------------------------+
| backplane                  | .1.3.6.1.4.1.12276.1.1.1.262144  |
+----------------------------+----------------------------------+
| txPwrHiAlarm               | .1.3.6.1.4.1.12276.1.1.1.262400  |
+----------------------------+----------------------------------+
| txPwrHiWarn                | .1.3.6.1.4.1.12276.1.1.1.262401  |
+----------------------------+----------------------------------+
| txPwrLoAlarm               | .1.3.6.1.4.1.12276.1.1.1.262402  |
+----------------------------+----------------------------------+
| txPwrLoWarn                | .1.3.6.1.4.1.12276.1.1.1.262403  |
+----------------------------+----------------------------------+
| rxPwrHiAlarm               | .1.3.6.1.4.1.12276.1.1.1.262404  |
+----------------------------+----------------------------------+
| rxPwrHiWarn                | .1.3.6.1.4.1.12276.1.1.1.262405  |
+----------------------------+----------------------------------+
| rxPwrLoAlarm               | .1.3.6.1.4.1.12276.1.1.1.262406  |
+----------------------------+----------------------------------+
| rxPwrLoWarn                | .1.3.6.1.4.1.12276.1.1.1.262407  |
+----------------------------+----------------------------------+
| txBiasHiAlarm              | .1.3.6.1.4.1.12276.1.1.1.262408  |
+----------------------------+----------------------------------+
| txBiasHiWarn               | .1.3.6.1.4.1.12276.1.1.1.262409  |
+----------------------------+----------------------------------+
| txBiasLoAlarm              | .1.3.6.1.4.1.12276.1.1.1.262410  |
+----------------------------+----------------------------------+
| txBiasLoWarn               | .1.3.6.1.4.1.12276.1.1.1.262411  |
+----------------------------+----------------------------------+
| ddmTempHiAlarm             | .1.3.6.1.4.1.12276.1.1.1.262412  |
+----------------------------+----------------------------------+
| ddmTempHiWarn              | .1.3.6.1.4.1.12276.1.1.1.262413  |
+----------------------------+----------------------------------+
| ddmTempLoAlarm             | .1.3.6.1.4.1.12276.1.1.1.262414  |
+----------------------------+----------------------------------+
| ddmTempLoWarn              | .1.3.6.1.4.1.12276.1.1.1.262415  |
+----------------------------+----------------------------------+
| ddmVccHiAlarm              | .1.3.6.1.4.1.12276.1.1.1.262416  |
+----------------------------+----------------------------------+
| ddmVccHiWarn               | .1.3.6.1.4.1.12276.1.1.1.262417  |
+----------------------------+----------------------------------+
| ddmVccLoAlarm              | .1.3.6.1.4.1.12276.1.1.1.262418  |
+----------------------------+----------------------------------+
| ddmVccLoWarn               | .1.3.6.1.4.1.12276.1.1.1.262419  |
+----------------------------+----------------------------------+

SNMP Trap Details
=================

This section provides examples of SNMP traps and their associated log messages, and what troubleshooting steps are recommended. Traps will be sent with either an **assert** when an alarm occurs, a **clear** when the alarm is cleared, or an **event** which is providing an update to a raised alarm event.

- assert(1) is reported in alertEffect when alarm is raised.
- clear(0) is reported in alertEffect when alarm is cleared.
- event(2) is updated in alertEffect when event notification is reported.

As an example, the following set of traps are from an LCD failure and recovery on an F5OS based rSeries device. Note, that first there are a bunch of alarms being raised noted by **(INTEGER alertEffect=1)**. Then there are follow-on events, which provide additional updates to those alarms that have been raised noted by **(INTEGER alertEffect=2)**. Finally, the alarms are cleared as noted by **(INTEGER alertEffect=0)**, as well as additional informational events related to the clear noted by **(INTEGER alertEffect=2)**.

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include 11-Jul-2022::06:32       
    <INFO> 11-Jul-2022::06:32:03.334 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440809 10.255.0.145:161 (TimeTicks sysUpTime=24905)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2022-07-11 06:32:03.331289309 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 11-Jul-2022::06:32:03.335 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440809 10.255.0.144:161 (TimeTicks sysUpTime=24905)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2022-07-11 06:32:03.331289309 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 11-Jul-2022::06:32:03.384 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440810 10.255.0.145:161 (TimeTicks sysUpTime=24910)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:03.331305808 UTC)(OCTET STRING alertDescription=LCD module communication error detected)
    <INFO> 11-Jul-2022::06:32:03.384 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440810 10.255.0.144:161 (TimeTicks sysUpTime=24910)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:03.331305808 UTC)(OCTET STRING alertDescription=LCD module communication error detected)
    <INFO> 11-Jul-2022::06:32:03.434 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440811 10.255.0.145:161 (TimeTicks sysUpTime=24915)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:03.335454678 UTC)(OCTET STRING alertDescription=LCD Health is Not OK)
    <INFO> 11-Jul-2022::06:32:03.434 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440811 10.255.0.144:161 (TimeTicks sysUpTime=24915)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:03.335454678 UTC)(OCTET STRING alertDescription=LCD Health is Not OK)
    <INFO> 11-Jul-2022::06:32:07.371 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440812 10.255.0.145:161 (TimeTicks sysUpTime=25309)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554447)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 11-Jul-2022::06:32:07.371 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440812 10.255.0.144:161 (TimeTicks sysUpTime=25309)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554447)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 11-Jul-2022::06:32:23.884 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440813 10.255.0.145:161 (TimeTicks sysUpTime=26960)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554448)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 11-Jul-2022::06:32:23.884 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440813 10.255.0.144:161 (TimeTicks sysUpTime=26960)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554448)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 11-Jul-2022::06:32:52.025 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440814 10.255.0.145:161 (TimeTicks sysUpTime=29774)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:52.020011073 UTC)(OCTET STRING alertDescription=Firmware update completed for lcd app)
    <INFO> 11-Jul-2022::06:32:52.025 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440814 10.255.0.144:161 (TimeTicks sysUpTime=29774)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:52.020011073 UTC)(OCTET STRING alertDescription=Firmware update completed for lcd app)
    <INFO> 11-Jul-2022::06:32:53.291 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440815 10.255.0.145:161 (TimeTicks sysUpTime=29901)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:53.287950254 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 11-Jul-2022::06:32:53.291 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440815 10.255.0.144:161 (TimeTicks sysUpTime=29901)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:53.287950254 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 11-Jul-2022::06:32:53.341 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440816 10.255.0.145:161 (TimeTicks sysUpTime=29906)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:53.287969529 UTC)(OCTET STRING alertDescription=LCD module communication is OK)
    <INFO> 11-Jul-2022::06:32:53.341 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440816 10.255.0.144:161 (TimeTicks sysUpTime=29906)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:53.287969529 UTC)(OCTET STRING alertDescription=LCD module communication is OK)
    <INFO> 11-Jul-2022::06:32:53.391 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440817 10.255.0.145:161 (TimeTicks sysUpTime=29911)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:53.292347336 UTC)(OCTET STRING alertDescription=LCD Health is OK)
    <INFO> 11-Jul-2022::06:32:53.391 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440817 10.255.0.144:161 (TimeTicks sysUpTime=29911)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:32:53.292347336 UTC)(OCTET STRING alertDescription=LCD Health is OK)


Generic SNMP Traps
------------------

**coldStart         	1.3.6.1.6.3.1.1.5.1**  


A coldStart trap signifies that the SNMP entity,supporting a notification originator application, is reinitializing itself and that its configuration may have been altered.

.. code-block:: bash

    r10900-2# file show log/system/snmp.log | include cold
    <INFO> 30-Apr-2024::10:30:40.348 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214784 10.255.80.251:162 (TimeTicks sysUpTime=456)(OBJECT IDENTIFIER snmpTrapOID=coldStart)


**link down         	1.3.6.1.6.3.1.1.5.3**  

A linkDown trap signifies that the SNMP entity, acting in an agent role, has detected that the ifOperStatus object for one of its communication links is about to enter the down state from some other state (but not from the notPresent state). This other state is indicated by the included value of ifOperStatus.

.. code-block:: bash

    r10900-2# file show log/system/snmp.log | include linkDown
    <INFO> 30-Apr-2024::10:32:21.589 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214828 10.255.80.251:162 (TimeTicks sysUpTime=10581)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554513)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 3-May-2024::15:51:52.365 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214841 10.255.80.251:162 (TimeTicks sysUpTime=27847659)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554453)(INTEGER ifAdminStatus.0.=2)(INTEGER ifOperStatus.0.=2)
    r10900-2#

Note: In F5OS-C 1.8.0 an additional F5OS enterprise trap has been added that will trigger in parallel with the generic linkup/down traps. The enterprise linkup/down traps adds a human readable interface name as seen below.

.. code-block:: bash

    <INFO> 3-May-2024::15:51:52.365 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214841 10.255.80.251:162 (TimeTicks sysUpTime=27847659)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554453)(INTEGER ifAdminStatus.0.=2)(INTEGER ifOperStatus.0.=2)

    <INFO> 3-May-2024::15:51:52.363 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214840 10.255.80.251:162 (TimeTicks sysUpTime=27847658)(OBJECT IDENTIFIER snmpTrapOID=down)(OCTET STRING alertSource=interface-13.0)(INTEGER alertEffect=1)(INTEGER alertSeverity=4)(OCTET STRING alertTimeStamp=2024-05-03 19:51:52.350979671 UTC)(OCTET STRING alertDescription=Interface down)

**link up         	1.3.6.1.6.3.1.1.5.4**  

A linkUp trap signifies that the SNMP entity, acting in an agent role, has detected that the ifOperStatus object for one of its communication links left the down state and transitioned into some other state (but not into the notPresent state). This other state is indicated by the included value of ifOperStatus.


.. code-block:: bash

    <INFO> 3-May-2024::15:59:54.373 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214845 10.255.80.251:162 (TimeTicks sysUpTime=27895859)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554453)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)

Note: In F5OS-C 1.8.0 an additional F5OS enterprise trap has been added that will trigger in parallel with the generic linkup/down traps. The enterprise linkup/down traps adds a human readable interface name as seen below.


.. code-block:: bash

    <INFO> 3-May-2024::15:59:54.373 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214845 10.255.80.251:162 (TimeTicks sysUpTime=27895859)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554453)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    
    <INFO> 3-May-2024::15:59:54.371 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214844 10.255.80.251:162 (TimeTicks sysUpTime=27895859)(OBJECT IDENTIFIER snmpTrapOID=up)(OCTET STRING alertSource=interface-13.0)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2024-05-03 19:59:54.359054296 UTC)(OCTET STRING alertDescription=Interface up)   

F5OS Specific Traps
------------------

Device Fault Traps
^^^^^^^^^^^^^^^^^^

**hardware-device-fault          .1.3.6.1.4.1.12276.1.1.1.65536**   

This set of taps may indicate a fault with various hardware components on the rSeries appliance like CPUs or fans. Examine the trap for specific details of what subsystem has failed to determine the proper troubleshooting steps to pursue. 

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include hardware-device-fault
    <INFO> 11-Jul-2022::06:29:16.529 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440640 10.255.0.145:161 (TimeTicks sysUpTime=8225)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.245012010 UTC)(OCTET STRING alertDescription=Deasserted: CPU HW correctable error)
    <INFO> 11-Jul-2022::06:29:16.529 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440640 10.255.0.144:161 (TimeTicks sysUpTime=8225)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.245012010 UTC)(OCTET STRING alertDescription=Deasserted: CPU HW correctable error)
    <INFO> 11-Jul-2022::06:29:17.332 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440650 10.255.0.145:161 (TimeTicks sysUpTime=8305)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-7)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.768784161 UTC)(OCTET STRING alertDescription=fan 7 at 27051 RPM)
    <INFO> 11-Jul-2022::06:29:17.333 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440650 10.255.0.144:161 (TimeTicks sysUpTime=8305)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-7)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.768784161 UTC)(OCTET STRING alertDescription=fan 7 at 27051 RPM)
    <INFO> 11-Jul-2022::06:29:17.433 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440651 10.255.0.145:161 (TimeTicks sysUpTime=8315)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-8)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.770124231 UTC)(OCTET STRING alertDescription=fan 8 at 26857 RPM)
    <INFO> 11-Jul-2022::06:29:17.433 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440651 10.255.0.144:161 (TimeTicks sysUpTime=8315)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-8)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.770124231 UTC)(OCTET STRING alertDescription=fan 8 at 26857 RPM)
    <INFO> 11-Jul-2022::06:29:18.237 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440659 10.255.0.145:161 (TimeTicks sysUpTime=8395)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-6)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.781064597 UTC)(OCTET STRING alertDescription=fan 6 at 27075 RPM)
    <INFO> 11-Jul-2022::06:29:18.237 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440659 10.255.0.144:161 (TimeTicks sysUpTime=8395)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-6)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.781064597 UTC)(OCTET STRING alertDescription=fan 6 at 27075 RPM)
    <INFO> 11-Jul-2022::06:29:19.041 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440667 10.255.0.145:161 (TimeTicks sysUpTime=8476)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.791114234 UTC)(OCTET STRING alertDescription=Deasserted: CPU thermal trip fault)
    <INFO> 11-Jul-2022::06:29:19.041 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440667 10.255.0.144:161 (TimeTicks sysUpTime=8476)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.791114234 UTC)(OCTET STRING alertDescription=Deasserted: CPU thermal trip fault)
    <INFO> 11-Jul-2022::06:29:19.643 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440675 10.255.0.145:161 (TimeTicks sysUpTime=8536)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-5)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.022807820 UTC)(OCTET STRING alertDescription=fan 5 at 26905 RPM)
    <INFO> 11-Jul-2022::06:29:19.643 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440675 10.255.0.144:161 (TimeTicks sysUpTime=8536)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-5)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.022807820 UTC)(OCTET STRING alertDescription=fan 5 at 26905 RPM)
    <INFO> 11-Jul-2022::06:29:20.446 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440683 10.255.0.145:161 (TimeTicks sysUpTime=8616)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.201227249 UTC)(OCTET STRING alertDescription=Deasserted: CPU hot fault)
    <INFO> 11-Jul-2022::06:29:20.446 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440683 10.255.0.144:161 (TimeTicks sysUpTime=8616)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.201227249 UTC)(OCTET STRING alertDescription=Deasserted: CPU hot fault)
    <INFO> 11-Jul-2022::06:29:20.546 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440684 10.255.0.145:161 (TimeTicks sysUpTime=8626)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-4)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.202497586 UTC)(OCTET STRING alertDescription=fan 4 at 26954 RPM)
    <INFO> 11-Jul-2022::06:29:20.546 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440684 10.255.0.144:161 (TimeTicks sysUpTime=8626)(OBJECT IDENTIFIER snmpTrapOID=hardware-device-fault)(OCTET STRING alertSource=fan-4)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.202497586 UTC)(OCTET STRING alertDescription=fan 4 at 26954 RPM)


**firmware-fault                 .1.3.6.1.4.1.12276.1.1.1.65537**

This set of taps may indicate a fault or temporary warning with the firmware upgrade process. Monitor the firmware upgrade process via SNMP traps, or via the CLI, API, or webUI alerts. These may occur as part of a software update to F5OS. Not every upgrade requires firmware to be updated. You may see different components having their firmware upgraded such as (lcd, bios, cpld, lop app, sirr, atse, asw, nso, nvme0, nvme1). It is important not to interrupt the firmware upgrade process. If you see a firmware update alert raised for a specific component, you should not make any changes to the system until each component returns a Firmware update completed message. In newer versions of F5OS, the webUI will display a banner at the top of the page while firmware updates run and will disappear when they complete. The banner will have a link to the **Alarms and Events** page which will show the current status of the firmware updates as seen below.


.. image:: images/rseries_monitoring_snmp/imagefirmwareupgrade.png
  :align: center
  :scale: 100%

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include firmware-fault
    <INFO> 11-Jul-2022::06:29:16.880 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440645 10.255.0.145:161 (TimeTicks sysUpTime=8260)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.265507257 UTC)(OCTET STRING alertDescription=Deasserted: Task stack warning)
    <INFO> 11-Jul-2022::06:29:16.881 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440645 10.255.0.144:161 (TimeTicks sysUpTime=8260)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.265507257 UTC)(OCTET STRING alertDescription=Deasserted: Task stack warning)
    <INFO> 11-Jul-2022::06:29:19.342 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440671 10.255.0.145:161 (TimeTicks sysUpTime=8506)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.797173242 UTC)(OCTET STRING alertDescription=Deasserted: Heap running low)
    <INFO> 11-Jul-2022::06:29:19.342 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440671 10.255.0.144:161 (TimeTicks sysUpTime=8506)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:15.797173242 UTC)(OCTET STRING alertDescription=Deasserted: Heap running low)
    <INFO> 11-Jul-2022::06:29:22.907 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440708 10.255.0.145:161 (TimeTicks sysUpTime=8862)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.233395912 UTC)(OCTET STRING alertDescription=Deasserted: ARM exception available)
    <INFO> 11-Jul-2022::06:29:22.907 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440708 10.255.0.144:161 (TimeTicks sysUpTime=8862)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:16.233395912 UTC)(OCTET STRING alertDescription=Deasserted: ARM exception available)
    <INFO> 11-Jul-2022::06:29:28.939 appliance-1 confd[127]: snmp snmpv2-trap reqid=1257440769 10.255.0.145:161 (TimeTicks sysUpTime=9466)(OBJECT IDENTIFIER snmpTrapOID=firmware-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-07-11 06:29:19.908471420 UTC)(OCTET STRING alertDescription=Deasserted: Watchdog timer warning)


**unknown-alarm                  .1.3.6.1.4.1.12276.1.1.1.65538**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include unknown-alarm

**memory-fault                   .1.3.6.1.4.1.12276.1.1.1.65539**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include memory-fault

**drive-fault                    .1.3.6.1.4.1.12276.1.1.1.65540**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include drive-fault

**cpu-fault                      .1.3.6.1.4.1.12276.1.1.1.65541**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include cpu-fault

**pcie-fault                     .1.3.6.1.4.1.12276.1.1.1.65542**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include pcie-fault

**aom-fault                      .1.3.6.1.4.1.12276.1.1.1.65543**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include aom-fault

**drive-capacity-fault           .1.3.6.1.4.1.12276.1.1.1.65544**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include drive-capacity-fault
    <INFO> 12-Apr-2023::11:54:10.563 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130731 10.255.8.22:6011 (TimeTicks sysUpTime=87079)(OBJECT IDENTIFIER snmpTrapOID=drive-capacity-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=2)(OCTET STRING alertTimeStamp=2023-04-12 11:54:10.558711877 UTC)(OCTET STRING alertDescription=Running out of drive capacity)
    <INFO> 12-Apr-2023::11:54:10.613 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130732 10.255.8.22:6011 (TimeTicks sysUpTime=87084)(OBJECT IDENTIFIER snmpTrapOID=drive-capacity-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:54:10.558725204 UTC)(OCTET STRING alertDescription=Drive usage exceeded 97%, used=100%)
    <INFO> 12-Apr-2023::11:54:35.167 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130733 10.255.8.22:6011 (TimeTicks sysUpTime=89540)(OBJECT IDENTIFIER snmpTrapOID=drive-capacity-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:54:35.162718848 UTC)(OCTET STRING alertDescription=Running out of drive capacity)
    <INFO> 12-Apr-2023::11:54:35.217 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130734 10.255.8.22:6011 (TimeTicks sysUpTime=89545)(OBJECT IDENTIFIER snmpTrapOID=drive-capacity-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:54:35.162734807 UTC)(OCTET STRING alertDescription=Drive usage with in range, used=54%)

**power-fault                    .1.3.6.1.4.1.12276.1.1.1.65545**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include power-fault
    <INFO> 10-Jul-2023::13:43:27.453 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423963 10.255.0.144:161 (TimeTicks sysUpTime=15326)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.975395131 UTC)(OCTET STRING alertDescription=Deasserted: SSD1 12V power fault)
    <INFO> 10-Jul-2023::13:43:27.755 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423966 10.255.0.144:161 (TimeTicks sysUpTime=15356)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.298853193 UTC)(OCTET STRING alertDescription=Deasserted: NSE +3.0V fault)
    <INFO> 10-Jul-2023::13:43:27.855 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423967 10.255.0.144:161 (TimeTicks sysUpTime=15366)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.300188096 UTC)(OCTET STRING alertDescription=Deasserted: ASW +1.12V VCCTGXB fault)
    <INFO> 10-Jul-2023::13:43:27.955 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423968 10.255.0.144:161 (TimeTicks sysUpTime=15376)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.301555964 UTC)(OCTET STRING alertDescription=Deasserted: ATSE2 +1.12V VCCRGXB fault)
    <INFO> 10-Jul-2023::13:43:28.056 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423969 10.255.0.144:161 (TimeTicks sysUpTime=15386)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.302869268 UTC)(OCTET STRING alertDescription=Deasserted: ATSE1 +1.12V VCCRGXB fault)
    <INFO> 10-Jul-2023::13:43:28.156 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423970 10.255.0.144:161 (TimeTicks sysUpTime=15396)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.304281027 UTC)(OCTET STRING alertDescription=Deasserted: CPU +1.0V PVCCANA fault)
    <INFO> 10-Jul-2023::13:43:28.257 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423971 10.255.0.144:161 (TimeTicks sysUpTime=15406)(OBJECT IDENTIFIER snmpTrapOID=power-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.306889907 UTC)(OCTET STRING alertDescription=Deasserted: SUS +1.05V PCH fault)


**thermal-fault                  .1.3.6.1.4.1.12276.1.1.1.65546**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include thermal-fault
    <INFO> 10-Jul-2023::13:43:24.288 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423931 10.255.0.144:161 (TimeTicks sysUpTime=15009)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:18.753307182 UTC)(OCTET STRING alertDescription=NSE_3 at +29.6 degC)
    <INFO> 10-Jul-2023::13:43:24.389 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423932 10.255.0.144:161 (TimeTicks sysUpTime=15019)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:18.754920066 UTC)(OCTET STRING alertDescription=NSE_1 at +30.6 degC)
    <INFO> 10-Jul-2023::13:43:24.489 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423933 10.255.0.144:161 (TimeTicks sysUpTime=15029)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.939393471 UTC)(OCTET STRING alertDescription=ATSE1_6 at +41.8 degC)
    <INFO> 10-Jul-2023::13:43:24.589 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423934 10.255.0.144:161 (TimeTicks sysUpTime=15039)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.941251711 UTC)(OCTET STRING alertDescription=NSE_0 at +30.2 degC)
    <INFO> 10-Jul-2023::13:43:24.690 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423935 10.255.0.144:161 (TimeTicks sysUpTime=15049)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.942774757 UTC)(OCTET STRING alertDescription=ATSE1_4 at +39.4 degC)
    <INFO> 10-Jul-2023::13:43:24.790 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423936 10.255.0.144:161 (TimeTicks sysUpTime=15059)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.944125549 UTC)(OCTET STRING alertDescription=ATSE1_3 at +38.7 degC)
    <INFO> 10-Jul-2023::13:43:24.891 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423937 10.255.0.144:161 (TimeTicks sysUpTime=15069)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.945482464 UTC)(OCTET STRING alertDescription=ATSE2_6 at +41.9 degC)
    <INFO> 10-Jul-2023::13:43:24.991 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423938 10.255.0.144:161 (TimeTicks sysUpTime=15080)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.946879630 UTC)(OCTET STRING alertDescription=ATSE1_1 at +40.0 degC)
    <INFO> 10-Jul-2023::13:43:25.092 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423939 10.255.0.144:161 (TimeTicks sysUpTime=15090)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:19.948228215 UTC)(OCTET STRING alertDescription=ATSE2_4 at +40.5 degC)
    <INFO> 10-Jul-2023::13:43:25.192 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423940 10.255.0.144:161 (TimeTicks sysUpTime=15100)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.940740589 UTC)(OCTET STRING alertDescription=ATSE1_0 at +37.0 degC)
    <INFO> 10-Jul-2023::13:43:25.293 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423941 10.255.0.144:161 (TimeTicks sysUpTime=15110)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.944627829 UTC)(OCTET STRING alertDescription=ATSE2_3 at +38.4 degC)
    <INFO> 10-Jul-2023::13:43:25.393 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423942 10.255.0.144:161 (TimeTicks sysUpTime=15120)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.946325820 UTC)(OCTET STRING alertDescription=outlet at +30.0 degC)
    <INFO> 10-Jul-2023::13:43:25.494 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423943 10.255.0.144:161 (TimeTicks sysUpTime=15130)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.947692185 UTC)(OCTET STRING alertDescription=ATSE2_1 at +40.6 degC)
    <INFO> 10-Jul-2023::13:43:25.594 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423944 10.255.0.144:161 (TimeTicks sysUpTime=15140)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.948945346 UTC)(OCTET STRING alertDescription=inlet at +20.5 degC)
    <INFO> 10-Jul-2023::13:43:25.695 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423945 10.255.0.144:161 (TimeTicks sysUpTime=15150)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.950209788 UTC)(OCTET STRING alertDescription=ATSE2_0 at +36.6 degC)
    <INFO> 10-Jul-2023::13:43:26.499 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423953 10.255.0.144:161 (TimeTicks sysUpTime=15230)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.962459089 UTC)(OCTET STRING alertDescription=Deasserted: VDDQ ABCD VR Hot)
    <INFO> 10-Jul-2023::13:43:26.600 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423954 10.255.0.144:161 (TimeTicks sysUpTime=15240)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.963782808 UTC)(OCTET STRING alertDescription=Deasserted: PCH VNN VR Hot)
    <INFO> 10-Jul-2023::13:43:28.458 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423973 10.255.0.144:161 (TimeTicks sysUpTime=15426)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.309752469 UTC)(OCTET STRING alertDescription=Deasserted: VDDQ EFGH VR Hot)
    <INFO> 10-Jul-2023::13:43:28.558 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423974 10.255.0.144:161 (TimeTicks sysUpTime=15436)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.311144082 UTC)(OCTET STRING alertDescription=Deasserted: EPO VNN VR Hot)
    <INFO> 10-Jul-2023::13:45:26.004 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423994 10.255.0.144:161 (TimeTicks sysUpTime=27181)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:45:25.950878479 UTC)(OCTET STRING alertDescription=CPU TCTL-Delta at -34.0 degC)
    <INFO> 10-Jul-2023::13:45:26.104 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423995 10.255.0.144:161 (TimeTicks sysUpTime=27191)(OBJECT IDENTIFIER snmpTrapOID=thermal-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:45:25.954328495 UTC)(OCTET STRING alertDescription=CPU at +53.0 degC)

**drive-thermal-throttle         .1.3.6.1.4.1.12276.1.1.1.65547**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include drive-thermal-throttle

**blade-thermal-fault            .1.3.6.1.4.1.12276.1.1.1.65548**

This SNMP Trap is for the VELOS system, and it monitors various temperature sensors on each VELOS blade. The sensors monitor CPU, FGPA, and memory temperatures and will warn if the temperature goes beyond recommended guidelines. If a thermal fault occurs you can verify if it has cleared due to a temporary condition. You can also check the system fans to ensure they are operating properly in the VELOS system via the command **show components component fantray-1**. You can also check the environment in which the VELOS system is running to ensure the data center is not operating at too high temperature.

.. code-block:: bash

    syscon-2-active# show components component fantray-1 
    components component fantray-1
    state firmware-version 1.02.798.0.1
    state software-version 2.00.960.0.1
    state serial-no  sub0772g006w
    state part-no    "SUB-0772-05 REV B"
    state empty      false
    properties fantray-state fantray-temperature 23.0
    properties fantray-state inlet-fan-1-speed 6768
    properties fantray-state inlet-fan-2-speed 6699
    properties fantray-state inlet-fan-3-speed 6743
    properties fantray-state exhaust-fan-1-speed 6715
    properties fantray-state exhaust-fan-2-speed 6744
    properties fantray-state exhaust-fan-3-speed 6793
    syscon-2-active#

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include blade-thermal-fault

**blade-hardware-fault           .1.3.6.1.4.1.12276.1.1.1.65549**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include blade-hardware-fault

**sensor-fault                   .1.3.6.1.4.1.12276.1.1.1.65577**

.. code-block:: bash

    syscon-1-active# file show log/confd/snmp.log | include sensor-fault        
    <INFO> 9-Nov-2023::19:21:08.938 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244105 10.255.0.139:162 (TimeTicks sysUpTime=271109396)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927022179 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:21:08.939 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244105 10.255.0.144:162 (TimeTicks sysUpTime=271109396)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927022179 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:21:08.942 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244106 10.255.0.144:162 (TimeTicks sysUpTime=271109396)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927022179 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:21:08.943 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244106 10.255.0.143:162 (TimeTicks sysUpTime=271109396)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927022179 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:21:08.988 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244107 10.255.0.139:162 (TimeTicks sysUpTime=271109401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927133721 UTC)(OCTET STRING alertDescription=Asserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:21:08.989 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244107 10.255.0.144:162 (TimeTicks sysUpTime=271109401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927133721 UTC)(OCTET STRING alertDescription=Asserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:21:08.993 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244108 10.255.0.144:162 (TimeTicks sysUpTime=271109401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927133721 UTC)(OCTET STRING alertDescription=Asserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:21:08.996 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244108 10.255.0.143:162 (TimeTicks sysUpTime=271109401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:21:08.927133721 UTC)(OCTET STRING alertDescription=Asserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:26:08.930 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244111 10.255.0.139:162 (TimeTicks sysUpTime=271139395)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911277769 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:26:08.931 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244111 10.255.0.144:162 (TimeTicks sysUpTime=271139395)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911277769 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:26:08.934 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244112 10.255.0.144:162 (TimeTicks sysUpTime=271139395)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911277769 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:26:08.935 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244112 10.255.0.143:162 (TimeTicks sysUpTime=271139395)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911277769 UTC)(OCTET STRING alertDescription=Sensor fault detected in hardware)
    <INFO> 9-Nov-2023::19:26:08.989 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244113 10.255.0.139:162 (TimeTicks sysUpTime=271139401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911332002 UTC)(OCTET STRING alertDescription=Deasserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:26:08.990 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244113 10.255.0.144:162 (TimeTicks sysUpTime=271139401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911332002 UTC)(OCTET STRING alertDescription=Deasserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:26:08.990 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244114 10.255.0.144:162 (TimeTicks sysUpTime=271139401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911332002 UTC)(OCTET STRING alertDescription=Deasserted: sensor fault: Inlet)
    <INFO> 9-Nov-2023::19:26:08.991 controller-1 confd[604]: snmp snmpv2-trap reqid=1548244114 10.255.0.143:162 (TimeTicks sysUpTime=271139401)(OBJECT IDENTIFIER snmpTrapOID=sensor-fault)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 00:26:08.911332002 UTC)(OCTET STRING alertDescription=Deasserted: sensor fault: Inlet)

**module-present                 .1.3.6.1.4.1.12276.1.1.1.66304**

.. code-block:: bash

    syscon-1-active# file show log/confd/snmp.log | include module-present
    <INFO> 31-Aug-2023::17:29:41.592 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087723 10.255.0.139:162 (TimeTicks sysUpTime=10937)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:16.554609619 UTC)(OCTET STRING alertDescription=Blade6 removed)
    <INFO> 31-Aug-2023::17:29:41.593 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087723 10.255.0.144:162 (TimeTicks sysUpTime=10937)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:16.554609619 UTC)(OCTET STRING alertDescription=Blade6 removed)
    <INFO> 31-Aug-2023::17:29:41.604 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087731 10.255.0.139:162 (TimeTicks sysUpTime=10938)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:16.596222405 UTC)(OCTET STRING alertDescription=Blade4 removed)
    <INFO> 31-Aug-2023::17:29:41.605 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087731 10.255.0.144:162 (TimeTicks sysUpTime=10938)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:16.596222405 UTC)(OCTET STRING alertDescription=Blade4 removed)
    <INFO> 31-Aug-2023::17:29:41.607 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087733 10.255.0.139:162 (TimeTicks sysUpTime=10938)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:16.618843267 UTC)(OCTET STRING alertDescription=Blade5 removed)
    <INFO> 31-Aug-2023::17:29:41.608 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087733 10.255.0.144:162 (TimeTicks sysUpTime=10938)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:16.618843267 UTC)(OCTET STRING alertDescription=Blade5 removed)
    <INFO> 31-Aug-2023::17:29:41.611 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087735 10.255.0.139:162 (TimeTicks sysUpTime=10939)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.006214637 UTC)(OCTET STRING alertDescription=Vpc1 present)
    <INFO> 31-Aug-2023::17:29:41.612 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087735 10.255.0.144:162 (TimeTicks sysUpTime=10939)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.006214637 UTC)(OCTET STRING alertDescription=Vpc1 present)
    <INFO> 31-Aug-2023::17:29:41.614 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087737 10.255.0.139:162 (TimeTicks sysUpTime=10939)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.018550834 UTC)(OCTET STRING alertDescription=Vpc2 present)
    <INFO> 31-Aug-2023::17:29:41.615 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087737 10.255.0.144:162 (TimeTicks sysUpTime=10939)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.018550834 UTC)(OCTET STRING alertDescription=Vpc2 present)
    <INFO> 31-Aug-2023::17:29:41.627 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087745 10.255.0.139:162 (TimeTicks sysUpTime=10940)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.040748272 UTC)(OCTET STRING alertDescription=Blade1 present)
    <INFO> 31-Aug-2023::17:29:41.628 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087745 10.255.0.144:162 (TimeTicks sysUpTime=10940)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.040748272 UTC)(OCTET STRING alertDescription=Blade1 present)
    <INFO> 31-Aug-2023::17:29:41.630 controller-1 confd[604]: snmp snmpv2-trap reqid=1410087747 10.255.0.139:162 (TimeTicks sysUpTime=10941)(OBJECT IDENTIFIER snmpTrapOID=module-present)(OCTET STRING alertSource=controller-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-08-31 21:29:17.051477248 UTC)(OCTET STRING alertDescription=Blade2 present)


**psu-fault                      .1.3.6.1.4.1.12276.1.1.1.66305**

This set of SNMP traps will relate to the health of the power supplies in the rSeries appliances. You may see traps related to insertion or removal of power supplies, inputs, and voltage thresholds. It is best to determine if the trap was a temporary condition, and if not and an error state persists, then determine if the inputs of the power supplies have become disconnected or changed. If the problem only occurs on one power supply, then you can try swapping inputs/power supplies (assuming dual power is installed) during a maintenance window to see if the issue follows the power supply or the input source. 

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include psu-fault
    <INFO> 10-Jul-2023::13:43:13.426 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423818 10.255.0.144:161 (TimeTicks sysUpTime=13923)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-2)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:12.676537826 UTC)(OCTET STRING alertDescription=Deasserted: PSU 2 present)
    <INFO> 10-Jul-2023::13:43:15.336 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423839 10.255.0.144:161 (TimeTicks sysUpTime=14114)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:13.026271463 UTC)(OCTET STRING alertDescription=Asserted: PSU 1 input OK)
    <INFO> 10-Jul-2023::13:43:15.437 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423840 10.255.0.144:161 (TimeTicks sysUpTime=14124)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:13.320285820 UTC)(OCTET STRING alertDescription=Asserted: PSU 1 output OK)
    <INFO> 10-Jul-2023::13:43:15.537 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423841 10.255.0.144:161 (TimeTicks sysUpTime=14134)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:13.695325153 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 unsupported)
    <INFO> 10-Jul-2023::13:43:21.823 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423906 10.255.0.144:161 (TimeTicks sysUpTime=14763)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:16.259904448 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input under-voltage warning)
    <INFO> 10-Jul-2023::13:43:21.924 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423907 10.255.0.144:161 (TimeTicks sysUpTime=14773)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:16.610661807 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input over-voltage warning)
    <INFO> 10-Jul-2023::13:43:22.046 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423908 10.255.0.144:161 (TimeTicks sysUpTime=14785)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:16.937159315 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input over-voltage fault)
    <INFO> 10-Jul-2023::13:43:22.178 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423910 10.255.0.144:161 (TimeTicks sysUpTime=14798)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:17.289095481 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 unit off for low input voltage)
    <INFO> 10-Jul-2023::13:43:22.279 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423911 10.255.0.144:161 (TimeTicks sysUpTime=14808)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:17.710166573 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input under-voltage fault)
    <INFO> 10-Jul-2023::13:43:22.781 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423916 10.255.0.144:161 (TimeTicks sysUpTime=14858)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:18.060160831 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input over-power warning)
    <INFO> 10-Jul-2023::13:43:22.882 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423917 10.255.0.144:161 (TimeTicks sysUpTime=14869)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:18.380302625 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input over-current warning)
    <INFO> 10-Jul-2023::13:43:22.982 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423918 10.255.0.144:161 (TimeTicks sysUpTime=14879)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:18.704106036 UTC)(OCTET STRING alertDescription=Deasserted: PSU 1 input over-current fault)
    <INFO> 10-Jul-2023::13:43:26.650 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423955 10.255.0.144:161 (TimeTicks sysUpTime=15245)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:21.965032296 UTC)(OCTET STRING alertDescription=Asserted: PSU 1 present)
    <INFO> 10-Jul-2023::13:43:27.554 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423964 10.255.0.144:161 (TimeTicks sysUpTime=15336)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-controller)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:22.295486581 UTC)(OCTET STRING alertDescription=Deasserted: PSU mismatch)
    <INFO> 10-Jul-2023::13:43:28.708 appliance-1 confd[130]: snmp snmpv2-trap reqid=1977423977 10.255.0.144:161 (TimeTicks sysUpTime=15451)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-2)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-07-10 17:43:23.951104145 UTC)(OCTET STRING alertDescription=Deasserted: PSU 2 input OK)


**lcd-fault                      .1.3.6.1.4.1.12276.1.1.1.66306**

This set of SNMP traps will relate to the health of the LCD subsystem on rSeries appliances. You may notice lcd-fault traps as the firmware on the LCD is updated as part of an upgrade as seen below. These should be temporary states and eventually the system will generate an **LCD Health is OK** trap. If the system continues to show an LCD fault, a support case should be opened to determine if there is a legitimate hardware issue.

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include lcd-fault
    <INFO> 15-Feb-2023::15:55:35.572 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418268 10.255.0.144:161 (TimeTicks sysUpTime=294)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:55:34.911681272 UTC)(OCTET STRING alertDescription=Fault detected in LCD module)
    <INFO> 15-Feb-2023::15:55:38.088 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418272 10.255.0.144:161 (TimeTicks sysUpTime=545)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:55:38.055131188 UTC)(OCTET STRING alertDescription=Firmware update is running for lcd app)
    <INFO> 15-Feb-2023::15:55:57.476 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418298 10.255.0.144:161 (TimeTicks sysUpTime=2484)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-02-15 20:55:57.472258315 UTC)(OCTET STRING alertDescription=Fault detected in LCD module)
    <INFO> 15-Feb-2023::15:55:57.526 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418299 10.255.0.144:161 (TimeTicks sysUpTime=2489)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:55:57.472273735 UTC)(OCTET STRING alertDescription=LCD Health is Not OK)
    <INFO> 15-Feb-2023::15:58:42.071 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418313 10.255.0.144:161 (TimeTicks sysUpTime=18944)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-02-15 20:58:42.066037341 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 15-Feb-2023::15:58:42.120 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418314 10.255.0.144:161 (TimeTicks sysUpTime=18949)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:58:42.066055066 UTC)(OCTET STRING alertDescription=LCD module communication error detected)
    <INFO> 15-Feb-2023::15:58:42.171 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418315 10.255.0.144:161 (TimeTicks sysUpTime=18954)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:58:42.068393086 UTC)(OCTET STRING alertDescription=Fault detected in LCD module)
    <INFO> 15-Feb-2023::15:58:42.221 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418316 10.255.0.144:161 (TimeTicks sysUpTime=18959)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:58:42.068409568 UTC)(OCTET STRING alertDescription=LCD Health is Not OK)
    <INFO> 15-Feb-2023::15:59:12.060 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418321 10.255.0.144:161 (TimeTicks sysUpTime=21943)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:59:12.056692654 UTC)(OCTET STRING alertDescription=Firmware update completed for lcd app)
    <INFO> 15-Feb-2023::15:59:14.590 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418322 10.255.0.144:161 (TimeTicks sysUpTime=22196)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:59:14.579441541 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 15-Feb-2023::15:59:14.635 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418323 10.255.0.144:161 (TimeTicks sysUpTime=22200)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:59:14.579463512 UTC)(OCTET STRING alertDescription=LCD module communication is OK)
    <INFO> 15-Feb-2023::15:59:14.685 appliance-1 confd[126]: snmp snmpv2-trap reqid=1413418324 10.255.0.144:161 (TimeTicks sysUpTime=22205)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-02-15 20:59:14.588063311 UTC)(OCTET STRING alertDescription=LCD Health is OK)


**module-communication-error     .1.3.6.1.4.1.12276.1.1.1.66307**

Power Supply Module

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include module-communication-error
    <INFO> 12-Apr-2023::11:48:24.877 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130717 10.255.8.22:6011 (TimeTicks sysUpTime=52511)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-04-12 11:48:24.872113844 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 12-Apr-2023::11:48:24.926 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130718 10.255.8.22:6011 (TimeTicks sysUpTime=52516)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:48:24.872136218 UTC)(OCTET STRING alertDescription=PSU communication error detected)
    <INFO> 12-Apr-2023::11:48:37.139 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130719 10.255.8.22:6011 (TimeTicks sysUpTime=53737)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:48:37.136351907 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 12-Apr-2023::11:48:37.189 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130720 10.255.8.22:6011 (TimeTicks sysUpTime=53742)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=psu-1)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:48:37.136369021 UTC)(OCTET STRING alertDescription=PSU communication is OK)


LCD Module

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include module-communication-error
    <INFO> 12-Apr-2023::11:51:32.363 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130725 10.255.8.22:6011 (TimeTicks sysUpTime=71259)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-04-12 11:51:32.359013061 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 12-Apr-2023::11:51:32.413 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130726 10.255.8.22:6011 (TimeTicks sysUpTime=71264)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:51:32.359032524 UTC)(OCTET STRING alertDescription=LCD module communication error detected)
    <INFO> 12-Apr-2023::11:51:32.463 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130727 10.255.8.22:6011 (TimeTicks sysUpTime=71269)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:51:32.361661313 UTC)(OCTET STRING alertDescription=LCD Health is Not OK)
    <INFO> 12-Apr-2023::11:51:45.155 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130728 10.255.8.22:6011 (TimeTicks sysUpTime=72538)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:51:45.150848562 UTC)(OCTET STRING alertDescription=Module communication error detected)
    <INFO> 12-Apr-2023::11:51:45.205 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130729 10.255.8.22:6011 (TimeTicks sysUpTime=72543)(OBJECT IDENTIFIER snmpTrapOID=module-communication-error)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:51:45.150869755 UTC)(OCTET STRING alertDescription=LCD module communication is OK)
    <INFO> 12-Apr-2023::11:51:45.255 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130730 10.255.8.22:6011 (TimeTicks sysUpTime=72549)(OBJECT IDENTIFIER snmpTrapOID=lcd-fault)(OCTET STRING alertSource=lcd)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 11:51:45.156764576 UTC)(OCTET STRING alertDescription=LCD Health is OK)


**coldStart     .1.3.6.1.6.3.1.1.5.1**

.. code-block:: bash

    <INFO> 24-Feb-2024::15:09:35.437 appliance-1 confd[124]: snmp snmpv2-trap reqid=1494830098 10.255.80.251:162 (TimeTicks sysUpTime=552)(OBJECT IDENTIFIER snmpTrapOID=coldStart)
    <INFO> 24-Feb-2024::15:09:35.438 appliance-1 confd[124]: snmp snmpv2-trap reqid=1494830098 10.255.0.144:161 (TimeTicks sysUpTime=552)(OBJECT IDENTIFIER snmpTrapOID=coldStart)


Firmware Update Status Traps
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**firmware-update-status         .1.3.6.1.4.1.12276.1.1.1.65550**

These traps provide indication of the beginning (Firmware update is running) and end (Firmware upgrade has completed) of firmware upgrades for different parts of the system. These may occur as part of a software update to F5OS. Not every upgrade requires firmware to be updated. You may see different components having their firmware upgraded such as (lcd, bios, cpld, lop app, sirr, atse, asw, nso, nvme0, nvme1). It is important not to interrupt the firmware upgrade process. If you see a firmware update alert raised for a specific component, you should not make any changes to the system until each component returns a Firmware update completed message. In newer versions of F5OS, the webUI will display a banner at the top of the page while firmware updates run and will disappear when they complete. The banner will have a link to the **Alarms and Events** page which will show the status of the firmware updates as seen below.


.. image:: images/rseries_monitoring_snmp/imagefirmwareupgrade.png
  :align: center
  :scale: 100%

The CLI command below shows how to filter the **snmp.log** file to only show firmware related events.


.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include firmware
    <INFO> 24-Feb-2022::15:03:43.201 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469870 10.255.0.144:6011 (TimeTicks sysUpTime=526)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:03:40.509604919 UTC)(OCTET STRING alertDescription=Firmware update is running for <no value> 0)
    <INFO> 24-Feb-2022::15:03:43.203 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469871 10.255.0.144:6011 (TimeTicks sysUpTime=526)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:03:43.151642139 UTC)(OCTET STRING alertDescription=Firmware update is running for cpld)
    <INFO> 24-Feb-2022::15:03:57.106 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469872 10.255.0.144:6011 (TimeTicks sysUpTime=1916)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:03:57.104520565 UTC)(OCTET STRING alertDescription=Firmware update completed for atse 0)
    <INFO> 24-Feb-2022::15:03:59.162 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469873 10.255.0.144:6011 (TimeTicks sysUpTime=2122)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:03:59.160527052 UTC)(OCTET STRING alertDescription=Firmware update is running for atse 1)
    <INFO> 24-Feb-2022::15:04:17.153 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469874 10.255.0.144:6011 (TimeTicks sysUpTime=3921)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:04:17.150451625 UTC)(OCTET STRING alertDescription=Firmware update completed for atse 1)
    <INFO> 24-Feb-2022::15:04:17.202 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469875 10.255.0.144:6011 (TimeTicks sysUpTime=3926)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:04:17.153133013 UTC)(OCTET STRING alertDescription=Firmware update is running for nso 0)
    <INFO> 24-Feb-2022::15:04:31.472 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469876 10.255.0.144:6011 (TimeTicks sysUpTime=5353)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:04:31.470147155 UTC)(OCTET STRING alertDescription=Firmware update completed for nso 0)
    <INFO> 24-Feb-2022::15:04:33.165 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469877 10.255.0.144:6011 (TimeTicks sysUpTime=5522)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:04:33.162670549 UTC)(OCTET STRING alertDescription=Firmware update is running for asw 0)
    <INFO> 24-Feb-2022::15:04:47.390 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469878 10.255.0.144:6011 (TimeTicks sysUpTime=6945)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:04:47.387614748 UTC)(OCTET STRING alertDescription=Firmware update completed for asw 0)
    <INFO> 24-Feb-2022::15:12:03.947 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469902 10.255.0.144:6011 (TimeTicks sysUpTime=50600)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:12:03.943198729 UTC)(OCTET STRING alertDescription=Firmware update completed for cpld)
    <INFO> 24-Feb-2022::15:12:05.152 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469903 10.255.0.144:6011 (TimeTicks sysUpTime=50721)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:12:05.150458751 UTC)(OCTET STRING alertDescription=Firmware update is running for lop app)
    <INFO> 24-Feb-2022::15:13:05.154 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469905 10.255.0.144:6011 (TimeTicks sysUpTime=56721)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:13:05.151861316 UTC)(OCTET STRING alertDescription=Firmware update completed for lop app)
    <INFO> 24-Feb-2022::15:13:05.204 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469906 10.255.0.144:6011 (TimeTicks sysUpTime=56726)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:13:05.157391392 UTC)(OCTET STRING alertDescription=Firmware update is running for sirr )
    <INFO> 24-Feb-2022::15:13:05.303 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469907 10.255.0.144:6011 (TimeTicks sysUpTime=56735)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:13:05.164284873 UTC)(OCTET STRING alertDescription=Firmware update completed for sirr )
    <INFO> 24-Feb-2022::15:13:05.347 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469908 10.255.0.144:6011 (TimeTicks sysUpTime=56740)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:13:05.169214668 UTC)(OCTET STRING alertDescription=Firmware update is running for bios)
    <INFO> 24-Feb-2022::15:16:24.434 appliance-1 confd[114]: snmp snmpv2-trap reqid=1908469910 10.255.0.144:6011 (TimeTicks sysUpTime=76649)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-02-24 15:16:24.432163279 UTC)(OCTET STRING alertDescription=Firmware update completed for bios)



Drive Utilization Traps
^^^^^^^^^^^^^^^^^^^^^^^

**drive-utilization              .1.3.6.1.4.1.12276.1.1.1.65551**

The system will monitor the storage utilization of the rSeries disks and warn if the disk usage gets too high. There are 3 levels of events that can occur as seen below:

- drive-capacity:critical-limit - Drive Usage exceeded 97%
- drive-capacity:failure-limit  - Drive Usage exceeded 90%
- drive-capacity:warning-limit  - Drive Usage exceeded 85%

You can use the **show system alarms** CLI command to see if the drive is in an overutilized state. 

.. code-block:: bash

    appliance-1# show system alarms
    ID RESOURCE SEVERITY TEXT TIME CREATED
    --------------------------------------------------------------------------------------------------
    65545 appliance EMERGENCY Power fault detected in hardware 2023-03-24 12:37:13.713715583 UTC
    65544 appliance CRITICAL Running out of drive capacity 2023-03-27 15:41:37.847817761 UTC
    65545 appliance EMERGENCY Power fault detected in hardware 2023-03-24 12:37:13.713715583 UTC

The **show system events** CLI command will provide more details of the drive events that have occurred.

.. code-block:: bash

    appliance-1# show system events | nomore
    system events event
    log "65544 appliance drive-capacity-fault ASSERT CRITICAL \"Running out of drive capacity\" \"2023-03-27 15:41:37.847817761 UTC\""
    system events event
    log "65544 appliance drive-capacity-fault EVENT NA \"Drive usage exceeded 97%, used=100%\" \"2023-03-27 15:41:37.847831437 UTC\""
    system events event
    log "65544 appliance drive-capacity-fault CLEAR CRITICAL \"Running out of drive capacity\" \"2023-03-27 15:42:32.655591036 UTC\""
    system events event
    log "65544 appliance drive-capacity-fault EVENT NA \"Drive usage with in range, used=54%\" \"2023-03-27 15:42:32.655608659 UTC\""

You can also view the snmp.log file to see the SNMP traps that have been issued for **drive-utilization**.

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include drive-utilization
    <INFO> 12-Apr-2023::12:00:00.042 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130742 10.255.8.22:6011 (TimeTicks sysUpTime=122027)(OBJECT IDENTIFIER snmpTrapOID=drive-utilization)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=4)(OCTET STRING alertTimeStamp=2023-04-12 12:00:00.037547416 UTC)(OCTET STRING alertDescription=Drive utilization growth rate is high)
    <INFO> 12-Apr-2023::12:00:00.092 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130743 10.255.8.22:6011 (TimeTicks sysUpTime=122032)(OBJECT IDENTIFIER snmpTrapOID=drive-utilization)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 12:00:00.037560232 UTC)(OCTET STRING alertDescription=Drive usage growth rate exceeded 10%, growth=13%)
    <INFO> 12-Apr-2023::12:00:52.838 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130745 10.255.8.22:6011 (TimeTicks sysUpTime=127307)(OBJECT IDENTIFIER snmpTrapOID=drive-utilization)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 12:00:52.834736965 UTC)(OCTET STRING alertDescription=Drive utilization growth rate is high)
    <INFO> 12-Apr-2023::12:00:52.888 appliance-1 confd[116]: snmp snmpv2-trap reqid=608130746 10.255.8.22:6011 (TimeTicks sysUpTime=127312)(OBJECT IDENTIFIER snmpTrapOID=drive-utilization)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-12 12:00:52.834754109 UTC)(OCTET STRING alertDescription=Drive usage growth rate with in range, growth=-10268%)



FIPS Related Traps
^^^^^^^^^^^^^^^^^^

**fips-fault                     .1.3.6.1.4.1.12276.1.1.1.196308**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include fips-fault
    <INFO> 14-Apr-2023::13:42:57.915 appliance-1 confd[115]: snmp snmpv2-trap reqid=1188695914 10.255.8.22:6011 (TimeTicks sysUpTime=461536)(OBJECT IDENTIFIER snmpTrapOID=fips-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-04-14 13:42:57.910341089 UTC)(OCTET STRING alertDescription=Fault detected in FIPS module)
    <INFO> 14-Apr-2023::13:43:27.924 appliance-1 confd[115]: snmp snmpv2-trap reqid=1188695915 10.255.8.22:6011 (TimeTicks sysUpTime=464537)(OBJECT IDENTIFIER snmpTrapOID=fips-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-14 13:43:27.917797625 UTC)(OCTET STRING alertDescription=Fault detected in FIPS module)
    <INFO> 14-Apr-2023::13:56:57.930 appliance-1 confd[115]: snmp snmpv2-trap reqid=1188695918 10.255.8.22:6011 (TimeTicks sysUpTime=545537)(OBJECT IDENTIFIER snmpTrapOID=fips-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2023-04-14 13:56:57.925072069 UTC)(OCTET STRING alertDescription=Fault detected in FIPS module)
    <INFO> 14-Apr-2023::13:57:27.924 appliance-1 confd[115]: snmp snmpv2-trap reqid=1188695919 10.255.8.22:6011 (TimeTicks sysUpTime=548537)(OBJECT IDENTIFIER snmpTrapOID=fips-fault)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-14 13:57:27.919985256 UTC)(OCTET STRING alertDescription=Fault detected in FIPS module)

**fipsError                      .1.3.6.1.4.1.12276.1.1.1.196608**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include fipsError

System Event Traps
^^^^^^^^^^^^^^^^^^

**core-dump                      .1.3.6.1.4.1.12276.1.1.1.327680**

This trap will indicate that the system has generated a core-dump file. A support case should be opened to diagnose the failure and a qkview should be taken and uploaded to iHealth to capture the diagnostic information for F5 support to analyze. Below is an example of an SNMP trap indicating that the orchestration manager has generated a core dump Files.

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include dump
    <INFO> 27-Apr-2023::07:59:10.169 appliance-1 confd[115]: snmp snmpv2-trap reqid=627600425 10.255.0.144:161 (TimeTicks sysUpTime=223591142)(OBJECT IDENTIFIER snmpTrapOID=core-dump)(OCTET STRING alertSource=Appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-04-27 11:59:10.166591016 UTC)(OCTET STRING alertDescription=Core dumped on Appliance. process=appliance_orche, location=/var/shared/core/container/core.appliance_orch.appliance_orchestration_manager.18120.1682596749.core.gz)

**reboot                         .1.3.6.1.4.1.12276.1.1.1.327681**

This trap will indicate that the system has rebooted. It's possible this was a planned reboot initiated by the administrator. Below is an example of a reboot trap.

.. code-block:: bash

    r10900-1# file show log/system/snmp.log
    <INFO> 17-Nov-2023::12:06:13.587 appliance-1 confd[130]: snmp snmpv2-trap reqid=1025467718 10.255.0.144:161 (TimeTicks sysUpTime=380496)(OBJECT IDENTIFIER snmpTrapOID=reboot)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-17 17:06:13.583723667 UTC)(OCTET STRING alertDescription=System reboot is triggered by user)
    <INFO> 17-Nov-2023::12:09:02.207 appliance-1 confd[117]: snmp snmpv2-trap reqid=1710762179 10.255.0.144:161 (TimeTicks sysUpTime=69)(OBJECT IDENTIFIER snmpTrapOID=coldStart)

**raid-event                     .1.3.6.1.4.1.12276.1.1.1.393216**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include raid-event
    <INFO> 10-Nov-2023::15:05:09.223 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680977 10.255.0.144:161 (TimeTicks sysUpTime=261782586)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=1)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.216697040 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_failed SSD:ssd2)
    <INFO> 10-Nov-2023::15:05:09.274 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680978 10.255.0.144:161 (TimeTicks sysUpTime=261782591)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.264314422 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_ok SSD:ssd1)
    <INFO> 10-Nov-2023::15:05:09.326 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680979 10.255.0.144:161 (TimeTicks sysUpTime=261782596)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=1)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.275871180 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_failed SSD:ssd2)
    <INFO> 10-Nov-2023::15:05:09.377 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680980 10.255.0.144:161 (TimeTicks sysUpTime=261782602)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.318350942 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_ok SSD:ssd1)
    <INFO> 10-Nov-2023::15:05:09.430 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680981 10.255.0.144:161 (TimeTicks sysUpTime=261782607)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=1)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.330028590 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_failed SSD:ssd2)
    <INFO> 10-Nov-2023::15:05:09.481 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680982 10.255.0.144:161 (TimeTicks sysUpTime=261782612)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.373077858 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_ok SSD:ssd1)
    <INFO> 10-Nov-2023::15:05:09.533 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680983 10.255.0.144:161 (TimeTicks sysUpTime=261782617)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=1)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.384442574 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_failed SSD:ssd2)
    <INFO> 10-Nov-2023::15:05:09.584 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680984 10.255.0.144:161 (TimeTicks sysUpTime=261782622)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.425790569 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_ok SSD:ssd1)
    <INFO> 10-Nov-2023::15:05:09.636 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680985 10.255.0.144:161 (TimeTicks sysUpTime=261782627)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=1)(INTEGER alertSeverity=1)(OCTET STRING alertTimeStamp=2023-11-10 20:05:09.437237512 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_failed SSD:ssd2)
    <INFO> 10-Nov-2023::15:07:15.992 appliance-1 confd[130]: snmp snmpv2-trap reqid=1889680986 10.255.0.144:161 (TimeTicks sysUpTime=261795263)(OBJECT IDENTIFIER snmpTrapOID=raidEvent)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=0)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2023-11-10 20:07:15.972123613 UTC)(OCTET STRING alertDescription=RAID STATUS:raid_ok SSD:ssd1)

**backplane                      .1.3.6.1.4.1.12276.1.1.1.262144**

.. code-block:: bash

    r10900-1# file show log/system/snmp.log | include backplane

Interface / Optic Related Traps
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The SNMP traps below will correspond the Digital Diagnostics Monitoring (DDM) that the F5OS layer runs to check the status and health of the fiberoptic transceivers installed. The **show portgroups** CLI command in F5OS will display the current ddm thresholds for warning and alarm as well as current values.


.. code-block:: bash

    r10900-1# show portgroups 
    portgroups portgroup 1
    state vendor-name      "F5 NETWORKS INC."
    state vendor-oui       009065
    state vendor-partnum   "OPT-0031        "
    state vendor-revision  A0
    state vendor-serialnum "X3CAU6G         "
    state transmitter-technology "850 nm VCSEL"
    state media            100GBASE-SR4
    state optic-state      QUALIFIED
    state ddm rx-pwr low-threshold alarm -14.0
    state ddm rx-pwr low-threshold warn -11.0
    state ddm rx-pwr instant val-lane1 -0.66
    state ddm rx-pwr instant val-lane2 -0.77
    state ddm rx-pwr instant val-lane3 -0.79
    state ddm rx-pwr instant val-lane4 -0.9
    state ddm rx-pwr high-threshold alarm 3.4
    state ddm rx-pwr high-threshold warn 2.4
    state ddm tx-pwr low-threshold alarm -10.0
    state ddm tx-pwr low-threshold warn -8.0
    state ddm tx-pwr instant val-lane1 -1.17
    state ddm tx-pwr instant val-lane2 -0.52
    state ddm tx-pwr instant val-lane3 -1.02
    state ddm tx-pwr instant val-lane4 -1.48
    state ddm tx-pwr high-threshold alarm 5.0
    state ddm tx-pwr high-threshold warn 3.0
    state ddm temp low-threshold alarm -5.0
    state ddm temp low-threshold warn 0.0
    state ddm temp instant val 34.5781
    state ddm temp high-threshold alarm 75.0
    state ddm temp high-threshold warn 70.0
    state ddm bias low-threshold alarm 0.003
    state ddm bias low-threshold warn 0.005
    state ddm bias instant val-lane1 0.007494
    state ddm bias instant val-lane2 0.007474
    state ddm bias instant val-lane3 0.007494
    state ddm bias instant val-lane4 0.00746
    state ddm bias high-threshold alarm 0.013
    state ddm bias high-threshold warn 0.011
    state ddm vcc low-threshold alarm 2.97
    state ddm vcc low-threshold warn 3.135
    state ddm vcc instant val 3.3162
    state ddm vcc high-threshold alarm 3.63
    state ddm vcc high-threshold warn 3.465

To keep a balance between the number of DDM alert types that need to be defined and the specifics of the alerts, the type, direction (high/low), and severity uniquely identify each DDM alert type. For example, ddmTempHiWarn is the alert that indicates a high temperature warning condition. Temperature and Voltage (Vcc) are both only specific to the fiber-optic transceiver and not the lanes within Transmitter power, Receiver power, and Transmitter bias are specific to each of the 4 lanes in a fiber-optic transceiver. The lanes that are involved in each alert are embedded at the front of the description string of the alert. A description string might look like: Lanes 1,3 Receiver power low alarm.

Below is an example of the rx-pwr ddm monitoring. There is a low warn threshold of -11.0 and a low alarm threshold of -14.0. There is also a high warn threshold of 2.4 and a high alarm threshold of 3.4. There are 4 lanes for this specific transceiver, and the current readings are all within acceptable ranges. If any of the lanes were to cross the low or high warn or alarm thresholds, then an SNMP trap would be generated.

.. code-block:: bash

    state ddm rx-pwr low-threshold alarm -14.0  <-- Will trigger SNMP Trap for Low Alarm
    state ddm rx-pwr low-threshold warn -11.0   <-- Will trigger SNMP Trap for Low Warn
    state ddm rx-pwr instant val-lane1 -0.66    <-- Current Reading
    state ddm rx-pwr instant val-lane2 -0.77    <-- Current Reading
    state ddm rx-pwr instant val-lane3 -0.79    <-- Current Reading
    state ddm rx-pwr instant val-lane4 -0.9     <-- Current Reading
    state ddm rx-pwr high-threshold alarm 3.4   <-- Will trigger SNMP Trap for High Alarm
    state ddm rx-pwr high-threshold warn 2.4    <-- Will trigger SNMP Trap for High Warn



   

**txPwrHiAlarm                   .1.3.6.1.4.1.12276.1.1.1.262400**

The transmit power threshold for a specific transceiver has reached high alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**txPwrHiWarn                    .1.3.6.1.4.1.12276.1.1.1.262401**

The transmit power threshold for a specific transceiver has reached high warn status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**txPwrLoAlarm                   .1.3.6.1.4.1.12276.1.1.1.262402**

The transmit power threshold for a specific transceiver has reached low alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash

    r10900-2# file show log/system/snmp.log | include tx
    <INFO> 3-May-2024::15:52:04.279 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214842 10.255.80.251:162 (TimeTicks sysUpTime=27848850)(OBJECT IDENTIFIER snmpTrapOID=txPwr)(OCTET STRING alertSource=Portgroup 13)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2024-05-03 19:52:04.263075276 UTC)(OCTET STRING alertDescription=Lanes: 1 Transmitter power low alarm)


**txPwrLoWarn                    .1.3.6.1.4.1.12276.1.1.1.262403**

The transmit power threshold for a specific transceiver has reached low txPwrLoWarn status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**rxPwrHiAlarm                   .1.3.6.1.4.1.12276.1.1.1.262404**

The receive power threshold for a specific transceiver has reached high alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**rxPwrHiWarn                    .1.3.6.1.4.1.12276.1.1.1.262405**

The receive power threshold for a specific transceiver has reached high warn status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**rxPwrLoAlarm                   .1.3.6.1.4.1.12276.1.1.1.262406**

The receive power threshold for a specific transceiver has reached low alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash

    r10900-1# file show log/system/platform.log | include Lane
    2024-01-24T22:28:03.462218-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Notice" version=1.0 msgid=0x2201000000000027 msg="Received alert assert." alert="262401 Portgroup 1 rxPwr ASSERT ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-01-25 03:28:03.458903005 UTC'".
    2024-01-24T22:28:03.564842-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Notice" version=1.0 msgid=0x2201000000000027 msg="Received alert assert." alert="262401 Portgroup 2 rxPwr ASSERT ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-01-25 03:28:03.489226080 UTC'".
    2024-01-25T02:08:32.951999-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Info" version=1.0 msgid=0x2201000000000028 msg="Received alert clear." alert="262401 Portgroup 1 rxPwr CLEAR ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-01-25 07:08:32.946588706 UTC'".
    2024-01-25T02:08:33.057317-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Info" version=1.0 msgid=0x2201000000000028 msg="Received alert clear." alert="262401 Portgroup 2 rxPwr CLEAR ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-01-25 07:08:32.977310897 UTC'".
    2024-02-10T10:23:04.177576-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Notice" version=1.0 msgid=0x2201000000000027 msg="Received alert assert." alert="262401 Portgroup 2 rxPwr ASSERT ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-02-10 15:23:04.165468894 UTC'".
    2024-02-10T10:23:33.732267-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Info" version=1.0 msgid=0x2201000000000028 msg="Received alert clear." alert="262401 Portgroup 2 rxPwr CLEAR ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-02-10 15:23:33.729144778 UTC'".
    2024-02-10T16:35:33.719480-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Notice" version=1.0 msgid=0x2201000000000027 msg="Received alert assert." alert="262401 Portgroup 1 rxPwr ASSERT ERROR 'Lanes: 1,2,3,4 Receiver power low alarm' '2024-02-10 21:35:33.716614320 UTC'".
    2024-02-10T16:35:33.821542-05:00 r10900-1.f5demo2.net alert-service[9]: priority="Notice" version=1.0 msgid=0x2201000000000027 msg="Received alert assert." alert="262401 Portgroup 2 rxPwr ASSERT WARNING 'Lanes: 1,2,3,4 Receiver power low warning' '2024-02-10 21:35:33.747312529 UTC'".
    r10900-1# 


**rxPwrLoWarn                    .1.3.6.1.4.1.12276.1.1.1.262407**

The receive power threshold for a specific transceiver has reached low warn status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**txBiasHiAlarm                  .1.3.6.1.4.1.12276.1.1.1.262408**

The transmit bias threshold for a specific transceiver has reached high alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**txBiasHiWarn                   .1.3.6.1.4.1.12276.1.1.1.262409**

The transmit bias threshold for a specific transceiver has reached high warn status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**txBiasLoAlarm                  .1.3.6.1.4.1.12276.1.1.1.262410**

The transmit bias threshold for a specific transceiver has reached low alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash

    r10900-2# file show log/system/snmp.log | include tx
    <INFO> 3-May-2024::15:52:04.382 r10900-2 confd[152]: snmp snmpv2-trap reqid=961214843 10.255.80.251:162 (TimeTicks sysUpTime=27848860)(OBJECT IDENTIFIER snmpTrapOID=txBias)(OCTET STRING alertSource=Portgroup 13)(INTEGER alertEffect=1)(INTEGER alertSeverity=3)(OCTET STRING alertTimeStamp=2024-05-03 19:52:04.263208264 UTC)(OCTET STRING alertDescription=Lanes: 1 Transmitter bias low alarm)

**txBiasLoWarn                   .1.3.6.1.4.1.12276.1.1.1.262411**

The transmit bias threshold for a specific transceiver has reached low warn status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**ddmTempHiAlarm                 .1.3.6.1.4.1.12276.1.1.1.262412**

The ddm temperature threshold for a specific transceiver has reached high alarm status. Run the show portgroups command to see what the current values are for that transceiver. 

.. code-block:: bash


**ddmTempHiWarn                  .1.3.6.1.4.1.12276.1.1.1.262413**

The ddm temperature threshold for a specific transceiver has reached high warn status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


**ddmTempLoAlarm                 .1.3.6.1.4.1.12276.1.1.1.262414**

The ddm temperature threshold for a specific transceiver has reached low alarm status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


**ddmTempLoWarn                  .1.3.6.1.4.1.12276.1.1.1.262415**

The ddm temperature threshold for a specific transceiver has reached low warn status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


**ddmVccHiAlarm                  .1.3.6.1.4.1.12276.1.1.1.262416**

The ddm vcc (Voltage) threshold for a specific transceiver has reached high alarm status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


**ddmVccHiWarn                   .1.3.6.1.4.1.12276.1.1.1.262417**

The ddm vcc (Voltage) threshold for a specific transceiver has reached high warn status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


**ddmVccLoAlarm                  .1.3.6.1.4.1.12276.1.1.1.262418**

The ddm vcc (Voltage) threshold for a specific transceiver has reached low alarm status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


**ddmVccLoWarn                   .1.3.6.1.4.1.12276.1.1.1.262419**

The ddm vcc (Voltage) threshold for a specific transceiver has reached low warn status. Run the show portgroups command to see what the current values are for that transceiver.

.. code-block:: bash


Enabling SNMP Traps
===================

Enabling SNMP Traps in the CLI for F5OS-A 1.2.0 or Later
--------------------------------------------------------

The SNMP trap CLI configuration has been simplified in the F5OS-A 1.2.0 release and later. Use the **system snmp target** command to configure the SNMP trap destination. The example below uses SNMP v2c and a community string.

.. code-block:: bash

    r5900-2(config)# system snmp targets target v2c-target config community public security-model v2c ipv4 address 10.255.0.144 port 162 
    r5900-2(config-target-v2c-target)# commit
    Commit complete.
    r5900-2(config-target-v2c-target)# 

This example below uses SNMP v3 and uses an SNMP user instead of a community string.

.. code-block:: bash

    r5900-2(config)# system snmp targets target snmp-trap-receiver config user snmpv3-user ipv4 address 10.255.0.144 port 162
    r5900-2(config-target-snmp-trap-receiver)# commit
    Commit complete.
    r5900-2(config-target-v2c-target)# 

You can then view the current SNMP configuration with the **show system snmp targets** command.

.. code-block:: bash

    r5900-2(config)# do show system snmp targets 
                                                                    SECURITY                                     
    NAME                NAME                USER         COMMUNITY  MODEL     ADDRESS       PORT  ADDRESS  PORT  
    -------------------------------------------------------------------------------------------------------------
    snmp-trap-receiver  snmp-trap-receiver  snmpv3-user  -          -         10.255.0.144  162   -        -     
    v2c-target          v2c-target          -            public     v2c       10.255.0.144  162   -        -     

    r5900-2(config)# 


Enabling SNMP Traps in the CLI for Releases Prior to F5OS-A 1.2.0
-----------------------------------------------------------------

For releases prior to F5OS-A 1.2.0, the configuration of SNMP was more difficult, and was done as outlined below. It is provided for reference, but the newer configuration above should be used instead.


Enter **config** mode and enter the following commands to enable SNMP traps for the F5OS-A layer. Specify your SNMP trap receiver's IP address and port after the **snmpTargetAddrTAddress** field. Make sure to **commit** any changes.

Note: The **snmpTargetAddrTAddress** is unintuitive in these earlier releases and is much simpler after upgrading to F5OS-A 1.2.0 or later. In the snmpTargetAddrTAddress, the 1st octet after the IP address is 161 >> 8 = 0, and 2nd octet 161 & 255 = 161. The IP address configuration for an IP address of 10.255.0.144 & 161 UDP port is **10.255.0.144.0.161**.


.. code-block:: bash

    r5900-2# config
    Entering configuration mode terminal
    r5900-2(config)# SNMP-NOTIFICATION-MIB snmpNotifyTable snmpNotifyEntry v2_trap snmpNotifyTag v2_trap snmpNotifyType trap snmpNotifyStorageType nonVolatile 
    r5900-2(config-snmpNotifyEntry-v2_trap)# exit
    r5900-2(config)# SNMP-TARGET-MIB snmpTargetAddrTable snmpTargetAddrEntry group2 snmpTargetAddrTDomain 1.3.6.1.6.1.1 snmpTargetAddrTAddress 10.255.0.144.0.161 snmpTargetAddrTimeout 1500 snmpTargetAddrRetryCount 3 snmpTargetAddrTagList v2_trap snmpTargetAddrParams group2 snmpTargetAddrStorageType nonVolatile snmpTargetAddrEngineID "" snmpTargetAddrTMask "" snmpTargetAddrMMS 2048 enabled
    r5900-2(config-snmpTargetAddrEntry-group2)# exit
    r5900-2(config)# SNMP-TARGET-MIB snmpTargetParamsTable snmpTargetParamsEntry group2 snmpTargetParamsMPModel 1 snmpTargetParamsSecurityModel 2 snmpTargetParamsSecurityName public snmpTargetParamsSecurityLevel noAuthNoPriv snmpTargetParamsStorageType nonVolatile
    r5900-2(config-snmpTargetParamsEntry-group2)# exit
    r5900-2(config)# commit
    Commit complete.
    r5900-2(config)# 

There are various SNMP show commands in the CLI to provide configuration and stats.

.. code-block:: bash

    appliance-1# show SNMP-FRAMEWORK-MIB 
    SNMP-FRAMEWORK-MIB snmpEngine snmpEngineID 80:00:61:81:05:01
    SNMP-FRAMEWORK-MIB snmpEngine snmpEngineBoots 26
    SNMP-FRAMEWORK-MIB snmpEngine snmpEngineTime 15215
    SNMP-FRAMEWORK-MIB snmpEngine snmpEngineMaxMessageSize 50000
    
    appliance-1# show SNMP-MPD-MIB      
    SNMP-MPD-MIB snmpMPDStats snmpUnknownSecurityModels 0
    SNMP-MPD-MIB snmpMPDStats snmpInvalidMsgs 0
    SNMP-MPD-MIB snmpMPDStats snmpUnknownPDUHandlers 0
   
    appliance-1# show SNMP-TARGET-MIB 
    SNMP-TARGET-MIB snmpTargetObjects snmpUnavailableContexts 0
    SNMP-TARGET-MIB snmpTargetObjects snmpUnknownContexts 0
    
    appliance-1# show SNMP-USER-BASED-SM-MIB 
    SNMP-USER-BASED-SM-MIB usmStats usmStatsUnsupportedSecLevels 0
    SNMP-USER-BASED-SM-MIB usmStats usmStatsNotInTimeWindows 0
    SNMP-USER-BASED-SM-MIB usmStats usmStatsUnknownUserNames 0
    SNMP-USER-BASED-SM-MIB usmStats usmStatsUnknownEngineIDs 0
    SNMP-USER-BASED-SM-MIB usmStats usmStatsWrongDigests 0
    SNMP-USER-BASED-SM-MIB usmStats usmStatsDecryptionErrors 0
    
    appliance-1# show SNMPv2-MIB            
    SNMPv2-MIB system sysDescr "Linux 3.10.0-1160.25.1.F5.1.el7_8.x86_64 : Appliance services version 1.1.0-3306"
    SNMPv2-MIB system sysObjectID 1.3.6.1.2.1.1
    SNMPv2-MIB system sysUpTime 1525114
    SNMPv2-MIB system sysServices 72
    SNMPv2-MIB system sysORLastChange 6
    SNMPv2-MIB snmp snmpInPkts 1
    SNMPv2-MIB snmp snmpInBadVersions 0
    SNMPv2-MIB snmp snmpInBadCommunityNames 1
    SNMPv2-MIB snmp snmpInBadCommunityUses 0
    SNMPv2-MIB snmp snmpInASNParseErrs 0
    SNMPv2-MIB snmp snmpSilentDrops 0
    SNMPv2-MIB snmp snmpProxyDrops 0
    SNMPv2-MIB snmpSet snmpSetSerialNo 1200461836
                                                                                                            SYS   
    SYS                                                                                                        ORUP  
    ORINDEX  SYS ORID             SYS ORDESCR                                                                  TIME  
    -----------------------------------------------------------------------------------------------------------------
    1        1.3.6.1.4.1.12276.1  F5 Networks enterprise Platform MIB                                          6     
    2        1.3.6.1.2.1.31       The MIB module to describe generic objects for network interface sub-layers  6     

    appliance-1# 



Enabling SNMP Traps in the webUI
--------------------------------

As of F5OS-A version 1.3.0 you can enable SNMP traps in the webUI. Go to the **System Settings** page, and then select **SNMP Configuration**. Under the **Targets** section, select **Add**. If you are going to use SNMPv3, you should setup an SNMP user first.


.. image:: images/rseries_monitoring_snmp/image6.png
  :align: center
  :scale: 70%

Enter the **Security Model**, **IP Address** and **Port** of the SNMP Trap receiver. You'll be required to add an **SNMP User** when selecting SNMPv3 as the security model.

.. image:: images/rseries_monitoring_snmp/image7.png
  :align: center
  :scale: 100%


Enabling SNMP Traps in the API
------------------------------

To enable SNMP traps via the API, send the following API call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/SNMP-NOTIFICATION-MIB:SNMP-NOTIFICATION-MIB

In the body of the API call include the following:

.. code-block:: json

    {
        "SNMP-NOTIFICATION-MIB:SNMP-NOTIFICATION-MIB": {
            "snmpNotifyTable": {
                "snmpNotifyEntry": [
                    {
                        "snmpNotifyName": "v2_trap",
                        "snmpNotifyTag": "v2_trap",
                        "snmpNotifyType": "trap",
                        "snmpNotifyStorageType": "nonVolatile"
                    }
                ]
            }
        }
    }



.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/SNMP-TARGET-MIB:SNMP-TARGET-MIB

.. code-block:: json

    {
        "SNMP-TARGET-MIB:SNMP-TARGET-MIB": {
            "snmpTargetAddrTable": {
                "snmpTargetAddrEntry": [
                    {
                        "snmpTargetAddrName": "group2",
                        "snmpTargetAddrTDomain": "1.3.6.1.6.1.1",
                        "snmpTargetAddrTAddress": "10.255.0.144.0.161",
                        "snmpTargetAddrTimeout": 1500,
                        "snmpTargetAddrRetryCount": 3,
                        "snmpTargetAddrTagList": "v2_trap",
                        "snmpTargetAddrParams": "group2",
                        "snmpTargetAddrStorageType": "nonVolatile",
                        "snmpTargetAddrEngineID": "",
                        "snmpTargetAddrTMask": "",
                        "snmpTargetAddrMMS": 2048,
                        "enabled": true
                    }
                ]
            },
            "snmpTargetParamsTable": {
                "snmpTargetParamsEntry": [
                    {
                        "snmpTargetParamsName": "group2",
                        "snmpTargetParamsMPModel": 1,
                        "snmpTargetParamsSecurityModel": 2,
                        "snmpTargetParamsSecurityName": "public",
                        "snmpTargetParamsSecurityLevel": "noAuthNoPriv",
                        "snmpTargetParamsStorageType": "nonVolatile"
                    }
                ]
            }
        }
    }






Polling SNMP Endpoints
=====================


Once SNMP is properly setup and allow-lists are enabled you can poll SNMP objects from remote endpoints. If you have an SNMP manager, it is recommended you download the appropriate MIBs from the rSeries appliance and compile them into you SNMP manager. Alternatively, you can use SNMP command line utilities from a remote client to validate the SNMP endpoints. You can then poll/query the appliance via SNMP to get stats from the system using the following SNMP OID’s:

SNMP System
-----------

You can view system parameters such as SysDescr, sysObjectID, sysUptime, sysContact, sysName, sysLocation, sysServices, sysORLastChange, sysORTable, sysDateAndTime by SNMP walking the following OID.

**SNMP System OID: .1.3.6.1.2.1.1**

Example output:

.. code-block:: bash

    prompt% snmpwalk -ObenU -v2c -c public 10.255.2.40 .1.3.6.1.2.1.1
    .1.3.6.1.2.1.1.1.0 = STRING: F5 rSeries-r10900 : Linux 3.10.0-1160.71.1.F5.1.el7_8.x86_64 : Appliance services version 1.8.0-9573
    .1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.12276.1.3.1.2
    .1.3.6.1.2.1.1.3.0 = Timeticks: (19392145) 2 days, 5:52:01.45
    .1.3.6.1.2.1.1.4.0 = STRING: jim@f5.com
    .1.3.6.1.2.1.1.5.0 = STRING: r10900-1.f5demo2.net
    .1.3.6.1.2.1.1.6.0 = STRING: Boston
    .1.3.6.1.2.1.1.7.0 = INTEGER: 72
    .1.3.6.1.2.1.1.8.0 = Timeticks: (6) 0:00:00.06
    .1.3.6.1.2.1.1.9.1.2.1 = OID: .1.3.6.1.4.1.12276.1
    .1.3.6.1.2.1.1.9.1.2.2 = OID: .1.3.6.1.2.1.31
    .1.3.6.1.2.1.1.9.1.3.1 = STRING: F5 Networks enterprise Platform MIB
    .1.3.6.1.2.1.1.9.1.3.2 = STRING: The MIB module to describe generic objects for network interface sub-layers
    .1.3.6.1.2.1.1.9.1.4.1 = Timeticks: (6) 0:00:00.06
    .1.3.6.1.2.1.1.9.1.4.2 = Timeticks: (6) 0:00:00.06
    prompt% 

SNMP ifTable & ifXTable
-----------------------

You can poll the following SNMP OIDs to get detailed Interface stats for each physical port on the rSeries appliances, and for Link Aggregation Groups that have been configured.  Below are table views of the ifTable and ifXTable, you can poll individual interfaces if needed.

**NOTE: Stats for LAG interfaces are not currently populated.**

.. code-block:: bash

    FLD-ML-00054045$ snmptable -v 2c -Cl -CB -Ci -OX -Cb -Cc 32 -Cw 500  -c public 10.255.2.40 ifTable
    SNMP table: IF-MIB::ifTable

    Index                           Descr                           Type                            Mtu                             Speed                           PhysAddress                     AdminStatus                     OperStatus                      LastChange                      InOctets                        InUcastPkts                     InNUcastPkts                    InDiscards                      InErrors                        InUnknownProtos                 
    OutOctets                       OutUcastPkts                    OutNUcastPkts                   OutDiscards                     OutErrors                       OutQLen                         Specific                        

    index: [1]
    1                               r10900 Interface mgmt           ethernetCsmacd                  0                               4294967295                      0:94:a1:69:59:2                 up                              up                              ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554433]
    33554433                        this isin 1                     ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:d                 up                              up                              ?                               ?                               ?                               ?                               3423143                         0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554434]
    33554434                        r10900 Interface 2.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:12                up                              down                            ?                               ?                               ?                               ?                               0                               43290                           ?                               
    ?                               ?                               ?                               0                               54                              ?                               ?                               

    index: [33554435]
    33554435                        r10900 Interface 3.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:e                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554436]
    33554436                        r10900 Interface 4.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:f                 down                            down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554437]
    33554437                        r10900 Interface 5.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:10                up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554438]
    33554438                        r10900 Interface 6.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:11                up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554439]
    33554439                        r10900 Interface 7.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:13                up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554440]
    33554440                        r10900 Interface 8.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:14                up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554441]
    33554441                        r10900 Interface 9.0            ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:15                up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554442]
    33554442                        r10900 Interface 10.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:16                up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554443]
    33554443                        r10900 Interface 11.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:3                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554444]
    33554444                        r10900 Interface 12.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:8                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554445]
    33554445                        r10900 Interface 13.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:4                 up                              up                              ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554446]
    33554446                        r10900 Interface 14.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:5                 up                              up                              ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554447]
    33554447                        r10900 Interface 15.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:6                 up                              up                              ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554448]
    33554448                        r10900 Interface 16.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:7                 up                              up                              ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554449]
    33554449                        test2                           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:9                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554450]
    33554450                        test2                           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:a                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554451]
    33554451                        r10900 Interface 19.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:b                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [33554452]
    33554452                        r10900 Interface 20.0           ethernetCsmacd                  9600                            4294967295                      0:94:a1:69:59:c                 up                              down                            ?                               ?                               ?                               ?                               0                               0                               ?                               
    ?                               ?                               ?                               0                               0                               ?                               ?                               

    index: [67108865]
    67108865                        LAG to Arista                   ieee8023adLag                   9600                            4294967295                      0:94:a1:69:59:24                up                              up                              ?                               ?                               ?                               ?                               ?                               ?                               ?                               
    ?                               ?                               ?                               ?                               ?                               ?                               ?                               

    index: [67108866]
    67108866                        LAG to other r10900             ieee8023adLag                   9600                            4294967295                      0:94:a1:69:59:25                up                              up                              ?                               ?                               ?                               ?                               ?                               ?                               ?                               
    ?                               ?                               ?                               ?                               ?                               ?                               ?                               
    FLD-ML-00054045$

Below is an example of the ifXTable on the rSeries appliance.

.. code-block:: bash

    FLD-ML-00054045$ snmptable -v 2c -Cl -CB -Ci -OX -Cb -Cc 16 -Cw 384  -c public 10.255.2.40 ifXTable
    SNMP table: IF-MIB::ifXTable

    Name            InMulticastPkts InBroadcastPkts OutMulticastPkt OutBroadcastPkt HCInOctets      HCInUcastPkts   HCInMulticastPk HCInBroadcastPk HCOutOctets     HCOutUcastPkts  HCOutMulticastP HCOutBroadcastP LinkUpDownTrapE HighSpeed       PromiscuousMode ConnectorPresen Alias           CounterDisconti 

    index: [1]
    mgmt            ?               ?               ?               ?               928786560       146850          1264798         6763048         52938269        144995          73              20575           ?               1000            ?               ?               ?               ?               

    index: [33554433]
    1.0             ?               ?               ?               ?               455173154       3309922         918021          662903          9392768         0               73381           0               ?               100000          ?               ?               ?               ?               

    index: [33554434]
    2.0             ?               ?               ?               ?               0               0               0               0               278528          0               2176            0               ?               100000          ?               ?               ?               ?               

    index: [33554435]
    3.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554436]
    4.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554437]
    5.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554438]
    6.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554439]
    7.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554440]
    8.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554441]
    9.0             ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554442]
    10.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554443]
    11.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               100000          ?               ?               ?               ?               

    index: [33554444]
    12.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               100000          ?               ?               ?               ?               

    index: [33554445]
    13.0            ?               ?               ?               ?               110217730       0               861080          0               110218498       0               861086          0               ?               25000           ?               ?               ?               ?               

    index: [33554446]
    14.0            ?               ?               ?               ?               110217730       0               861080          0               110218498       0               861086          0               ?               25000           ?               ?               ?               ?               

    index: [33554447]
    15.0            ?               ?               ?               ?               110216320       0               861065          0               110217088       0               861071          0               ?               25000           ?               ?               ?               ?               

    index: [33554448]
    16.0            ?               ?               ?               ?               110216320       0               861065          0               110218498       0               861086          0               ?               25000           ?               ?               ?               ?               

    index: [33554449]
    17.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554450]
    18.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554451]
    19.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               25000           ?               ?               ?               ?               

    index: [33554452]
    20.0            ?               ?               ?               ?               0               0               0               0               0               0               0               0               ?               10000           ?               ?               ?               ?               

    index: [67108865]
    Arista          ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               276447232       ?               ?               ?               ?               

    index: [67108866]
    HA-Interconnect ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               ?               276447232       ?               ?               ?               ?               
    FLD-ML-00054045$




SNMP CPU Processor Stats
---------------------------

The CPU Processor Stats Table provides details on the Intel CPU processors which are running in the rSeries appliance. It displays the core and thread counts, as well as the cache size, frequency and model number.

**F5-PLATFORM-STATS-MIB:cpuProcessorStatsTable OID: .1.3.6.1.4.1.12276.1.2.1.1.1**


Below is an example polling the F5-PLATFORM-STATS-MIB:cpuProcessorStatsTable on an rSeries appliance. Note, that the output below is from an r10900 appliance which has 24 CPU cores which are hyperthreaded, so there are 48 cpuThreadCnt for the whole appliance.

.. code-block:: bash

    prompt%  snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:cpuProcessorStatsTable
    SNMP table: F5-PLATFORM-STATS-MIB::cpuProcessorStatsTable

        index cpuIndex cpuCacheSize cpuCoreCnt       cpuFreq cpuStepping cpuThreadCnt                              cpuModelName
    platform        0    36864(KB)         24 3099.902(MHz)           6           48 Intel(R) Xeon(R) Gold 6312U CPU @ 2.40GHz
    prompt%

SNMP CPU Utilization Stats Table
-------------------------------

The table below shows the total CPU utilization for an rSeries appliance over 5 seconds, 1 minute, and 5 minutes averages as well as the current value.

**F5-PLATFORM-STATS-MIB:cpuUtilizationStatsTable OID: .1.3.6.1.4.1.12276.1.2.1.1.2**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:cpuUtilizationStatsTable
    SNMP table: F5-PLATFORM-STATS-MIB::cpuUtilizationStatsTable

    cpuCore   cpuCurrent cpuTotal5secAvg cpuTotal1minAvg cpuTotal5minAvg
        cpu 1 percentage    1 percentage    1 percentage    1 percentage
    prompt% 

SNMP CPU Core Stats Table
---------------------------

The CPU Core Stats Table shows the total CPU utilization per CPU within an rSeries appliance over 5 seconds, 1 minute, and 5 minutes averages.


**F5-PLATFORM-STATS-MIB:cpuCoreStatsTable OID: .1.3.6.1.4.1.12276.1.2.1.1.3**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:cpuCoreStatsTable
    SNMP table: F5-PLATFORM-STATS-MIB::cpuCoreStatsTable

    coreIndex coreName   coreCurrent coreTotal5secAvg coreTotal1minAvg coreTotal5minAvg
            0     cpu0  0 percentage     0 percentage     0 percentage     1 percentage
            1     cpu1  0 percentage     0 percentage     0 percentage     1 percentage
            2     cpu2  0 percentage     0 percentage     0 percentage     1 percentage
            3     cpu3  0 percentage     0 percentage     0 percentage     1 percentage
            4     cpu4  0 percentage     0 percentage     0 percentage     1 percentage
            5     cpu5  0 percentage     0 percentage     1 percentage     1 percentage
            6     cpu6  1 percentage     2 percentage     1 percentage     1 percentage
            7     cpu7  0 percentage     1 percentage     2 percentage     2 percentage
            8     cpu8  2 percentage     2 percentage     2 percentage     2 percentage
            9     cpu9  1 percentage     1 percentage     2 percentage     1 percentage
            10    cpu10  2 percentage     1 percentage     1 percentage     1 percentage
            11    cpu11  2 percentage     2 percentage     2 percentage     1 percentage
            12    cpu12  1 percentage     3 percentage     2 percentage     1 percentage
            13    cpu13  0 percentage     2 percentage     2 percentage     2 percentage
            14    cpu14  1 percentage     1 percentage     1 percentage     1 percentage
            15    cpu15  0 percentage     1 percentage     1 percentage     1 percentage
            16    cpu16  0 percentage     1 percentage     2 percentage     2 percentage
            17    cpu17  2 percentage     1 percentage     1 percentage     1 percentage
            18    cpu18  1 percentage     1 percentage     1 percentage     1 percentage
            19    cpu19  1 percentage     3 percentage     1 percentage     1 percentage
            20    cpu20  1 percentage     2 percentage     1 percentage     1 percentage
            21    cpu21  3 percentage     2 percentage     1 percentage     1 percentage
            22    cpu22  4 percentage     2 percentage     1 percentage     2 percentage
            23    cpu23  0 percentage     1 percentage     1 percentage     1 percentage
            24    cpu24  2 percentage     2 percentage     2 percentage     2 percentage
            25    cpu25  1 percentage     1 percentage     1 percentage     1 percentage
            26    cpu26  0 percentage     1 percentage     1 percentage     1 percentage
            27    cpu27  0 percentage     0 percentage     1 percentage     1 percentage
            28    cpu28  0 percentage     1 percentage     1 percentage     1 percentage
            29    cpu29  0 percentage     1 percentage     1 percentage     1 percentage
            30    cpu30  0 percentage     3 percentage     4 percentage     2 percentage
            31    cpu31  0 percentage     3 percentage     2 percentage     1 percentage
            32    cpu32  0 percentage     3 percentage     4 percentage     2 percentage
            33    cpu33  1 percentage     1 percentage     1 percentage     1 percentage
            34    cpu34  0 percentage     3 percentage     2 percentage     3 percentage
            35    cpu35  1 percentage     2 percentage     1 percentage     1 percentage
            36    cpu36  1 percentage     2 percentage     3 percentage     2 percentage
            37    cpu37  0 percentage    15 percentage     4 percentage     3 percentage
            38    cpu38  0 percentage     5 percentage     1 percentage     2 percentage
            39    cpu39  0 percentage     1 percentage     1 percentage     2 percentage
            40    cpu40  0 percentage     2 percentage     4 percentage     3 percentage
            41    cpu41  3 percentage     1 percentage     1 percentage     2 percentage
            42    cpu42  2 percentage     1 percentage     2 percentage     2 percentage
            43    cpu43  3 percentage    19 percentage     3 percentage     2 percentage
            44    cpu44  0 percentage     1 percentage     1 percentage     2 percentage
            45    cpu45  0 percentage     1 percentage     1 percentage     2 percentage
            46    cpu46 29 percentage     8 percentage     1 percentage     2 percentage
            47    cpu47  1 percentage     1 percentage     1 percentage     2 percentage
    prompt%

SNMP Disk Info Table
------------------

The following table displays information about the disks installed on an rSeries appliance.

**F5-PLATFORM-STATS-MIB:diskInfoTable OID: .1.3.6.1.4.1.12276.1.2.1.2.1**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:diskInfoTable
    SNMP table: F5-PLATFORM-STATS-MIB::diskInfoTable

    diskName           diskModel diskVendor diskVersion       diskSerialNo diskSize diskType
    nvme0n1 INTEL SSDPE2KX010T8      Intel    VDV10184 PHLJ1082028K1P0FGN 735.00GB     nvme
    nvme1n1 INTEL SSDPE2KX010T8      Intel    VDV10184 PHLJ108203XB1P0FGN 735.00GB     nvme
    prompt%

SNMP Disk Utilization Stats Table
---------------------------------

The table below shows the current disk utilization and performance of the disk on an rSeries appliance.

**F5-PLATFORM-STATS-MIB:diskUtilizationStatsTable OID: .1.3.6.1.4.1.12276.1.2.1.2.2**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:diskUtilizationStatsTable
    SNMP table: F5-PLATFORM-STATS-MIB::diskUtilizationStatsTable

    diskPercentageUsed diskTotalIops diskReadIops diskReadMerged      diskReadBytes diskReadLatencyMs diskWriteIops diskWriteMerged     diskWriteBytes diskWriteLatencyMs
        40 percentage        0 IOPs 7808367 IOPs         430038 180736036864 bytes        1280537 ms 94675416 IOPs        73036236 899072663040 bytes         6921453 ms
        40 percentage        0 IOPs 5505364 IOPs         374782 137111059968 bytes         984633 ms 94675420 IOPs        73036232 899072663040 bytes         6832726 ms
    prompt% 

SNMP Temperature Stats Table
----------------------------

The table below shows the temperature stats for the system.

**F5-PLATFORM-STATS-MIB:temperatureStatsTable OID: .1.3.6.1.4.1.12276.1.2.1.3.1**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:temperatureStatsTable
    SNMP table: F5-PLATFORM-STATS-MIB::temperatureStatsTable

        tempCurrent     tempAverage     tempMinimum     tempMaximum
    26.7 centigrade 27.1 centigrade 26.4 centigrade 29.6 centigrade
    prompt% 

SNMP Memory Stats Table
----------------------

This MIB displays the memory utilization for the system.

**F5-PLATFORM-STATS-MIB:memoryStatsTable OID:.1.3.6.1.4.1.12276.1.2.1.4.1**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:memoryStatsTable
    SNMP table: F5-PLATFORM-STATS-MIB::memoryStatsTable

        memAvailable           memFree memPercentageUsed  memPlatformTotal  memPlatformUsed
    17778171904 bytes 13109547008 bytes     93 percentage 26845536256 bytes 8436613120 bytes
    prompt% 


SNMP FPGA Table
---------------

The FPGA Stats table shows the current FPGA versions. Depending on the rSeries appliance model there may be one or more FPGAs installed. The r2000/r4000 models have no FPGAs. The r5000 models have one Application Traffic Service Engine (ATSE) and one Appliance SWitch (ASW) FPGA. The r10000 and r12000 models have 2 ATSE FPGAs, one ASW FPGA, and an additional FPGA called the Network SOcket (NSO). The output below is from an r10900.

**F5-PLATFORM-STATS-MIB:fpgaTable OID: .1.3.6.1.4.1.12276.1.2.1.5.1**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:fpgaTable
    SNMP table: F5-PLATFORM-STATS-MIB::fpgaTable

    fpgaIndex fpgaVersion
        asw_0      71.4.2
        nso_0      70.4.1
        atse_0      72.5.1
        atse_1      72.5.1
    prompt% 

SNMP Firmware Table
----------------------

This MIB provides the current firmware status and version for all firmware subsystems.

**F5-PLATFORM-STATS-MIB:fwTable OID: 1.3.6.1.4.1.12276.1.2.1.6.1**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:fwTable
    SNMP table: F5-PLATFORM-STATS-MIB::fwTable

                        fwName                         fwVersion configurable fwUpdateStatus
                          QAT0 Lewisburg C62X Crypto/Compression        false              ?
                          QAT1 Lewisburg C62X Crypto/Compression        false              ?
                          QAT2 Lewisburg C62X Crypto/Compression        false              ?
                          QAT3 Lewisburg C62X Crypto/Compression        false              ?
                          QAT4 Lewisburg C62X Crypto/Compression        false              ?
                          QAT5 Lewisburg C62X Crypto/Compression        false              ?
               fw-version-bios                        2.02.145.1        false           none
               fw-version-cpld                          02.0B.00        false           none
               fw-version-sirr                            1.1.72        false           none
            f w-version-lcd-ui                           1.13.12        false           none
            fw-version-bios-me                         4.4.4.603        false           none
            fw-version-lcd-app                     1.01.069.00.1        false              ?
            fw-version-lop-app                      2.00.357.0.1        false           none
        fw-version-drive-nvme0                          VDV10170        false           none
        fw-version-drive-nvme1                          VDV10170        false           none
     fw-version-lcd-bootloader                     1.01.027.00.1        false           none
     fw-version-lop-bootloader                      1.02.062.0.1        false           none
    fw-version-drive-u.2.slot1                          VDV10184        false           none
    fw-version-drive-u.2.slot2                          VDV10184        false           none
    prompt%

SNMP Fantray Stats Table
----------------------

Query the following SNMP OID to get detailed fan speeds.

**F5-PLATFORM-STATS-MIB:fantrayStatsTable  OID: .1.3.6.1.4.1.12276.1.2.1.7.1**

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-PLATFORM-STATS-MIB:fantrayStatsTable                       
    SNMP table: F5-PLATFORM-STATS-MIB::fantrayStatsTable

    fan-1-speed fan-2-speed fan-3-speed fan-4-speed fan-5-speed fan-6-speed fan-7-speed fan-8-speed fan-9-speed fan-10-speed fan-11-speed fan-12-speed
    16251 RPM   16384 RPM   16375 RPM   16242 RPM   16277 RPM   16330 RPM   16366 RPM   16224 RPM   16233 RPM    16207 RPM    16322 RPM    16286 RPM
    prompt% 


SNMP LLDP Configuration Table
-----------------------------

Query the following SNMP OID to get detailed LLDP configuration table.

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-OS-LLDP-MIB:lldpIfConfigTable 
    SNMP table: F5-OS-LLDP-MIB::lldpIfConfigTable

    lldpIfName lldpIfEnabled lldpIfTlvAdvertisement lldpIfTlvmap
            1.0          true                   txrx       130943
            2.0          true                   txrx       130943
            6.0          true                   txrx       130943
            13.0         true                   txrx       130943
            14.0         true                   txrx       130943
            15.0         true                   txrx       130943
            16.0         true                   txrx       130943
    prompt%


SNMP LLDP Neighbors Table
-----------------------------

Query the following SNMP OID to get detailed LLDP neighbors table.

.. code-block:: bash

    prompt% snmptable -v 2c  -c public -m ALL 10.255.2.40 F5-OS-LLDP-MIB:lldpNeighborsTable
    SNMP table: F5-OS-LLDP-MIB::lldpNeighborsTable

    lldpLocalInterface lldpNeighborPortId lldpNeighborChassisId    lldpNeighborPortDesc lldpNeighborSysName     lldpNeighborSysDesc lldpNeighborSysCap lldpNeighborMgmtAddr lldpNeighborPvid lldpNeighborPpvid lldpNeighborVlanName lldpNeighborVlanTag lldpNeighborProtocolIdentity lldpNeighborAutoNego lldpNeighborPmd lldpNeighborMau lldpNeighborAggStatus lldpNeighborAggPortid lldpNeighborMfs lldpNeighborF5ProductModel
                13.0               13.0          f5-wjex-ngkt Jim McCarron's r10900-2  r10900-2.f5demo.net Jim McCarron's r10900-2            1310740                   ::                0                 1                    ?                   0                            1                    1               0               0                     1                     0            9600                     r10900
                14.0               14.0          f5-wjex-ngkt Jim McCarron's r10900-2  r10900-2.f5demo.net Jim McCarron's r10900-2            1310740                   ::                0                 1                    ?                   0                            1                    1               0               0                     1                     0            9600                     r10900
                15.0               15.0          f5-wjex-ngkt Jim McCarron's r10900-2  r10900-2.f5demo.net Jim McCarron's r10900-2            1310740                   ::                0                 1                    ?                   0                            1                    1               0               0                     1                     0            9600                     r10900
                16.0               16.0          f5-wjex-ngkt Jim McCarron's r10900-2  r10900-2.f5demo.net Jim McCarron's r10900-2            1310740                   ::                0                 1                    ?                   0                            1                    1               0               0                     1                     0            9600                     r10900
    prompt% 



Troubleshooting SNMP
====================

There are SNMP logs within each appliance. SNMP information is captured in the **snmp.log** file located with the **/log/system** directory in the F5OS layer:

**Note: The CLI and webUI abstract the full paths for logs so that they are easier to find. If using root access to the bash shell, then the full path to the system controller SNMP logs is **/var/F5/system/log/snmp.log**

To list the files in the **log/system** directory in the CLI use the **file list path log/system** command:

.. code-block:: bash

    r5900-2# file list path log/system/
    entries {
        name 
    audit.log
    confd.log
    devel.log
    devel.log.1
    lcd.log
    lcd.log.1
    lcd.log.2.gz
    lcd.log.3.gz
    lcd.log.4.gz
    lcd.log.5.gz
    logrotate.log
    logrotate.log.1
    logrotate.log.2.gz
    platform.log
    reprogram_chassis_network.log
    rsyslogd_init.log
    snmp.log
    startup.log
    startup.log.prev
    trace/
    vconsole_auth.log
    vconsole_startup.log
    velos.log
    webui/
    }
    r5900-2# 

SNMP information (requests/traps) are captured in the **snmp.log** file located with the **log** directory of each appliance. This is very useful for diagnosing issues with SNMP connectivity. The SNMP logs get rotated, aggregated, and zipped. If SNMP requests are not being logged, be sure that the system doing the SNMP polling has been added to the allowed IP list so that it can access the F5OS layer.


.. code-block:: bash

    appliance-1# file tail -n 30 log/system/snmp.log
    <INFO> 2-Apr-2022::17:10:52.656 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379603 10.255.0.144:6011 (TimeTicks sysUpTime=5013)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-2)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-04-02 17:10:52.654777039 UTC)(OCTET STRING alertDescription=Deasserted: PSU 2 output OK)
    <INFO> 2-Apr-2022::17:10:54.057 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379604 10.255.0.144:6011 (TimeTicks sysUpTime=5153)(OBJECT IDENTIFIER snmpTrapOID=psu-fault)(OCTET STRING alertSource=psu-2)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-04-02 17:10:54.056039741 UTC)(OCTET STRING alertDescription=Deasserted: PSU 2 input OK)
    <INFO> 2-Apr-2022::17:10:58.057 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379605 10.255.0.144:6011 (TimeTicks sysUpTime=5553)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-04-02 17:10:58.054795136 UTC)(OCTET STRING alertDescription=Firmware update completed for nso 0)
    <INFO> 2-Apr-2022::17:10:58.106 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379606 10.255.0.144:6011 (TimeTicks sysUpTime=5558)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-04-02 17:10:58.061700377 UTC)(OCTET STRING alertDescription=Firmware update is running for asw 0)
    <INFO> 2-Apr-2022::17:11:12.639 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379607 10.255.0.144:6011 (TimeTicks sysUpTime=7012)(OBJECT IDENTIFIER snmpTrapOID=firmware-update-status)(OCTET STRING alertSource=appliance)(INTEGER alertEffect=2)(INTEGER alertSeverity=8)(OCTET STRING alertTimeStamp=2022-04-02 17:11:12.637515513 UTC)(OCTET STRING alertDescription=Firmware update completed for asw 0)
    <INFO> 2-Apr-2022::17:11:18.931 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379608 10.255.0.144:6011 (TimeTicks sysUpTime=7641)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554442)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:11:18.940 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379609 10.255.0.144:6011 (TimeTicks sysUpTime=7642)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554443)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:11:18.949 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379610 10.255.0.144:6011 (TimeTicks sysUpTime=7643)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554444)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:11:18.952 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379611 10.255.0.144:6011 (TimeTicks sysUpTime=7643)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554445)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:11:26.107 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379612 10.255.0.144:6011 (TimeTicks sysUpTime=8358)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=1)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:12:11.111 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379613 10.255.0.144:6011 (TimeTicks sysUpTime=12859)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554442)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:12:11.114 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379614 10.255.0.144:6011 (TimeTicks sysUpTime=12859)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554443)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:12:11.116 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379615 10.255.0.144:6011 (TimeTicks sysUpTime=12859)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554444)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:12:11.117 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379616 10.255.0.144:6011 (TimeTicks sysUpTime=12859)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554445)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 2-Apr-2022::17:12:32.813 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379617 10.255.0.144:6011 (TimeTicks sysUpTime=15029)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554442)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:12:44.644 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379618 10.255.0.144:6011 (TimeTicks sysUpTime=16212)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554442)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:08.822 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379619 10.255.0.144:6011 (TimeTicks sysUpTime=18630)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554443)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:10.676 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379620 10.255.0.144:6011 (TimeTicks sysUpTime=18815)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554443)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:20.832 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379621 10.255.0.144:6011 (TimeTicks sysUpTime=19831)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554444)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:36.847 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379622 10.255.0.144:6011 (TimeTicks sysUpTime=21432)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554451)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:39.694 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379623 10.255.0.144:6011 (TimeTicks sysUpTime=21717)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554451)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:44.867 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379624 10.255.0.144:6011 (TimeTicks sysUpTime=22234)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554445)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:57.724 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379625 10.255.0.144:6011 (TimeTicks sysUpTime=23520)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554444)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:13:58.891 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379626 10.255.0.144:6011 (TimeTicks sysUpTime=23637)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554456)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 2-Apr-2022::17:14:07.747 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379627 10.255.0.144:6011 (TimeTicks sysUpTime=24522)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554445)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 3-Apr-2022::03:36:20.153 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379628 10.255.0.144:6011 (TimeTicks sysUpTime=3757763)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554456)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 11-Apr-2022::09:22:48.457 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379629 10.255.0.144:6011 (TimeTicks sysUpTime=74956593)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554456)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 12-Apr-2022::14:55:59.513 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379630 10.255.0.144:6011 (TimeTicks sysUpTime=85595699)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554456)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    <INFO> 12-Apr-2022::16:18:01.054 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379631 10.255.0.144:6011 (TimeTicks sysUpTime=86087853)(OBJECT IDENTIFIER snmpTrapOID=linkUp)(INTEGER ifIndex.0.=33554456)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=1)
    <INFO> 12-Apr-2022::16:18:02.471 appliance-1 confd[104]: snmp snmpv2-trap reqid=1799379632 10.255.0.144:6011 (TimeTicks sysUpTime=86087995)(OBJECT IDENTIFIER snmpTrapOID=linkDown)(INTEGER ifIndex.0.=33554456)(INTEGER ifAdminStatus.0.=1)(INTEGER ifOperStatus.0.=2)
    appliance-1# 

Downloading SNMP Logs from the API
----------------------------------

You can download various logs from the F5OS layer using the F5OS API. To list the current log files in the **log/system/** directory use the following API call.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/f5-utils-file-transfer:file/list

In the body of the API call, add the virtual path you want to list.

.. code-block:: json
 
    {
    "f5-utils-file-transfer:path": "log/system/"
    }

To download a specific log file use the following API call.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/f5-utils-file-transfer:file/f5-file-download:download-file/f5-file-download:start-download

In the body of the API call select **form-data**, and then enter the key/value pairs as seen below. The example provided will download the **snmp.log** file that resides in the **log/system** directory.

.. image:: images/rseries_monitoring_snmp/snmplogdownload.png
  :align: center
  :scale: 70%

If you are using Postman, instead of clicking **Send**, click on the arrow next to Send, and then select **Send and Download**. You will then be prompted to save the file to your local file system.

.. image:: images/rseries_monitoring_snmp/sendanddownload.png
  :align: center
  :scale: 70%











