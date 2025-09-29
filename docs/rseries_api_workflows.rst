=====================
rSeries API Workflows
=====================

There are two main points of management within the rSeries appliances: the F5OS-C platform layer, and the individual tenants. Both support their own CLI, webUI, and API access.

At the F5OS-A platform layer level, initial configuration consists of defining static management IP addresses, routing, and other system parameters like DNS and NTP. Licensing is also configured at the F5OS-A platform layer level and is similar to iSeries in that it is applied at the appliance level and inherited by all tenants.

For more information about configuring your system, see rSeries Systems: Getting Started and rSeries Systems: Administration and Configuration at support.f5.com.

You can use a RESTful API client like Postman to configure the F5 rSeries platform.

Download the F5 rSeries F5OS-A platform Postman Collection **Coming Soon**.

These workflows assume that the initial out-of-band management configuration has been completed.

Workflows
=========

Initial Setup of rSeries F5OS Platform Layer
--------------------------------------------

`Configure System Settings via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_platform_layer.html#system-settings-via-the-api>`_

`Manual Licensing via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_platform_layer.html#manual-licensing-via-api>`_

`Automatic Licensing via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_platform_layer.html#automatic-licensing-via-api>`_

Initial Setup of the rSeries Network Layer
------------------------------------------

`Configuring PortGroups via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_network_layer.html#configuring-portgroups-from-the-api>`_

`Configuring Interfaces via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_network_layer.html#configuring-interfaces-from-the-api>`_

`Configuring VLANs via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_network_layer.html#configuring-vlans-from-the-api>`_

`Configuring LAGs via API <https://clouddocs.f5.com/training/community/rseries-training/html/initial_setup_of_rseries_network_layer.html#configuring-lags-from-the-api>`_

Deploying an rSeries Tenant
---------------------------


`Loading Tenant Images from a Remote Server via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_deploying_a_tenant.html#loading-tenant-images-from-a-remote-server-via-api>`_

`Uploading Tenant Images from a Client Machine via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_deploying_a_tenant.html#uploading-tenant-images-from-a-client-machine-via-the-api>`_

`Creating a Tenant via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_deploying_a_tenant.html#creating-a-tenant-via-api>`_

`Validating Tenant Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_deploying_a_tenant.html#validating-tenant-status-via-api>`_

`Expanding a Tenant via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_deploying_a_tenant.html#expanding-a-tenant-via-api>`_

`Deleting a Tenant via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_deploying_a_tenant.html#deleting-a-tenant-via-the-api>`_


Securing / Hardening F5OS on rSeries
------------------------------------


`Adding Allow List Entries via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#adding-allow-list-entries-via-api>`_

`Managing Device Certificates, Keys, CSRs, and CAs via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#managing-device-certificates-keys-csrs-and-cas-via-api>`_

`Enabling F5OS Appliance Mode via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#enabling-f5os-appliance-mode-via-the-api>`_

`Enabling BIG-IP Tenant Appliance Mode via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#enabling-big-ip-tenant-appliance-mode-via-the-api>`_

`Validating Tenant Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#validating-tenant-status-via-api>`_

`Resource-Admin & Guest User Role via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#resource-admin-guest-user-role-via-api>`_

`Configuring SSH and CLI Timeouts & Deny Root SSH Settings via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#configuring-ssh-and-cli-timeouts-deny-root-ssh-settings-via-api>`_

`Token Lifetime via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#token-lifetime-via-api>`_

`Disabling Basic Auth via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#disabling-basic-auth-via-the-api>`_

`Setting Password Policies via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#setting-password-policies-via-api>`_

`Configuring Login Banner / MoTD via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#configuring-login-banner-motd-via-api>`_

`Enabling NTP Authentication via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#enabling-ntp-authentication-via-api>`_

`Configuring Management Ciphers via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#configuring-management-ciphers-via-api>`_

`Configuring Client Certificate Authentication via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#configuring-client-certificate-authentication-via-api>`_

`Proxy Server via API for Licensing and Qkview Uploads to iHealth <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#proxy-server-via-api-for-licensing-and-qkview-uploads-to-ihealth>`_

`Configuration of Audit Logs via F5OS API (F5OS-A 1.4.0 and Later) <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#configuration-of-audit-logs-via-f5os-api-f5os-a-1-4-0-and-later>`_

`Viewing Audit Logs via F5OS API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#viewing-audit-logs-via-f5os-api>`_

`Downloading Audit Logs via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_security.html#downloading-audit-logs-via-api>`_


rSeries Software Upgrades
-------------------------


`Importing F5OS-A Images from a Remote Server via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_software_upgrades.html#importing-f5os-a-images-from-a-remote-server-via-the-api>`_

`Uploading F5OS-A Images from a Client Machine via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_software_upgrades.html#uploading-f5os-a-images-from-a-client-machine-via-the-api>`_

`Upgrading F5OS via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_software_upgrades.html#upgrading-f5os-via-the-api>`_

`Loading Tenant Images from a Remote Server via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_software_upgrades.html#loading-tenant-images-from-a-remote-server-via-api>`_

`Uploading Tenant Images from a Client Machine via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_software_upgrades.html#uploading-tenant-images-from-a-client-machine-via-the-api>`_

rSeries Configuration Backup and Restore
----------------------------------------

`Backing Up F5OS via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#backing-up-f5os-via-api>`_

`Exporting F5OS Backup via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#exporting-f5os-backup-via-api>`_

`Downloading an F5OS Backup via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#downloading-an-f5os-backup-via-api>`_

`Resetting the system via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#resetting-the-system-via-api>`_

`Changing the Default Password and Importing F5OS Backups via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#changing-the-default-password-and-importing-f5os-backups-via-api>`_

`Importing an F5OS Backup from a Remote Server via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#importing-an-f5os-backup-from-a-remote-server-via-api>`_

`Uploading an F5OS Backup from a Client Machine via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#uploading-an-f5os-backup-from-a-client-machine-via-api>`_

`Restore via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_f5os_configuration_backup_and_restore.html#restore-using-the-api>`_

Diagnostics
-----------

`qkview Creation and Upload to iHealth via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#qkview-creation-and-upload-to-ihealth-via-api>`_

`qkview Download to Client via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#qkview-download-to-client-via-api>`_

`Downloading Logs from the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#downloading-logs-from-the-api>`_

`Viewing Event Logs via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#viewing-event-logs-from-the-api>`_

`Changing the Software Component Log Levels via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#changing-the-software-componenet-log-levels-via-api>`_

`Enabling AOM Access for SSH via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#enabling-aom-access-for-ssh-via-api>`_

`TCPDUMP Download to Client via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#tcpdump-download-to-client-via-api>`_

Monitoring rSeries Health & Alert Status
----------------------------------------

`Checking Active Alerts via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries_health_status.html#checking-active-alerts-via-api>`_

`Checking System Health via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries_health_status.html#checking-system-health-via-api>`_

`Filter to Get a Summary of System Health via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries_health_status.html#filter-to-get-a-summary-of-system-health-via-api>`_

Monitoring
----------

`Hardware and System Component Monitoring via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#hardware-and-system-component-monitoring-from-the-api>`_

`Appliance Component Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#appliance-component-status-from-the-api>`_

`LCD Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#lcd-status-from-the-api>`_

`Power Supply Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#power-supply-status-from-the-api>`_

`Storage Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#storage-status-from-the-api>`_

`CPU Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#cpu-status-from-the-api>`_

`Temperature Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#temperature-status-from-the-api>`_

`Memory Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#memory-status-from-the-api>`_

`Trusted Protection Module Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#trusted-protection-module-status-from-the-api>`_

`Software Health and Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#software-health-and-status-from-the-api>`_

`F5 Cluster Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#f5-cluster-status-via-api>`_

`F5 Service Instances Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#f5-service-instances-status>`_

`F5 Services Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#f5-services-status>`_

`Layer2 FDB Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#layer2-fdb-status>`_

`F5 Service-Pods Status via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#f5-service-pods-status>`_

`System Health via API <https://clouddocs.f5.com/training/community/rseries-training/html/monitoring_rseries.html#system-health>`_

rSeries F5OS-A SNMP Monitoring and Alerting
-------------------------------------------

`Downloading MIBs via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#downloading-mibs-via-api>`_

`Exporting MIBs to a Remote Server via the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#exporting-mibs-to-a-remote-server-via-the-api>`_

`Adding Allowed IPs for SNMP via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#adding-allowed-ips-for-snmp-via-api>`_

`Adding Interface and LAG descriptions via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#adding-interface-and-lag-descriptions-via-api>`_

`Configuring SNMP Access via API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#configuring-snmp-access-via-api>`_

`Enabling SNMP Traps in the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#enabling-snmp-traps-in-the-api>`_

`Downloading SNMP Logs from the API <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html#downloading-snmp-logs-from-the-api>`_