=====================
Migration to rSeries
=====================


F5 understands migrating configurations to new platforms can be a challenge and we’ve developed tooling that will help customers migrate existing BIG-IP configurations into rSeries tenants. Originally, F5 promoted the F5 BIG-IP Journeys application that assisted with migrating UCS based configurations from older platforms such as iSeries to current TMOS versions running as tenants on the rSeries platforms.

The Journeys tool was originally hosted on GitHub, but has now been moved to downloads.f5.com. It can still be used for migrations to specific supported TMOS versions, but it is being phased out for migrations to newer TMOS versions in favor of the platform-migrate utility. More details about the changes can be found in the following solution article.

`K000137313: F5 Journeys - BIG-IP upgrade and migration utility <https://my.f5.com/manage/s/article/K000137313>`_

Going forward, F5 has and will continue to enhance the native UCS based migration utility by enhancing the **platform-migrate** option to incorporate some the reporting that the Journeys tool provided. Specifically, a new **validate** option has been added to the platform-migrate utility in TMOS version 21.1.0. The platform-migrate utility will be easier to use than Journeys as it does not require maintaining an external Docker environment and runs native within the BIG-IP TMOS software. 

**tmsh load sys ucs platform-migrate validate**

More details can be found in the following solution article:

`K82540512: Overview of the UCS archive 'platform-migrate' option <https://my.f5.com/manage/s/article/K82540512>`_

In addition to the Journeys tool and the platform-migrate utility, there is a migration based Ansible collection that can be leveraged to both setup F5OS on a target rSeries system, and migrate a configuration into an F5OS based TMOS tenant:

`Modernizing F5 Platforms with Ansible <https://community.f5.com/kb/technicalarticles/modernizing-f5-platforms-with-ansible/341973>`_








