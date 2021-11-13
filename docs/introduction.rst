=============
Introduction
=============

rSeries is F5’s next generation appliance-based solution that will replace the current iSeries platforms. The rSeries platforms have many advantages over the current iSeries architecture. This guide will highlight the differences between the two architectures and then provide details on how to configure, monitor and troubleshoot the new platforms so that customers considering adoption understand how rSeries will fit within their environment. 


rSeries Overview
===============

-------------------------------
Kubernetes Based Platform Layer
-------------------------------

The major difference between rSeries and iSeries is the introduction of a new Kubernetes-based platform layer (called F5OS) that will allow for some exciting new capabilities. Luckily customers won’t need to learn Kubernetes in order to manage the new appliances, it will be abstracted from the administrator who will be able to manage the new platform layer via familiar F5 CLI, GUI, or API interfaces. 

rSeries will continue to provide hardware acceleration and offload capabilities in a similar way that iSeries did, however more modern FPGA, CPU, and crypto offload capabilities have been introduced. The new F5OS platform layer will allow rSeries to run different types of tenants within the same appliance. As an example, rSeries will be able to run:

•	Existing TMOS/BIG-IP tenants*
•	Future support for Next-generation BIG-IP software tenants (BIG-IP MA/Modular Architecture)
•	In the future the possibility of running approved 3rd party tenants 

 * specific software releases

.. image:: images//image1.png
  :align: center
  :scale: 70%



Customers will be able to migrate existing BIG-IP devices, or vCMP guests into a tenant running on VELOS. A tenant is conceptually similar to a vCMP guest on the VIPRION platform. Once inside the tenant, the management experience will be similar to the experience on existing BIG-IP platforms. The BIG-IP tenant will be managed just as a vCMP guest is managed today on VIPRION. The administrator can connect directly to the tenant’s GUI, CLI, or API and have the same experience as they have with their existing platforms. 

In the future BIG-IP MA tenants will be able to be provisioned within the same chassis, which will allow customers to leverage the next generation of BIG-IP software side-by-side with the existing BIG-IP software. What will differ from an administrator’s perspective is the initial setup of the F5OS platform layer. We’ll look at some additional architecture differences between VELOS and VIPRION before getting into how to manage and monitor the new F5OS platform layer. 

---------------------------------
Smaller Footprint, Higher Density
---------------------------------

The physical architecture of VELOS differs from the VIPRION platform in several ways. First, we’ve shrunk the size of the blades and now support double the number of slots in the same 4RU chassis footprint. The VELOS CX410 chassis supports 8 slots vs. only 4 on the VIPRION C2400 chassis.

.. image:: images//image2.png
  :align: center
  :scale: 70%

The second major difference vs. VIPRION is the introduction of centralized/redundant SX410 system controllers. The picture above shows the two redundant system controllers that ship by default with the VELOS 4RU CX410 chassis:

The system controllers are responsible for providing non-blocking connections and layer 2 switching between the 8 slots within the system. The system controllers are star-wired with multiple connections to each slot.  Each BX110 blade has one 100Gb backplane connection to each system controller (200Gb total). Future generation line cards may leverage additional connections. The picture below shows the backplane interconnections of a fully populated 8 slot CX410 chassis with 8 BX110 blades installed. 

.. image:: images//image3.png
  :align: center
  :scale: 40%

While both system controllers are active, they provide a non-blocking 1.6Tbs backplane between the 8 slots. Note that the BX110 linecards currently have a L4/L7 throughput rating of 95Gbs, but that is not a limitation of the backplane. If one of the system controllers were to fail, traffic would immediately switch over to the remaining system controller and the backplane bandwidth would be cut in half to 800Gbps. The backplane ports are aggregated together using link aggregation during normal operation, and traffic will be distributed according to the hashing algorithm of the Link Aggregation Group (LAG) thus utilizing both controllers for forwarding between slots.

A VIPRION chassis in comparison does not have a centralized switch fabric, and all blades are connected across the passive backplane in a full mesh fashion. The backplane in VIPRION was blocking, meaning the front panel bandwidth of a blade was greater than the blades backplane connectivity. Below is an example of the VIPRION C2400 chassis with B2250 blades. Each blade had a single 40Gb connection to every other blade. The total backplane bandwidth is 6 x 40 Gb = 240 Gb.

.. image:: images//image4.png
  :align: center
  :scale: 70%

The system controllers in VELOS are also the central point of management for the entire chassis. VIPRION required a dedicated out-of-band Ethernet management port and console connection for each blade inserted in the chassis. This meant more cabling, layer2 switch ports, and external terminal servers in order to fully manage the VIPRION chassis as seen below:

.. image:: images//image5.png
  :align: center
  :scale: 40%


With VELOS only the system controllers need to be cabled for out-of-band management and console connections. This reduces the amount of cabling, layer2 switch ports, and external terminal servers required for full chassis management as seen below:

.. image:: images//image6.png
  :align: center
  :scale: 40%

Additionally, the out-of-band Ethernet ports on the system controllers can be bundled together inside of a Link Aggregation Group.

----------------------------
The Kubernetes Control Plane
----------------------------

In addition to being the centralized layer2 switch fabric for the entire chassis, the system controllers also host the Kubernetes control plane that is responsible for provisioning resources/workloads within the chassis. VELOS utilizes an opensource distribution of Kubernetes called OpenShift, and specifically it uses the OKD project/distribution. This is largely abstracted away from the administrator as they won’t be configuring or monitoring containers or Kubernetes components. In the future some Kubernetes like features will start to be exposed, but it will likely be done through the VELOS F5OS CLI, GUI, or API’s. 

A combination of Docker Compose and Kubernetes is used within the F5OS layer.  Docker Compose is used to bring up the system controller and chassis partition software stacks as they need to be fully functional early in the startup process. Then Kubernetes takes over and is responsible for deploying workloads to the blades. One of the system controllers will be chosen to serve as primary and the other secondary from a Kubernetes control plane perspective. The central VELOS chassis F5OS API, CLI and GUI are served up from the primary system controller. The floating IP address will always follow the primary controller so CLI, GUI, and API access should not be prevented due to a controller failure.

.. image:: images//image7.png
  :align: center
  :scale: 40%

The diagram above is somewhat simplified as it shows a single software stack for the Kubernetes control plane. In reality there are multiple instances that run on the system controllers. There is a software stack for the system controllers themselves which provides F5OS CLI, GUI, and API management for the controllers as well as chassis partition (a grouping of blades) lifecycle management. There is also a unique stack for every chassis partition in the system. This software stack resides on the system controllers and can fail over from one controller to the other for added redundancy. It provides the F5OS CLI, GUI, and API functions for the chassis partition, as well as support for the networking services such as stpd, lldpd, lacpd, that get deployed as workloads on the blades.

The Kubernetes control plane is responsible for deploying workloads to the blades. This would happen when tenants or **chassis partitions** (see next section) are configured. We won’t get too deep into the Kubernetes architecture as its not required to manage the VELOS chassis. Know that the Kubernetes platform layer will allow F5 to introduce exciting new features in the future, but F5 will continue to provide abstracted interfaces for ease of management. By leveraging microservices and containers, F5 may be able to introduce new options such as shared multitenancy and dynamic scaling in the future. These are features that wer not supported on VIPRION.

------------------
Chassis Partitions
------------------

Another exciting new feature is the notion of grouping multiple VELOS blades together to form “mini VIPRIONS” within the same VELOS chassis. This will allow for another layer of isolation in addition to tenancy (similar to vCMP guests) that VIPRION didn’t support. This could be used to separate production from dev/test environments or to provide different security zones for different classes of applications. Within a VELOS chassis an administrator can group together one or more blades to form a chassis partition. A chassis may contain multiple chassis partitions and a blade may belong to only one chassis partition at a time. The minimum unit for a chassis partition is one blade and the maximum is 8 blades within the CX410 chassis.
 
**Note: Chassis partitions are not related to TMOS admin partitions which are typically used to provide admin separation within a TMOS instance.** 
 
A chassis partition runs its own unique F5OS software image, has a unique set of users/authentication, and is accessed via its own GUI, CLI and API. The chassis partition can be further divided to support multiple BIG-IP tenants. A tenant operates in a similar manner to how vCMP guests operated within the VIPRION chassis. It is assigned dedicated vCPU and memory resources and is restricted to specific VLANs for network connectivity. 

Below is an example of a VELOS CX410 chassis divided into 3 chassis partitions (Red, Green, and Blue). These chassis partitions are completely isolated from each other and the system controllers ensure no traffic can bleed from one chassis partition to another.  Once a chassis partition is created individual tenants can be deployed and they will be restricted to only the resources within that chassis partition. 

.. image:: images//image8.png
  :align: center
  :scale: 40%

-------
Tenants
-------

Tenancy is required to deploy any BIG-IP resources. VELOS is a multitenant chassis by default, there is no bare-metal mode, although it can be configured to emulate this mode with a single large tenant. You can configure one big chassis partition and assign all blades in the system to this resource. In fact, there is a “Default” partition that all blades are part of when inserted. You may change the slots assigned to the chassis partition by removing it from default and assigning to a new or existing chassis partition. A tenant could then be assigned to utilize all CPU and memory across that chassis partition. This would emulate a VIPRION system running “bare metal” where vCMP is not provisioned. 

When configuring HA between two VELOS chassis, there is no HA relationship across chassis at the F5OS layer where the system controllers or chassis partitions are configured. All HA is configured at the tenant level using Device Service Clustering, similar to how HA is configured between vCMP guests in separate VIPRION chassis. 

.. image:: images//image9.png
  :align: center
  :scale: 60%


