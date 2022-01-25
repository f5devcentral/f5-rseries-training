==================
rSeries Networking
==================

Platform Layer Isolation
========================

Management of the new F5OS platform layer is completely isolated from in-band traffic networking and VLANs. It is purposely isolated so that it is only accessible via the out-of-band management network. In fact, there are no in-band IP addresses assigned to the F5OS layer, only tenants will have in-band management IP addresses and access. Tenants also have out-of-band connectivity.

This allows customers to run a secure/locked-down out-of-band management network where access is tightly restricted. The diagram below shows the out-of-band management access entering the rSeries appliance through **MGMT** port. The external MGMT port is bridged to an internal out-of-band network that connects to all tenants within the rSeries appliance. 

.. image:: images/rseries_networking/image1.png
  :align: center


Port Groups
===========

The portgroup component is used to control the mode of the physical ports. This controls whether a port is bundled or unbundled and the port speed. Currently the high speed ports do not support unbundling. **Adjacent** high speed ports (**1.0** & **2.0** on both the r5000/r10000 series) and (**11.0** & **12.0** on the r10000 series) must be configured in the same mode and speed currently. Either both are configured for 40Gb or both configured for 100Gb, you cannot mix and match. You cannot break out these ports to lower speeds (25Gb or 10Gb) via a breakout cables as this is currently unsupported. Low speed 25Gb/10Gb ports (**3.0** - **10.0** on both the r5000/r10000 series) and (**13.0** - **20.0** on the r10000 series) can be configured independently, and adjacent low speed ports can have different speed values. The term portgroup is used rather than simply “port” because some front panel ports may accept different types of SFPs. Depending on the portgroup mode value, a different FPGA version is loaded, and the speed of the port is adjusted accordingly. The user can modify the portgroup mode as needed through the F5OS CLI, GUI or API.


.. image:: images/rseries_networking/image2.png
  :align: center


Below is an example of the F5OS GUI **Port Groups** screen on a r10000 Series appliance. Note that any changes in configuration will require a reboot of the appliance to load a new FPGA bitstream image.

.. image:: images/rseries_networking/image3.png
  :align: center

.. image:: images/rseries_networking/image4.png
  :align: center
  :scale: 50%

Interfaces
==========

Interfaces will always be numbered starting with **1.0** and ending with the maximum number of ports on the appliance (**10.0** on the r5000 series and **20.0** on the r10000 series appliances). The type of optic in combination with the **port group** setting will dictate the speed of the interface. Interfaces can be run independetly or bundled together in Link Aggregation Groups. VLANs will be assigned to independent interfaces, or at the LAG configuration level if multiple interfaces are bundled together.


Supported Optics
================

Only F5 branded optics are officially supported on rSeries appliances. On rSeries r2000/r4000 models speeds of 1Gb, 10Gb, or 25Gb are supported. On the r5000/r10000 models speeds of 10Gb, 25Gb, 40Gb, or 100Gb are supported depending on the type of optics used and the port group configuration. Note the r5000/r10000 appliances do not support 1Gb connectivity. rSeries high speed interfaces will accept F5 approved QSFP+ & QSFP28 optics, while low speed ports will accept SFP28 and SFP+ optics. 3rd party optics are not officially supported per F5’s support policies: 

https://support.f5.com/csp/article/K8153. 

More details on each optic can be found in the F5 Platforms Accessories Guide:

https://techdocs.f5.com/en-us/hw-platforms/f5-plat-accessories.html


rSeries 1GB SFP+ Options (r2000 & r4000 only)
---------------------------------------------

+------------------------+------------+----------------------------------------------------------------------------------+
| 25GBASE-SR (SFP28)     | OPT-0053   | TRANSCEIVER, SFP28, 25G-SR, 100M, LC, MMF, 1W, F5 BRANDED                        |
+------------------------+------------+----------------------------------------------------------------------------------+
| 25GBASE-LR (SFP28)     | OPT-0054   | TRANSCEIVER, SFP28, 25G-LR, 10KM, LC, SMF, 1.5W, F5 BRANDED                      |
+------------------------+------------+----------------------------------------------------------------------------------+


rSeries 10GB SFP+ Options
-------------------------

+------------------------+------------+------------------------------------------------------------------------------------------+
| 10G Active DAC (SFP+)  | CBL-0138   | CABLE ASSEMBLY, SFP+ ACTIVE, COPPER, 10GBPS, 30 AWG, 3.0m, M-M, 3.3V, GEN 2, F5 BRANDED  |
+------------------------+------------+------------------------------------------------------------------------------------------+
| 10GBASE-LR (SFP+)      | OPT-0017   | TRANSCEIVER, SFP+, 10GIG, 1310nm, 10Km, LC, SMF.LIMITING, DDM, -5/70C, F5 BRANDED        |
+------------------------+------------+------------------------------------------------------------------------------------------+
| 10GBASE-SR (SFP+)      | OPT-0016   | TRANSCEIVER, SFP+, 10GIG, 850nm, 300m, LC, MMF, LIMITING, DDM, F5 BRANDED                |
+------------------------+------------+------------------------------------------------------------------------------------------+
| ??/     | OPT-0016   | TRANSCEIVER, SFP+, 10GIG, 850nm, 300m, LC, MMF, LIMITING, DDM, F5 BRANDED                |
+------------------------+------------+------------------------------------------------------------------------------------------+

rSeries 25GB SFP28 Options
--------------------------

+------------------------+------------+----------------------------------------------------------------------------------+
| 25GBASE-SR (SFP28)     | OPT-0053   | TRANSCEIVER, SFP28, 25G-SR, 100M, LC, MMF, 1W, F5 BRANDED                        |
+------------------------+------------+----------------------------------------------------------------------------------+
| 25GBASE-LR (SFP28)     | OPT-0054   | TRANSCEIVER, SFP28, 25G-LR, 10KM, LC, SMF, 1.5W, F5 BRANDED                      |
+------------------------+------------+----------------------------------------------------------------------------------+

rSeries 40GB QSFP+ Options
--------------------------


+------------------------+------------+------------------------------------------------------------------------------+
| 40GBASE-LR4 (QSFP+)    | OPT-0030   | TRANSCEIVER, QSFP+, 40G-LR4, 10KM, LC, SMF, DDM, F5 BRANDED                  |
+------------------------+------------+------------------------------------------------------------------------------+
| 40GBASE-SR4 (QSFP+)    | OPT-0036   | TRANSCEIVER, QSFP+, 40GIG-SR4, 850NM, 100M, MPO, RESET, MMF, DDM, F5 BRANDED |
+------------------------+------------+------------------------------------------------------------------------------+
| 40G BiDi (QSFP+)       | OPT-0043   | TRANSCEIVER, QSFP+, 2X20G BIDI 850NM-900NM, 100M, LC, MMF, DDM, F5 BRANDED   |
+------------------------+------------+------------------------------------------------------------------------------+
| 40G-PSM4 (QSFP+)       | OPT-0045   | TRANSCEIVER, QSFP+, 40GIG-PSM4, 1310NM, 10KM, MPO, SMF, DDM, F5 BRANDED      |
+------------------------+------------+------------------------------------------------------------------------------+

rSeries 100GB QSFP28 Options
----------------------------

+------------------------+------------+----------------------------------------------------------------------------------+
| 100GBASE-SR4 (QSFP28)  | OPT-0031   | TRANSCEIVER, QSFP28, 100G-SR4, 850NM, MMF, MPO, DDM, BRANDED                     |
+------------------------+------------+----------------------------------------------------------------------------------+
| 100GBASE-LR4 (QSFP28)  | OPT-0052   | TRANSCEIVER, QSFP28, 100G-LR4, 10KM, LC, SMF, 4.5W, DDM, VELOCITY SDK, BRANDED   |
+------------------------+------------+----------------------------------------------------------------------------------+
| 100G-PSM4 (QSFP28)     | OPT-0055   | TRANSCEIVER, QSFP28, 100GIG-PSM4, 1310NM, 500M, MPO, SMF, F5 BRANDED             |
+------------------------+------------+----------------------------------------------------------------------------------+
| 100G BIDI (QSFP28)     | OPT-0047   | TRANSCEIVER, QSFP28, 100G BIDI, 100M, LC, MMF, F5 BRANDED                        |
+------------------------+------------+----------------------------------------------------------------------------------+


rSeries Optics SKU's 
---------------------

**Note: 100G BiDi is planned (please contact product management to discuss your requirements, as there are different standards available in the market)**

rSeries 1GB SFP SKU's
--------------------------

1Gb Optics are only supported on the r2000/r4000 platforms:

+----------------------+----------------------------------------------------------------------------------------+
| F5-UPG-SFP-R         | Field Upgrade: SFP Fiber Connector (1G - LC/850nm) ROHS                                |
+----------------------+----------------------------------------------------------------------------------------+
| F5-UPG-SFPLX-R       | Field Upgrade: SFP LX Fiber Connector (1G - LC/1310nm) ROHS                            |
+----------------------+----------------------------------------------------------------------------------------+


rSeries 10GB SFP+ SKU's
--------------------------

10Gb Optics are supported on all rSeries (r2000/r4000/r5000/r10000) platforms:

+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-SFPC-R        | Field Upgrade: SFP Copper Connector (10/100/1000 RJ45)) ROHS                          |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-SFP+-R        | Field Upgrade: SFP+ Fiber Connector (10G-LC/850nm) ROHS                               |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-SFP+LR-R      | Field Upgrade: SFP+LR Fiber Connector (10G-LC/1310nm) ROHS                            |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-SFPC+-3M-8    | Field Upgrade: Copper SFP+ 10G Direct Attach 8-Pack 3M (8900, 11000, B4200, B2100)    |
+----------------------+---------------------------------------------------------------------------------------+


rSeries 25GB SFP28 SKU's
--------------------------

25Gb Optics are supported on all rSeries (r2000/r4000/r5000/r10000) platforms:

+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-SFP28-SR      | Field Upgrade: Transceiver SFP28, 25G-SR, 100M, LC, MMF, DDM (rSeries ONLY)           |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-SFP28-LR      | Field Upgrade: Transceiver SFP28, 25G-LR, 100M, LC, MMF, DDM (rSeries ONLY)           |
+----------------------+---------------------------------------------------------------------------------------+


rSeries 40GB QSFP+ SKU's
--------------------------

40Gb Optics are only supported on the r5000/r10000 platforms:

+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP+SR4      | Field Upgrade: QSFP+ Transceiver (40G-SR4, 850NM, 100M, MPO, DDM Support)             |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP+LR4      | Field Upgrade: QSFP+ Transceiver (40G-LR4, 1310NM, 10KM, LC, SMF, DDM Support)        |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP+PSM4     | Field Upgrade: QSFP+ Transceiver (40G-PSM4, 4x10LR, 1310NM, 10KM, MPO/APC, SMF, DDM)  |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP+BD       | Field Upgrade: Transceiver QSFP+, 2X20G BIDI 850NM-900NM, 100M, LC, MMF, DDM          |
+----------------------+---------------------------------------------------------------------------------------+

rSeries 100GB QSFP28 SKU's
--------------------------

100Gb Optics are only supported on the r5000/r10000 platforms:

+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP28-SR4    | Field Upgrade: QSFP28 Transceiver (100G-SR4, 850NM, 70M/100M, OM3/OM4, MMF, MPO, DDM) |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP28-LR4    | Field Upgrade: QSFP28 Transceiver (100G-LR4, 10KM, LC, SMF, 4.5W, DDM)                |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP28-PSM4   | Field Upgrade: QSFP28 Transceiver (100G-PSM4, 500M, MPO/APC, SMF, DDM) ROHS           |
+----------------------+---------------------------------------------------------------------------------------+
| F5-UPG-QSFP28-BD     | Field Upgrade: Transceiver QSFP28, 100G BIDI, 100M, LC, MMF, DDM (rSeries ONLY)       |
+----------------------+---------------------------------------------------------------------------------------+


**Note: The QSFP+ & QSFP28 optics cannot be configured for unbundled mode - 4 x 25Gb (with a 100Gb QSFP28 optic) or 4 x 10Gb (with a 40Gb QSFP+ optic).  The following breakout cable SKU’s are not supported on rSeries currently.**

**THESE ARE UNSUPPORTED**

+---------------------+------+--------------------------------------------------------------------------------------------+
| F5-UPGVELSR4XSR3M   | CN   | VELOS Field Upgrade: QSFP28-QSFP+ Breakout Cable for SR4 ONLY MPO to 4LC (3 Meter 2 Pack)  |
+---------------------+------+--------------------------------------------------------------------------------------------+
| F5-UPGVELSR4XSR1M   | CN   | VELOS Field Upgrade: QSFP28-QSFP+ Breakout Cable for SR4 ONLY MPO to 4LC (1 Meter 2 Pack)  |
+---------------------+------+--------------------------------------------------------------------------------------------+
| F5-UPGVELSR4XSR10M  | CN   | VELOS Field Upgrade: QSFP28-QSFP+ Breakout Cable for SR4 ONLY MPO to 4LC (10 Meter 2 Pack) |
+---------------------+------+--------------------------------------------------------------------------------------------+

Breakout for 40G PSM4 or 100G PSM4 transceivers *ONLY* (Note these are not 2 pack):

**THESE ARE UNSUPPORTED**

+---------------------+------+----------------------------------------------------------------------------------------------+
| F5-UPG-VELPSMXLR10M | CN   | VELOS Field Upgrade: QSFP28-QSFP+ Breakout Cable for PSM4 ONLY. MPO/APC to 4LC (10 Meter)    |
+---------------------+------+----------------------------------------------------------------------------------------------+
| F5-UPG-VELPSM4XLR3M | CN   | VELOS Field Upgrade: QSFP28-QSFP+ Breakout Cable for PSM4 ONLY. MPO/APC to 4LC (3 Meter)     |
+---------------------+------+----------------------------------------------------------------------------------------------+

VLANs
=====

rSeries supports both 802.1Q tagged and untagged VLAN interfaces. In the current F5OS releases, double VLAN tagging (802.1Q-in-Q) is not supported. VLANs can be added to any individual port, or to a Link Aggregation Group. BIG-IP tenants can share the same VLANs if needed.


Link Aggregation Groups
=======================

rSeries allows for bonding of interfaces into Link Aggregation Groups or LAG’s. LAG’s can span across any port as long as they are configured to support the same speed. Links within a LAG must be the same type and speed. LAG’s may be configured for static or lacp mode.

An admin can configure the **LACP Type** to **LACP** or **Static**, the **LACP Mode** to be **Active** or **Passive**, and the **LACP Interval** to **Slow** or **Fast**.

Pipelines
=========

The r10000 and r5000 series of appliances expose internal pipelines (connection paths between internal FPGA's) to the user so that they can plan for the most optimal network connectivity to rSeries to avoid oversubscription. rSeries appliances will have multiple pipelines between FPGA's and each pipeline supports a max bandwidth of 100Gb. Front panel ports are statically mapped to different internal pipelines to distribute load, ideally proper knowlwedge of pipelines and planning will avoid any possible internal oversubscription scenarios.

If all ports are utilized and running at max bandwidth capacity simulataneously this may result in an oversubsciprion if the maximum bandwidth for the internal pipelines are achieved. By exposing the internal pipelines to the user, they can plan ahead and spread external network connections into specific ports to maximize pipeline bandwidth and avoid oversubscription. Currently the mapping of ports to internal piepleines is static and not configurable, although F5 may make this a configurable option in the future.

Below is an example of the total external front panel possible bandwidth exceeding internal pipeline bandwidth:

.. image:: images/rseries_networking/image5.png
  :align: center
  :scale: 120%

There are static mappings of external ports to specific internal pipelines. If you are not using all ports you can spread the used ports over the diffferent pipelines by chossing different front panel ports to avoid possible oversubscription scenarios.

.. image:: images/rseries_networking/image6.png
  :align: center
  :scale: 120%

Below shows the total piplines and ports for both the r5000 and r10000 series appliances.

.. image:: images/rseries_networking/image7.png
  :align: center
  :scale: 120%

You can view the front panel port to pipeline mapping in the CLI, GUI, or API of F5OS.

.. image:: images/rseries_networking/image8.png
  :align: center
  :scale: 50%

.. code-block:: bash


  Boston-r10900-1# show port-mappings 
                                                                              NUM                                             
                                          CAPACITY  ALLOCATED  OVERSUBSCRIBE   ALLOCATED  MAX                                  
  NAME       INDEX       PIPELINE GROUP   BW        BW         STATUS          PORTS      PORTS  PORTS                         
  -----------------------------------------------------------------------------------------------------------------------------
  default-1  PIPELINE-1  PIPELINEGROUP-1  100       200        OVERSUBSCRIBED  5          8      [ 1.0 3.0 4.0 5.0 6.0 ]       
             PIPELINE-2  PIPELINEGROUP-1  100       200        OVERSUBSCRIBED  5          8      [ 10.0 2.0 7.0 8.0 9.0 ]      
  default-2  PIPELINE-3  PIPELINEGROUP-2  100       200        OVERSUBSCRIBED  5          8      [ 11.0 13.0 14.0 15.0 16.0 ]  
             PIPELINE-4  PIPELINEGROUP-2  100       185        OVERSUBSCRIBED  5          8      [ 12.0 17.0 18.0 19.0 20.0 ]    