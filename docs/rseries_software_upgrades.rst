=========================
rSeries Software Upgrades
=========================


F5OS-A Platform Layer Upgrades
==============================

rSeries appliances ship with a new platform layer software called F5OS. There are two different flavors of F5OS, one for VELOS chassis based systems called F5OS-C (C is for Chassis), and F5OS-A (A is for Appliances) for rSeries appliances. The F5OS platform layer is upgraded independently from the tenant layer, and has its own unique image which can be downloaded from downloads.f5.com. Below is the default page for downloads.f5.com, note the **F5OS** section under **F5 Product Family** and under **Product Line** select **F5OS Appliance Software**. Click the hyperlink for this section.


.. image:: images/rseries_software_upgrades/image1.png
  :align: center
  :scale: 70%

Here you will see a drop down for the various supported versions available for download. Select the desired version, and accept the user agreement.

.. image:: images/rseries_software_upgrades/image2.png
  :align: center
  :scale: 70%

Here you can download the ISO file for F5OS along with checksum files and release notes:

.. image:: images/rseries_software_upgrades/image3.png
  :align: center
  :scale: 70%


Uploading F5OS Images via the GUI
---------------------------------

You can do this from the **System Settings -> Software Management** page.

.. image:: images/rseries_software_upgrades/image4.png
  :align: center
  :scale: 70%

Here you'll have the option to **Upload** or **Import** a new F5OS image. If you choose **Import** you will be prompted to provide the configuration details for a remote HTTPS server where the images can be downloaded from. If you choose **Upload** then you'll be able to upload an image directly from your local client machine via the browser. 

Alternatively, you may also upload images to the controller through the **System Settings -> File Utilities** page. You can select the **images/staging** option from the drop-down menu to import new F5OS-A images. Once uploaded into the staging area they will be imported and made available for upgrades after a brief delay.

.. image:: images/rseries_software_upgrades/image5.png
  :align: center
  :scale: 70%



After the upload completes, it will take some time for it to be fully imported and verified. At that point it should show up in the CLI and GUI. If you don’t see it immediately be patient and wait a few minutes for it to show up as it will not show up until the file has been verified. Inside the **ISO** file are two different types of software, the **OS** and the **Service**. All three types should be displayed in the **Software Management** page. In the example below the ISO for version 1.0.0-11432 consists of an **OS** and **Service** with the same version number. 

.. image:: images/rseries_software_upgrades/image6.png
  :align: center
  :scale: 70%

When upgrading the F5OS platform layer, you will have a choice of upgrading either a **Bundled** release meaning **OS** and **Services** are bundled together in an ISO image or **Unbundled** where you can upgrade service and/or OS independently. Note currently F5 has not released any service only or OS only releases, but they may be an option in the future. For now it is recommended to choose **Bundled** upgrades.

.. image:: images/rseries_software_upgrades/image7.png
  :align: center
  :scale: 70%

.. image:: images/rseries_software_upgrades/image8.png
  :align: center
  :scale: 70%  

Uploading F5OS Images via the CLI
---------------------------------

If you would prefer to upload the F5OS image via the CLI this can be done with the **file import** command. Use the **file import** command to get the F5OS image file from a remote HTTPS server or from a remote server over SCP or SFTP. Below is an example importing from a remote HTTPS server. Note the target directory should be **images/staging**:

.. code-block:: bash

    Boston-r10900-1# file import remote-host 10.255.0.142 remote-file /upload/F5OS-A-1.0.0-11432.R5R10.iso local-file images/staging/F5OS-A-1.0.0-11432.R5R10.iso username corpuser insecure
    Value for 'password' (<string>): ********
    result File transfer is initiated.(images/staging/F5OS-A-1.0.0-11432.R5R10.iso)
    Boston-r10900-1#

If a remote HTTPS server is not available you may also copy the file from the CLI over SCP by adding the **protocol scp** option to the command line:

.. code-block:: bash

    Boston-r10900-1# file import remote-host 10.255.0.142 remote-file /var/www/server/1/upload/F5OS-A-1.0.0-11432.R5R10.iso local-file images/staging/F5OS-A-1.0.0-11432.R5R10.iso username root insecure protocol scp
    Value for 'password' (<string>): ********
    result File transfer is initiated.(images/staging/F5OS-A-1.0.0-11432.R5R10.iso)


The command **file transfer-status** will provide details of the transfer progress and any errors:

.. code-block:: bash

    Boston-r10900-1# file transfer-status
    result 
    S.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                
    1    |Import file|HTTPS   |images/staging/F5OS-A-1.0.0-11432.R5R10.iso                 |10.255.0.142        |/upload/F5OS-A-1.0.0-11432.R5R10.iso                        |In Progress (86.0%)|Wed Jan  5 20:02:56 2022

    Boston-r10900-1# file transfer-status
    result 
    S.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                
    1    |Import file|HTTPS   |images/staging/F5OS-A-1.0.0-11432.R5R10.iso                 |10.255.0.142        |/upload/F5OS-A-1.0.0-11432.R5R10.iso                        |         Completed|Wed Jan  5 20:03:03 2022

    Boston-r10900-1# 


You can view the current F5OS images and their status in the F5OS CLI by using the **show system image** command:

.. code-block:: bash

    Boston-r10900-1# show system image 
                                    IN    
    VERSION OS   STATUS  DATE        USE   
    ---------------------------------------
    1.0.0-11432  ready   2021-12-03  true  

    VERSION                          IN    
    SERVICE      STATUS  DATE        USE   
    ---------------------------------------
    1.0.0-11432  ready   2021-12-03  true  

                                    IN     
    VERSION ISO  STATUS  DATE        USE    
    ----------------------------------------
    1.0.0-11432  ready   2021-12-03  false  


You can alternatively copy the F5OS images into the management IP address of F5OS from a client machine over SCP. You would use the **root** account and the target directory should be **/var/import/staging/**.

.. code-block:: bash

    root@xubuntu-vm:/var/www/server/1/upload# scp F5OS-A-1.0.0-11433.R5R10.iso root@10.255.0.132:/var/import/staging
    root@10.255.0.132's password: 
    F5OS-A-1.0.0-11433.R5R10.iso                                                                                                                                                                                         100% 5291MB 110.2MB/s   00:48 

Uploading F5OS-A Images via the API
------------------------------------

When uploading or importing F5OS-A images into the rSeries appliance, the files should be imported into the **images/staging** directory. Once the file import is initiated you can check its status using the **file transfer-status** API calls. Below are API calls to upload and monitor status.

List the current F5OS images in the **images/staging** directory via API calls:

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/restconf/data/f5-utils-file-transfer:file/list

.. code-block:: json

    {
    "f5-utils-file-transfer:path": "images/staging"
    }

Below is an example output:

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "entries": [
                {
                    "name": "\nF5OS-A-1.0.0-11432.R5R10.iso\nF5OS-A-1.0.0-8722.R2R4.NSIT.iso"
                }
            ]
        }
    }

You can then query the **images/import** directory for various image types like ISO, Service, or OS:

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/restconf/data/f5-utils-file-transfer:file/list

For the body you can enter the specific path you want to query. i.e. /images/import/<path> where path can be **iso**, **service**, or **os**:

.. code-block:: json

    {
    "f5-utils-file-transfer:path": "images/import/iso"
    }

Below is an example output:

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "entries": [
                {
                    "name": "\nF5OS-A-1.0.0-11432.R5R10.iso\nF5OS-A-1.0.0-8722.R2R4.NSIT.iso"
                }
            ]
        }
    }

To import an F5OS-A image, use the following API example:

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/api/data/f5-utils-file-transfer:file/import

.. code-block:: json

    {
        "input": [
            {
                "remote-host": "10.255.0.142",
                "remote-file": "/upload/{{Controller_ISO_Image_Full}}",
                "local-file": "images/staging/",
                "insecure": "",
                "f5-utils-file-transfer:username": "corpuser",
                "f5-utils-file-transfer:password": "Passw0rd!!"
            }
        ]
    }


You can then check on the file transfer status with the following API call:

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/restconf/data/f5-utils-file-transfer:file/transfer-status

A response like the one below will provide the status of the transfer:

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "result": "\nS.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                \n1    |Import file|HTTPS   |images/import/iso/F5OS-A-1.1.0-0188.R5R10.CANDIDATE.iso     |artifactory.f5net.com|artifactory/velocity-os-generic-dev/F5OS-A/candidate-testing/1.1.0-0188.VF12.4_Candidate_2.6646b1d9/results/appliance/images/F5OS-A-1.1.0-0188.R5R10.CANDIDATE.iso|         Completed|Thu Jan  6 02:58:41 2022\n2    |Import file|HTTPS   |images/staging/F5OS-A-1.0.0-11432.R5R10.iso                 |10.255.0.142        |/upload/F5OS-A-1.0.0-11432.R5R10.iso                        |         Completed|Wed Jan  5 20:03:03 2022\n3    |Import file|SCP     |images/tenant/F5OS-A-1.0.0-11432.R5R10.iso                  |10.255.0.142        |/var/www/server/1/upload/F5OS-A-1.0.0-11432.R5R10.iso       |         Completed|Wed Jan  5 20:08:24 2022\n"
        }
    }

After transferring the file you can view the contents of the images/staging directory. The file will then go through an import process before it is ready for use.

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/restconf/data/f5-utils-file-transfer:file/list

.. code-block:: json

    {
    "f5-utils-file-transfer:path": "images/staging"
    }

You will see all the files in the images/staging directory:

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "entries": [
                {
                    "name": "\nF5OS-A-1.0.0-11432.R5R10.iso\nF5OS-A-1.0.0-8722.R2R4.NSIT.iso"
                }
            ]
        }
    }

You can then monitor the images/import/iso directory to see when the file is ready to use for upgrade:

.. code-block:: bash

    POST https://{{Appliance1_IP}}:8888/restconf/data/f5-utils-file-transfer:file/list

.. code-block:: json

    {
    "f5-utils-file-transfer:path": "images/import/iso"
    }

You’ll see output similar to the example below. Once the file shows up here you are ready to upgrade.

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "entries": [
                {
                    "name": "\nF5OS-A-1.0.0-11432.R5R10.iso\nF5OS-A-1.0.0-8722.R2R4.NSIT.iso"
                }
            ]
        }
    }


You can then query the image status via the API:

.. code-block:: bash

    GET https://{{Appliance1_IP}}:8888//api/data/openconfig-system:system/f5-system-image:image/state

The output will show the status for the OS, Service, ISO, and Install Status.

.. code-block:: json

    {
        "f5-system-image:state": {
            "os": {
                "os": [
                    {
                        "version-os": "1.1.0-0188",
                        "status": "ready",
                        "date": "2021-11-24",
                        "in-use": false
                    },
                    {
                        "version-os": "1.0.0-11432",
                        "status": "ready",
                        "date": "2021-12-03",
                        "in-use": true
                    }
                ]
            },
            "services": {
                "service": [
                    {
                        "version-service": "1.1.0-0188",
                        "status": "ready",
                        "date": "2021-11-24",
                        "in-use": false
                    },
                    {
                        "version-service": "1.0.0-11432",
                        "status": "ready",
                        "date": "2021-12-03",
                        "in-use": true
                    }
                ]
            },
            "iso": {
                "iso": [
                    {
                        "version-iso": "1.1.0-0188",
                        "status": "ready",
                        "date": "2021-11-24",
                        "in-use": false
                    },
                    {
                        "version-iso": "1.0.0-11432",
                        "status": "ready",
                        "date": "2021-12-03",
                        "in-use": false
                    }
                ]
            },
            "install": {
                "install-os-version": "1.0.0-11432",
                "install-service-version": "1.0.0-11432",
                "install-status": "success"
            }
        }
    }



Upgrading F5OS via GUI
----------------------

Once the new images are loaded you can perform the upgrade from the **System Settings > Controller Management** screen. Currently it is recommended you use the **Bundled** option to upgrade using the ISO. In the future there may be cases where **Unbundled** (separate OS or Service upgrades) are recommended. Once you click Save the upgrade process will begin. For F5OS versions 1.1.x there is no rolling upgrade support and both controllers will reboot immediately taking the entire chassis offline. For F5OSv1.2 rolling upgrade support has been added, but you must be on a v1.2.x release or later to take advantage of this new functionality.

.. image:: images/rseries_software_upgrades/image7.png
  :align: center
  :scale: 70%

Chassis Partition Upgrades
==========================

-----------------------------------------
Upgrading a Chassis Partition via the GUI
-----------------------------------------

Upgrade of chassis partitions is performed from the system controller GUI **Partition Management** screen. You must first click the checkmark next to the chassis partition you wish to upgrade and then click **Edit**. You’ll now be able perform either a **bundled** or **unbundled** software upgrade of the chassis partition. At this time a bundled upgrade using the ISO image is recommended. Once you click **Save**, the partition will be brought offline and back online after the upgrade. All tenants will be suspended during this time so an outage will occur for this chassis partition only. 

.. image:: images/velos_software_upgrades/image8.png
  :align: center
  :scale: 70%

-----------------------------------------
Upgrading a Chassis Partition via the CLI
-----------------------------------------

In the system controller CLI you can use the **show image** command to see the currently installed software versions for each chassis partition.

.. code-block:: bash

    syscon-1-active# show image
    VERSION OS                                   IN     
    CONTROLLER   CONTROLLER  STATUS  DATE        USE    
    ----------------------------------------------------
    1.1.2-6101   1           ready               false  
    1.2.0-10357  1           ready   2021-08-21  false  
    1.2.1-10692  1           ready   2021-08-30  false  
    1.2.1-10781  1           ready   2021-09-01  true   

    VERSION                                             
    SERVICE                                      IN     
    CONTROLLER   CONTROLLER  STATUS  DATE        USE    
    ----------------------------------------------------
    1.1.0-6101   1           ready   2021-05-09  false  
    1.1.2-6101   1           ready   2021-05-09  false  
    1.2.0-10357  1           ready   2021-08-21  false  
    1.2.1-10692  1           ready   2021-08-30  false  
    1.2.1-10781  1           ready   2021-09-01  true   

    VERSION ISO                                  IN     
    CONTROLLER   CONTROLLER  STATUS  DATE        USE    
    ----------------------------------------------------
    1.1.2-6101   1           ready   2021-05-09  false  
    1.2.0-10357  1           ready   2021-08-21  false  
    1.2.1-10692  1           ready   2021-08-30  false  
    1.2.1-10781  1           ready   2021-09-01  false  

    VERSION OS                                   IN     
    CONTROLLER   CONTROLLER  STATUS  DATE        USE    
    ----------------------------------------------------
    1.1.2-6101   2           ready   2021-05-09  false  
    1.2.0-10357  2           ready   2021-08-21  false  
    1.2.1-10692  2           ready   2021-08-30  false  
    1.2.1-10781  2           ready   2021-09-01  true   

    VERSION                                             
    SERVICE                                      IN     
    CONTROLLER   CONTROLLER  STATUS  DATE        USE    
    ----------------------------------------------------
    1.1.0-6101   2           ready   2021-05-09  false  
    1.1.2-6101   2           ready   2021-05-09  false  
    1.2.0-10357  2           ready   2021-08-21  false  
    1.2.1-10692  2           ready   2021-08-30  false  
    1.2.1-10781  2           ready   2021-09-01  true   

    VERSION ISO                                  IN     
    CONTROLLER   CONTROLLER  STATUS  DATE        USE    
    ----------------------------------------------------
    1.1.2-6101   2           ready   2021-05-09  false  
    1.2.0-10357  2           ready   2021-08-21  false  
    1.2.1-10692  2           ready   2021-08-30  false  
    1.2.1-10781  2           ready   2021-09-01  false  

    VERSION OS                                   IN               
    PARTITION    CONTROLLER  STATUS  DATE        USE    NAME  ID  
    --------------------------------------------------------------
    1.2.0-10357  1           ready   2021-08-21  false            
    1.2.1-10692  1           ready   2021-08-30  false            

    VERSION                                                          
    SERVICE                                      IN                  
    PARTITION    CONTROLLER  STATUS  DATE        USE    NAME     ID  
    -----------------------------------------------------------------
    1.2.0-10357  1           ready   2021-08-21  false               
    1.2.1-10692  1           ready   2021-08-30  true   default  1   

    VERSION ISO                                  IN                         
    PARTITION    CONTROLLER  STATUS  DATE        USE    NAME            ID  
    ------------------------------------------------------------------------
    1.2.0-10357  1           ready   2021-08-21  false                      
    1.2.1-10692  1           ready   2021-08-30  true   bigpartition    2   
                                                        default         1   
                                                        smallpartition  3   

    VERSION OS                                   IN               
    PARTITION    CONTROLLER  STATUS  DATE        USE    NAME  ID  
    --------------------------------------------------------------
    1.2.0-10357  2           ready   2021-08-21  false            
    1.2.1-10692  2           ready   2021-08-30  false            

    VERSION                                                          
    SERVICE                                      IN                  
    PARTITION    CONTROLLER  STATUS  DATE        USE    NAME     ID  
    -----------------------------------------------------------------
    1.2.0-10357  2           ready   2021-08-21  false               
    1.2.1-10692  2           ready   2021-08-30  true   default  1   

    VERSION ISO                                  IN                         
    PARTITION    CONTROLLER  STATUS  DATE        USE    NAME            ID  
    ------------------------------------------------------------------------
    1.2.0-10357  2           ready   2021-08-21  false                      
    1.2.1-10692  2           ready   2021-08-30  true   bigpartition    2   
                                                        default         1   
                                                        smallpartition  3   

    syscon-1-active# 

The command **show running-config image** will show the current configuration for software images. You can enter **config** mode and change the configuration using the **system image set-version** command and then commit to initiate an upgrade.

.. code-block:: bash

    syscon-1-active# show running-config image 
    image controller config os os 1.1.2-6101
    !
    image controller config os os 1.2.0-10357
    !
    image controller config os os 1.2.1-10692
    !
    image controller config os os 1.2.1-10781
    !
    image controller config services service 1.1.0-6101
    !
    image controller config services service 1.1.2-6101
    !
    image controller config services service 1.2.0-10357
    !
    image controller config services service 1.2.1-10692
    !
    image controller config services service 1.2.1-10781
    !
    image controller config iso iso 1.1.2-6101
    service 1.1.2-6101
    os      1.1.2-6101
    !
    image controller config iso iso 1.2.0-10357
    service 1.2.0-10357
    os      1.2.0-10357
    !
    image controller config iso iso 1.2.1-10692
    service 1.2.1-10692
    os      1.2.1-10692
    !
    image controller config iso iso 1.2.1-10781
    service 1.2.1-10781
    os      1.2.1-10781
    !
    image partition config os os 1.2.0-10357
    !
    image partition config os os 1.2.1-10692
    !
    image partition config services service 1.2.0-10357
    !
    image partition config services service 1.2.1-10692
    !
    image partition config iso iso 1.2.0-10357
    service 1.2.0-10357
    os      1.2.0-10357
    !
    image partition config iso iso 1.2.1-10692
    service 1.2.1-10692
    os      1.2.1-10692
    !
    syscon-1-active# 

.. code-block:: bash

    syscon-1-active(config)# system image set-version iso-version 1.2.1-10781 
    response Controller iso version has been set
    syscon-1-active(config)# 


An upgrade of the system controllers should automatically start after the above command is entered. You can follow the upgrade progress by issuing the command **show system image**:

.. code-block:: bash

    syscon-2-active# show system image 
                        SERVICE      ISO      INSTALL      
    NUMBER  OS VERSION   VERSION      VERSION  STATUS       
    --------------------------------------------------------
    1       1.2.1-10692  1.2.1-10692  -        in-progress  
    2       1.2.1-10692  1.2.1-10692  -        pending      

    syscon-2-active# 

-----------------------------------------
Upgrading a Chassis Partition via the API
-----------------------------------------

To upgrade a chassis partition via the API you must first run the check version API call with the version you want to update to:

.. code-block:: bash

 POST https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-system-partition:partitions/partition=bigpartition/check-version

.. code-block:: json

    {
        "input": {
            "iso-version": "{{Partition_ISO_Image}}"
        }
    }

If the compatability check then you will get a message like the one below, and it is safe to install the new image via the set-version API call:

.. code-block:: json

    {
        "f5-system-partition:output": {
            "result": "Partition upgrade compatibility check succeeded."
        }
    }

This is the Set Version API call that will initiate the upgrade:

.. code-block:: bash

    POST https://{{Chassis1_System_Controller_IP}}:8888/restconf/data/f5-system-partition:partitions/partition=bigpartition/set-version

.. code-block:: json

    {
        "input": {
            "iso-version": "{{Partition_ISO_Image}}"
        }
    }

If the upgrad is successful, you will get notification like the message below:

.. code-block:: json

    {
        "f5-system-partition:output": {
            "result": "Version update successful."
        }
    }



Tenant Images and Upgrades
==========================

-------------------------------------
Loading Tenant Images for New Tenants
-------------------------------------

Tenant software images are loaded directly into the F5OS platform layer for use in creating new tenants. The first release of rSeries only supports TMOS v15.1.5. No other TMOS versions are supported other than hotfixes or rollups based on those versions of software. Tenant upgrades take place inside the tenants themslves, and images don't need to be loaded into the F5OS layer.

Before deploying any tenant, you must ensure you have a proper tenant software release loaded into F5OS. Under **Tenant Management** there is a page for uploading tenant software images. There are TMOS images specifically for rSeries. Only supported VELOS TMOS releases should be loaded into this system. Do not attempt to load older or even newer images unless there are officially supported on rSeries. 

There is an option to **Add** new releases which will open a pop-up window that will ask for remote host, path and optional authentication parameters. You may only upload from a remote HTTPS server using the GUI in the current VELOS release. The **Tenant Images** page will also indicate of an image is in use by a tenant, and if it is replicated to other blades in the chassis partition.

.. image:: images/rseries_software_upgrades/image9.png
  :align: center
  :scale: 70%


.. image:: images/rseries_software_upgrades/image10.png
  :align: center
  :scale: 70%

If an HTTPS server is not available, you may upload a tenant image using scp directly to the chassis partition. Simply scp an image to the system controllers out-of-band management IP address using the admin account and a path of **IMAGES**. 

.. code-block:: bash

    scp BIGIP-bigip14.1.x-tmos-bugfix-14.1.3.1-0.0.586.ALL-VELOS.qcow2.zip.bundle admin@10.255.0.148:IMAGES

You may also import the tenant image file from the chassis partition CLI. Use the file import command to get the tenant image file from a remote HTTPS server. You must do this for each chassis partition:

.. code-block:: bash

    bigpartition# file import remote-host 10.255.0.142 remote-file /upload/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle local-file images/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle username corpuser insecure

Repeat for other chassis partitions:

.. code-block:: bash

    smallpartition# file import remote-host 10.255.0.142 remote-file /upload/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle local-file images/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle username corpuser insecure


The command **file transfer-status** will provide details of the transfer progress and any errors:

.. code-block:: bash

    bigpartition-1# file import remote-host 10.255.0.142 remote-file /upload/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle local-file images/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle username corpuser insecure
    Value for 'password' (<string>): ********
    result File transfer is initiated.(images/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle)



    bigpartition-1# file transfer-status 
    result 
    S.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                
    1    |Import file|HTTPS   |images/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle       |10.255.0.142        |/upload/BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle      |In Progress (15.0%)|Fri Sep 17 15:36:29 2021


You can view the current tenant images and their status in the chassis partition CLI:

.. code-block:: bash

    bigpartition-2# show images
                                                                            IN                 
    NAME                                                                       USE    STATUS      
    ----------------------------------------------------------------------------------------------
    BIGIP-14.1.3-0.0.7.ALL-VELOS.qcow2.zip.bundle                              false  replicated  
    BIGIP-14.1.4-0.0.619.ALL-VELOS.qcow2.zip.bundle                            true   replicated  
    BIGIP-bigip14.1.x-tmos-bugfix-14.1.3.1-0.0.586.ALL-VELOS.qcow2.zip.bundle  false  replicated

To copy a tenant image into a chassis partition, use the following API call to the chassis partition IP address:

.. code-block:: bash

    POST https://{{Chassis1_BigPartition_IP}}:8888/api/data/f5-utils-file-transfer:file/import

.. code-block:: json

    {
        "input": [
            {
                "remote-host": "10.255.0.142",
                "remote-file": "upload/{{Tenant_Image}}",
                "local-file": "images/{{Tenant_Image}}",
                "insecure": "",
                "f5-utils-file-transfer:username": "corpuser",
                "f5-utils-file-transfer:password": "Passw0rd!!"
            }
        ]
    }

To list the current tenant images available on the chassis partition use the following API Call:

.. code-block:: bash

    GET https://{{Chassis1_BigPartition_IP}}:8888/restconf/data/f5-tenant-images:images

Below is output generated from the previous command:

.. code-block:: json

    {
        "f5-tenant-images:images": {
            "image": [
                {
                    "name": "BIGIP-15.1.4-0.0.46.ALL-VELOS.qcow2.zip.bundle",
                    "in-use": true,
                    "status": "replicated"
                },
                {
                    "name": "BIGIP-15.1.4-0.0.47.ALL-VELOS.qcow2.zip.bundle",
                    "in-use": false,
                    "status": "replicated"
                }
            ]
        }
    }

---------------
Tenant Upgrades
---------------

Tenants are upgraded via the normal TMOS upgrade process. Find the proper ISO image and ensure it is of a supported rSeries release, and upload it into the TMOS tenant. Once uploaded you can upgrade and boot into the new version. Currently rSeries does not allow an upgrade of the tenant from inside the F5OS layer, you must perform the upgrade from inside the tenant.

**NOTE: Currently rSeries does not provide a shared image repository for all tenants to upgrade from. With vCMP guests, iSeries allowed for an image to be loaded once into the host layer, and all tenants had access to that repository to use to upgrade.**

