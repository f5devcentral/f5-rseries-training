=====================
Migration to rSeries
=====================


F5 understands migrating configurations to new platforms can be a challenge and we’ve developed tooling that will help customers migrate existing BIG-IP configurations into rSeries tenants. Originally, F5 promoted the F5 BIG-IP Journeys application that assisted with migrating UCS based configurations from older platforms such as iSeries to current TMOS versions running as tenants on the rSeries platforms.

The Journeys tool was originally hosted on GitHub but has now been moved to downloads.f5.com. It can still be used for migrations to specific supported TMOS versions, but it is being phased out for migrations to newer TMOS versions in favor of the platform-migrate utility. More details about the changes can be found in the following solution article.

`K000137313: F5 Journeys - BIG-IP upgrade and migration utility <https://my.f5.com/manage/s/article/K000137313>`_

Going forward, F5 has and will continue to enhance the native UCS based migration utility by enhancing the **platform-migrate** option to incorporate some the reporting that the Journeys tool provided. Specifically, a new **validate** option has been added to the platform-migrate utility in TMOS version 21.1.0. The platform-migrate utility will be easier to use than Journeys as it does not require maintaining an external Docker environment and runs native within the BIG-IP TMOS software. 

**tmsh load sys ucs platform-migrate validate**

`K82540512: Overview of the UCS archive 'platform-migrate' option <https://my.f5.com/manage/s/article/K82540512>`_

You can migrate UCS configuration files from iSeries and VIPRION BIG-IP systems to F5OS BIG-IP platforms using the platform-migrate option of the tmsh load sys ucs command. Beginning in version 21.1.0, the validate (dry run) capability enhances this workflow by checking UCS compatibility before migration and generating a report of unsupported or modified configuration elements.

`UCS Migration Workflow for F5OS BIG-IP Platforms <https://techdocs.f5.com/en-us/bigip-21-1-0/big-ip-system-migrating-devices-and-configurations-between-different-platforms/ucs-migration-workflow-for-f5os-big-ip-platforms.html>`_

Some additional important considerations about UCS files can be found in the following solution articles:

To avoid current Management IP change, you can specify the keep-current-management-ip option to the load sys ucs command.

`Overview of the UCS archive 'keep-current-management-ip' option <https://my.f5.com/manage/s/article/K000132494>`_

When migrating a BIG-IP configuration to an F5OS-A (rSeries) platform using a UCS file, the import may fail if the UCS contains configurations that are not supported on F5OS-A. Specifically, UCS files exported from BIG-IP devices that use trunk (LAG) interfaces require manual editing before they can be successfully imported on an rSeries tenant. 

`Step-by-Step Guide: How to Manually Fix UCS files for F5OS migration to rSeries platforms <https://my.f5.com/manage/s/article/K000160543>`_

This is addressed in version 21.0.0.1 and 14.4.1, but for prior versions an edit of the UCS will be required.

`Bug ID 658943: Errors when platform migration process is loading UCS using trunks on vCMP guest/F5OS Tenants <https://cdn.f5.com/product/bugtracker/ID658943.html>`_

During migrations, care should be taken to avoid Layer2 loops if shared VLANs are in use:

`Best Practices for VLAN Configuration on F5OS-Appliance (rSeries): Avoiding Layer 2 Loops <https://my.f5.com/manage/s/article/K000152319>`_

In addition to the platform-migrate utility, there is a migration based Ansible collection that can be leveraged to both setup F5OS on a target rSeries system, and migrate a configuration into an F5OS based TMOS tenant:

`Modernizing F5 Platforms with Ansible <https://community.f5.com/kb/technicalarticles/modernizing-f5-platforms-with-ansible/341973>`_








