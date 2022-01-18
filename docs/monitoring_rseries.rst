==================
Monitoring rSeries
==================


With the introduction of a new F5OS platform layer, anyone deploying rSeries will need to know the important things for them to monitor to ensure proper health and performance of the system. In addition to getting F5’s recommendation on what to monitor, administrators will require details on how to get access to that information. 

Some admins may want CLI commands to monitor, or API calls to query the system, and others may prefer the GUI. Many customers also use SNMP to monitor and be alerted of system issues and events. For SNMP integrations F5 will provide specific SNMP OID’s that an admin can monitor, and what traps are available for altering. The following sections will outline what sort of monitoring and alerting is available with the new F5OS layer in rSeries.

Accessing the F5OS API
======================

The F5OS platform API’s for the can be reached on port 8888. In this document we will use the Postman tool to access rSeries F5OS platform layer API’s. You can download the Postman tool at:

https://www.postman.com/downloads/

You may also use Curl to get API status. The following curl command is pointed at the system controller floating IP address on port 8888. The example below is using basic authentication. You can request output in JSON by using the following Accept header:

.. code-block:: bash

    $ curl -k https://<Appliance1-IP>:8888/restconf/yang-library-version --header 'Accept: application/yang-data+json' -u admin:<password>

Or you can alter the Accept header to receive output in XML format:

.. code-block:: bash

    $ curl -k https://<Appliance1-IP>:8888/restconf/yang-library-version --header 'Accept: application/yang-data+xml' -u admin:<password>


Appliance Level and System Component Monitoring
===============================================

------------------------------------------
System Inventory / Components from the CLI
------------------------------------------

High level appliance status can be obtained by using the **show components ** command, this will include all the subsystems:

.. code-block:: bash

    appliance-1# show components 
    components component lcd
    state serial-no sub0872g00d5
    state empty false
    components component platform
    state description    "BIG-IP r5900"
    state serial-no      f5-vdvh-bfwi
    state part-no        "200-0411-02 REV 2"
    state empty          false
    state tpm-integrity-status Valid
    state memory available 6973112320
    state memory free 1112055808
    state memory used-percent 95
    state temperature current 25.9
    state temperature average 25.7
    state temperature minimum 24.5
    state temperature maximum 26.8
                                                                                    UPDATE  
    NAME                        NAME  VALUE                              CONFIGURABLE  STATUS  
    -------------------------------------------------------------------------------------------
    QAT0                        -     Lewisburg C62X Crypto/Compression  false         -       
    QAT1                        -     Lewisburg C62X Crypto/Compression  false         -       
    QAT2                        -     Lewisburg C62X Crypto/Compression  false         -       
    fw-version-bios             -     1.02.108.1                         false         none    
    fw-version-bios-me          -     4.4.4.58                           false         none    
    fw-version-cpld             -     02.0A.00                           false         none    
    fw-version-drive-m.2.slot1  -     EDA7602Q                           false         none    
    fw-version-drive-nvme0      -     EDA7602Q                           false         none    
    fw-version-lcd-app          -     1.01.057.00.1                      false         none    
    fw-version-lcd-bootloader   -     1.01.027.00.1                      false         none    
    fw-version-lcd-ui           -     1.5.1                              false         none    
    fw-version-lop-app          -     1.00.214.0.1                       false         none    
    fw-version-lop-bootloader   -     1.02.062.0.1                       false         none    
    fw-version-sirr             -     1.1.29                             false         none    

                                                                                                                            READ                             WRITE    
    DISK                                                                                    TOTAL  READ    READ    READ     LATENCY  WRITE  WRITE   WRITE    LATENCY  
    NAME     MODEL                       VENDOR   VERSION   SERIAL NO       SIZE      TYPE  IOPS   IOPS    MERGED  BYTES    MS       IOPS   MERGED  BYTES    MS       
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------
    nvme0n1  SAMSUNG MZ1LB960HAJQ-00007  Samsung  EDA7602Q  S435NA0NA05748  733.00GB  nvme  10000  106370  90831   5018531  31839    24160  28636   1340660  35097    

    cpu state cpu-utilization thread cpu
    cpu state cpu-utilization current 3
    cpu state cpu-utilization five-second-avg 2
    cpu state cpu-utilization one-minute-avg 2
    cpu state cpu-utilization five-minute-avg 2
    CPU               CORE                           THREAD                                              
    INDEX  CACHESIZE  CNT   FREQ           STEPPING  CNT     MODELNAME                                   
    -----------------------------------------------------------------------------------------------------
    0      24576(KB)  16    2899.951(MHz)  6         32      Intel(R) Xeon(R) Silver 4314 CPU @ 2.40GHz  

                            FIVE    ONE     FIVE    
    THREAD                   SECOND  MINUTE  MINUTE  
    INDEX   THREAD  CURRENT  AVG     AVG     AVG     
    -------------------------------------------------
    0       cpu0    2        1       1       1       
    1       cpu1    1        1       1       1       
    2       cpu2    1        1       1       1       
    3       cpu3    1        1       3       3       
    4       cpu4    1        1       4       3       
    5       cpu5    2        1       3       3       
    6       cpu6    5        2       3       3       
    7       cpu7    2        2       3       3       
    8       cpu8    2        3       2       3       
    9       cpu9    1        3       3       2       
    10      cpu10   2        2       3       3       
    11      cpu11   3        2       2       3       
    12      cpu12   1        1       2       3       
    13      cpu13   0        2       3       3       
    14      cpu14   2        5       3       3       
    15      cpu15   8        3       3       3       
    16      cpu16   3        1       2       2       
    17      cpu17   0        0       1       1       
    18      cpu18   0        0       1       1       
    19      cpu19   5        2       4       3       
    20      cpu20   3        1       2       3       
    21      cpu21   1        1       2       2       
    22      cpu22   0        1       1       3       
    23      cpu23   0        1       3       2       
    24      cpu24   4        1       3       2       
    25      cpu25   4        2       2       3       
    26      cpu26   13       6       2       3       
    27      cpu27   9        3       2       2       
    28      cpu28   10       4       3       3       
    29      cpu29   4        2       2       2       
    30      cpu30   2        1       2       4       
    31      cpu31   19       7       3       2       

    FPGA             
    INDEX   VERSION  
    -----------------
    asw_0   71.2.7   
    atse_0  72.2.5   

    components component psu-1
    state serial-no S92031RC1991
    state part-no M1845
    state empty false
    psu-stats psu-current-in 1.718
    psu-stats psu-current-out 26.562
    psu-stats psu-voltage-in 204.5
    psu-stats psu-voltage-out 12.062
    psu-stats psu-temperature-1 33.0
    psu-stats psu-temperature-2 44.0
    psu-stats psu-temperature-3 48.0
    psu-stats psu-fan-1-speed 9760
    appliance-1# 

If you just want the state and not all the detials:

.. code-block:: bash

    appliance-1# show components component state  
    components component lcd
    state serial-no sub0872g00d5
    state empty false
    components component platform
    state description    "BIG-IP r5900"
    state serial-no      f5-vdvh-bfwi
    state part-no        "200-0411-02 REV 2"
    state empty          false
    state tpm-integrity-status Valid
    state memory available 6973259776
    state memory free 1123016704
    state memory used-percent 95
    state temperature current 26.0
    state temperature average 25.7
    state temperature minimum 24.5
    state temperature maximum 26.8
    components component psu-1
    state serial-no S92031RC1991
    state part-no M1845
    state empty false
    appliance-1# 

Or just the properties:

.. code-block:: bash

    appliance-1# show components component properties 
                                                                                                UPDATE  
    NAME      NAME                        NAME  VALUE                              CONFIGURABLE  STATUS  
    -----------------------------------------------------------------------------------------------------
    lcd                                                                                                  
    platform  QAT0                        -     Lewisburg C62X Crypto/Compression  false         -       
            QAT1                        -     Lewisburg C62X Crypto/Compression  false         -       
            QAT2                        -     Lewisburg C62X Crypto/Compression  false         -       
            fw-version-bios             -     1.02.108.1                         false         none    
            fw-version-bios-me          -     4.4.4.58                           false         none    
            fw-version-cpld             -     02.0A.00                           false         none    
            fw-version-drive-m.2.slot1  -     EDA7602Q                           false         none    
            fw-version-drive-nvme0      -     EDA7602Q                           false         none    
            fw-version-lcd-app          -     1.01.057.00.1                      false         none    
            fw-version-lcd-bootloader   -     1.01.027.00.1                      false         none    
            fw-version-lcd-ui           -     1.5.1                              false         none    
            fw-version-lop-app          -     1.00.214.0.1                       false         none    
            fw-version-lop-bootloader   -     1.02.062.0.1                       false         none    
            fw-version-sirr             -     1.1.29                             false         none    
    psu-1                                                                                                

    appliance-1# 


Or you can view individual subsystems. High level power supply status can be obtained by using the **show components component <psu-#>** command:

.. code-block:: bash

    appliance-1# show components component psu-
        PSU      PSU      PSU      PSU      PSU          PSU          PSU          PSU    
        CURRENT  CURRENT  VOLTAGE  VOLTAGE  TEMPERATURE  TEMPERATURE  TEMPERATURE  FAN 1  
    NAME   IN       OUT      IN       OUT      1            2            3            SPEED  
    -----------------------------------------------------------------------------------------
    psu-1  1.703    26.312   204.5    12.078   33.0         44.0         48.0         9792   

    appliance-1# show components component psu-1
    components component psu-1
    state serial-no S92031RC1991
    state part-no M1845
    state empty false
    psu-stats psu-current-in 1.718
    psu-stats psu-current-out 26.75
    psu-stats psu-voltage-in 204.0
    psu-stats psu-voltage-out 12.062
    psu-stats psu-temperature-1 33.0
    psu-stats psu-temperature-2 44.0
    psu-stats psu-temperature-3 48.0
    psu-stats psu-fan-1-speed 9760
    appliance-1# 


High level power supply status can be obtained by using the **show components component psu-stats** command.

.. code-block:: bash

    appliance-1# show components component psu-stats 
        PSU      PSU      PSU      PSU      PSU          PSU          PSU          PSU    
        CURRENT  CURRENT  VOLTAGE  VOLTAGE  TEMPERATURE  TEMPERATURE  TEMPERATURE  FAN 1  
    NAME   IN       OUT      IN       OUT      1            2            3            SPEED  
    -----------------------------------------------------------------------------------------
    psu-1  1.703    26.0     204.0    12.062   33.0         44.0         48.0         9792   

    appliance-1# 


High level chassis LCD status can be obtained by using the **show components component lcd** command:

.. code-block:: bash

    appliance-1# show components component lcd
    components component lcd
    state serial-no sub0872g00d5
    state empty false
    appliance-1# 

You can view stats on the platfrom CPU and bascic utilization:

.. code-block:: bash

    appliance-1# show components component cpu      
                            FIVE    ONE     FIVE                                                                                                                                  FIVE    ONE     FIVE    
                            SECOND  MINUTE  MINUTE  CPU               CORE                           THREAD                                              THREAD                   SECOND  MINUTE  MINUTE  
    NAME      THREAD  CURRENT  AVG     AVG     AVG     INDEX  CACHESIZE  CNT   FREQ           STEPPING  CNT     MODELNAME                                   INDEX   THREAD  CURRENT  AVG     AVG     AVG     
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    platform  cpu     2        1       2       2       0      24576(KB)  16    2899.951(MHz)  6         32      Intel(R) Xeon(R) Silver 4314 CPU @ 2.40GHz  0       cpu0    0        0       1       1       
                                                                                                                                                            1       cpu1    0        0       1       1       
                                                                                                                                                            2       cpu2    0        1       1       1       
                                                                                                                                                            3       cpu3    1        2       3       3       
                                                                                                                                                            4       cpu4    2        1       2       3       
                                                                                                                                                            5       cpu5    2        3       3       3       
                                                                                                                                                            6       cpu6    4        2       2       3       
                                                                                                                                                            7       cpu7    4        3       3       3       
                                                                                                                                                            8       cpu8    17       5       3       3       
                                                                                                                                                            9       cpu9    4        2       2       3       
                                                                                                                                                            10      cpu10   3        1       3       3       
                                                                                                                                                            11      cpu11   3        1       3       3       
                                                                                                                                                            12      cpu12   2        1       3       3       
                                                                                                                                                            13      cpu13   2        1       4       3       
                                                                                                                                                            14      cpu14   3        1       3       3       
                                                                                                                                                            15      cpu15   2        1       3       3       
                                                                                                                                                            16      cpu16   3        2       2       2       
                                                                                                                                                            17      cpu17   0        1       1       1       
                                                                                                                                                            18      cpu18   0        0       1       1       
                                                                                                                                                            19      cpu19   2        1       2       2       
                                                                                                                                                            20      cpu20   2        2       3       3       
                                                                                                                                                            21      cpu21   3        1       3       3       
                                                                                                                                                            22      cpu22   1        1       2       2       
                                                                                                                                                            23      cpu23   4        1       3       2       
                                                                                                                                                            24      cpu24   7        3       4       3       
                                                                                                                                                            25      cpu25   1        1       6       4       
                                                                                                                                                            26      cpu26   1        1       2       3       
                                                                                                                                                            27      cpu27   1        1       3       2       
                                                                                                                                                            28      cpu28   3        1       2       3       
                                                                                                                                                            29      cpu29   2        1       3       3       
                                                                                                                                                            30      cpu30   2        1       3       3       
                                                                                                                                                            31      cpu31   2        1       2       2       

    appliance-1# 

You can view stats on the storage subsystem:

.. code-block:: bash

    appliance-1# show components component storage  
                                                                                                                                    READ                             WRITE    
            DISK                                                                                    TOTAL  READ    READ    READ     LATENCY  WRITE  WRITE   WRITE    LATENCY  
    NAME      NAME     MODEL                       VENDOR   VERSION   SERIAL NO       SIZE      TYPE  IOPS   IOPS    MERGED  BYTES    MS       IOPS   MERGED  BYTES    MS       
    ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    platform  nvme0n1  SAMSUNG MZ1LB960HAJQ-00007  Samsung  EDA7602Q  S435NA0NA05748  733.00GB  nvme  10000  106370  90831   5018531  31839    24160  28636   1340660  35097    

    appliance-1# 

------------------------------------------
System Inventory / Components from the API
------------------------------------------

Chassis Status
--------------

The overall chassis status can be queried via the following API command:

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-platform:components/component=chassis

.. code-block:: json

    {
        "openconfig-platform:component": [
            {
                "name": "chassis",
                "config": {
                    "name": "chassis"
                },
                "state": {
                    "description": "VELOS CX410",
                    "serial-no": "chs600148s",
                    "part-no": "400-0087-01 REV 1",
                    "empty": false,
                    "f5-platform:nebs": {
                        "capable": false,
                        "enabled": false
                    }
                }
            }
        ]
    }


LCD Status
----------

The chassis LCD panel status can be queried via the following API command:

.. code-block:: bash

    GET https://{{Appliance1-IP}}:8888/restconf/data/openconfig-platform:components/component=lcd

.. code-block:: json

    {
        "openconfig-platform:component": [
            {
                "name": "lcd",
                "config": {
                    "name": "lcd"
                },
                "state": {
                    "serial-no": "sub0872g00d5",
                    "empty": false
                }
            }
        ]
    }


Fantray Status
--------------

The chassis fantray status can be queried via the following API command:

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-platform:components/component=fantray-1

.. code-block:: json

    {
        "openconfig-platform:component": [
            {
                "name": "fantray-1",
                "config": {
                    "name": "fantray-1"
                },
                "state": {
                    "firmware-version": "1.02.798.0.1",
                    "software-version": "1.00.824.0.1",
                    "serial-no": "sub0772g002f",
                    "part-no": "SUB-0772-04 REV A",
                    "empty": false
                }
            }
        ]
    }


Power Supply Status
-------------------

The CX410 chassis can have up to 4 individual power supplies installed. Each can be queried via the following API command. Substitute psu-1, or psu-2 (for dual power systems) at the end of the API call:

.. code-block:: bash

    GET https://{{Appliance1-IP}}:8888/restconf/data/openconfig-platform:components/component=psu-1

.. code-block:: json

{
    "openconfig-platform:component": [
        {
            "name": "psu-1",
            "config": {
                "name": "psu-1"
            },
            "state": {
                "serial-no": "S92031RC1991",
                "part-no": "M1845",
                "empty": false
            },
            "f5-fan-psu-stats:psu-stats": {
                "psu-current-in": "1.718",
                "psu-current-out": "25.937",
                "psu-voltage-in": "204.0",
                "psu-voltage-out": "12.062",
                "psu-temperature-1": "33.0",
                "psu-temperature-2": "43.0",
                "psu-temperature-3": "48.0",
                "psu-fan-1-speed": 9760
            }
        }
    ]
}

Blade Status
------------

There can be up to 8 blades installed in the CX410 chassis. Each one can be queried by changing the blade number at the end:

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-platform:components/component=blade-1

.. code-block:: json

    {
        "openconfig-platform:component": [
            {
                "name": "blade-1",
                "config": {
                    "name": "blade-1"
                },
                "state": {
                    "description": "VELOS BX110",
                    "serial-no": "bld422435s",
                    "part-no": "400-0086-02 REV 2",
                    "empty": false,
                    "f5-platform:nebs": {
                        "capable": true,
                        "enabled": true
                    }
                }
            }
        ]
    }


System Controller 1 & 2 Status
------------------------------

There are 2 redundant system controllers in the CX410 chassis. Each one can be queried using the following API call. Substitute controller=2 to query the second system controller: 

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-platform:components/component=controller-1

Or:

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-platform:components/component=controller-2




The output of the API call above will be broken out into the following detail:

The beginning of the output highlights any equipment failures or mismatches and whether or not the chassis is NEBS enabled. Next is the current status of the platform memory for this system controller showing available, used, and used-precent. Next are the thermal readings for temperature showing **current**, **average**, **minimum**, & **maximum** readings.

.. code-block:: json

    {
        "openconfig-platform:component": [
            {
                "name": "controller-1",
                "config": {
                    "name": "controller-1"
                },
                "state": {
                    "description": "VELOS SX410",
                    "serial-no": "bld422584s",
                    "part-no": "SUB-0881-00 REV B",
                    "empty": false,
                    "f5-platform:tpm-integrity-status": "Valid",
                    "f5-platform:nebs": {
                        "capable": true,
                        "enabled": false
                    },
                    "f5-platform:memory": {
                        "available": "25571659776",
                        "free": "13131718656",
                        "used-percent": 24
                    },
                    "f5-platform:temperature": {
                        "current": "24.1",
                        "average": "24.6",
                        "minimum": "22.9",
                        "maximum": "28.0"
                    }
                },


Next in the output is properties which tracks the various software and BIOS versions:

.. code-block:: json


                "properties": {
                    "property": [
                        {
                            "name": "fw-version-bios",
                            "config": {
                                "name": "fw-version-bios"
                            },
                            "state": {
                                "value": "1.03.006.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-bios-me",
                            "config": {
                                "name": "fw-version-bios-me"
                            },
                            "state": {
                                "value": "4.0.4.211",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-cpld",
                            "config": {
                                "name": "fw-version-cpld"
                            },
                            "state": {
                                "value": "01.03.0A",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-lcd-app",
                            "config": {
                                "name": "fw-version-lcd-app"
                            },
                            "state": {
                                "value": "2.02.113.00.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-lcd-bootloader",
                            "config": {
                                "name": "fw-version-lcd-bootloader"
                            },
                            "state": {
                                "value": "2.01.109.00.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-lop-app",
                            "config": {
                                "name": "fw-version-lop-app"
                            },
                            "state": {
                                "value": "1.00.1067.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-lop-bootloader",
                            "config": {
                                "name": "fw-version-lop-bootloader"
                            },
                            "state": {
                                "value": "1.02.1019.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-vfc-app-fanCtrl1",
                            "config": {
                                "name": "fw-version-vfc-app-fanCtrl1"
                            },
                            "state": {
                                "value": "1.00.824.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-vfc-bootloader-fanCtrl1",
                            "config": {
                                "name": "fw-version-vfc-bootloader-fanCtrl1"
                            },
                            "state": {
                                "value": "1.02.798.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-vpc-app-psuCtrl1",
                            "config": {
                                "name": "fw-version-vpc-app-psuCtrl1"
                            },
                            "state": {
                                "value": "1.00.694.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-vpc-app-psuCtrl2",
                            "config": {
                                "name": "fw-version-vpc-app-psuCtrl2"
                            },
                            "state": {
                                "value": "1.00.694.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-vpc-bootloader-psuCtrl1",
                            "config": {
                                "name": "fw-version-vpc-bootloader-psuCtrl1"
                            },
                            "state": {
                                "value": "1.02.669.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        },
                        {
                            "name": "fw-version-vpc-bootloader-psuCtrl2",
                            "config": {
                                "name": "fw-version-vpc-bootloader-psuCtrl2"
                            },
                            "state": {
                                "value": "1.02.669.0.1",
                                "configurable": false,
                                "f5-platform:update-status": "none"
                            }
                        }
                    ]
                },

The next section covers the storage details of the system:

.. code-block:: json

    "storage": {
                    "state": {
                        "f5-platform:disks": {
                            "disk": [
                                {
                                    "disk-name": "nvme0n1",
                                    "state": {
                                        "model": "SAMSUNG MZ1LB960HAJQ-00007",
                                        "vendor": "Samsung",
                                        "version": "EDA7502Q",
                                        "serial-no": "S435NE0MA00234",
                                        "size": "683.00GB",
                                        "type": "nvme"
                                    }
                                },
                                {
                                    "disk-name": "sda",
                                    "state": {
                                        "model": "USB 3.0",
                                        "vendor": "PNY",
                                        "version": "FD",
                                        "serial-no": "",
                                        "size": "57.00GB",
                                        "type": "usb"
                                    }
                                }
                            ]
                        }
                    }
                },

The last section of this output shows CPU state and stats. There are 8 CPU cores on each system controller, the output below is truncated as the stats are the same for each CPU (0-7). The output shows overall platform CPU utilization including current, five-second-avg, one-minute-avg, and five-minute-avg.

.. code-block:: json

    "cpu": {
                    "state": {
                        "f5-platform:processors": {
                            "processor": [
                                {
                                    "cpu-index": 1,
                                    "state": {
                                        "cachesize": "2048(KB)",
                                        "core-cnt": "8",
                                        "freq": "2200.000(MHz)",
                                        "stepping": "1",
                                        "thread-cnt": "8",
                                        "modelname": "Intel(R) Atom(TM) CPU C3758 @ 2.20GHz"
                                    }
                                }
                            ]
                        },
                        "f5-platform:cpu-utilization": {
                            "core": "cpu",
                            "current": 44,
                            "five-second-avg": 31,
                            "one-minute-avg": 47,
                            "five-minute-avg": 43
                        },
                        "f5-platform:cpu-cores": {
                            "cpu-core": [
                                {
                                    "core-index": 0,
                                    "core": "cpu0",
                                    "current": 34,
                                    "five-second-avg": 24,
                                    "one-minute-avg": 49,
                                    "five-minute-avg": 44
                                },
                                {
                                    "core-index": 1,
                                    "core": "cpu1",
                                    "current": 49,
                                    "five-second-avg": 33,
                                    "one-minute-avg": 44,
                                    "five-minute-avg": 42
                                },
                                {
                                    "core-index": 2,
                                    "core": "cpu2",
                                    "current": 55,
                                    "five-second-avg": 33,
                                    "one-minute-avg": 49,
                                    "five-minute-avg": 44
                                },
                                {
                                    "core-index": 3,
                                    "core": "cpu3",
                                    "current": 36,
                                    "five-second-avg": 34,
                                    "one-minute-avg": 48,
                                    "five-minute-avg": 43
                                },
                                {
                                    "core-index": 4,
                                    "core": "cpu4",
                                    "current": 56,
                                    "five-second-avg": 26,
                                    "one-minute-avg": 46,
                                    "five-minute-avg": 43
                                },
                                {
                                    "core-index": 5,
                                    "core": "cpu5",
                                    "current": 43,
                                    "five-second-avg": 38,
                                    "one-minute-avg": 48,
                                    "five-minute-avg": 43
                                },
                                {
                                    "core-index": 6,
                                    "core": "cpu6",
                                    "current": 44,
                                    "five-second-avg": 33,
                                    "one-minute-avg": 46,
                                    "five-minute-avg": 44
                                },
                                {
                                    "core-index": 7,
                                    "core": "cpu7",
                                    "current": 38,
                                    "five-second-avg": 27,
                                    "one-minute-avg": 46,
                                    "five-minute-avg": 44
                                }
                            ]
                        }
                    }
                }
            }
        ]
    }

--------------------------------------------------
System Inventory / Components Alerting and Logging
--------------------------------------------------

From the system controller GUI there is a high-level status and alerting of any faults for the chassis level components.

.. image:: images/monitoring_velos/image2.png
  :align: center
  :scale: 70%


System Alerts via API
---------------------

Recent system level alerts can be accessed via the API. 

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-system:system/f5-event-log:events

.. code-block:: json


    {
        "f5-event-log:events": {
            "event": [
                {
                    "log": "65543 controller-2 aom-fault EVENT NA \"LOP Runtime fault detected: LOP is not receiving health reports from all installed VFC cards\" \"2021-03-05 04:48:14.485125925 UTC\""
                },
                {
                    "log": "65543 controller-2 aom-fault CLEAR ERROR \"Fault detected in the AOM\" \"2021-03-05 04:48:14.605547335 UTC\""
                },
                {
                    "log": "65543 controller-2 aom-fault EVENT NA \"No LOP Runtime fault detected: LOP is not receiving health reports from all installed VFC cards\" \"2021-03-05 04:48:14.605590242 UTC\""
                },


System Controller Monitoring via CLI
------------------------------------

To see if the openshift cluster is up and running use the **show cluster** command. You should see status for each installed blade and controller in the **Ready** state. Each section under **Stage Name** should show a **Status** of **Done**. During the bootup process you can monitor the status of the individual stages. The most recent openshift logs are displayed, and you can determine if the chassis is healthy or having issues.

.. code-block:: bash

    syscon-2-active# show cluster 
    NAME          STATUS  TIME CREATED          ROLES         CPU  PODS  MEMORY      HUGEPAGES  
    --------------------------------------------------------------------------------------------
    blade-1       Ready   2021-01-30T21:50:32Z  compute       28   250   26112340Ki  102890Mi   
    blade-2       Ready   2021-01-16T08:20:08Z  compute       28   250   26112340Ki  102890Mi   
    blade-3       Ready   2021-01-30T21:50:31Z  compute       28   250   26112340Ki  102890Mi   
    controller-1  Ready   2020-12-08T21:09:45Z  infra,master  -    -     -           -          
    controller-2  Ready   2020-12-08T21:09:45Z  infra,master  -    -     -           -          

    STAGE NAME               STATUS  
    ---------------------------------
    AddingBlade              Done    
    HealthCheck              Done    
    HostedInstall            Done    
    MasterAdditionalInstall  Done    
    MasterInstall            Done    
    NodeBootstrap            Done    
    NodeJoin                 Done    
    Prerequisites            Done    
    ServiceCatalogInstall    Done    
    etcdInstall              Done    

    cluster cluster-status summary-status "Openshift cluster is healthy, and all controllers and blades are ready."
    INDEX  STATUS                                                                                      
    ---------------------------------------------------------------------------------------------------
    0      2021-02-06 18:19:59.445387 -  Orchestration manager startup.                                
    1      2021-02-06 18:20:15.219686 -  Orchestration manager transitioning to active.                
    2      2021-02-06 18:20:16.476607 -  Can now ping controller-1.chassis.local (10.1.3.51).          
    3      2021-02-06 18:20:26.863054 -  Can now ping controller-2.chassis.local (10.1.3.52).          
    4      2021-02-06 18:20:27.727600 -  Successfully ssh'd to CC controller-1.chassis.local.          
    5      2021-02-06 18:20:28.311630 -  Successfully ssh'd to CC controller-2.chassis.local.          
    6      2021-02-06 18:20:43.329803 -  Found valid DNS configuration on controller-2.chassis.local.  
    7      2021-02-06 18:21:23.039277 -  Can now ping blade blade-1.chassis.local (10.1.3.1).          
    8      2021-02-06 18:21:23.274312 -  Can now ping blade blade-2.chassis.local (10.1.3.2).          
    9      2021-02-06 18:21:23.520862 -  Can now ping blade blade-3.chassis.local (10.1.3.3).          
    10     2021-02-06 18:21:56.539448 -  Controller 1 is ready in openshift cluster.                   
    11     2021-02-06 18:21:56.539547 -  Controller 2 is ready in openshift cluster.                   
    12     2021-02-06 18:21:56.539583 -  Blade 1 is ready in openshift cluster.                        
    13     2021-02-06 18:21:56.539618 -  Blade 2 is ready in openshift cluster.                        
    14     2021-02-06 18:21:56.539652 -  Blade 3 is ready in openshift cluster.                        
    15     2021-02-06 18:21:56.539687 -  Openshift cluster is ready.                                   
    16     2021-02-06 18:21:56.541546 -  Successfully SSH'd to blade blade-1.chassis.local.            
    17     2021-02-06 18:21:56.970645 -  Successfully SSH'd to blade blade-2.chassis.local.            
    18     2021-02-06 18:21:57.492814 -  Successfully SSH'd to blade blade-3.chassis.local.            
    19     2021-02-06 18:21:58.312127 -  Openshift cluster is NOT ready.                               
    20     2021-02-06 18:22:19.060573 -  Openshift cluster is ready.                              


In the GUI a high-level status of the system controller HA state, and the ability to force a failover can be done from the **System Settings -> Controller Management** screen. Here you can see system controller 1 & 2 status, and role. You can optionally configure the type of failover with either auto (recommended) or Preferred node.  You can also force a failover from one system controller to the next and perform controller software upgrades. 

.. image:: images/monitoring_velos/image3.png
  :align: center
  :scale: 70%

The dashboard in the system controller GUI also provides high level status of each controller and its current role.

.. image:: images/monitoring_velos/image4.png
  :align: center
  :scale: 70%

Active alarms & events can be viewed form the system controllers **System Settings > Alarms & Events** page:

.. image:: images/monitoring_velos/image5.png
  :align: center
  :scale: 70%


Monitoring the Layer2 Switch Fabric on the System Controllers
-------------------------------------------------------------

This section will outline what status should and can be monitored for the Layer2 switch fabric function on the system controllers. Administrators will want to monitor the internal and external interfaces and LAGs for both status and to view stats to understand current utilization. They will be looking to understand what the utilization of each port is and how is traffic balanced between the two switch fabrics on the system controllers. This section will detail what sort of monitoring is currently supported via CLI, GUI, API, and SNMP, and will also detail any altering, logging, or SNMP traps that are available.

Before getting into what monitoring is supported, it is important to understand how things connect together and their labeling. The diagram below provides the internal interface numbering on the system controllers so that an admin can monitor the status and statistics of each interface. This will give them visibility into the traffic distribution across the backplane and dual switch fabrics.  Link Aggregation is configured on the blade side of the connection, but not on the system controller side. Note that the blade in slot 1 will have two connections, one to system controller 1 interface **1/3.1** and one to system controller 2 interface **2/3.1**, the numbering follows the same logic for other slots:

.. image:: images/monitoring_velos/image6.png
  :align: center
  :scale: 70%

There are also separate control plane connections to each blade which are also put into Link Aggregation Group. Note that the blade in slot 1 will have two connections, one to system controller 1 interface **1/1.1** and one to system controller 2 interface **2/1.1**, the numbering follows the same logic for other slots:

.. image:: images/monitoring_velos/image7.png
  :align: center
  :scale: 70%

Those ports will be joined together in a LAG (Link Aggregation) bundle on the system controller side. Note the LAG connecting to slot 1 is labeled **cplagg_1.1**, slot2 is labeled **cplagg_1.2** etc…:

.. image:: images/monitoring_velos/image8.png
  :align: center
  :scale: 70%

CLI Monitoring of the Layer2 Switch Fabric on the System Controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is a CLI command to monitor all the internal and external ports and LAGs on the dual system controllers as well as the out-of-band management ports. Below is a command to view the stats for one of the backplane ports of the system controller:

.. code-block:: bash

    syscon-2-active# show interfaces interface 1/1.1
    interfaces interface 1/1.1
    state name    1/1.1
    state type    ethernetCsmacd
    state loopback-mode false
    state enabled
    state ifindex 10
    state admin-status UP
    state oper-status UP
    state last-change 61612666625
    state counters in-octets 14937303301
    state counters in-pkts 64279377
    state counters in-unicast-pkts 46181461
    state counters in-broadcast-pkts 3495683
    state counters in-multicast-pkts 14602233
    state counters in-discards 553
    state counters in-errors 0
    state counters in-unknown-protos 0
    state counters in-fcs-errors 0
    state counters out-octets 13859445595
    state counters out-pkts 69051486
    state counters out-unicast-pkts 51154295
    state counters out-broadcast-pkts 13115083
    state counters out-multicast-pkts 4782108
    state counters out-discards 0
    state counters out-errors 0
    hold-time state up 0
    hold-time state down 0
    ethernet state mac-address 5a:a5:5a:01:01:01
    ethernet state auto-negotiate true
    ethernet state duplex-mode FULL
    ethernet state port-speed SPEED_10GB
    ethernet state enable-flow-control false
    ethernet state hw-mac-address 5a:a5:5a:01:01:01
    ethernet state counters in-mac-pause-frames 0
    ethernet state counters in-oversize-frames 3398952
    ethernet state counters in-jabber-frames 0
    ethernet state counters in-fragment-frames 0
    ethernet state counters in-8021q-frames 0
    ethernet state counters in-crc-errors 0
    ethernet state counters out-mac-pause-frames 0
    ethernet state counters out-8021q-frames 0


The **show lacp** CLI command will show both external LAG interfaces if the management ports are bonded together, and internal LAG’s to each slot. In the output below there are 3 blades installed in slots 1-3. They will be labeled **cplagg_1.<slot#>**. The **mgmt_aggr** is a name provided by the admin when the LAG for the external management piorts were configured. This name will be different depending on what the admin chooses for a name.

.. code-block:: bash

    syscon-1-active# show lacp
                                                                                                                                                                                                                                    PARTNER  LACP    LACP    LACP    LACP    LACP             
                                    LACP                     SYSTEM                                                                                                                       OPER                     PARTNER  PORT  PORT     IN      OUT     RX      TX      UNKNOWN  LACP    
    NAME        NAME        INTERVAL  MODE    SYSTEM ID MAC    PRIORITY  INTERFACE  INTERFACE  ACTIVITY  TIMEOUT  SYNCHRONIZATION  AGGREGATABLE  COLLECTING  DISTRIBUTING  SYSTEM ID        KEY   PARTNER ID         KEY      NUM   NUM      PKTS    PKTS    ERRORS  ERRORS  ERRORS   ERRORS  
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    cplagg_1.1  cplagg_1.1  FAST      ACTIVE  0:a:49:ff:96:2   53248     1/1.1      1/1.1      ACTIVE    SHORT    IN_SYNC          true          true        true          0:a:49:ff:96:2   2     0:a:49:ff:96:2     0        4225  2        261162  259897  0       -       -        -       
                                                                        2/1.1      2/1.1      ACTIVE    SHORT    IN_SYNC          true          true        true          0:a:49:ff:96:2   2     0:a:49:ff:96:2     0        8321  4        260829  259557  0       -       -        -       
    cplagg_1.2  cplagg_1.2  FAST      ACTIVE  0:a:49:ff:95:22  53248     1/1.2      1/1.2      ACTIVE    SHORT    IN_SYNC          true          true        true          0:a:49:ff:95:22  3     0:a:49:ff:95:22    0        4226  2        261162  259897  0       -       -        -       
                                                                        2/1.2      2/1.2      ACTIVE    SHORT    IN_SYNC          true          true        true          0:a:49:ff:95:22  3     0:a:49:ff:95:22    0        8322  4        260829  259557  0       -       -        -       
    cplagg_1.3  cplagg_1.3  FAST      ACTIVE  0:a:49:ff:92:62  53248     1/1.3      1/1.3      ACTIVE    SHORT    IN_SYNC          true          true        true          0:a:49:ff:92:62  4     0:a:49:ff:92:62    0        4227  2        261162  259897  0       -       -        -       
                                                                        2/1.3      2/1.3      ACTIVE    SHORT    IN_SYNC          true          true        true          0:a:49:ff:92:62  4     0:a:49:ff:92:62    0        8323  4        260829  259558  0       -       -        -       
    cplagg_1.4  cplagg_1.4  FAST      ACTIVE  -                -                                                                                                                                                                                                                              
    cplagg_1.5  cplagg_1.5  FAST      ACTIVE  -                -                                                                                                                                                                                                                              
    cplagg_1.6  cplagg_1.6  FAST      ACTIVE  -                -                                                                                                                                                                                                                              
    cplagg_1.7  cplagg_1.7  FAST      ACTIVE  -                -                                                                                                                                                                                                                              
    cplagg_1.8  cplagg_1.8  FAST      ACTIVE  -                -                                                                                                                                                                                                                              
    mgmt-aggr   mgmt-aggr   SLOW      ACTIVE  0:94:a1:8e:d0:0  53248     1/mgmt0    1/mgmt0    ACTIVE    LONG     IN_SYNC          true          true        true          0:94:a1:8e:d0:0  10    44:4c:a8:bc:ca:77  10       4608  12       8708    259835  0       -       -        -       
                                                                        2/mgmt0    2/mgmt0    ACTIVE    LONG     IN_SYNC          true          true        true          0:94:a1:8e:d0:0  10    44:4c:a8:bc:ca:77  10       8704  11       8700    259506  0       -       -        -       

    syscon-1-active# 

GUI Monitoring of the Layer2 Switch Fabric on the System Controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the current release there is no backplane interface or LAG monitoring in the system controller GUI. You’ll need to use the CLI or API to get stats/status of the backplane ports or external management ports.

API Monitoring of the Layer2 Switch Fabric on the System Controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following API command will show all system controller Ethernet interfaces and link aggregation (both internal and external) as well as out-of-band management Interfaces.

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-interfaces:interfaces

.. code-block:: json

    {
        "openconfig-interfaces:interfaces": {
            "interface": [
                {
                    "name": "1/1.1",
                    "config": {
                        "name": "1/1.1",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.1",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 10,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92280482278",
                        "counters": {
                            "in-octets": "25930763576",
                            "in-pkts": "81611721",
                            "in-unicast-pkts": "80283080",
                            "in-broadcast-pkts": "1044199",
                            "in-multicast-pkts": "284442",
                            "in-discards": "234",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "4756887206",
                            "out-pkts": "14131402",
                            "out-unicast-pkts": "3522019",
                            "out-broadcast-pkts": "4997025",
                            "out-multicast-pkts": "5612358",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.1"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:01",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:01",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "3768462",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.2",
                    "config": {
                        "name": "1/1.2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.2",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 11,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92285255781",
                        "counters": {
                            "in-octets": "56277978006",
                            "in-pkts": "88976020",
                            "in-unicast-pkts": "88696511",
                            "in-broadcast-pkts": "2220",
                            "in-multicast-pkts": "277289",
                            "in-discards": "161",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "13206772277",
                            "out-pkts": "32586877",
                            "out-unicast-pkts": "16699631",
                            "out-broadcast-pkts": "5189699",
                            "out-multicast-pkts": "10697547",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.2"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:02",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:02",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "12417630",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.3",
                    "config": {
                        "name": "1/1.3",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.3",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 2,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92275142893",
                        "counters": {
                            "in-octets": "354634359",
                            "in-pkts": "2641151",
                            "in-unicast-pkts": "2368952",
                            "in-broadcast-pkts": "2095",
                            "in-multicast-pkts": "270104",
                            "in-discards": "108",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "2988733076",
                            "out-pkts": "10658051",
                            "out-unicast-pkts": "6410863",
                            "out-broadcast-pkts": "3858086",
                            "out-multicast-pkts": "389102",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.3"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:03",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:03",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.4",
                    "config": {
                        "name": "1/1.4",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.4",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 3,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.4"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:04",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:04",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.5",
                    "config": {
                        "name": "1/1.5",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.5",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 12,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.5"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:05",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:05",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.6",
                    "config": {
                        "name": "1/1.6",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.6",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 13,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.6"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:06",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:06",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.7",
                    "config": {
                        "name": "1/1.7",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.7",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 4,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.7"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:07",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:07",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/1.8",
                    "config": {
                        "name": "1/1.8",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/1.8",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 5,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.8"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:01:08",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:01:08",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/2.2",
                    "config": {
                        "name": "1/2.2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/2.2",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 16,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92290207170",
                        "counters": {
                            "in-octets": "35926728797",
                            "in-pkts": "62373338",
                            "in-unicast-pkts": "61424422",
                            "in-broadcast-pkts": "948836",
                            "in-multicast-pkts": "80",
                            "in-discards": "1420",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "18308927413",
                            "out-pkts": "75484207",
                            "out-unicast-pkts": "52192600",
                            "out-broadcast-pkts": "7630195",
                            "out-multicast-pkts": "15661412",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:02:02",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:02:02",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "777070",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/2.3",
                    "config": {
                        "name": "1/2.3",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/2.3",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 17,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92294856723",
                        "counters": {
                            "in-octets": "35903131962",
                            "in-pkts": "62358618",
                            "in-unicast-pkts": "61409038",
                            "in-broadcast-pkts": "949512",
                            "in-multicast-pkts": "68",
                            "in-discards": "1408",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "10144687997",
                            "out-pkts": "48994975",
                            "out-unicast-pkts": "48994866",
                            "out-broadcast-pkts": "53",
                            "out-multicast-pkts": "56",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:02:03",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:02:03",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/2.4",
                    "config": {
                        "name": "1/2.4",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/2.4",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 18,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92299535013",
                        "counters": {
                            "in-octets": "35887746555",
                            "in-pkts": "62347891",
                            "in-unicast-pkts": "61400563",
                            "in-broadcast-pkts": "947247",
                            "in-multicast-pkts": "81",
                            "in-discards": "1412",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "9947112430",
                            "out-pkts": "47524506",
                            "out-unicast-pkts": "47524403",
                            "out-broadcast-pkts": "53",
                            "out-multicast-pkts": "50",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:02:04",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:02:04",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/2.5",
                    "config": {
                        "name": "1/2.5",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/2.5",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 20,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "525159521784",
                        "counters": {
                            "in-octets": "39187630636",
                            "in-pkts": "156665808",
                            "in-unicast-pkts": "146241632",
                            "in-broadcast-pkts": "5146811",
                            "in-multicast-pkts": "5277365",
                            "in-discards": "357",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "131905256742",
                            "out-pkts": "191440059",
                            "out-unicast-pkts": "187741861",
                            "out-broadcast-pkts": "3664321",
                            "out-multicast-pkts": "33877",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:02:05",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:02:05",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "67713732",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/2.6",
                    "config": {
                        "name": "1/2.6",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/2.6",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 21,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "525170483866",
                        "counters": {
                            "in-octets": "3506406963",
                            "in-pkts": "11491114",
                            "in-unicast-pkts": "284319",
                            "in-broadcast-pkts": "896187",
                            "in-multicast-pkts": "10310608",
                            "in-discards": "278",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "56632515503",
                            "out-pkts": "73518479",
                            "out-unicast-pkts": "72742834",
                            "out-broadcast-pkts": "755683",
                            "out-multicast-pkts": "19962",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:02:06",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:02:06",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "12398399",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.1",
                    "config": {
                        "name": "1/3.1",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.1",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 1,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "95362054354",
                        "counters": {
                            "in-octets": "4439535522366",
                            "in-pkts": "52226052570",
                            "in-unicast-pkts": "52147390412",
                            "in-broadcast-pkts": "363",
                            "in-multicast-pkts": "78661795",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "4439666219327",
                            "out-pkts": "52227806487",
                            "out-unicast-pkts": "52149161038",
                            "out-broadcast-pkts": "1542",
                            "out-multicast-pkts": "78643907",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:01",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:01",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "32",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.2",
                    "config": {
                        "name": "1/3.2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.2",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 105,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "95371605787",
                        "counters": {
                            "in-octets": "4439666243467",
                            "in-pkts": "52227806760",
                            "in-unicast-pkts": "52149161311",
                            "in-broadcast-pkts": "1542",
                            "in-multicast-pkts": "78643907",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "4439535546761",
                            "out-pkts": "52226052867",
                            "out-unicast-pkts": "52147390709",
                            "out-broadcast-pkts": "363",
                            "out-multicast-pkts": "78661795",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:02",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:02",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "32",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.3",
                    "config": {
                        "name": "1/3.3",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.3",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 85,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "95367435112",
                        "counters": {
                            "in-octets": "2403516",
                            "in-pkts": "11032",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "1905",
                            "in-multicast-pkts": "9127",
                            "in-discards": "11032",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:03",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:03",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.4",
                    "config": {
                        "name": "1/3.4",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.4",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 53,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:04",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:04",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.5",
                    "config": {
                        "name": "1/3.5",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.5",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 9,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:05",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:05",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.6",
                    "config": {
                        "name": "1/3.6",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.6",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 102,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:06",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:06",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.7",
                    "config": {
                        "name": "1/3.7",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.7",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 69,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:07",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:07",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/3.8",
                    "config": {
                        "name": "1/3.8",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/3.8",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 41,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:03:08",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:03:08",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/4.1",
                    "config": {
                        "name": "1/4.1",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/4.1",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 34,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "92294733496",
                        "counters": {
                            "in-octets": "228",
                            "in-pkts": "2",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "2",
                            "in-discards": "2",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:01:04:01",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:01:04:01",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "1/mgmt0",
                    "config": {
                        "name": "1/mgmt0",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "1/mgmt0",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 15,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "94788507060",
                        "counters": {
                            "in-octets": "124504572",
                            "in-pkts": "903148",
                            "in-unicast-pkts": "293862",
                            "in-broadcast-pkts": "538983",
                            "in-multicast-pkts": "70303",
                            "in-discards": "92",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "159838430",
                            "out-pkts": "993469",
                            "out-unicast-pkts": "553394",
                            "out-broadcast-pkts": "180102",
                            "out-multicast-pkts": "259973",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "mgmt-aggr"
                        },
                        "state": {
                            "mac-address": "00:94:a1:8e:d0:7d",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_1GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "00:94:a1:8e:d0:7d",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.1",
                    "config": {
                        "name": "2/1.1",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.1",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 10,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91721269332",
                        "counters": {
                            "in-octets": "4372564708",
                            "in-pkts": "17107339",
                            "in-unicast-pkts": "6258191",
                            "in-broadcast-pkts": "287493",
                            "in-multicast-pkts": "10561655",
                            "in-discards": "300",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "60252233307",
                            "out-pkts": "85000926",
                            "out-unicast-pkts": "84350738",
                            "out-broadcast-pkts": "129631",
                            "out-multicast-pkts": "520557",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.1"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:01",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:01",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "12394712",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.2",
                    "config": {
                        "name": "2/1.2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.2",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 11,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91727028538",
                        "counters": {
                            "in-octets": "3456428138",
                            "in-pkts": "19902188",
                            "in-unicast-pkts": "13280905",
                            "in-broadcast-pkts": "1136594",
                            "in-multicast-pkts": "5484689",
                            "in-discards": "166",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "25052369345",
                            "out-pkts": "74559178",
                            "out-unicast-pkts": "73908994",
                            "out-broadcast-pkts": "129627",
                            "out-multicast-pkts": "520557",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.2"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:02",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:02",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "3747325",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.3",
                    "config": {
                        "name": "2/1.3",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.3",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 2,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91713346120",
                        "counters": {
                            "in-octets": "822985336",
                            "in-pkts": "7420328",
                            "in-unicast-pkts": "6108978",
                            "in-broadcast-pkts": "1041683",
                            "in-multicast-pkts": "269667",
                            "in-discards": "103",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "2467739747",
                            "out-pkts": "4752812",
                            "out-unicast-pkts": "4102624",
                            "out-broadcast-pkts": "129631",
                            "out-multicast-pkts": "520557",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.3"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:03",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:03",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.4",
                    "config": {
                        "name": "2/1.4",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.4",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 3,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.4"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:04",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:04",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.5",
                    "config": {
                        "name": "2/1.5",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.5",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 12,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.5"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:05",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:05",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.6",
                    "config": {
                        "name": "2/1.6",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.6",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 13,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.6"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:06",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:06",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.7",
                    "config": {
                        "name": "2/1.7",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.7",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 4,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.7"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:07",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:07",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/1.8",
                    "config": {
                        "name": "2/1.8",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/1.8",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 5,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "cplagg_1.8"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:01:08",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:01:08",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/2.2",
                    "config": {
                        "name": "2/2.2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/2.2",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 16,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91731943025",
                        "counters": {
                            "in-octets": "10498555489",
                            "in-pkts": "43643415",
                            "in-unicast-pkts": "42715957",
                            "in-broadcast-pkts": "927390",
                            "in-multicast-pkts": "68",
                            "in-discards": "24",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "40025701457",
                            "out-pkts": "78104847",
                            "out-unicast-pkts": "54792115",
                            "out-broadcast-pkts": "7677447",
                            "out-multicast-pkts": "15635285",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:02:02",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:02:02",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "775779",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/2.3",
                    "config": {
                        "name": "2/2.3",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/2.3",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 17,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91736600968",
                        "counters": {
                            "in-octets": "10496541001",
                            "in-pkts": "43643033",
                            "in-unicast-pkts": "42715235",
                            "in-broadcast-pkts": "927735",
                            "in-multicast-pkts": "63",
                            "in-discards": "23",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "32274013723",
                            "out-pkts": "53814882",
                            "out-unicast-pkts": "53814779",
                            "out-broadcast-pkts": "59",
                            "out-multicast-pkts": "44",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:02:03",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:02:03",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/2.4",
                    "config": {
                        "name": "2/2.4",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/2.4",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 18,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91745418204",
                        "counters": {
                            "in-octets": "10495853111",
                            "in-pkts": "43642401",
                            "in-unicast-pkts": "42715561",
                            "in-broadcast-pkts": "926784",
                            "in-multicast-pkts": "56",
                            "in-discards": "27",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "32111449258",
                            "out-pkts": "53850019",
                            "out-unicast-pkts": "53849911",
                            "out-broadcast-pkts": "62",
                            "out-multicast-pkts": "46",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:02:04",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:02:04",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/2.5",
                    "config": {
                        "name": "2/2.5",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/2.5",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 20,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91749376611",
                        "counters": {
                            "in-octets": "130408111774",
                            "in-pkts": "248849392",
                            "in-unicast-pkts": "245152416",
                            "in-broadcast-pkts": "3663488",
                            "in-multicast-pkts": "33488",
                            "in-discards": "93",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "39155071852",
                            "out-pkts": "147371690",
                            "out-unicast-pkts": "136951024",
                            "out-broadcast-pkts": "5144992",
                            "out-multicast-pkts": "5275674",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:02:05",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:02:05",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "66746774",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/2.6",
                    "config": {
                        "name": "2/2.6",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/2.6",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 21,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91753495222",
                        "counters": {
                            "in-octets": "56621094489",
                            "in-pkts": "85388246",
                            "in-unicast-pkts": "84613492",
                            "in-broadcast-pkts": "755358",
                            "in-multicast-pkts": "19396",
                            "in-discards": "58",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "3504829058",
                            "out-pkts": "11485860",
                            "out-unicast-pkts": "282611",
                            "out-broadcast-pkts": "895811",
                            "out-multicast-pkts": "10307438",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:02:06",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:02:06",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "12395527",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.1",
                    "config": {
                        "name": "2/3.1",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.1",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 1,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "94942086460",
                        "counters": {
                            "in-octets": "5101782656244",
                            "in-pkts": "57305206157",
                            "in-unicast-pkts": "28824166",
                            "in-broadcast-pkts": "363",
                            "in-multicast-pkts": "57276381628",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "5100404767374",
                            "out-pkts": "57297833348",
                            "out-unicast-pkts": "26860585",
                            "out-broadcast-pkts": "1533",
                            "out-multicast-pkts": "57270971230",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:01",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:01",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "24",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.2",
                    "config": {
                        "name": "2/3.2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.2",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 105,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "94950666557",
                        "counters": {
                            "in-octets": "5100404793807",
                            "in-pkts": "57297833644",
                            "in-unicast-pkts": "26860585",
                            "in-broadcast-pkts": "1533",
                            "in-multicast-pkts": "57270971526",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "5101782683033",
                            "out-pkts": "57305206458",
                            "out-unicast-pkts": "28824166",
                            "out-broadcast-pkts": "363",
                            "out-multicast-pkts": "57276381929",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:02",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:02",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "24",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.3",
                    "config": {
                        "name": "2/3.3",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.3",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 85,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "94946614936",
                        "counters": {
                            "in-octets": "2355615",
                            "in-pkts": "10806",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "1896",
                            "in-multicast-pkts": "8910",
                            "in-discards": "10806",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:03",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_UNKNOWN",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:03",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.4",
                    "config": {
                        "name": "2/3.4",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.4",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 53,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:04",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:04",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.5",
                    "config": {
                        "name": "2/3.5",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.5",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 9,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:05",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:05",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.6",
                    "config": {
                        "name": "2/3.6",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.6",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 102,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:06",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:06",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.7",
                    "config": {
                        "name": "2/3.7",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.7",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 69,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:07",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:07",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/3.8",
                    "config": {
                        "name": "2/3.8",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/3.8",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 41,
                        "admin-status": "UP",
                        "oper-status": "DOWN",
                        "counters": {
                            "in-octets": "0",
                            "in-pkts": "0",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "0",
                            "in-discards": "0",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:03:08",
                            "auto-negotiate": false,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_25GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:03:08",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/4.1",
                    "config": {
                        "name": "2/4.1",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/4.1",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 34,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "91755988351",
                        "counters": {
                            "in-octets": "228",
                            "in-pkts": "2",
                            "in-unicast-pkts": "0",
                            "in-broadcast-pkts": "0",
                            "in-multicast-pkts": "2",
                            "in-discards": "2",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "0",
                            "out-pkts": "0",
                            "out-unicast-pkts": "0",
                            "out-broadcast-pkts": "0",
                            "out-multicast-pkts": "0",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL"
                        },
                        "state": {
                            "mac-address": "5a:a5:5a:02:04:01",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_10GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "5a:a5:5a:02:04:01",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "2/mgmt0",
                    "config": {
                        "name": "2/mgmt0",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    "state": {
                        "name": "2/mgmt0",
                        "type": "iana-if-type:ethernetCsmacd",
                        "loopback-mode": false,
                        "enabled": true,
                        "ifindex": 15,
                        "admin-status": "UP",
                        "oper-status": "UP",
                        "last-change": "94362333415",
                        "counters": {
                            "in-octets": "2257420268",
                            "in-pkts": "2699262",
                            "in-unicast-pkts": "1682315",
                            "in-broadcast-pkts": "793147",
                            "in-multicast-pkts": "223800",
                            "in-discards": "182",
                            "in-errors": "0",
                            "in-unknown-protos": "0",
                            "in-fcs-errors": "0",
                            "out-octets": "60492741",
                            "out-pkts": "493007",
                            "out-unicast-pkts": "233377",
                            "out-broadcast-pkts": "27",
                            "out-multicast-pkts": "259603",
                            "out-discards": "0",
                            "out-errors": "0"
                        }
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "config": {
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "openconfig-if-aggregate:aggregate-id": "mgmt-aggr"
                        },
                        "state": {
                            "mac-address": "00:94:a1:8e:d0:7e",
                            "auto-negotiate": true,
                            "duplex-mode": "FULL",
                            "port-speed": "openconfig-if-ethernet:SPEED_1GB",
                            "enable-flow-control": false,
                            "hw-mac-address": "00:94:a1:8e:d0:7e",
                            "counters": {
                                "in-mac-pause-frames": "0",
                                "in-oversize-frames": "0",
                                "in-jabber-frames": "0",
                                "in-fragment-frames": "0",
                                "in-8021q-frames": "0",
                                "in-crc-errors": "0",
                                "out-mac-pause-frames": "0",
                                "out-8021q-frames": "0"
                            }
                        }
                    }
                },
                {
                    "name": "cplagg_1.1",
                    "config": {
                        "name": "cplagg_1.1",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        },
                        "state": {
                            "lag-speed": 1240240968
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.2",
                    "config": {
                        "name": "cplagg_1.2",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        },
                        "state": {
                            "lag-speed": 1240240968
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.3",
                    "config": {
                        "name": "cplagg_1.3",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        },
                        "state": {
                            "lag-speed": 1240240968
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.4",
                    "config": {
                        "name": "cplagg_1.4",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.5",
                    "config": {
                        "name": "cplagg_1.5",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.6",
                    "config": {
                        "name": "cplagg_1.6",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.7",
                    "config": {
                        "name": "cplagg_1.7",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "cplagg_1.8",
                    "config": {
                        "name": "cplagg_1.8",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP",
                            "min-links": 1
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                },
                {
                    "name": "mgmt-aggr",
                    "config": {
                        "name": "mgmt-aggr",
                        "type": "iana-if-type:ieee8023adLag"
                    },
                    "state": {
                        "loopback-mode": false,
                        "enabled": true
                    },
                    "hold-time": {
                        "state": {
                            "up": 0,
                            "down": 0
                        }
                    },
                    "openconfig-if-aggregate:aggregation": {
                        "config": {
                            "lag-type": "LACP"
                        },
                        "state": {
                            "lag-speed": 1240240968
                        }
                    },
                    "openconfig-if-ethernet:ethernet": {
                        "state": {
                            "auto-negotiate": true,
                            "enable-flow-control": false
                        }
                    }
                }
            ]
        }
    }


Link Aggregation Status of System Controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following API call will list the current status of all backplane LACP interfaces, as well as front panel management port lacp interfaces:

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/openconfig-lacp:lacp

.. code-block:: json

    {
        "openconfig-lacp:lacp": {
            "interfaces": {
                "interface": [
                    {
                        "name": "cplagg_1.1",
                        "config": {
                            "name": "cplagg_1.1",
                            "interval": "FAST"
                        },
                        "state": {
                            "name": "cplagg_1.1",
                            "interval": "FAST",
                            "lacp-mode": "ACTIVE",
                            "system-id-mac": "0:94:a1:8e:d0:0",
                            "system-priority": 53248
                        },
                        "members": {
                            "member": [
                                {
                                    "interface": "1/1.1",
                                    "state": {
                                        "activity": "ACTIVE",
                                        "timeout": "SHORT",
                                        "synchronization": "IN_SYNC",
                                        "aggregatable": true,
                                        "collecting": true,
                                        "distributing": true,
                                        "system-id": "0:94:a1:8e:d0:0",
                                        "oper-key": 2,
                                        "partner-id": "0:a:49:ff:96:2",
                                        "partner-key": 0,
                                        "port-num": 4225,
                                        "partner-port-num": 2,
                                        "counters": {
                                            "lacp-in-pkts": "1173835",
                                            "lacp-out-pkts": "1170433",
                                            "lacp-rx-errors": "0"
                                        }
                                    }
                                },
                                {
                                    "interface": "2/1.1",
                                    "state": {
                                        "activity": "ACTIVE",
                                        "timeout": "SHORT",
                                        "synchronization": "IN_SYNC",
                                        "aggregatable": true,
                                        "collecting": true,
                                        "distributing": true,
                                        "system-id": "0:94:a1:8e:d0:0",
                                        "oper-key": 2,
                                        "partner-id": "0:a:49:ff:96:2",
                                        "partner-key": 0,
                                        "port-num": 8321,
                                        "partner-port-num": 4,
                                        "counters": {
                                            "lacp-in-pkts": "1174757",
                                            "lacp-out-pkts": "1170443",
                                            "lacp-rx-errors": "0"
                                        }
                                    }
                                }
                            ]
                        }
                    },

Alerting and Logging for the Layer2 Switch Fabric on the System Controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

API Monitoring of Chassis Cluster Status from the System Controller
-------------------------------------------------------------------

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/f5-chassis-cluster:cluster

.. code-block:: json

    {
        "f5-chassis-cluster:cluster": {
            "nodes": {
                "node": [
                    {
                        "name": "blade-1",
                        "status": "Ready",
                        "time-created": "2021-08-31T00:16:13Z",
                        "roles": "compute",
                        "info": {
                            "cpu": 28,
                            "pods": 250,
                            "memory": "26112340Ki",
                            "hugepages": "102890Mi"
                        }
                    },
                    {
                        "name": "blade-2",
                        "status": "Ready",
                        "time-created": "2021-08-31T00:16:12Z",
                        "roles": "compute",
                        "info": {
                            "cpu": 28,
                            "pods": 250,
                            "memory": "26112340Ki",
                            "hugepages": "102890Mi"
                        }
                    },
                    {
                        "name": "blade-3",
                        "status": "Ready",
                        "time-created": "2021-08-31T00:16:11Z",
                        "roles": "compute",
                        "info": {
                            "cpu": 28,
                            "pods": 250,
                            "memory": "26112340Ki",
                            "hugepages": "102890Mi"
                        }
                    },
                    {
                        "name": "controller-1",
                        "status": "Ready",
                        "time-created": "2021-08-30T23:30:41Z",
                        "roles": "infra,master"
                    },
                    {
                        "name": "controller-2",
                        "status": "Ready",
                        "time-created": "2021-09-15T20:16:48Z",
                        "roles": "infra,master"
                    }
                ]
            },
            "install-progress": {
                "install-progress": [
                    {
                        "stage-name": "AddingBlade",
                        "status": "Done"
                    },
                    {
                        "stage-name": "AddingController",
                        "status": "Done"
                    },
                    {
                        "stage-name": "AddingEtcd",
                        "status": "Done"
                    },
                    {
                        "stage-name": "HealthCheck",
                        "status": "Done"
                    },
                    {
                        "stage-name": "HostedInstall",
                        "status": "Done"
                    },
                    {
                        "stage-name": "MasterAdditionalInstall",
                        "status": "Done"
                    },
                    {
                        "stage-name": "MasterInstall",
                        "status": "Done"
                    },
                    {
                        "stage-name": "NodeBootstrap",
                        "status": "Done"
                    },
                    {
                        "stage-name": "NodeJoin",
                        "status": "Done"
                    },
                    {
                        "stage-name": "Prerequisites",
                        "status": "Done"
                    },
                    {
                        "stage-name": "ServiceCatalogInstall",
                        "status": "Done"
                    },
                    {
                        "stage-name": "etcdInstall",
                        "status": "Done"
                    }
                ]
            },
            "orchestration-manager": {
                "cluster-initialized": true,
                "cluster-ready": true,
                "active-node": "controller-1.chassis.local",
                "etcd-ha-initialized": true,
                "etcd-ha-running": true,
                "controller-status": [
                    {
                        "index": 1,
                        "name": "controller-1.chassis.local",
                        "inserted": true,
                        "in-cluster": true,
                        "ready-cluster": true,
                        "able-to-ping": true,
                        "able-to-ssh": true,
                        "state": "In Cluster"
                    },
                    {
                        "index": 2,
                        "name": "controller-2.chassis.local",
                        "inserted": true,
                        "in-cluster": true,
                        "ready-cluster": true,
                        "able-to-ping": true,
                        "able-to-ssh": true,
                        "state": "In Cluster"
                    }
                ],
                "blade-status": [
                    {
                        "index": 1,
                        "name": "blade-1.chassis.local",
                        "inserted": true,
                        "in-cluster": true,
                        "ready-cluster": true,
                        "able-to-ping": true,
                        "able-to-ssh": true,
                        "state": "In Cluster",
                        "partition-label": "partition-2"
                    },
                    {
                        "index": 2,
                        "name": "blade-2.chassis.local",
                        "inserted": true,
                        "in-cluster": true,
                        "ready-cluster": true,
                        "able-to-ping": true,
                        "able-to-ssh": true,
                        "state": "In Cluster",
                        "partition-label": "partition-2"
                    },
                    {
                        "index": 3,
                        "name": "blade-3.chassis.local",
                        "inserted": true,
                        "in-cluster": true,
                        "ready-cluster": true,
                        "able-to-ping": true,
                        "able-to-ssh": true,
                        "state": "In Cluster",
                        "partition-label": "partition-3"
                    }
                ]
            },
            "cluster-status": {
                "summary-status": "Check DNS server configuration. Openshift cluster is healthy, and all controllers and blades are ready.",
                "cluster-status": [
                    {
                        "status": "2021-09-17 02:37:43.730946 -  Orchestration manager startup."
                    },
                    {
                        "status": "2021-09-17 02:38:03.741536 -  Can now ping controller-1.chassis.local (100.65.3.51)."
                    },
                    {
                        "status": "2021-09-17 02:38:03.749433 -  Can now ping controller-2.chassis.local (100.65.3.52)."
                    },
                    {
                        "status": "2021-09-17 02:38:03.800254 -  Successfully ssh'd to CC controller-1.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:38:04.030608 -  Successfully ssh'd to CC controller-2.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:38:04.037921 -  Can now ping blade blade-1.chassis.local (100.65.3.1)."
                    },
                    {
                        "status": "2021-09-17 02:38:04.266296 -  Successfully SSH'd to blade blade-1.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:38:04.274272 -  Can now ping blade blade-2.chassis.local (100.65.3.2)."
                    },
                    {
                        "status": "2021-09-17 02:38:04.477978 -  Successfully SSH'd to blade blade-2.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:38:04.484810 -  Can now ping blade blade-3.chassis.local (100.65.3.3)."
                    },
                    {
                        "status": "2021-09-17 02:38:04.683856 -  Successfully SSH'd to blade blade-3.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:39:05.398483 -  Invalid DNS server configured on controller-1.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:39:05.424236 -  Orchestration manager transitioning to active."
                    },
                    {
                        "status": "2021-09-17 02:39:55.229251 -  Can NOT ping controller-2.chassis.local (100.65.3.52)."
                    },
                    {
                        "status": "2021-09-17 02:45:11.056259 -  Can now ping controller-2.chassis.local (100.65.3.52)."
                    },
                    {
                        "status": "2021-09-17 02:45:11.297634 -  Successfully ssh'd to CC controller-2.chassis.local."
                    },
                    {
                        "status": "2021-09-17 02:46:13.207504 -  Controller 1 is ready in openshift cluster."
                    },
                    {
                        "status": "2021-09-17 02:46:13.207610 -  Controller 2 is ready in openshift cluster."
                    },
                    {
                        "status": "2021-09-17 02:46:13.207654 -  Blade 1 is ready in openshift cluster."
                    },
                    {
                        "status": "2021-09-17 02:46:13.207697 -  Blade 2 is ready in openshift cluster."
                    },
                    {
                        "status": "2021-09-17 02:46:13.207739 -  Blade 3 is ready in openshift cluster."
                    },
                    {
                        "status": "2021-09-17 02:46:13.207782 -  Openshift cluster is ready."
                    },
                    {
                        "status": "2021-09-17 02:47:02.228486 -  Openshift cluster is NOT ready."
                    },
                    {
                        "status": "2021-09-17 02:47:21.461242 -  Openshift cluster is ready."
                    }
                ]
            }
        }
    }


API Monitoring of Chassis Partitions from the System Controller
---------------------------------------------------------------

.. code-block:: bash

    GET https://{{System-Controller-IP}}:8888/restconf/data/f5-system-slot:slots

.. code-block:: json


    {
        "f5-system-slot:slots": {
            "slot": [
                {
                    "slot-num": 1,
                    "enabled": true,
                    "partition": "bigpartition"
                },
                {
                    "slot-num": 2,
                    "enabled": true,
                    "partition": "bigpartition"
                },
                {
                    "slot-num": 3,
                    "enabled": true,
                    "partition": "smallpartition"
                },
                {
                    "slot-num": 4,
                    "enabled": true,
                    "partition": "default"
                },
                {
                    "slot-num": 5,
                    "enabled": true,
                    "partition": "default"
                },
                {
                    "slot-num": 6,
                    "enabled": true,
                    "partition": "default"
                },
                {
                    "slot-num": 7,
                    "enabled": true,
                    "partition": "default"
                },
                {
                    "slot-num": 8,
                    "enabled": true,
                    "partition": "default"
                }
            ]
        }
    }

GUI Monitoring of Chassis Partitions from the System Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dashboard:

.. image:: images/monitoring_velos/image9.png
  :align: center
  :scale: 70%

The are some basic visuals in the System Controller GUI for Chassis Partitions with an Operational State, views of the partitions, and the ability to do some basic configuration from the System Controller. You can connect directly to one of the Chassis Partitions to get more specific details.

The GUI screen below shows Chassis Partition visualization/configuration. An admin can see which blades belong to which chassis partitions as well as the chassis partition operational status:

.. image:: images/monitoring_velos/image10.png
  :align: center
  :scale: 70%

CLI Monitoring of Chassis Partitions from the System Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The CLI command show partitions will show the current chassis partitions ID’s and their status on each system controller:

.. code-block:: bash

    syscon-2-active# show partitions 
                            PARTITION  PARTITION  
    NAME        CONTROLLER  ID         STATUS     
    ----------------------------------------------
    bigpart     1           2          running    
                2           2          running    
    blade3part  1           3          running    
                2           3          running    
    default     1           1          running    
                2           1          running    
    none        1           0          disabled   
                2           0          disabled

The CLI command **show running-config slots** will show the which slots are configured to participate in specific chassis partitions:

.. code-block:: bash

    syscon-2-active# show running-config slots 
    slots slot 1
    enabled
    partition bigpart
    !
    slots slot 2
    enabled
    partition bigpart
    !
    slots slot 3
    enabled
    partition blade3part
    !
    slots slot 4
    enabled
    partition default
    !
    slots slot 5
    enabled
    partition default
    !
    slots slot 6
    enabled
    partition default
    !
    slots slot 7
    enabled
    partition default
    !
    slots slot 8
    enabled
    partition default
    !
  

Alerting and Logging of Chassis Partitions Events from the System Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


From the System Controller CLI you may also view the System Controller and individual Blade Status using the **show cluster** or the shortened **show cluster nodes** command:

.. code-block:: bash

    controller-1# show cluster nodes
    NAME          STATUS  TIME CREATED          ROLES         CPU  PODS  MEMORY      HUGEPAGES  
    --------------------------------------------------------------------------------------------
    blade-1       Ready   2020-08-06T06:18:08Z  compute       28   250   26112340Ki  102890Mi   
    blade-2       Ready   2020-08-06T06:18:09Z  compute       28   250   26112340Ki  102890Mi   
    controller-1  Ready   2020-08-06T05:52:45Z  infra,master  -    -     -           -          
    controller-2  Ready   2020-08-06T05:52:51Z  infra,master  -    -     -           -          


    controller-1# show cluster 
    NAME          STATUS  TIME CREATED          ROLES         CPU  PODS  MEMORY      HUGEPAGES  
    --------------------------------------------------------------------------------------------
    blade-1       Ready   2020-08-06T06:18:08Z  compute       28   250   26112340Ki  102890Mi   
    blade-2       Ready   2020-08-06T06:18:09Z  compute       28   250   26112340Ki  102890Mi   
    controller-1  Ready   2020-08-06T05:52:45Z  infra,master  -    -     -           -          
    controller-2  Ready   2020-08-06T05:52:51Z  infra,master  -    -     -           -          

    STAGE NAME               STATUS  
    ---------------------------------
    AddingBlade              Done    
    HealthCheck              Done    
    HostedInstall            Done    
    MasterAdditionalInstall  Done    
    MasterInstall            Done    
    NodeBootstrap            Done    
    NodeJoin                 Done    
    Prerequisites            Done    
    ServiceCatalogInstall    Done    
    etcdInstall              Done    




