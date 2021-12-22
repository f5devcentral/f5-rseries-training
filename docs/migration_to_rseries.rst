=====================
Migration to rSeries
=====================


F5 understands migrating configurations to new platforms can be a challenge and we’ve developed a tool that will help customers migrate existing BIG-IP configurations into rSeries tenants. F5 BIG-IP Journeys app assists with migrating a configuration from any BIG-IP between versions 11.5.0 and 14.1.3 onto the rSeries platforma, which as of now runs on BIG-IP version 15.1.4. It allows

•	Flagging source configuration feature parity gaps and fixing them with custom or F5 recommended solutions, automated deployment of the updated configuration to rSeries tenant and post-deployment validation.
•	List of possibly unsupported configuration features for the rSeries platform that the journeys app is able to catch

The Journeys app is available for download at F5’s DevCentral Github site. It has recently been enhanced to add additional migration use cases such as per-app migration and AS3 conversion:

https://github.com/f5devcentral/f5-journeys