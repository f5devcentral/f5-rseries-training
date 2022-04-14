===========================================
rSeries F5OS-A SNMP Monitoring and Alerting
===========================================


Within rSeries tenants, SNMP support remains unchanged from existing BIG-IPs. SNMP monitoring and SNMP traps are supported in a similar manner as they are within a vCMP guest. F5OS-A handles the lower level networking, and SNMP MIBs and Traps are supported at this layer. F5OS-A currently supports SNMP v1 and v2c versions. SNMPv3 is currently unsupported in the F5OS-A layer, but is being added in the Q3CY22 timeframe.

In the F5OS-A v1.x.x versions, SNMP support is limited to SNMP Trap support for certain events like link up/down traps, and **IF-MIB** support for the physical interfaces. **IF-MIB**, **EtherLike-MIB**, and the **PLATFORM-STATS-MIB**.

As of F5OS-A 1.x.x the following netSNMP MIBs available are available:

- TRANSPORT-ADDRESS-MIB
- SNMPv2-TC
- SNMPv2 SMI
- SNMPv2-MIB
- SNMPv2-CONF 
- SNMP-VIEW-BASED-ACM-MIB
- SNMP-USER-BASED-SM-MIB
- SNMP-TARGET-MIB
- SNMP-NOTIFICATION-MIB
- SNMP-MPD-MIB
- SNMP-FRAMEWORK-MIB
- SNMP-COMMUNITY-MIB
- RFC1213-MIB
- IPV6-MIB
- IF-MIB
- IANAifType-MIB
- HOST-RESOURCES-MIB
- EtherLike-MIB

As of F5OS-A 1.x.x the following F5OS Appliance MIBs available are available:

- F5-ALERT-DEF-MIB
- F5-COMMON-SMI-MIB
- F5OS-APPLIANCE-ALERT-NOTIF-MIB


As of F5OS-A 1.x.x the following alerts and traps are available:

- Interface UP
- Interface DOWN
- Cold Start
- Hardware device fault detected
- Firmware diagnostic state fault detected
- Unregistered alarm detected
- Fault in memory detected
- Fault in drive detected
- CPU fault detected
- Fault in PCIe device detected
- Running out of drive capacity
- Power fault detected in hardware
- Thermal fault detected in hardware
- Drive has entered a thermal throttle condition
- Thermal fault detected in blade
- Hardware fault detected in blade
- Firmware update status
- Drive utilization growth rate is high
- Service health status
- Change detected in appliance module presence
- PSU fault detected
- Fault detected in LCD module
- Module communication error detected
- Crypto error identified in one or more services
- Detected process crash on the system


Adding Allowed IPs for SNMP
===========================

Adding Allowed IPs for SNMP via CLI
-----------------------------------

By default SNMP traffic is not allowed into the F5OS layer. Before enabling SNMP, you'll need to open up the out-of-band management port on F5OS-A to allow SNMP traffic. Below is an example of allowing an SNMP endpoint at 10.255.0.144 to SNMP poll the system on port 161.


.. code-block:: bash

    appliance-1(config)# system allowed-ips allowed-ip SNMP-144 config ipv4 address 10.255.0.144 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 

Currently you can add one ip address/port pair per **allowed-ip** name. If you require more than one IP address you can add it under another name as seen below. 

.. code-block:: bash

    appliance-1(config)# system allowed-ips allowed-ip SNMP-144 config ipv4 address 10.255.0.144 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


    appliance-1(config)# system allowed-ips allowed-ip SNMP-145 config ipv4 address 10.255.0.145 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 

The **allowed-ips** currently allows a specific IP address, and doesn't support CIDR configurations. This is being added, and will be available in an upcoming F5OS-A release.

Adding Allowed IPs for SNMP via API
-----------------------------------

By default SNMP traffic is not allowed into the F5OS layer. Before enabling SNMP you'll need to open up the out-of-band management port on F5OS-A to allow SNMP traffic. Below is an example of allowing an multiple SNMP endpoints at to access SNMP on the system on port 161.

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

Within the body of the API call, specific IP address/port combinations can be added under a given name. In the current release, you are limited to one IP address/port per name. 

.. code-block:: json

    {
        "allowed-ip": [
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
            },
            {
                "name": "SNMP-142",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.142",
                        "port": 161
                    }
                }
            }
        ]
    }



To view the allowed IP's in the API use the following call.

.. code-block:: bash

    GET https://{{Appliance1_IP}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

The output will show the previously configured allowed-ip's.


.. code-block:: json

    {
        "f5-allowed-ips:allowed-ips": {
            "allowed-ip": [
                {
                    "name": "SNMP",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.143",
                            "port": 161
                        }
                    }
                },
                {
                    "name": "SNMP-WIN-10",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.144",
                            "port": 161
                        }
                    }
                },
                {
                    "name": "SNMP2",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.142",
                            "port": 161
                        }
                    }
                }
            ]
        }
    }


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


If Link Aggregation Groups (LAGs) are configured, decriptions should be added to the LAG interfaces as well:

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

To add descriptions for both the in-band, and out-of-band management ports in the CLI, follow the examples below. The API example below is for the r10000 modela which have 20 interfaces, and one managment port. For the r5000 series models you should adjust for 10 interfaces and one managment.

.. code-block:: bash

    PATCH https://{{Appliance1_IP}}:8888/restconf/data/

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




Configuring SNMP Access
=======================

To enable SNMP you'll need to configure basic SNMP parameters like sytem contact, location and name. Then you'll configure access for specific SNMP communities and versions. Currently SNMP can be setup via CLI or API, but not the webUI. Adding SNMP configuraiton support for the webUI is currently targeted for ??????

Configuring SNMP Access via CLI
-------------------------------

You can configure the SNMP System parameters including the System Contact, System Location, and System Name as seen below:

.. code-block:: bash

    appliance-1(config)# SNMPv2-MIB system sysContact jim@f5.com sysLocation Boston sysName r5900-2
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 

Setting up SNMP can de done from the CLI by enabling the Public SNMP community. Below is an example of enabling SNMP monitoring at the F5OS layer. F5OS only supports read-only access for SNMP monitoring. 

.. code-block:: bash


    appliance-1# config
    Entering configuration mode terminal
    appliance-1(config)# SNMP-COMMUNITY-MIB snmpCommunityTable snmpCommunityEntry public snmpCommunityName public snmpCommunitySecurityName public
    appliance-1(config-snmpCommunityEntry-public)# exit
    appliance-1(config)# SNMP-VIEW-BASED-ACM-MIB vacmSecurityToGroupTable vacmSecurityToGroupEntry 2 public vacmGroupName read-access
    appliance-1(config-vacmSecurityToGroupEntry-2/public)# exit
    appliance-1(config)# SNMP-VIEW-BASED-ACM-MIB vacmSecurityToGroupTable vacmSecurityToGroupEntry 1 public vacmGroupName read-access
    appliance-1(config-vacmSecurityToGroupEntry-1/public)# exit
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 



Configuring SNMP Access via API
-------------------------------

Enabling SNMP Traps
===================

Enabling SNMP Traps in the CLI
------------------------------

Enter **config** mode and enter the following commands to enable SNMP traps for the F5OS-A layer. Specifiy, your SNMP trap receiver's IP address and port after the **snmpTargetAddrTAddress** field. Make sure to **commit** any changes.

**Note: The **snmpTargetAddrTAddress** is currently unintuitive and an enhancement request has been filed to simplify the IP address and port configuration. The Trap target IP receiver, The 1st octet is 161 >> 8 = 0, and 2nd octet 161 & 255 = 161. The IP address configuration for an IP address of 10.255.0.144 & 161 UDP port is "10.255.0.144.0.161".**


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









Polling SNMP Endpoints
=====================


You can then poll the appliance via SNMP to get stats from the system using the following SNMP OID's:

-----------
SNMP System
-----------

SNMP System OID: .1.3.6.1.2.1.1

Exmaple output:

.. code-block:: bash

    sysDescr.0	Linux 3.10.0-862.14.4.el7.centos.plus.x86_64 : Partition services version 1.2.1-10781	OctetString	10.255.0.148:161
    sysObjectID.0	system	OID	10.255.0.148:161
    sysUpTime.0	1 hour 13 minutes 13.88 seconds (439388)	TimeTicks	10.255.0.148:161
    sysContact.0	jim@f5.com	OctetString	10.255.0.148:161
    sysName.0	VELOS-bigpartition	OctetString	10.255.0.148:161
    sysLocation.0	Boston	OctetString	10.255.0.148:161
    sysServices.0	72	Integer	10.255.0.148:161
    .1.3.6.1.2.1.1.8.0	190 milliseconds (19)	TimeTicks	10.255.0.148:161
    .1.3.6.1.2.1.1.9.1.2.1	platform	OID	10.255.0.148:161
    .1.3.6.1.2.1.1.9.1.2.2	.1.3.6.1.2.1.31	OID	10.255.0.148:161

------------
SNMP ifXIndex
------------

You can poll the following SNMP OID to get detailed interface stats for each physical port on the rSeries appliances and also for Link Aggregation Groups that have been configured. 

**NOTE: Stats for LAG interfaces are not currently populated.**

SNMP ifIndex OID: .1.3.6.1.2.1.2.2.1


+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+
| **ifIndex** | **ifDescr**         | **ifType**     | **ifMtu** | **ifSpeed** | **ifPhysAddress**  | **ifAdminStatus** | **ifOperStatus** | **ifLastChange** | **ifInOctets** | **ifInUcastPkts** | **ifInNUcastPkts** | **ifInDiscards** | **ifInErrors** | **ifInUnknownProtos** | **ifOutOctets** | **ifOutUcastPkts** | **ifOutNUcastPkts** | **ifOutDiscards** | **ifOutErrors** | **ifOutQLen** | **ifSpecific** | **Index Value** |
+=============+=====================+================+===========+=============+====================+===================+==================+==================+================+===================+====================+==================+================+=======================+=================+====================+=====================+===================+=================+===============+================+=================+
| 33554441    | Interface-1/1.0     | ethernetCsmacd | 9600      | 4294967295  | 00-94-A1-8E-D0-00  | up                | up               | 0                | 0              | 0                 | 0                  | 33554441         |                |                       |                 |                    |                     |                   |                 |               |                |                 |
+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+
| 33554442    | Interface-1/2.0     | ethernetCsmacd | 9600      | 4294967295  | 00-94-A1-8E-D0-01  | up                | up               | 0                | 0              | 0                 | 0                  | 33554441         |                |                       |                 |                    |                     |                   |                 |               |                |                 |
+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+
| 33554449    | Interface-2/1.0     | ethernetCsmacd | 9600      | 4294967295  | 00-94-A1-8E-D0-80  | up                | up               | 0                | 0              | 0                 | 0                  | 33554441         |                |                       |                 |                    |                     |                   |                 |               |                |                 |
+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+
| 33554450    | Interface-2/2.0     | ethernetCsmacd | 9600      | 4294967295  | 00-94-A1-8E-D0-81  | up                | up               | 0                | 0              | 0                 | 0                  | 33554441         |                |                       |                 |                    |                     |                   |                 |               |                |                 |
+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+
| 67108865    | Arista LAG          | ieee8023adLag  | 9600      | 4294967295  | 00-94-A1-8E-D0-0B  | up                | up               | 0                | 0              | 0                 | 0                  | 33554441         |                |                       |                 |                    |                     |                   |                 |               |                |                 |
+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+
| 67108866    | HA-Interconnect LAG | ieee8023adLag  | 9600      | 4294967295  | 00-94-A1-8E-D0-0C  | up                | up               | 0                | 0              | 0                 | 0                  | 33554441         |                |                       |                 |                    |                     |                   |                 |               |                |                 |
+-------------+---------------------+----------------+-----------+-------------+--------------------+-------------------+------------------+------------------+----------------+-------------------+--------------------+------------------+----------------+-----------------------+-----------------+--------------------+---------------------+-------------------+-----------------+---------------+----------------+-----------------+

---------------------
Chassis Partition CPU
--------------------- 

The CPU Processor Stats Table provides details on the Intel CPU processors, which are running in the BX100 line card. It displays the Core and Thread Counts, as well as the Cache Size, Frequency and Model Number.

SNMP Chassis Partition CPU Processor Stats Table OID: .1.3.6.1.4.1.12276.1.2.1.1.1

+-----------+--------------+------------------+----------------+---------------+-----------------+------------------+------------------------------------------+-----------------------------+
| **Index** | **cpuIndex** | **cpuCacheSize** | **cpuCoreCnt** | **cpuFreq**   | **cpuStepping** | **cpuThreadCnt** | **cpuModelName**                         | **Index Value**             |
+===========+==============+==================+================+===============+=================+==================+==========================================+=============================+
| blade-1   | 0            | 19712(KB)        | 14             | 2552.893(MHz) | 4               | 28               | Intel(R) Xeon(R) D-2177NT CPU @ 1.90GHz  | 7.98.108.97.100.101.45.49.0 |
+-----------+--------------+------------------+----------------+---------------+-----------------+------------------+------------------------------------------+-----------------------------+
| blade-2   | 0            | 19712(KB)        | 14             | 2370.593(MHz) | 4               | 28               | Intel(R) Xeon(R) D-2177NT CPU @ 1.90GHz  | 7.98.108.97.100.101.45.50.0 |
+-----------+--------------+------------------+----------------+---------------+-----------------+------------------+------------------------------------------+-----------------------------+

---------------------------
CPU Utilization Stats Table
---------------------------

The table below shows the total CPU Utilization per blade within a chassis parition over 5 seconds, 1 minute, and 5 minutes averages, as well as the current value.

SNMP CPU Utilization Stas Table OID: .1.3.6.1.4.1.12276.1.2.1.1.2

+-------------+----------------+---------------------+---------------------+---------------------+---------------------------+
| **cpuCore** |	**cpuCurrent** | **cpuTotal5secAvg** | **cpuTotal1minAvg** | **cpuTotal5minAvg** | **Index Value**           |
+=============+================+=====================+=====================+=====================+===========================+
| cpu         | 3              | 4                   | 4                   | 4                   | 7.98.108.97.100.101.45.49 |
+-------------+----------------+---------------------+---------------------+---------------------+---------------------------+
| cpu         | 3              | 4                   | 4                   | 4                   | 7.98.108.97.100.101.45.50 |
+-------------+----------------+---------------------+---------------------+---------------------+---------------------------+

---------------------------
CPU Core Stats Table
---------------------------

The table below shows the total CPU Utilization per vCPU within a chassis parition over 5 seconds, 1 minute, and 5 minutes averages. Below is an example of a 2-blade chassis partition. Each blade has 28 vCPU's or Cores:

SNMP CPU Core Stas Table OID: .1.3.6.1.4.1.12276.1.2.1.1.3


+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| **CoreIndex** | **CoreName** | **CoreCurrent** | **CoreTotal5secAvg** | **CoreTotal1minAvg** | **CoreTotal5minAvg** | **Index Value**               |
+===============+==============+=================+======================+======================+======================+===============================+
| 0             | cpu0         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.0   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 1             | cpu1         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.1   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 2             | cpu2         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.2   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 3             | cpu3         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.3   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 4             | cpu4         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.4   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 5             | cpu5         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.5   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 6             | cpu6         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.6   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 7             | cpu7         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.7   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 8             | cpu8         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.8   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 9             | cpu9         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.9   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 10            | cpu10        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.10  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 11            | cpu11        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.11  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 12            | cpu12        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.12  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 13            | cpu13        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.13  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 14            | cpu14        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.14  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 15            | cpu15        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.15  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 16            | cpu16        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.16  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 17            | cpu17        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.17  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 18            | cpu18        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.18  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 19            | cpu19        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.19  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 20            | cpu20        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.20  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 21            | cpu21        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.21  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 22            | cpu22        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.22  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 23            | cpu23        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.23  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 24            | cpu24        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.24  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 25            | cpu25        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.25  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 26            | cpu26        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.26  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 27            | cpu27        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.49.27  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 0             | cpu0         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.0   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 1             | cpu1         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.1   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 2             | cpu2         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.2   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 3             | cpu3         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.3   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 4             | cpu4         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.4   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 5             | cpu5         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.5   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 6             | cpu6         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.6   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 7             | cpu7         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.7   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 8             | cpu8         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.8   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 9             | cpu9         | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.9   |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 10            | cpu10        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.10  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 11            | cpu11        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.11  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 12            | cpu12        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.12  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 13            | cpu13        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.13  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 14            | cpu14        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.14  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 15            | cpu15        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.15  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 16            | cpu16        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.16  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 17            | cpu17        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.17  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 18            | cpu18        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.18  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 19            | cpu19        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.19  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 20            | cpu20        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.20  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 21            | cpu21        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.21  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 22            | cpu22        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.22  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 23            | cpu23        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.23  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 24            | cpu24        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.24  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 25            | cpu25        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.25  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 26            | cpu26        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.26  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+
| 27            | cpu27        | 7               | 8                    | 8                    | 8                    | 7.98.108.97.100.101.45.50.27  |
+---------------+--------------+-----------------+----------------------+----------------------+----------------------+-------------------------------+

---------------
Disk Info Table
---------------

The following table displays information about the disks installed on each blade in the current chassis partition.

SNMP Disk Info Table OID: .1.3.6.1.4.1.12276.1.2.1.2.1

+--------------+----------------------------+----------------+-----------------+------------------+----------------+--------------+-------------------------------------------------------+
| **diskName** | **diskModel**              | **diskVendor** | **diskVersion** | **diskSerialNo** | **diskSize**   | **diskType** | **Index Value**                                       |
+==============+============================+================+=================+==================+================+==============+=======================================================+
| nvme0n1      | SAMSUNG MZ1LB960HAJQ=00007 | Samsung        | EDA7502Q        | S435NE0MA02828   | 733.00GB       | nvme         | 7.98.108.97.100.101.45.49.7.110.118.109.101.48.110.49 |
+--------------+----------------------------+----------------+-----------------+------------------+----------------+--------------+-------------------------------------------------------+
| nvme0n1      | SAMSUNG MZ1LB960HAJQ=00007 | Samsung        | EDA7502Q        | S435NE0MA00227   | 733.00GB       | nvme         | 7.98.108.97.100.101.45.50.7.110.118.109.101.48.110.49 |
+--------------+----------------------------+----------------+-----------------+------------------+----------------+--------------+-------------------------------------------------------+

----------------------------
Disk Utilization Stats Table
----------------------------

The table below shows the current disk utilzation and performance of the disk on each BX110 blade within the current chassis partition.

SNMP Disk Utilization Stats Table OID: .1.3.6.1.4.1.12276.1.2.1.2.2


+------------------------+-------------------+------------------+--------------------+-------------------+-----------------------+-------------------+---------------------+--------------------+-------------------------+-------------------------------------------------------+
| **diskPercentageUsed** | **diskTotalIops** | **diskReadIops** | **diskReadMerged** | **diskReadBytes** | **diskReadLatencyMs** | **diskWriteIops** | **diskWriteMerged** | **diskWriteBytes** | **diskWriteLatencyMs**  | **Index Value**                                       |                            
+========================+===================+==================+====================+===================+=======================+===================+=====================+====================+=========================+=======================================================+
|                        | 4495              | 0                | 0                  | 4390905           | 13695                 | 20511             | 32907               | 2195945            | 56163                   | 7.98.108.97.100.101.45.49.7.110.118.109.101.48.110.49 |
+------------------------+-------------------+------------------+--------------------+-------------------+-----------------------+-------------------+---------------------+--------------------+-------------------------+-------------------------------------------------------+
|                        | 4495              | 0                | 0                  | 4390905           | 13695                 | 20511             | 32907               | 2195945            | 56163                   | 7.98.108.97.100.101.45.50.7.110.118.109.101.48.110.49 |
+------------------------+-------------------+------------------+--------------------+-------------------+-----------------------+-------------------+---------------------+--------------------+-------------------------+-------------------------------------------------------+

-----------------------
Temperature Stats Table
-----------------------

The table below shows the temperature stats for the current chassis partition.

SNMP Temperature Stats Table OID: .1.3.6.1.4.1.12276.1.2.1.3.1


+----------------+-----------------+-----------------+-----------------+---------------------------+
| **tempCurent** | **tempAverage** | **tempMinimum** | **tempMaximum** | **Index Value**           |                            
+================+=================+=================+=================+===========================+
| 29.0           | 25.8            | 24.0            | 29.0            | 7.98.108.97.100.101.45.49 |
+----------------+-----------------+-----------------+-----------------+---------------------------+
| 29.0           | 26.2            | 24.0            | 30.0            | 7.98.108.97.100.101.45.50 |        
+----------------+-----------------+-----------------+-----------------+---------------------------+

------------------
Memory Stats Table
------------------

SNMP Memory Stats Table OID:.1.3.6.1.4.1.12276.1.2.1.4.1

----------------
FPGA Stats Table
----------------

The FPGA Stats table shows the current FPGA version. There are two different FPGA's on each BX110 line card: The ATSE (Application Traffic Service Engine) and the VQF (VELOS Queuing FPGA). 

SNMP FPGA Stats Table OID: .1.3.6.1.4.1.12276.1.2.1.5.1

+---------------+-----------------+--------------------------------------------------+
| **fpgaIndex** | **fpgaVersion** | **Index Value**                                  |                            
+===============+=================+==================================================+
| vqf_0         | 8.7.12          | 7.98.108.97.100.101.45.49.5.118.113.102.95.48    |
+---------------+-----------------+--------------------------------------------------+
| atse_0        | 7.7.3           | 7.98.108.97.100.101.45.49.6.97.116.115.101.95.48 |  
+---------------+-----------------+--------------------------------------------------+
| vqf_0         | 8.7.12          | 7.98.108.97.100.101.45.49.5.118.113.102.95.48    |
+---------------+-----------------+--------------------------------------------------+
| atse_0        | 7.7.3           | 7.98.108.97.100.101.45.49.6.97.116.115.101.95.48 |  
+---------------+-----------------+--------------------------------------------------+


SNMP Trap Support in F5OS
========================

You can enable SNMP traps for the F5OS layer. The **F5-CTRLR-ALERT-NOTIF-MIB* & the **F5-PARTITION-ALERT-NOTIF-MIB** provide details about supported system controller and chassis partition SNMP traps. Below is the current full list of traps support by F5OS: 



For the system controllers, the following SNMP Traps are supported as of F5OS 1.2.x as defined in the **F5-CTRLR-ALERT-NOTIF-MIB.txt**:

SNMP Trap events that note a fault should also trigger an Alert that can be viewed in the show alters output, in the CLI, WebUI, and API. Once the clear SNMP Trap is sent, it should clear the event form the show events output.

+----------------------------+----------------------------------+
| **Alert**                  | **OID**                          |                            
+============================+==================================+
| lcd-fault                  | .1.3.6.1.4.1.12276.1.1.1.65792   |
+----------------------------+----------------------------------+
| psu-fault                  | .1.3.6.1.4.1.12276.1.1.1.65793   |
+----------------------------+----------------------------------+
| module-present             | .1.3.6.1.4.1.12276.1.1.1.65794   |
+----------------------------+----------------------------------+
| module-communication-error | .1.3.6.1.4.1.12276.1.1.1.65795   |
+----------------------------+----------------------------------+
| psu-redundancy-fault       | .1.3.6.1.4.1.12276.1.1.1.65796   |
+----------------------------+----------------------------------+
| arbitration-state          | .1.3.6.1.4.1.12276.1.1.1.66048   |
+----------------------------+----------------------------------+
| switch-status              | .1.3.6.1.4.1.12276.1.1.1.66049   |
+----------------------------+----------------------------------+
| link-state                 | .1.3.6.1.4.1.12276.1.1.1.66050   |
+----------------------------+----------------------------------+
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
| service-health             | .1.3.6.1.4.1.12276.1.1.1.65552   |
+----------------------------+----------------------------------+
| fipsError                  | .1.3.6.1.4.1.12276.1.1.1.196608  |
+----------------------------+----------------------------------+
| core-dump                  | .1.3.6.1.4.1.12276.1.1.1.327680  |
+----------------------------+----------------------------------+


For the chassis partitions, the following SNMP Traps are supported as of F5OS 1.2.x as defined in the **F5-PARTITION-ALERT-NOTIF-MIB.txt**:

+----------------------------+-----------------------------------+
| **Alert**                  | **OID**                           |                            
+============================+===================================+
| hardware-device-fault      |  .1.3.6.1.4.1.12276.1.1.1.65536   |
+----------------------------+-----------------------------------+
| firmware-fault             |  .1.3.6.1.4.1.12276.1.1.1.65537   |
+----------------------------+-----------------------------------+
| unknown-alarm              |  .1.3.6.1.4.1.12276.1.1.1.65538   |
+----------------------------+-----------------------------------+
| memory-fault               |  .1.3.6.1.4.1.12276.1.1.1.65539   |
+----------------------------+-----------------------------------+
| drive-fault                |  .1.3.6.1.4.1.12276.1.1.1.65540   |
+----------------------------+-----------------------------------+
| cpu-fault                  |  .1.3.6.1.4.1.12276.1.1.1.65541   |
+----------------------------+-----------------------------------+
| pcie-fault                 |  .1.3.6.1.4.1.12276.1.1.1.65542   |
+----------------------------+-----------------------------------+
| aom-fault                  |  .1.3.6.1.4.1.12276.1.1.1.65543   |
+----------------------------+-----------------------------------+
| drive-capacity-fault       |  .1.3.6.1.4.1.12276.1.1.1.65544   |
+----------------------------+-----------------------------------+
| power-fault                |  .1.3.6.1.4.1.12276.1.1.1.65545   |
+----------------------------+-----------------------------------+
| thermal-fault              |  .1.3.6.1.4.1.12276.1.1.1.65546   |
+----------------------------+-----------------------------------+
| drive-thermal-throttle     |  .1.3.6.1.4.1.12276.1.1.1.65547   |
+----------------------------+-----------------------------------+
| blade-thermal-fault        |  .1.3.6.1.4.1.12276.1.1.1.65548   |
+----------------------------+-----------------------------------+
| blade-hardware-fault       |  .1.3.6.1.4.1.12276.1.1.1.65549   |
+----------------------------+-----------------------------------+
| firmware-update-status     |  .1.3.6.1.4.1.12276.1.1.1.65550   |
+----------------------------+-----------------------------------+
| fipsError                  |  .1.3.6.1.4.1.12276.1.1.1.196608  |
+----------------------------+-----------------------------------+
| core-dump                  |  .1.3.6.1.4.1.12276.1.1.1.327680  |
+----------------------------+-----------------------------------+


Troubleshooting SNMP
====================

There are SNMP logs within each appliance. SNMP information is captured in the **snmp.log** file located with the **/log/system** directory in the F5OS layer:

**Note: The CLI and webUI abstract the full paths for logs so that they are easier to find. If using root access to the bash shell, then the full path to the system controller snmp logs is **/var/F5/system/log/snmp.log**

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

SNMP information (requests/traps) are captured in the **snmp.log** file located with the **log** directory of each appliance. This is very usefule for diagnosing issues with SNMP connectivity. The SNMP logs get rotated, aggregated and zipped.


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











