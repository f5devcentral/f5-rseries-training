============================================
Initial Setup of rSeries F5OS Platform Layer
============================================


Connect a console or terminal server to the console port of the rSeries appliance. Login as admin/admin (you’ll be prompted to change the password) and access the F5OS CLI. F5OS utilizes **ConfD** for configuration management of F5OS and will be a familiar navigation experience if you have used it on other products. The CLI supports command completion and online help and is easy to navigate. There are **show** commands to display current configurations and status, and a **config** mode to alter current configuration.

Once logged in you can display the current running configuration by issuing the command **show running-config**.

.. code-block:: bash

  Boston-r10900-1# show running-config 
  cluster disk-usage-threshold config warning-limit 85
  cluster disk-usage-threshold config error-limit 90
  cluster disk-usage-threshold config critical-limit 97
  cluster disk-usage-threshold config growth-rate-limit 10
  cluster disk-usage-threshold config interval 60
  cluster nodes node node-1
  config name node-1
  config enabled
  !
  fpga-tables xbar-ports xbar-port port0_mod0
  !
  fpga-tables xbar-ports xbar-port port1_mod0
  !
  fpga-tables xbar-ports xbar-port port2_mod3
  !
  fpga-tables xbar-ports xbar-port port3_mod3
  !
  fpga-tables xbar-ports xbar-port port4_mod4
  !
  fpga-tables xbar-ports xbar-port port5_mod5
  !
  fpga-tables xbar-ports xbar-port port6_mod1
  !
  cluster disk-usage-threshold config warning-limit 85
  cluster disk-usage-threshold config error-limit 90
  cluster disk-usage-threshold config critical-limit 97
  cluster disk-usage-threshold config growth-rate-limit 10
  cluster disk-usage-threshold config interval 60
  cluster nodes node node-1
  config name node-1
  config enabled
  !

To alter any configuration, you must enter config mode:

.. code-block:: bash

  syscon-2-active# config
  Entering configuration mode terminal
  syscon-2-active(config)#

 And so save any configuration you must enter *commit**.

.. code-block:: bash

  Boston-r10900-1(config)# commit


----------------------------
Internal Appliance IP Ranges
----------------------------

The rSeries appliances ship with a default internal RFC6598 address space of 100.64.0.0/12. This should be sufficient for most production environments. You can verify this with the following command.

.. code-block:: bash

  Boston-r10900-1# show system network 
  system network state configured-network-range-type RFC6598
  system network state configured-network-range 100.64.0.0/12
  system network state active-network-range-type RFC6598
  system network state active-network-range 100.64.0.0/12
  Boston-r10900-1# 

This address range never leaves the inside of the appliance and will not interfere with any communication outside the rSeries device. There can however be address collisions if a device trying to manage rSeries via the out-of-band management port falls within this range, or an external management device or service falls within this range and communicated with rSeries over its out-of-band networking. This may result in rSeries not able to communicate with those devices.

Some examples would be any client trying to access the F5OS or tenant out-of-band interfaces to reach its’ CLI, GUI, or API. Other examples would be external services such as SNMP, DNS, NTP, SNMP, Authentication that have addresses that fall within the RFC6598 address space. You may experience connectivity problems with these types of clients/services if there is an overlap. Note that this does not affect the data plane / in-band interfaces, it only affects communication to the out-of-band interfaces. 

If there is the potential for conflict with external devices that fall within this range that need to communicate with rSeries, then there are options to change the configured-network-range-type to one of sixteen different blocks within the RFC1918 address space. Changing this will require a complete appliance power-cycle, rebooting is not sufficient.  Please consult with F5 prior to making any changes to the internal addresses.

.. code-block:: bash

  Boston-r10900-1(config)# system network config network-range-type RFC < Hit TAB>
  Possible completions:
    RFC1918   System uses 10.[0-15]/12 as specified by RFC1918
    RFC6598   System uses 100.64/10 as specified by RFC6598
  Boston-r10900-1(config)# system network config network-range-type RFC

If changing to one of the RFC1918 address spaces, you will need to choose from one of 16 prefix ranges as seen below. You should ensure that this will not overlap with current address space deployed within the environment:

.. code-block:: bash

  Boston-r10900-1(config)# system network config network-range-type RFC1918 prefix ?
  Description: 
  The network prefix index is used to select the range of IP addresses
  used internally within the appliance. The network prefix should be
  selected such that internal appliance addresses do not overlap with
  site-local addresses that are accessible to the appliance.

  Network Prefix Index       Appliance Network Range
  0                          10.[0-15].0.0/16
  1                          10.[16-31].0.0/16
  2                          10.[32-47].0.0/16
  3                          10.[48-63].0.0/16
  4                          10.[64-79].0.0/16
  5                          10.[80-95].0.0/16
  6                          10.[96-111].0.0/16
  7                          10.[112-127].0.0/16
  8                          10.[128-143].0.0/16
  9                          10.[144-159].0.0/16
  10                         10.[160-175].0.0/16
  11                         10.[176-191].0.0/16
  12                         10.[192-207].0.0/16
  13                         10.[208-223].0.0/16
  14                         10.[224-239].0.0/16
  15                         10.[240-255].0.0/16
  Possible completions:
    <unsignedByte, 0 .. 15>[0]
  Boston-r10900-1(config)# system network config network-range-type RFC1918 prefix 15
  Boston-r10900-1(config)# commit
  Commit complete.

**Note: This change will not take effect until the appliance is power cycled. A complete power cycle is required in order to convert existing internal address space to the new address space, a reboot is not sufficient.**

-------------------------------
IP Address Assignment & Routing
-------------------------------

Each system controller requires its own unique IP address, and a floating IP address also needs to be configured. The floating IP address will follow the primary system controller. The IP addresses can be statically defined or acquired via DHCP. In addition to the IP addresses a default route and subnet mask/prefix length is defined. For the initial 1.1.x versions of F5OS only IPv4 IP addresses are supported on the out-of-band interfaces of the system controllers. IPv6 and dual stack IPv4/v6 support requires F5OS be running 1.2.x or later. Note the tenants themselves support IPv4/IPv6 management today.

.. image:: images/initial_setup_of_velos_system_controllers/image1.png
  :align: center
  :scale: 70%

Once logged in you will configure the static IP addresses (unless DHCP is preferred).

.. code-block:: bash

  syscon-2-active(config)# system mgmt-ip config ipv4 controller-1 address 10.255.0.212
  syscon-2-active(config)# system mgmt-ip config ipv4 controller-2 address 10.255.0.213
  syscon-2-active(config)# system mgmt-ip config ipv4 floating address 10.255.0.214
  syscon-2-active(config)# system mgmt-ip config ipv4 prefix-length 24
  syscon-2-active(config)# system mgmt-ip config ipv4 gateway 10.255.0.1

In order to make these changes active you must commit the changes. No configuration changes are executed until the commit command is issued. 

.. code-block:: bash

  syscon-2-active(config)# commit

Now that the out-of-band addresses and routing are configured you can attempt to access the system controller GUI via the floating IP address that has been defined. You should see a screen similar to the one below, and you can verify your management interface settings.

.. image:: images/initial_setup_of_velos_system_controllers/image2.png
  :align: center
  :scale: 70%

-------------------------------------------------------
Interface Aggregation for System Controllers (Optional)
-------------------------------------------------------

As seen in previous diagrams each system controller has its own independent out-of-band 10Gb ethernet connection. These can run independently of each other and should be connected to the same layer2 VLAN so that the floating IP address can move from primary to standby in the event of a failure. You may optionally configure these two interfaces into a single Link Aggregation Group (LAG) for added resiliency which is recommended. This would allow direct access to either static IP address on the system controllers in the event one link should fail. Below is a depiction of each system controllers OOB interface bonded together in a single LAG:

.. image:: images/initial_setup_of_velos_system_controllers/image3.png
  :align: center
  :scale: 70%

To enable this feature, you would need to enable link aggregation on the system controllers via the CLI, GUI or API, and then make changes to your upstream layer2 switching infrastructure to ensure the two ports are put into the same LAG. To configure the management ports of both system controllers to run in a LAG configure as follows:

On the active controller create a managment LACP interface:

.. code-block:: bash

  lacp interfaces interface mgmt-aggr
  config name mgmt-aggr
  !
 
Next create a management aggregate interface and set the **config type** to **ieee8023adLag** and set the **lag-type** to **LACP**.

.. code-block:: bash

  interfaces interface mgmt-aggr
  config name mgmt-aggr
  config type ieee8023adLag
  aggregation config lag-type LACP
  !

Finally add the aggregate that you created by name to each of the management interfaces on the two controllers: 

.. code-block:: bash

  !
  interfaces interface 1/mgmt0
  config name 1/mgmt0
  config type ethernetCsmacd
  ethernet config aggregate-id mgmt-aggr
  !
 
 
  interfaces interface 2/mgmt0
  config name 2/mgmt0
  config type ethernetCsmacd
  ethernet config aggregate-id mgmt-aggr

---------------
System Settings
---------------

Once the IP addresses have been defined system settings such as DNS servers, NTP, and external logging should be defined. This can be done from the CLI, GUI, or API.

**From the CLI:**

.. code-block:: bash

  syscon-2-active# config
  Entering configuration mode terminal
  syscon-2-active(config)# system dns servers server 192.168.19.1 config address 192.168.10.1
  syscon-2-active(config-server-192.168.19.1)# exit
  syscon-2-active(config)# system ntp config enabled 
  syscon-2-active(config)# system ntp servers server time.f5net.com config address time.f5net.com
  syscon-2-active(config-server-time.f5net.com)# exit
  syscon-2-active(config)# system logging remote-servers remote-server 10.255.0.142 selectors selector LOCAL0 WARNING
  syscon-2-active(config-remote-server-10.255.0.142)# exit
  syscon-2-active(config)# commit

**From the GUI:**

You can configure the DNS and Time settings from the GUI if preferred. DNS is configured under **Network Settings > DNS**. Here you can add DNS lookup servers, and optional search domains. This will be needed for the VELOS chassis to resolve hostnames that may be used for external services like ntp, authentication servers, or to reach iHealth for qkview uploads.

.. image:: images/initial_setup_of_velos_system_controllers/image4.png
  :align: center
  :scale: 70%

  Configuring Network Time Protocol is highly recommended so that the VELOS systems clock is sync’d and accurate. In addition to configure NTP time sources, you can set the local timezone for this chassis location.

.. image:: images/initial_setup_of_velos_system_controllers/image5.png
  :align: center
  :scale: 70%

  It’s also a good idea to have the VELOS system send logs to an external syslog server. This can be configured in the System Settings > Log Settings screen. Here you can configure remote servers, the logging facility, and severity levels. You can also configure logging subsystem level individually. The remote logging severity level will override and component logging levels if they are higher, but only for logs sent remotely. Local logging levels will follow however the component levels are configured here.

.. image:: images/initial_setup_of_velos_system_controllers/image6.png
  :align: center
  :scale: 70%

**From the API:**

If you would prefer to automate the setup of the VELOS chassis, there are API calls for all of the examples above. To set the DNS configuration for the system controllers use the following API call:

.. code-block:: bash

  PATCH https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/

.. code-block:: json

  {
      "openconfig-system:system": {
          "clock": {
              "config": {
                  "timezone-name": "America/New_York"
              }
          },
          "dns": {
              "config": {
                  "search": "olympus.f5net.com"
              },
              "servers": {
                  "server": [
                      {
                          "address": "8.8.8.8",
                          "config": {
                              "address": "8.8.8.8"
                          }
                      },
                      {
                          "address": "192.168.10.1",
                          "config": {
                              "address": "192.168.10.1"
                          }
                      },
                      {
                          "address": "192.168.11.1",
                          "config": {
                              "address": "192.168.11.1"
                          }
                      }
                  ]
              }
          }
      }
  }


To set System Time settings use the following API call as an example:

.. code-block:: bash

  PATCH https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/

.. code-block:: json

  {
      "openconfig-system:system": {
          "clock": {
              "config": {
                  "timezone-name": "America/New_York"
              }
          },
          "ntp": {
              "config": {
                  "enabled": "true"
              },
              "servers": {
                  "server": [
                      {
                          "address": "time.f5net.com",
                          "config": {
                              "address": "time.f5net.com"
                          }
                      }
                  ]
              }
          }
      }
  }

To set a Remote Logging destination:

.. code-block:: bash

  PATCH https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/

.. code-block:: json

  {
      "openconfig-system:system": {
          "logging": {
              "remote-servers": {
                  "remote-server": [
                      {
                          "host": "10.255.0.142",
                          "config": {
                              "host": "10.255.0.142",
                              "remote-port": "514"
                          },
                          "selectors": {
                              "selector": [
                                  {
                                      "facility": "LOCAL0",
                                      "severity": "INFORMATIONAL",
                                      "config": {
                                          "facility": "LOCAL0",
                                          "severity": "INFORMATIONAL"
                                      }
                                  }
                              ]
                          }
                      }
                  ]
              }
          }
      }
  }

---------------------------
Licensing the VELOS Chassis
---------------------------

Licensing for the VELOS system is handled at the chassis level. This is similar to how VIPRION licensing is implemented where the system is licensed once, and all subsystems inherit their licensing from the chassis. With VELOS, licensing is applied at the system controller level and all chassis partitions and tenants will inherit their licenses from the base system. There is no need to procure add-on licenses for MAX SSL/Compression or for tenancy/vCMP. This is different than VIPRION where there was an extra charge for virtualization/vCMP and in some cases for MAX SSL/Compression. For VELOS these are included in the base license at no extra cost. VELOS does not run vCMP, and instead runs tenancy.

Licenses can be applied via CLI, GUI, or API. A base registration key and optional add-on keys are needed, and it follows the same manual or automatic licensing capabilities of other BIG-IP systems. Licensing is accessible under the **System Settings > Licensing** page. **Automatic** will require proper routing and DNS connectivity to the Internet to reach F5’s licensing server. If this is not possible to reach the licensing server use the **Manual** method.

.. image:: images/initial_setup_of_velos_system_controllers/image7.png
  :width: 45%

.. image:: images/initial_setup_of_velos_system_controllers/image8.png
  :width: 45%

You can activate and display the current license in the GUI, CLI or API. To license the VELOS chassis automatically from the CLI:

.. code-block:: bash

  syscon-1-active(config)# system licensing install registration-key I1234-12345-12345-12345-1234567
  result License installed successfully.

To license the VELOS chassis manually you’ll need to get the dossier first:

.. code-block:: bash

  syscon-2-active(config)# system licensing get-dossier
  b9a9936886bada077d93843a281ce4c34bf78db0d6c32c40adea3a5329db15edd413fe7d7f8143fd128ebe2d97642b4ed9192b530788fe3965593e3b42131c66220401b16843476159414ceeba8af5fb67a39fe2a2f408b9…

You can then access F5’s licensing server (license.f5.com) and paste in the dossier when prompted:

.. image:: images/initial_setup_of_velos_system_controllers/image9.png
  :align: center
  :scale: 70%

.. image:: images/initial_setup_of_velos_system_controllers/image10.png
  :align: center
  :scale: 70%


This should generate a license that can be saved or pasted into the VELOS system using the command **system licensing manual-install license**:

.. image:: images/initial_setup_of_velos_system_controllers/image11.png
  :align: center
  :scale: 70%

.. code-block:: bash

  syscon-2-active(config)# system licensing manual-install license 
  Value for 'license' (<string>): 
  [Multiline mode, exit with ctrl-D.]
  >

You should paste in the license and when finished hit **<CTRL> D**.

.. code-block:: bash

  #
  > #-----------------------------------------
  > # Copyright 1996-2021, F5 Networks, Inc.
  > # All rights reserved. 
  > #-----------------------------------------
  > 
  result License installed successfully.
  syscon-2-active(config)# 

You can also view the EULA via the CLI:

.. code-block:: bash

  syscon-2-active(config)# system licensing get-eula 
  eula-text END USER LICENSE AGREEMENT

  DOC-0355-16

  IMPORTANT " READ BEFORE INSTALLING OR OPERATING THIS PRODUCT

  YOU AGREE TO BE BOUND BY THE TERMS OF THIS LICENSE BY INSTALLING,
  HAVING INSTALLED, COPYING, OR OTHERWISE USING THE SOFTWARE.  IF YOU
  DO NOT AGREE, DO NOT INSTALL OR USE THE SOFTWARE.

The CLI command **show system licensing** will display the chassis level licensing:

.. code-block:: bash

  syscon-2-active# show system licensing 
  system licensing license 
                          Licensed version    1.2.0
                          Registration Key    V0453-12345-12345-12345-1234567
                          Licensed date       2020/12/08
                          License start       2020/12/07
                          License end         2021/01/08
                          Service check date  2020/12/08
                          Platform ID         F101
                          Appliance SN        chs600148s
                          
                          Active Modules
                            Local Traffic Manager, CX410 (E428722-4444383)
                            Best Bundle, CX410
                            APM-Lite
                            Carrier Grade NAT (AFM ONLY)
                            Max Compression, CX410
                            Rate Shaping
                            Max SSL, CX410
                            DNS, Max QPS, CX410
                            Advanced Routing, CX410
                            Advanced Firewall Manager, CX410
                            Advanced Web Application Firewall, CX410
                            Access Policy Manager, Base, CX410
                            Anti-Virus Checks
                            Base Endpoint Security Checks
                            Firewall Checks
                            Machine Certificate Checks
                            Network Access
                            Protected Workspace
                            Secure Virtual Keyboard
                            APM, Web Application
                            App Tunnel
                            Remote Desktop

**Note: VELOS supports AWAF vs. ASM licensing, and modules like AAM are not supported on the VELOS platform since it has reached End-of-Life.**

https://support.f5.com/csp/article/K70113407

To get the current licensing status via API use the following API call. Issue a **GET** to the floating IP address of the system controllers:

.. code-block:: bash

  GET https://{System-Controller-IP}:8888/restconf/data/openconfig-system:system/f5-system-licensing:licensing

.. code-block:: json

  {
      "f5-system-licensing:licensing": {
          "config": {
              "registration-key": {
                  "base": "L0747-54464-34113-50785-0819580"
              },
              "dossier": "0159d9644a6701c807e04ee146cdb9fcc604f42baa4c48f1c35682c6691dde0e30640626b90fbee83cda8384b1dbe4b92cbf3112426d441acff042617c6b8380e837698714159c6931cf874350f23c24fe1a783b0216ede8368626f9910e1908e0f6a541d8e61746d92f49ba897ca7579bf29de282767821465df467f409d8140bd928b103a1c621",
              "license": "#\nAuth vers :                        5b\n#\n#\n#       BIG-IP System License Key File\n#       DO NOT EDIT THIS FILE!!\n#\n#       Install this file as \"/config/bigip.license\".\n#\n#       Contact information in file /CONTACTS\n#\n#\n#       Warning: Changing the system time while this system is running\n#                with a time-limited license may make the system unusable.\n#\nUsage :                            F5 Internal Product Development\n#\n#\n#  Only the specific use referenced above is allowed. Any other uses are prohibited.\n#\nVendor :                           F5 Networks, Inc.\n#\n#       Module List \n#\nactive module :                    Best Bundle, CX410|Q163449-3930707|Max Compression, CX410|Rate Shaping|Max SSL, CX410|DNS, Max QPS, CX410|Advanced Firewall Manager, CX410|Advanced Web Application Firewall, CX410|Access Policy Manager, Base, CX410|Carrier Grade NAT (AFM ONLY)|Advanced Routing, CX410\noptional module :                  Advanced Protocols, CX410\noptional module :                  Anti-Bot Mobile, CX410\noptional module :                  APM, 1000 VPN Users\noptional module :                  APM, 10000 VPN Users\noptional module :                  APM, 25000 VPN Users\noptional module :                  APM, 500 VPN Users\noptional module :                  APM, 5000 VPN Users\noptional module :                  Basic Policy Enforcement Manager, CX410\noptional module :                  BPEM, Traffic Classification, CX410\noptional module :                  DataSafe, CX410\noptional module :                  Dynamic Policy Provisioning, CX410\noptional module :                  External Interface and Network HSM\noptional module :                  FIPS 140-2 Compliant Mode, CX410\noptional module :                  FIX Low Latency\noptional module :                  Intrusion Prevention System, CX410\noptional module :                  IP Intelligence, 1Yr, CX410\noptional module :                  IP Intelligence, 3Yr, CX410\noptional module :                  IPS Signatures, 1Yr, CX410\noptional module :                  IPS Signatures, 3Yr, CX410\noptional module :                  Multicast Routing, CX410\noptional module :                  PEM URL Filtering, 1Yr, CX410\noptional module :                  PEM URL Filtering, 3Yr, CX410\noptional module :                  PEM, Quota Management, CX410\noptional module :                  Policy Enforcement Manager, CX410\noptional module :                  Privileged User Access, 100 End-Points\noptional module :                  Privileged User Access, 1000 End-Points\noptional module :                  Privileged User Access, 250 End-Points\noptional module :                  Privileged User Access, 50 End-Points\noptional module :                  Privileged User Access, 500 End-Points\noptional module :                  Secure Web Gateway, 1Yr, 30K Sessions, CX410\noptional module :                  Secure Web Gateway, 1Yr, 60K Sessions, CX410\noptional module :                  Secure Web Gateway, 3Yr, 30K Sessions, CX410\noptional module :                  Secure Web Gateway, 3Yr, 60K Sessions, CX410\noptional module :                  SM2_SM3_SM4\noptional module :                  SSL Orchestrator, CX410\noptional module :                  Subscriber Discovery, CX410\noptional module :                  Threat Campaigns, 1Yr, CX410\noptional module :                  Threat Campaigns, 3Yr, CX410\noptional module :                  Traffic Classification, CX410\noptional module :                  URL Filtering, 1Yr, 30K Sessions, CX410\noptional module :                  URL Filtering, 1Yr, 60K Sessions, CX410\noptional module :                  URL Filtering, 3Yr, 30K Sessions, CX410\noptional module :                  URL Filtering, 3Yr, 60K Sessions, CX410\n#\n#       Accumulated Tokens for Module\n#       Max SSL, CX410  perf_SSL_Mbps 1  key Q163449-3930707\n#\nperf_SSL_Mbps :                    1\n#\n#       Accumulated Tokens for Module\n#       Access Policy Manager, Base, CX410  apm_access_sessions 100000000  key Q163449-3930707\n#\n#       Accumulated Tokens for Module\n#       Access Policy Manager, Base, CX410  apm_sessions 500  key Q163449-3930707\n#\n#       Accumulated Tokens for Module\n#       Access Policy Manager, Base, CX410  apm_urlf_limited_sessions 100000000  key Q163449-3930707\n#\napm_access_sessions :              100000000\napm_sessions :                     500\napm_urlf_limited_sessions :        100000000\n#\n#       License Tokens for Module Advanced Web Application Firewall, CX410 key Q163449-3930707\n#\nwaf_gc :                           enabled\nmod_waf :                          enabled\nmod_datasafe :                     enabled\nmod_asm :                          enabled\nltm_persist_cookie :               enabled\nltm_persist :                      enabled\nltm_lb_rr :                        enabled\nltm_lb_ratio :                     enabled\nltm_lb_priority :                  enabled\nltm_lb_pool_member_limit :         UNLIMITED\nltm_lb_least_conn :                enabled\nltm_lb_l3_addr :                   enabled\nltm_lb :                           enabled\nasm_apps :                         unlimited\n#\n#       License Tokens for Module Best Bundle, CX410 key Q163449-3930707\n#\nperf_vcmp_max_guests :             100000\nperf_PVA_dram_limit :              enabled\nnw_vlan_groups :                   enabled\nmod_ltm :                          enabled\nmod_lbl :                          enabled\nmod_ilx :                          enabled\nltm_network_virtualization :       enabled\nfpga_performance :                 enabled\n#\n#       License Tokens for Module Advanced Firewall Manager, CX410 key Q163449-3930707\n#\nperf_SSL_total_TPS :               UNLIMITED\nnw_l2_transparent :                enabled\nmod_afw :                          enabled\nmod_afm :                          enabled\nltm_netflow_switching :            enabled\nltm_monitor_rule :                 enabled\n#\n#       License Tokens for Module Max SSL, CX410 key Q163449-3930707\n#\nperf_SSL_per_core :                enabled\nperf_SSL_cmp :                     enabled\n#\n#       License Tokens for Module Max Compression, CX410 key Q163449-3930707\n#\nperf_http_compression_Mbps :       UNLIMITED\nperf_http_compression_hw :         enabled\n#\n#       License Tokens for Module Advanced Routing, CX410 key Q163449-3930707\n#\nnw_routing_rip :                   enabled\nnw_routing_ospf :                  enabled\nnw_routing_isis :                  enabled\nnw_routing_bgp :                   enabled\nnw_routing_bfd :                   enabled\n#\n#       License Tokens for Module DNS, Max QPS, CX410 key Q163449-3930707\n#\nmod_dnsgtm :                       enabled\nltm_dnssec :                       enabled\nltm_dns_v13 :                      enabled\nltm_dns_rate_limit :               UNLIMITED\nltm_dns_rate_fallback :            UNLIMITED\nltm_dns_lite :                     enabled\nltm_dns_licensed_objects :         UNLIMITED\ngtm_rate_limit :                   UNLIMITED\ngtm_rate_fallback :                UNLIMITED\ngtm_licensed_objects :             UNLIMITED\n#\n#       License Tokens for Module Carrier Grade NAT (AFM ONLY) key Q163449-3930707\n#\nmod_cgnat :                        enabled\nltm_network_map :                  enabled\nltm_monitor_udp :                  enabled\nltm_monitor_tcp_ho :               enabled\nltm_monitor_tcp :                  enabled\nltm_monitor_radius :               enabled\nltm_monitor_icmp :                 enabled\nltm_monitor_gateway_icmp :         enabled\ndslite :                           enabled\ncgnat :                            enabled\n#\n#       License Tokens for Module Access Policy Manager, Base, CX410 key Q163449-3930707\n#\nmod_apm :                          enabled\napm_web_applications :             enabled\napm_remote_desktop :               enabled\napm_pingaccess :                   enabled\napm_na :                           enabled\napm_logon_page_fraud_protection :  enabled\napm_ep_svk :                       enabled\napm_ep_pws :                       enabled\napm_ep_machinecert :               enabled\napm_ep_fwcheck :                   enabled\napm_ep_avcheck :                   enabled\napm_ep :                           enabled\napm_app_tunnel :                   enabled\napm_api_protection :               enabled\napi_protection_infra :             enabled\n#\n#       License Tokens for Module Rate Shaping key Q163449-3930707\n#\nltm_bandw_rate_tosque :            enabled\nltm_bandw_rate_fairque :           enabled\nltm_bandw_rate_classl7 :           enabled\nltm_bandw_rate_classl4 :           enabled\nltm_bandw_rate_classes :           enabled\n#\n# Debug Msg - Is sol18346625 affected; Usage, \"2021-02-09 00.00.00\", started after requirement date \"2016-04-15 00.00.00\"\n#\n# LC disabled in accordance with https://support.f5.com/kb/en-us/solutions/public/k/18/sol18346625.html\n#\ngtm_lc :                           disabled\n#\n#       Licensing Information \n#\nLicensed date :                    20210209\nLicense start :                    20210208\nLicense end :                      20210406\nService check date :               20210307\n#\n#       Platform Information \n#\nRegistration Key :                 L0747-54464-34113-50785-0819580\nLicensed version :                 1.0.0\nPlatform ID :                      F101\nAppliance SN :                     chs600032s\n#\n#       Outbound License Dossier Validation\n#\nDossier :                          0159d9644a6701c807e04ee146cdb9fcc612426d441acff042617c6b8380e837698714159c6931cf874350f23c24fe1a783b0216ede8368626f9910e1908e0f6a541d8e6\n#\n#       Outbound License Authorization Signature\n#\nAuthorization :                    b03537af28c9542b1d215b22e232c6f59bb75a52d84934d42f5004a02b1c90874774b57c4da87952a4db78091ab13ab1971acdcbbddf8c56532f7593bddc70718f24b04b8060c34e798e60c9a462db4b385a00ff50af956809f4754d3d21315899e7cefe9d484f5a2522c05c2ab4e4f6d55952a43abf16799075a25f1c98155a579b9f562ac6f6acc14035ebb522792ef2cc7c3d6f873d933f2940c904dfd2f8293644b4b962630cc6fa73633a3dd0b3b309d40f066bca9ba5503a0670c1be9170df2a05bdf6d99d882ae8ad5d8148d9f040e05694ed31ee708fe2db110b780d812fcf8ed7a9ce61d3283f3182b2447e4e92ca44ac15db2b0871583be66a7f7e\n#\n#-----------------------------------------\n# Copyright 1996-2021, F5 Networks, Inc.\n# All rights reserved. \n#-----------------------------------------\n"
          },
          "state": {
              "license": "\nLicensed version    1.0.0\nRegistration Key    L0747-54464-34113-50785-0819580\nLicensed date       2021/02/09\nLicense start       2021/02/08\nLicense end         2021/04/06\nService check date  2021/03/07\nPlatform ID         F101\nAppliance SN        chs600032s\n\nActive Modules\n Best Bundle, CX410 (Q163449-3930707)\n  Max Compression, CX410\n  Rate Shaping\n  Max SSL, CX410\n  DNS, Max QPS, CX410\n  Advanced Firewall Manager, CX410\n  Advanced Web Application Firewall, CX410\n  Access Policy Manager, Base, CX410\n  Carrier Grade NAT (AFM ONLY)\n  Advanced Routing, CX410\n"
          }
      }
  }

--------------------------
Chassis Partition Creation
--------------------------

Once the base level networking, licensing, and system settings are defined, the next step is to create the chassis partitions that will be used. The system ships with all 8 slots defined within the **default** chassis partition. If there is no need for more than one chassis partition, you can just utilize the default partition and any blades installed will automatically cluster together. Multiple tenants can be defined within the chassis partition and they can be restricted to specific vCPU’s as well as restricted to a single blade or be allowed to span across multiple blades. Conceptually this is similar to how vCMP guests are defined, but the underlying technology is different. 

If you decide to utilize the default partition you will need to assign an out-of-band management IP address, prefix and default route so that it can be managed. You can also define what release of F5OS software the chassis partition should run. Note this is different than the software the tenants will actually run. Once the management IP address is assigned you would then connect directly to that chassis partition to manage its networking, users and authentication, and tenants.

If you want to utilize other chassis partitions so that you can isolate all the networking for use by different groups, or for specific use cases you must first edit the default partition and remove any slots that you want to add to other partitions. Once a slot is removed from the default partition an option to create new chassis partitions is enabled, slots that are not tied to an existing chassis partition may be added to it. In the GUI screen below note that there is only the default partition, and all 8 slots are assigned to it. There are 3 blades installed in the chassis, and the rest show as **Empty**. 

.. image:: images/initial_setup_of_velos_system_controllers/image12.png
  :align: center
  :scale: 50%

For this configuration we will remove slots 1, 2, & 3 from the default chassis partition first. Once they are removed from default partition they can be assigned to a new partition. Select the checkbox next to the default partition and then click **Edit**.

.. image:: images/initial_setup_of_velos_system_controllers/image13.png
  :align: center
  :scale: 70%


You can then click on the slots in the graphic that you want to remove (slots 1, 2, & 3), and they will change from dark blue to light blue. Also note the **Selected Slots** will remove these slots as indicated by **4,5,6,7,8**. Click **Save** to complete removing slots 1-3 from the default partition.

.. image:: images/initial_setup_of_velos_system_controllers/image14.png
  :align: center
  :scale: 70%

The slots will turn white indicating they are not assigned to any chassis partition.

.. image:: images/initial_setup_of_velos_system_controllers/image15.png
  :align: center
  :scale: 70%

Next, create a new chassis partition that includes slots 1 & 2, and it will be named **bigpartition**. In the graphic click on slots 1 & 2 and they should turn grey, then select **Create**. 

.. image:: images/initial_setup_of_velos_system_controllers/image16.png
  :align: center
  :scale: 70%

The partition name must start with a letter, and cannot contain any special characters, only alpha-numeric characters are allowed. Fill in the **Name, IP Address, Prefix Length,** and **Gateway** fields. Finally select a Partition Image which defines the software release for the chassis partition. If there are no releases to choose from you must upload a valid chassis **partition image** into the system controller. You may download F5OS images form downloads.f5.com. When done click **Save** to create the new chassis partition.  

.. image:: images/initial_setup_of_velos_system_controllers/image17.png
  :align: center
  :scale: 70%

You can monitor the chassis partition status; it will go from **Disabled** to **Starting** to **Running**. 

.. image:: images/initial_setup_of_velos_system_controllers/image18.png
  :align: center
  :scale: 70%

Next repeat the process and create another chassis partition for slot3 naming it **smallpartition** and supplying IP address, prefix, and gateway along with a partition image.

.. image:: images/initial_setup_of_velos_system_controllers/image19.png
  :align: center
  :scale: 70%

You’ll then see a summary of all 3 partitions each with a unique **partition ID**, along with their **Operational State**.

.. image:: images/initial_setup_of_velos_system_controllers/image20.png
  :align: center
  :scale: 50%

If you click on the **Dashboard**, you’ll see a graphical representation that has slots color coded based on the partition they are assigned to:

.. image:: images/initial_setup_of_velos_system_controllers/image21.png
  :align: center
  :scale: 70%

Creating a Chassis Partition via the CLI
----------------------------------------

Before creating a chassis partition via the CLI you must first ensure that there are available slots in order to create a partition. You can issue the command show running-config slots to see what slots are available.  If all the slots are assigned to a partition like **default**, then you’ll need to move some of the slots to the partition **none** before they can be added to a new partition. 

.. code-block:: bash

  syscon-2-active# show running-config slots
  slots slot 1
  enabled
  partition default
  !
  slots slot 2
  enabled
  partition default
  !
  slots slot 3
  enabled
  partition default
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

In this case we will mimic the flow in the GUI section where there are 3 blades installed in slots 1-3. Enter config mode and configure each slot to be in the partition none and then commit the changes. 

.. code-block:: bash

  syscon-2-active(config)# slots slot 1 partition none 
  syscon-2-active(config-slot-1)# exit
  syscon-2-active(config)# slots slot 2 partition none 
  syscon-2-active(config-slot-2)# exit
  syscon-2-active(config)# slots slot 3 partition none 
  syscon-2-active(config-slot-3)# exit
  syscon-2-active(config)# commit

Now these slots are available to be assigned to a new partition. Enter config mode and add the partition by defining a name, adding a management IP address, prefix, and gateway.

.. code-block:: bash

  syscon-2-active(config)# partitions partition bigpartition config mgmt-ip ipv4 address 5.5.5.5 prefix-length 24 gateway 5.5.5.254

Next set the software version you want the partition to run, and enable the partition and then commit changes:

.. code-block:: bash

  syscon-2-active(config)# partitions partition bigpartition set-version iso-version 1.1.0-3698


  syscon-2-active(config)# partitions partition bigpartition config enable  


You can use the command show running-config partitions to see how each partition is configured:

.. code-block:: bash

  syscon-2-active# show running-config partitions 
  partitions partition Number2
  !
  partitions partition bigpartition
  config enabled
  config iso-version 1.1.0-3698
  config mgmt-ip ipv4 address 10.255.0.148
  config mgmt-ip ipv4 prefix-length 24
  config mgmt-ip ipv4 gateway 10.255.0.1
  !
  partitions partition default
  config enabled
  config iso-version 1.1.0-3698
  config pxe-server internal
  config mgmt-ip ipv4 address 0.0.0.0
  config mgmt-ip ipv4 prefix-length 0
  config mgmt-ip ipv4 gateway 0.0.0.0
  !
  partitions partition none
  config disabled
  !
  partitions partition smallpartition
  config enabled
  config iso-version 1.1.0-3698
  config mgmt-ip ipv4 address 10.255.0.141
  config mgmt-ip ipv4 prefix-length 24
  config mgmt-ip ipv4 gateway 10.255.0.1
  !

Creating a Chassis Partition via the API
----------------------------------------

Before creating any new chassis partitions you should ensure you have the proper F5OS partition images loaded onto the system controller. You can query the system controller to see what images are currently available on the system:

.. code-block:: bash

  GET https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-system-image:image/partition/config

.. code-block:: json

  {
      "f5-system-image:partition": {
          "config": {
              "os": {
                  "os": [
                      {
                          "version": "1.0.0-12251"
                      },
                      {
                          "version": "1.1.0-2954"
                      }
                  ]
              },
              "services": {
                  "service": [
                      {
                          "version": "1.0.0-12251"
                      },
                      {
                          "version": "1.1.0-2954"
                      }
                  ]
              },
              "iso": {
                  "iso": [
                      {
                          "version": "1.0.0-12251",
                          "service": "1.0.0-12251",
                          "os": "1.0.0-12251"
                      },
                      {
                          "version": "1.1.0-2954",
                          "service": "1.1.0-2954",
                          "os": "1.1.0-2954"
                      }
                  ]
              }
          },
          "state": {
              "controllers": {
                  "controller": [
                      {
                          "number": 1,
                          "os": {
                              "os": [
                                  {
                                      "version-os-partition": "1.1.0-2954",
                                      "controller": 1,
                                      "status": "ready",
                                      "date": "2021-02-06",
                                      "in-use": false
                                  },
                                  {
                                      "version-os-partition": "1.0.0-12251",
                                      "controller": 1,
                                      "status": "ready",
                                      "date": "2020-11-05",
                                      "in-use": false
                                  }
                              ]
                          },
                          "services": {
                              "service": [
                                  {
                                      "version-service-partition": "1.1.0-2954",
                                      "controller": 1,
                                      "status": "ready",
                                      "date": "2021-02-06",
                                      "in-use": false
                                  },
                                  {
                                      "version-service-partition": "1.0.0-12251",
                                      "controller": 1,
                                      "status": "ready",
                                      "date": "2020-11-05",
                                      "in-use": false
                                  }
                              ]
                          },
                          "iso": {
                              "iso": [
                                  {
                                      "version-iso-partition": "1.1.0-2954",
                                      "controller": 1,
                                      "status": "ready",
                                      "date": "2021-02-06",
                                      "in-use": true,
                                      "partitions": {
                                          "partition": [
                                              {
                                                  "name": "default",
                                                  "id": 1
                                              },
                                              {
                                                  "name": "bigpartition",
                                                  "id": 2
                                              },
                                              {
                                                  "name": "smallpartition",
                                                  "id": 3
                                              }
                                          ]
                                      }
                                  },
                                  {
                                      "version-iso-partition": "1.0.0-12251",
                                      "controller": 1,
                                      "status": "ready",
                                      "date": "2020-11-05",
                                      "in-use": false
                                  }
                              ]
                          }
                      },
                      {
                          "number": 2,
                          "os": {
                              "os": [
                                  {
                                      "version-os-partition": "1.1.0-2954",
                                      "controller": 2,
                                      "status": "ready",
                                      "date": "2021-02-06",
                                      "in-use": false
                                  },
                                  {
                                      "version-os-partition": "1.0.0-12251",
                                      "controller": 2,
                                      "status": "ready",
                                      "date": "2020-11-05",
                                      "in-use": false
                                  }
                              ]
                          },
                          "services": {
                              "service": [
                                  {
                                      "version-service-partition": "1.1.0-2954",
                                      "controller": 2,
                                      "status": "ready",
                                      "date": "2021-02-06",
                                      "in-use": false
                                  },
                                  {
                                      "version-service-partition": "1.0.0-12251",
                                      "controller": 2,
                                      "status": "ready",
                                      "date": "2020-11-05",
                                      "in-use": false
                                  }
                              ]
                          },
                          "iso": {
                              "iso": [
                                  {
                                      "version-iso-partition": "1.1.0-2954",
                                      "controller": 2,
                                      "status": "ready",
                                      "date": "2021-02-06",
                                      "in-use": true,
                                      "partitions": {
                                          "partition": [
                                              {
                                                  "name": "default",
                                                  "id": 1
                                              },
                                              {
                                                  "name": "bigpartition",
                                                  "id": 2
                                              },
                                              {
                                                  "name": "smallpartition",
                                                  "id": 3
                                              }
                                          ]
                                      }
                                  },
                                  {
                                      "version-iso-partition": "1.0.0-12251",
                                      "controller": 2,
                                      "status": "ready",
                                      "date": "2020-11-05",
                                      "in-use": false
                                  }
                              ]
                          }
                      }
                  ]
              }
          }
      }
  }

Next import the desired image into the system controller floating IP address using the path /var/import/staging. You will need to import from a remote HTTPS server. There is an insecure option if you don’t want to use certificate-based authentication to the remote HTTPS server. 

.. code-block:: bash

  POST https://{{Chassis1_System_Controller_IP}}:8888/api/data/f5-utils-file-transfer:file/import

.. code-block:: json

    {
        "input": [
            {
                "remote-host": "10.255.0.142",
                "remote-file": "/upload/{{Partition_ISO_Image_Full}}",
                "local-file": "images/staging/",
                "insecure": "",
                "f5-utils-file-transfer:username": "corpuser",
                "f5-utils-file-transfer:password": "Passw0rd!!"
            }
        ]
    }

You should see confirmation similar to below:

.. code-block:: json

  {
      "f5-utils-file-transfer:output": {
          "result": "File transfer is initiated.(import/staging/F5OS-C-1.1.0-2391.PARTITION.CANDIDATE.iso)"
      }
  }


You may also check the transfer status via the API:

.. code-block:: bash

  POST https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-utils-file-transfer:file/transfer-status

You will see a response similar to the one below showing status:

.. code-block:: json

  {
      "f5-utils-file-transfer:output": {
          "result": "\nS.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            \n1    |Import file|HTTPS   |/var/import/staging/F5OS-C-1.1.0-2391.PARTITION.CANDIDATE.iso|10.255.0.142        |F5OS-C-1.1.0-2391.PARTITION.CANDIDATE.iso                   |Peer certificate cannot be authenticated with given CA certificates\n"
      }
  }

The system ships with all slots configured in the default chassis partition.  Before you can create a new chassis partition you must remove any slots you want to add to it from the default partition. To view the current assignment of slots to partitions use the following API command: 

.. code-block:: bash

  GET https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-system-slot:slots

.. code-block:: json

  {
      "f5-system-slot:slots": {
          "slot": [
              {
                  "slot-num": 1,
                  "enabled": true,
                  "partition": "default"
              },
              {
                  "slot-num": 2,
                  "enabled": true,
                  "partition": "default"
              },
              {
                  "slot-num": 3,
                  "enabled": true,
                  "partition": "default"
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

Next remove the default partition from the slots you’d like to assign to any new chassis partitions. In this case we’ll assign the partition none to slots 1, 2, 3. Once the slots are unassigned (in the none partition), they can be added to a new chassis partition.

.. code-block:: bash

  PATCH https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/

.. code-block:: json

  {
      "f5-system-slot:slots": {
          "slot": [
              {
                  "slot-num": 1,
                  "enabled": true,
                  "partition": "none"
              },
              {
                  "slot-num": 2,
                  "enabled": true,
                  "partition": "none"
              },
              {
                  "slot-num": 3,
                  "enabled": true,
                  "partition": "none"
              }
              
          ]
      }
  }

Next a chassis partition called bigpartition will be created. It will be assigned an out-of-band management IP address, mask and gateway, along with an F5OS ISO version that must be loaded before the partition can be created.

.. code-block:: bash

  POST https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-system-partition:partitions

.. code-block:: json

    {
        "partition":
            {
        "name": "bigpartition",
            "config": {
                    "enabled": false,
                        "iso-version": "{{Partition_ISO_Image}}",
                        "mgmt-ip": {
                            "ipv4": {
                                "address": "{{Chassis1_BigPartition_IP}}",
                                "prefix-length": 24,
                                "gateway": "{{OutofBand_DFGW}}"
                            }
                        }
                    }
            }
    }


Next slots 1 & 2 will be assigned to the chassis partition called bigpartition:

.. code-block:: bash

  PATCH https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/

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
              }
              
          ]
      }
  }

Finally, the bigpartition containing slots 1 & 2 will be enabled:

.. code-block:: bash

  PATCH https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-system-partition:partitions/partition=bigpartition/config/enabled

.. code-block:: json

  {
      "enabled": true        
  }

The chassis partition will have a default username/password of admin/admin. When using the GUI you would be prompted on first login to change the password. To do this via the API use the following API call:

.. code-block:: bash

  POST https://{{Chassis1_BigPartition_IP}}:8888/restconf/operations/openconfig-system:system/aaa/authentication/users/user=admin/config/change-password

.. code-block:: json

    {
        "input": [
            {
                "old-password": "admin",
                "new-password": "{{Chassis_Partition_Password}}",
                "confirm-password": "{{Chassis_Partition_Password}}"
            }
        ]
    }

System Controller Configuration Options
=======================================

You can go back and review or edit various settings for the system controllers. 

-----------------------------------------
Network Settings -> Management Interfaces
-----------------------------------------

Under **Network Settings** you can view/edit the IP addresses both floating and static for the system controllers. If you would prefer to use DHCP for automatic assignment of these addresses, this may also be configured. You may also choose to configure Link Aggregation for the two out-of-band management interfaces for added redundancy. The LAG will consist of the out-of-band management interface from each controller. It has been designed to appear as a single device, so you can connect them to the same switch/LAG, or to a VPC where the LAG is across multiple switches that logically appear as one switch.

**NOTE: For the initial 1.1.x versions of F5OS ronly IPv4 IP addressing is supported for the F5OS platform layer. IPv4/IPv6 dual stack support has been added in the F5OS 1.2.x release. This limitation is only for the F5OS platform layer v1.1.x versions, BIG-IP tenants are capable of IPv4/v6 dual stack management.** 

.. image:: images/initial_setup_of_velos_system_controllers/image22.png
  :align: center
  :scale: 70%

-----------------------
Network Settings -> DNS
-----------------------

External **DNS Lookup Servers** and **Search Domains** can be configured. This will be required for things like automatic license activation, NTP server domain resolution, and iHealth integration and it is recommended to be configured. 

.. image:: images/initial_setup_of_velos_system_controllers/image23.png
  :align: center
  :scale: 70%

---------------------------------------
Software Management -> Partition Images
---------------------------------------

Each chassis partition will require an F5OS software release to be specified when enabled. You may also upgrade chassis partitions as needed. Chassis partition releases are loaded into the system controllers via the **Software Management > Partition Images** GUI page. F5OS releases are available on downloads.f5.com.

.. image:: images/initial_setup_of_velos_system_controllers/image24.png
  :align: center
  :scale: 70%

----------------------------------------
Software Management -> Controller Images
----------------------------------------

System controllers also run a unique F5OS software version. Both system controllers will need to run the same SW version. You can upload new F5 OS controller images via the **Software Management > Controller Images** GUI screen.

.. image:: images/initial_setup_of_velos_system_controllers/image25.png
  :align: center
  :scale: 70%

----------------------------------
System Settings -> Alarms & Events
----------------------------------

Alarms and Events can be viewed via the **System Settings > Alarms & Events** GUI page. You may optionally choose different severity levels to see more or less events. 

.. image:: images/initial_setup_of_velos_system_controllers/image26.png
  :align: center
  :scale: 70%

You may also change timeframe to see historical events, and optional refresh the screen via the controls on the right-hand side of the page:

.. image:: images/initial_setup_of_velos_system_controllers/image27.png
  :align: center

----------------------------------------
System Settings -> Controller Management
----------------------------------------

System controller status, HA state, and software upgrades are managed via the **System Settings > Controller Management** GUI page. The **High Availability Status** refers to the Kubernetes control plane status which operates in an Active / Standby manner. Only one controller will be active from a control plane perspective. This does not reflect the status of the layer2 switch fabric on the controllers which operates in an active/active mode.

An administrator can failover from one system controller to the other, and also perform software upgrades to the controllers as needed. You may perform a bundled upgrade which combines both the OS and F5 service components, or they can be upgraded independently. An upgrade which includes the OS, will be more disruptive timewise vs. an upgrade that only updates the F5 services. F5 support would recommend which type of upgrade may be needed for a particular fix, or feature. Ideally F5 expects to have to update the OS less frequently in the long term than the F5 Services. For the initial releases of F5OS updating the full ISO is recommended.

**NOTE: Intial v1.1.x F5OS versions do not support rolling upgrades for the system controllers. Any upgrade that is initiated will update both controllers in parallel which will result in an outage for the entire chassis. A proper outage window should be planned for any upgrades and updating the standby chassis first is recommended if possible. Rolling upgrade support has been added to the 1.2.x release of F5OS. Once the system controllers are starting from a 1.2.x release rolling upgrade is supported.** 

.. image:: images/initial_setup_of_velos_system_controllers/image28.png
  :align: center
  :scale: 70% 

-----------------------------------
System Settings -> System Inventory
-----------------------------------

The **System Settings > System Inventory** page provides status, part numbers and serial numbers for the different physical components including controllers, blades, fan trays, power supply controller units, power supplies, and LCD.

.. image:: images/initial_setup_of_velos_system_controllers/image29.png
  :align: center
  :scale: 70% 

-------------------------------
System Settings -> Log Settings
-------------------------------

Under **System Settings > Log Settings** you may add remote log servers for the F5OS system controllers. You can also specify the **Software Component Log Levels** which may be useful when troubleshooting specific issues.

.. image:: images/initial_setup_of_velos_system_controllers/image30.png
  :align: center
  :scale: 70% 

---------------------------------
System Settings -> File Utilities
---------------------------------

The **System Settings > File Utilities** page allows for importing or exporting specific types of files to and from the system controllers. Logs from the various log directories log can be exported, cores and qkviews can be imported/exported from diags/shared and system controller and chassis partition SW images can be imported into import/staging.

.. image:: images/initial_setup_of_velos_system_controllers/image31.png
  :align: center
  :scale: 70% 

The Import/Export utility requires an external HTTPS server to copy to/from. A pop-up will be displayed asking for remote HTTPS server information. The utility does not currently support downloading directly through a browser to your desktop. Download directly to a browser is planned for a future release.

.. image:: images/initial_setup_of_velos_system_controllers/image32.png
  :align: center
  :scale: 70% 

--------------------------------
System Settings -> Time Settings
--------------------------------

Under the **System Settings > Time Settings** page Network Time Protocol servers can be added so that the system controller time sources are sync’d to a reliable time source. The Time Zone may also be set.

.. image:: images/initial_setup_of_velos_system_controllers/image33.png
  :align: center
  :scale: 70% 

-------------------------------------
System Settings -> Device Certificate
-------------------------------------

Device certificates and keys can be uploaded via the **Systems Settings > Device Certificates** page.

.. image:: images/initial_setup_of_velos_system_controllers/image34.png
  :align: center
  :scale: 70% 

---------------------------------
System Settings -> System Reports
---------------------------------

The **System Settings > System Reports** page allows an admin to generate QKViews and optionally upload them to iHealth. 

.. image:: images/initial_setup_of_velos_system_controllers/image35.png
  :align: center
  :scale: 70% 

To generate a QKView click on the button in the upper right-hand corner. It will take some time for the QKview to be generated.  

.. image:: images/initial_setup_of_velos_system_controllers/image36.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image37.png
  :align: center
  :scale: 70% 

Once the QKView is generated, you can click the checkbox next to it, and then select **Upload to iHealth**. Your iHealth credentials will automatically fill in if entered them previously and be cleared if you want to use another account, you can optionally add an **F5 Support Case Number** and **Description**.

.. image:: images/initial_setup_of_velos_system_controllers/image38.png
  :align: center
  :scale: 70% 

If you would like to store iHealth credentials within the configuration you may do so via the system controller CLI. Enter config mode, and then use the system diagnostics ihealth config command to configure a username and password.

.. code-block:: bash

  syscon-1-active(config)# system diagnostics ihealth config ?
  Possible completions:
    authserver   Server for Authentication server of iHealth ex:- https://api.f5.com/auth/pub/sso/login/ihealth-api
    password     password to login to iHealth
    server       Server for iHealth ex:- https://ihealth-api.f5.com/qkview-analyzer/api/qkviews?visible_in_gui=True
    username     username to login to iHealth
  syscon-1-active(config)# system diagnostics ihealth config 

---------------------------------------
System Settings -> Configuration Backup
---------------------------------------

You may backup the confd configuration databases for the system controller via the GUI. The backups can then be copied off-box using the file utilities GUI option. Currently the GUI does not support the restoration of confd backups, this must be done via the CLI or API. 

.. image:: images/initial_setup_of_velos_system_controllers/image39.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image40.png
  :align: center
  :scale: 70% 

----------------------------
System Settings -> Licensing
----------------------------

Licensing for the VELOS system is handled at the chassis level. This is similar to how VIPRION licensing is handled, where the system is licensed once, and all subsystems inherit their licensing from the chassis. With VELOS licensing is applied at the system controller and all chassis partitions and tenants will inherit their licenses from the base system. There is no need to add-on license for MAX SSL/Compression or for tenancy. This is different than VIPRION where there was an extra charge for virtualization/vCMP and in some cases for MAX SSL/Compression. For VELOS these are included in the base license at no extra cost.

Licenses can be applied via CLI, GUI, or API. A base registration key and optional add-on keys are needed, and it follows the same manual or automatic licensing capabilities of other BIG-IP systems. 

.. image:: images/initial_setup_of_velos_system_controllers/image41.png
  :align: center
  :scale: 70% 

------------------------------------------
System Settings -> Software Install Status
------------------------------------------

The **System Settings -> Software Install Status** is used to verify and observe software updates of the system controllers and the chassis partitions. You can view the various stages of the install process.

.. image:: images/initial_setup_of_velos_system_controllers/image42.png
  :align: center
  :scale: 70% 

--------------------------
System Settings -> General
--------------------------

The **System Settings > General** page allows you to configure Appliance mode for the system controllers. Appliance mode is a security feature where all root and bash shell access are disabled. A user will only be able to utilize the F5OS CLI when Appliance mode is enabled. The page also displays the Systems Operation and Status which includes the Base OS and Service Versions currently running on the system controllers as well as the chassis partitions.

.. image:: images/initial_setup_of_velos_system_controllers/image43.png
  :align: center
  :scale: 70% 

--------------------------------
User Management -> Auth Settings
--------------------------------

Each layer of F5OS has its own user and authentication management. This allows for a separate set of users that have access to the system controllers, and each chassis partition. You may define local users and/or remote authentication via LDAP & RADIUS. Active Directory and TACACS remote auth are not currently supported in the v1.1.x versions of the F5OS layer. These have been added to the v1.2.x F5OS release and are currently configurable via the CLI only. GUI configuration support for TACACS and Active Directory will be added in a subsequent release. 

**Note: VELOS tenants running TMOS support Active Directory and TACACS for remote auth. The limitation is only for the F5OS v1.1.x platform layer.**

.. image:: images/initial_setup_of_velos_system_controllers/image44.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image45.png
  :align: center
  :scale: 70% 

--------------------------------
User Management -> Server Groups
--------------------------------

You may define Server Groups which are collections of remote auth servers that the VELOS platform layer will use to authenticate against. Currently LDAP and RADIUS are supported. For LDAP you may choose to authenticate of TCP or SSL. You can configure the remote host’s IP address and port. 

.. image:: images/initial_setup_of_velos_system_controllers/image46.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image47.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image48.png
  :align: center
  :scale: 70% 

------------------------
User Management -> Users
------------------------

Local Users may be defined, passwords set or changed, and then assigned to specific roles (Admin or Operator). An account may also be locked, and that may be changed here.

.. image:: images/initial_setup_of_velos_system_controllers/image49.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image50.png
  :align: center
  :scale: 70% 

.. image:: images/initial_setup_of_velos_system_controllers/image51.png
  :align: center
  :scale: 70% 

