==========================
BIG-IQ Support for rSeries
==========================

Currently rSeries support in BIG-IQ will mimic iSeries functionality when vCMP is in use. In iSeries a vCMP guest can be created via the host CLI, GUI, or API and it can then be imported into BIG-IQ as a device. From there statistics/analytics can be gathered, and L4-7 configurations can be managed in a variety of ways. An rSeries tenant will behave identical to a iSeries vCMP guest from a BIG-IQ perspective, meaning you can import it after it has been created to manage configuration or get analytics.

When an rSeries tenant is created it can be imported as a device into BIG-IQ. It will import just like any other BIG-IP instance or device.  Once imported it will show up with a Type of **BIG-IP Tenant**.

.. image:: images/bigiq_support_for_velos/image1.png
  :align: center
  :scale: 70%

rSeries tenants can also be onboarded in BIG-IQ using Declarative Onboarding (DO). Once A tenant is created via one of the rSeries F5OS interfaces you can run a DO declaration like the one below to BIG-IQ to provision, configure and import it. In the DO declaration you will specify a **targetHost** which is the IP address of the tenant to be onboarded. The following is an example of a DO declaration for onboarding an rSeries tenant:

.. code-block:: bash

    POST https://{{BigIQ_Mgmt}}/mgmt/shared/declarative-onboarding

.. code-block:: json

    {
        "class": "DO",
        "declaration": {
            "schemaVersion": "1.5.0",
            "class": "Device",
            "async": true,
            "Common": {
                "class": "Tenant",
                "myProvision": {
                    "class": "Provision",
                    "avr": "nominal",
                    "ltm": "nominal",
                    "asm": "nominal",
                    "apm": "nominal"
                },
                "myDns": {
                    "class": "DNS",
                    "nameServers": [
                        "10.192.50.10",
                        "10.192.50.11"
                    ]
                },
                "myNtp": {
                    "class": "NTP",
                    "servers": [
                        "time.f5net.com"
                    ],
                    "timezone": "UTC"
                },
                "internal-self": {
                    "class": "SelfIp",
                    "address": "10.10.11.11/24",
                    "vlan": "vlan-444",
                    "allowService": "all",
                    "trafficGroup": "traffic-group-local-only"
                },
                "external-self": {
                    "class": "SelfIp",
                    "address": "10.10.12.11/24",
                    "vlan": "vlan-555",
                    "trafficGroup": "traffic-group-local-only",
                    "allowService": "default"
                },
                "myDbVariables": {
                    "class": "DbVariables",
                    "ui.advisory.enabled": "true",
                    "ui.advisory.color": "red",
                    "ui.advisory.text": "Configuration deployed with AS3. Do not make any change directly on the BIG-IP or those changes may be lost."
                },
                "admin": {
                    "class": "User",
                    "userType": "regular",
                    "shell": "bash",
                    "partitionAccess": {
                        "all-partitions": {
                            "role": "admin"
                        }
                    },
                    "password": "{{Chassis_Partition_Password}}"
                },
                "root": {
                    "class": "User",
                    "userType": "root",
                    "newPassword": "{{Chassis_Partition_Password}}",
                    "oldPassword": "{{Chassis_Partition_Password}}"
                },
                "hostname": "tenant1.chassis1.f5demo.net"
            }
        },
        "targetHost": "{{Chassis1_Tenant1_IP}}",
        "targetUsername": "admin",
        "targetPassphrase": "admin",
        "bigIqSettings": {
            "failImportOnConflict": false,
            "conflictPolicy": "USE_BIGIQ",
            "deviceConflictPolicy": "USE_BIGIP",
            "versionedConflictPolicy": "KEEP_VERSION",
            "statsConfig": {
                "enabled": true,
                "zone": "default"
            },
            "accessModuleProperties": {
                "cm:access:access-group-name": "tenant1",
                "cm:access:import-shared": true
            },
            "snapshotWorkingConfig": false
        }
    }

Shortly after the declaration is sent to BIG-IQ you can see a new onboarding task. This will take a while to complete as it may require reboots of the tenant for module provisioning. After the tenant is onboarded it will be imported into BIG-IQ.

.. image:: images/bigiq_support_for_velos/image2.png
  :align: center
  :scale: 70%


