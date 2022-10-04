==============================
rSeries Performance and Sizing
==============================


rSeries is a new generation of hardware appliances using the latest Intel CPUs for processing in addition to Field Programmable Gate Arrays (FPGA's) for hardware offload (on the r10000 and r5000 Series). Intel CPUs perform SSL processing and compression offload as was done with previous generation BIG-IP solutions such as iSeries and the VIPRION B4450. Older VIPRION blades such as the B2100, B2150, and B2250 use Intel processing, but use Cavium Nitrox for SSL offload. The newer generation Intel chipsets provide more modern SSL cipher support and can offload ECC (Elliptical Curve) based ciphers in hardware, which most previous generations of VIPRION blades and older appliances could not.

In addition to more modern Intel chipsets, the mid-range (r5000) and high-end (r10000) rSeries appliances also have extensive FPGA support. The r2000 and r4000 rSeries models do not include FPGA's and instead perform these functions in software with some specialized offload. In previous generations of F5 hardware the ePVA (FPGA) was used to offload varying workloads from FASTL4 to DDoS mitigation, and that functionality is brought forward and expanded upon in the new generation of rSeries hardware. 

Some additional links on the benefits of hardware offload using the ePVA in previous generation BIG-IP solutions:

https://techdocs.f5.com/content/dam/f5/kb/global/solutions/sol12837_pdf.html/12837.pdf

https://devcentral.f5.com/s/articles/F5-Fast-L4-Acceleration-and-the-F5-Smart-Coprocessor-prioritized-Fast-L4-Acceleration

In rSeries there are now multiple FPGA’s, the **Application Traffic Services Engine** (ATSE), and the **Appliance Switch** (ASW), and the **Network Services Socket** (NSO). In addition to supporting previous functions done by the ePVA there are also additional functions that were performed in software or 3rd party chipsets that are now handled within the FPGA’s. Below is an architectural diagram of the r10000 Series appliance. 

.. image:: images/rseries_performance_and_sizing/image1.png
  :align: center
  :scale: 40%

The r5000 appliance has a similar architecture but since it hits a different price/performance point than the r10000 it has fewer FPGA's, CPUs, and fewer physical ports.

.. image:: images/rseries_performance_and_sizing/image2.png
  :align: center
  :scale: 40%

Both the r4000 and r2000 appliances have a slightly different hardware architecture than the r5000 and r10000 appliances. They still run F5OS-A software, but they do not utilize FPGA's for hardware offload, and instead perform these functions in software and leverage SR-IOV. This means that CPUs do not need to be dedicated to the F5OS layer, leaving more CPU for tenants. These platforms also run a different class of Intel processing, and do not utilize hyperthreading like the higher end platforms do. These appliances are positioned for smaller scale environments and they do not support 40Gb or 100Gb interfaces. Instead they support 1Gb, 10Gb, and 25Gb interfaces. Below is the architecture of the r4000 appliance.

.. image:: images/rseries_performance_and_sizing/image3.png
  :align: center
  :scale: 40%

The r2000 appliance has a similar architecture to the r4000, but it has less memory and fewer CPU cores.

.. image:: images/rseries_performance_and_sizing/image4.png
  :align: center
  :scale: 40%  

**Note: In the initial 1.0.x versions of F5OS-A (for rSeries appliances), not all FPGA HW offload functions are enabled. Many will be added in the subsequent TMOS and F5OS releases. AFM DDoS mitigation offload is not fully supported in v1.0.x versions of F5OS-A and will run in software similar to how it would run in a BIG-IP VE. SSL and Compression HW offload are fully supported in the initial v1.0.x F5OS-A releases, as is FASTL4 HW offload. CGNAT, PEM, SPDAG, 802.1Q-in-Q (double) VLAN tagging, vWire, VLAN Groups, are not supported in the initial F5OS-A 1.0.x releases, and are being prioritized for a future release.**

When comparing rSeries to the previous generation iSeries appliances, it is important to note that rSeries provides more options for network connectivity including 25GB and 100Gb Ethernet support. rSeries appliances are generally providing up to 2x more performance than the previous generation iSeries appliances.

Looking at comparisons of iSeries i10800 versus the r10000 or the iSeries i5800 versus the r5000 you can see a 1.2x-2.4x increase in performance depending on which metric is looked at. From an SSL perspective the increase is 2.3x-10x for RSA based ciphers, and for Elliptical Curve, rSeries will offload that processing to hardware; some older BIG-IP appliances may have had to process more modern ciphers in software.

.. image:: images/rseries_performance_and_sizing/image5.png
  :align: center
  :scale: 40%

The performance numbers for rSeries already include any overhead for multitenancy as the platform is multitenant by default. There is nothing to switch on to enable multitenancy. VIPRION or iSeries on the other hand has the option of running multitenancy by enabling vCMP. Published data sheet numbers for VIPRION or iSeries are for bare-metal mode, where no virtualization (vCMP) is enabled. Enabling vCMP on VIPRION or iSeries has overhead and will reduce the overall performance of a blade or appliance as the hypervisor takes up CPU and memory resources.

How much performance drops can vary for different metrics, but F5 has always sized environments using a rule-of-thumb of ~20% hit on performance for enabling virtualization/vCMP. With rSeries the published data sheet numbers are with multitenancy enabled, so there is no need to calculate in an additional 20% drop due to virtualization being enabled.  

Platform vCPU Sizing
====================

r10000 vCPU Sizing
------------------

Each rSeries 10900 model has 48 vCPUs, but 12 of those vCPUs are reserved for use by the F5OS platform layer. This is different from iSeries where each vCPU gave a portion of its processing and memory to the hypervisor when vCMP was enabled. In the r10900, 36 vCPUs are available for tenants since the other 12 are reserved by F5OS. The diagram below depicts the r10900 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image10.png
  :align: center
  :scale: 30%

The r10800 model has 48 vCPUs, but 12 of those vCPUs are reserved for use by the F5OS platform layer and 8 vCPUs are disabled via licensing. This is different than iSeries where each vCPU gave a portion of its processing and memory to the hypervisor when vCMP was enabled. In the r10800, 28 vCPUs are available for tenants since 12 are reserved for F5OS, and 8 are disabled via licensing. The diagram below depicts the r10800 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image11.png
  :align: center
  :scale: 60%


The r10600 model has 48 vCPUs, but 12 of those vCPUs are reserved for use by the F5OS platform layer. This is different than iSeries where each vCPU gave a portion of its processing and memory to the hypervisor when vCMP was enabled. In the r10600, 24 vCPUs are available for tenants since the other 12 are reserved for F5OS, and 12 are disabled via licensing. The diagram below depicts the r10600 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image12.png
  :align: center
  :scale: 30%


r5000 vCPU Sizing
------------------

Each rSeries 5900 model has 32 vCPUs, but 6 of those vCPUs are reserved for use by the F5OS platform layer. This is different from iSeries where each vCPU gave a portion of its processing and memory to the hypervisor when vCMP was enabled. In the r5900, 26 vCPUs are available for tenants since the other 6 are reserved. The diagram below depicts the r5900 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image13.png
  :align: center
  :scale: 70%

The r5800 model has 32 vCPUs, but 6 of those vCPUs are reserved for use by the F5OS platform layer and 8 vCPUs are disabled via licensing. This is different from iSeries where each vCPU gave a portion of its processing and memory to the hypervisor when vCMP was enabled. In the r5800, 18 vCPUs are available for tenants since 6 are reserved for F5OS, and 8 are disabled via licensing. The diagram below depicts the r5800 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image14.png
  :align: center
  :scale: 70%

The r5600 model has 32 vCPUs, but 6 of those vCPUs are reserved for use by the F5OS platform layer. This is different than iSeries where each vCPU gave a portion of its processing and memory to the hypervisor when vCMP was enabled. In the r5600, 12 vCPUs are available for tenants since the other 6 are reserved for F5OS, and 14 are disabled via licensing. Note there is a limit of 8 tenants on thr r5600. The diagram below depicts the r5600 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image15.png
  :align: center
  :scale: 90%

r4000 vCPU Sizing
------------------

Each rSeries 4800 model has 16 CPUs (The 4000 platform does not utilize hyperhreading/vCPUs). No CPUs are dedicated to the F5OS platform layer which is different from the mid-range and high-end rSeries appliances. In the r4800 16 CPUs are available to be assigned to tenants. The diagram below depicts the r4800 CPU allocation: 

.. image:: images/rseries_performance_and_sizing/image16.png
  :align: center
  :scale: 90%

The r4600 model has 16 CPUs (The 4000 platform does not utilize hyperhreading/vCPUs). No CPUs are dedicated to the F5OS platform layer which is different from the mid-range and high-end rSeries appliances. In the r4600 12 CPUs are available to be assigned to tenants and 4 are disabled via licensing. The diagram below depicts the r4600 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image17.png
  :align: center
  :scale: 90%

r2000 vCPU Sizing
------------------

Each rSeries 2800 model has 8 CPUs (The 2000 platform does not utilize hyperhreading/vCPUs). No CPUs are dedicated to the F5OS platform layer which is different from the mid-range and high-end rSeries appliances. In the r2800 8 CPUs are available to be assigned to tenants (and only one tenant is supported). The diagram below depicts the r2800 CPU allocation: 

.. image:: images/rseries_performance_and_sizing/image18.png
  :align: center
  :scale: 70%

The r2600 model has 8 CPUs (The 2000 platform does not utilize hyperhreading/vCPUs). No CPUs are dedicated to the F5OS platform layer which is different from the mid-range and high-end rSeries appliances. In the r2600 4 CPUs are available to be assigned to tenants and 4 are disabled via licensing. The diagram below depicts the r2600 vCPU allocation: 

.. image:: images/rseries_performance_and_sizing/image19.png
  :align: center
  :scale: 70%


Memory Sizing
=============

In general migrating from an iSeries to the equivalent rSeries model in the mid-range will mean either 1.3x or 2.6x more memory. For the high-end it will either be 2.x more memory, or the same amount of memory (when comparing the 11600/11800).

.. image:: images/rseries_performance_and_sizing/image34.png
  :width: 45%

.. image:: images/rseries_performance_and_sizing/image35.png
  :width: 45%

Breaking down memory to get per vCPU numbers will help when dealing with current vCMP guest configurations where memory is allocated based on the number of vCPUs assigned to the guest. Because rSeries has a different architecture than iSeries there is a formula for calculating how much memory a vCPU will receive. The chart below shows the default RAM per vCPU allocation with 1vCPU tenant. 

  min-memory = (3.5 * 1024 * vcpu-cores-per-node) + 512


With rSeries the amount of RAM per vCPU will change slightly as more vCPUs are added to the tenant. Below are the default values for total RAM, and RAM per vCPU for the rSeries tenants. These are **Recommended** values, but rSeries provides **Advanced** options where memory per tenant can be customized to allocate more memory without having to allocate mor vCPU. See the Multitennancy section for more details on memory customization.

For resource provisioning you can use **Recommended** settings or **Advanced** settings. Recommended will allocate memory in proportion the number of vCPUs assigned to the tenant. Advanced mode will allow you to customize the memory allocation for this tenant. This is something not possible in previous generation iSeries appliances, but now you can overprovision memory assigned to the tenant. The default memory allocations for Recommended mode are shown below. Note: Not all rSeries appliances support the maximum number of vCPUs; this will vary by platform. Below is for the r10900 platform which supports up to 36 vCPUs for tenancy.

+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| **Tenant Size**       | **Physical Cores** | **Logical Cores (vCPU)** | **Min GB RAM**  | **RAM/vCPU**    |
+=======================+====================+==========================+=================+=================+
| rSeries 1vCPU Tenant  | 0.5                |  1                       | 4,096,000,000   | 4,096,000,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 2vCPU Tenant  | 1                  |  2                       | 7,680,000,000   | 3,840,000,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 4vCPU Tenant  | 2                  |  4                       | 14,848,000,000  | 3,712,000,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 6vCPU Tenant  | 3                  |  6                       | 22,016,000,000  | 3,669,333,333   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 8vCPU Tenant  | 4                  |  8                       | 29,184,000,000  | 3,648,000,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 10vCPU Tenant | 5                  |  10                      | 36,352,000,000  | 3,635,200,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 12vCPU Tenant | 6                  |  12                      | 43,520,000,000  | 3,626,666,667   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 14vCPU Tenant | 7                  |  14                      | 50,688,000,000  | 3,620,571,429   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 16vCPU Tenant | 8                  |  16                      | 57,856,000,000  | 3,616,000,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 18vCPU Tenant | 9                  |  18                      | 65,024,000,000  | 3,612,444,444   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 20vCPU Tenant | 10                 |  20                      | 72,192,000,000  | 3,609,600,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 22vCPU Tenant | 11                 |  22                      | 79,360,000,000  | 3,607,272,727   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 24vCPU Tenant | 12                 |  24                      | 86,528,000,000  | 3,605,333,333   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 26vCPU Tenant | 13                 |  26                      | 93,696,000,000  | 3,603,692,308   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 28vCPU Tenant | 14                 |  28                      | 100,864,000,000 | 3,602,285,714   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 30vCPU Tenant | 15                 |  30                      | 108,032,000,000 | 3,601,066,667   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 32vCPU Tenant | 16                 |  32                      | 115,200,000,000 | 3,600,000,000   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 34vCPU Tenant | 17                 |  34                      | 122,368,000,000 | 3,599,058,824   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+
| rSeries 36vCPU Tenant | 18                 |  36                      | 129,536,000,000 | 3,598,222,222   |
+-----------------------+--------------------+--------------------------+-----------------+-----------------+

Each rSeries appliance has an overall amount of memory for the appliance, and the F5OS layer will take a portion of RAM, leaving the rest for use by tenants. Below is the amount of memory used by F5OS on each of the rSeries appliances. The table also displays the total minimum amount of RAM allocated using the recommended values, and how much extra RAM is available for tenants beyond the recommended values.

Using the minimum Recommended values per tenant ~129GB of RAM will be allocated for the r10000 Series tenants, leaving ~15GB of additional RAM. You may over-allocate RAM to any tenant until the extra 15GB of RAM is depleted. There is a formula for figuring out the minimum amount of RAM a particular tenant size will receive using the recommended values:

**min-memory = (3.5 * 1024 * vcpu-cores-per-node) + 512**


+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| **rSeries Platform**  | **Memory per System** | **Memory use by F5OS**  | **Memory Available to Tenants**  | **Mininimum RAM used (Max vCPU)**          |  **Extra RAM Available for Tenants**  |
+=======================+=======================+=========================+==================================+============================================+=======================================+
| r10900 Series         | 256GB RAM             | 25GB                    | 231,906,000,000                  | 129,536,000,000                            | 102,370,000,000                       |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r10800 Series         | 256GB RAM             | 25GB                    | 231,906,000,000                  | 108,032,000,000                            | 123,874,000,000                       |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r10600 Series         | 256GB RAM             | 25GB                    | 231,906,000,000                  | 86,528,000,000                             | 145,378,000,000                       |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r5900 Series          | 128GB RAM             | 15GB                    | 113,132,000,000                  | 93,696,000,000                             | 19,436,000,000                        |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r5800 Series          | 128GB RAM             | 15GB                    | 113,132,000,000                  | 65,024,000,000                             | 48,108,000,000                        |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r5600 Series          | 128GB RAM             | 15GB                    | 113,132,000,000                  | 43,520,000,000                             | 69,612,000,000                        |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r4800 Series          | 64GB RAM              | 8GB                     | 56GB                             | TBD                                        | TBD                                   |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r4600 Series          | 64GB RAM              | 8GB                     | 56GB                             | TBD                                        | TBD                                   |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r2800 Series          | 32GB RAM              | 8GB                     | 24GB                             | TBD                                        | TBD                                   |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
| r2600 Series          | 32GB RAM              | 8GB                     | 24GB                             | TBD                                        | TBD                                   |
+-----------------------+-----------------------+-------------------------+----------------------------------+--------------------------------------------+---------------------------------------+
