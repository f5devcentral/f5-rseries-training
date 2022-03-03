=========================
rSeries Inside the Tenant
=========================


Once a tenant is deployed you can connect/communicate directly to one of its CLI, webUI, or API interfaces. At this layer you are interacting with TMOS, i.e. the experience should be almost identical to a vCMP guest with some minor exceptions. Day to day management of the tenant will use the same CLI (tmsh), API (iControl) and webUI as TMOS instances or hardware devices running within customer envronments today. If you are using vCMP today, then many of the concepts will be familiar. If you run and iSeries appliance or VIPRION in a bare metal mode, then there will be some differences as rSeries will be configured with at least one tenant, and lower layer configuration and stats will come from the F5OS layer.

VLAN Behavior
=============

In both VIRPION and iSeries with vCMP, VLANs are created at the vCMP host layer and added to physical interfaces or trunks. The VLANs are then assigned to a vCMP guest at creation time using the VLAN name. VLAN names are passed through to the vCMP guest as-is, meaning the names are unaltered. 

rSeries follows a similar behavior as far as tenants inheriting VLANs from the F5OS layer. At tenant creation time the admin will assign VLANs to the tenant based on VLAN ID. The rSeries tenant will inherit the VLAN names just as a vCMP guest will. Below is an rSeries tenant showing the VLAN names being passed from the F5OS layer that the tenant was configured for: 

.. image:: images/rseries_inside_the_tenant/image1.png
  :align: center
  :scale: 70%

These are the VLANs as they appear in the F5OS platform layer. Notice that the tenant does not see all VLAN’s, only the ones that are assigned to it by the administrator:

.. image:: images/rseries_inside_the_tenant/image2.png
  :align: center
  :scale: 70%


Interface Behavior
==================

The number of interfaces within a tenant will be based upon the number of vCPUs assigned to the tenant. The screenshot below shows the interfaces inside the tenant lining up with the number of physical cores per tenant. In the example there are 36 vCPUs assigned to a single tenant, this will equate to 18 physical CPU’s. 


.. image:: images/rseries_inside_the_tenant/image4.png
  :align: center
  :scale: 70%


.. image:: images/rseries_inside_the_tenant/image3.png
  :align: center
  :scale: 70%

Trunk / HA Group Behavior
=========================

Within a vCMP guest Trunks can be used as part of the **HA Group** functionality to determine when a guest should fail over to its peer. 

An HA group is a specification of certain pools or host trunks (or any combination of these) that a guest administrator associates with a traffic group instance. The most common reason to configure an HA group is to ensure that failover is triggered when some number of trunk members become unavailable.




