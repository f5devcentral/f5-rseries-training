===============================
Points of Management in rSeries
===============================

There are two main points of management within the rSeries appliances: The **F5OS platform layer**, and the individual **tenants**. Each support their own CLI, webUI, and API access and have their own authentication and user configuration. 

.. image:: images/rseries_points_of_management/image1.png
  :align: center
  :scale: 80%

Additionally, they each run their own version of software; tenants are able to run specific versions of TMOS, which have been approved to run on the rSeries platform. The intial supported version is 15.1.5 for the r10000 and r5000 appliances and 15.1.6 for the r4000 and r2000 appliances. The F5OS platform layer runs its own version of F5OS, which is unique to the rSeries appliances. On downloads.f5.com the rSeries versions of F5OS are referred to as **F5OS-A** where the **A** stands for **Appliances**. The VELOS chassis also runs F5OS, but that version is designated as **F5OS-C**, where **C** stands for **Chassis**.

.. image:: images/rseries_points_of_management/image2.png
  :align: center
  :scale: 80%

At the F5OS platform layer, initial configuration consists of out-of-band management IP addresses, routing, and other system parameters like DNS & NTP. Licensing is also configured at the F5OS layer and is similar to iSeries with vCMP configured in that it is applied at the appliance level and inherited by all tenants. In-band networking (VLANs, Interfaces, Link Aggregation Groups) are also configured within the F5OS platform layer. Once networking is set up tenants can be provisioned and deployed from the F5OS management interfaces. Once the tenant is deployed, it is managed like any other BIG-IP. This is very similar to how vCMP guests are managed on iSeries or VIPRION.  Please refer to the **rSeries Systems Administration webUIde** on askf5.com for more detailed information.

https://techdocs.f5.com/en-us/f5os-a-1-0-0/f5-rseries-systems-administration-configuration.html





  
