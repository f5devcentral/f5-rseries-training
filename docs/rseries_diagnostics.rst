===================
rSeries Diagnostics
===================

This section will go through some of the diagnostic capabilities within the new F5OS layer. Inside the tenant the same BIG-IP diagnostic utilities that customers are used to are sill available.

Qkviews
=======

rSeries appliances support the ability to generate qkviews to collect and bundle configuration and diagnostic data that can be sent to F5 support or uploaded to iHealth. It is important to understand the rSeries architecture when generating qkviews. Generating a qkview from the F5OS platform layer will capture OS data, container information and info related to the health of the underlying F5OS layer. To capture tenant level information, you’ll need to run a qkview inside the TMOS layer of the tenant. The following links provide more details:

https://support.f5.com/csp/article/K76100544

https://support.f5.com/csp/article/K04756153

In general, you can use the qkview utility on rSeries systems to automatically collect configuration and diagnostic information from the system. The qkview utility provided in F5OS-A software captures diagnostic information from the rSeries system and associated containers. 

Note: The qkview utility on the rSeries system does not capture diagnostic data from tenant BIG-IP systems. To generate diagnostic data for a tenant BIG-IP, log in to the tenant system and perform the relevant procedure in K12878: Generating diagnostic data using the qkview utility.

The qkview utility on the rSeries system generates machine-readable JavaScript Object Notation (JSON) diagnostic data and combines the data into a single compressed Tape ARchive (TAR) format file. The single TAR file is comprised of embedded TAR files containing the diagnostic data of individual containers running on the system, as well as diagnostic data from the rSeries system. You can upload this file, called a QKView file, to iHealth, or give it to F5 Support to help them troubleshoot any issues.

Note: F5 Support requires a QKView file in all cases in which remote access to the product is not available.

Qkview Creation and Upload via GUI
----------------------------------


A QKView for the F5OS layer can be generated from the **System Settings > System Reports** page. Once finished it can also be uploaded them to iHealth. 

.. image:: images/rseries_diagnostics/image1.png
  :align: center
  :scale: 70%

To generate a qkview click on the button in the upper right-hand corner. It will take some time for the qkview to be generated.  Once the qkview is generated, you can click the checkbox next to it, and then select **Upload to iHealth**. Your iHealth credentials will automatically fill in if you entered them previously and can be cleared if you want to use another account, you can optionally add an **F5 Support Case Number** and **Description** when uploading to iHealth.


.. image:: images/rseries_diagnostics/image2.png
  :align: center
  :scale: 70%

.. image:: images/rseries_diagnostics/image3.png
  :align: center
  :scale: 70%

Qkview Creation and Upload via CLI
----------------------------------

If you would like to store iHealth credentials within the configuration you may do so via the F5OS CLI. Enter config mode, and then use the **system diagnostics ihealth config** command to configure a **username** and **password**.

.. code-block:: bash

    appliance-1# config
    Entering configuration mode terminal
    appliance-1(config)# system diagnostics ihealth config username test@f5.com password 
    (<AES encrypted string>): ********
    appliance-1(config)# commit
    Commit complete.
    appliance-1(config)# 

To generate a qkview from the CLI run the command **system diagnostics qkview capture**.

.. code-block:: bash

    appliance-1(config)# system diagnostics qkview capture 
    result  Warning: Qkview may contain sensitive data such as secrets, passwords and core files. Handle with care. Please send this file to F5 support. 
    Qkview file appliance-1.qkview is being collected.
    return code 200
    
    resultint 0
    appliance-1(config)# 
 
You can view the status of the capture using the command **system diagnostics qkview status**.

.. code-block:: bash

    appliance-1# system diagnostics qkview status
    result  {"Busy":true,"Percent":97,"Status":"collating","Message":"Collating data","Filename":"appliance-1.qkview"}
    
    resultint 0
    appliance-1# 

You may also confirm the file has been created by using the **file list** command, or the command **system diagnostics qkview list** to see more details about the size and creation date of the file:

.. code-block:: bash

    appliance-1# file list path diags/shared/qkview/
    entries {
        name 
    appliance-1.qkview
    }
    appliance-1# 

    appliance-1# system diagnostics qkview list 
    result  {"Qkviews":[{"Filename":"appliance-1.qkview","Date":"2022-01-16T16:57:22.983013886Z","Size":208510806}]}
    
    resultint 0
    appliance-1# 

To upload the qkview file to iHealth using the CLI use the following command **system diagnostics ihealth upload qkview-file <file-name> description "Text for description" service-request-number <SR Number>**.

.. code-block:: bash

    appliance-1# system diagnostics ihealth upload qkview-file appliance-1.qkview description "This is a test" 
    message HTTP/1.1 202 Accepted
    Location: /support/ihealth/status/Z3HydOfa
    Date: Sun, 16 Jan 2022 17:02:36 GMT
    Content-Length: 0


    errorcode false
    appliance-1# 


Qkview Creation and Upload via API
----------------------------------

To generate a qkview from the API POST the following API call to the F5OS out-of-band management IP.

.. code-block:: bash

    POST https://{{Appliance4_IP}}:8888/restconf/data/openconfig-system:system/f5-system-diagnostics-qkview:diagnostics/f5-system-diagnostics-qkview:qkview/f5-system-diagnostics-qkview:capture

In the body of the API call supply the filename for the qkview:

.. code-block:: json

    {
        "f5-system-diagnostics-qkview:filename": "my-qkview4.tgz"
    }

Below is the following output showing successfukl intiation of the qkview:

.. code-block:: json


    {
        "f5-system-diagnostics-qkview:output": {
            "result": " Warning: Qkview may contain sensitive data such as secrets, passwords and core files. Handle with care. Please send this file to F5 support. \nQkview file my-qkview4.tgz is being collected.\nreturn code 200\n ",
            "resultint": 0
        }
    }

To view the qkview status via the API POST the following API call:

.. code-block:: bash

    POST https://{{Appliance4_IP}}:8888/restconf/data/openconfig-system:system/f5-system-diagnostics-qkview:diagnostics/f5-system-diagnostics-qkview:qkview/f5-system-diagnostics-qkview:status

The output will display the percentage complete, error, or complete status:

.. code-block:: json

    {
        "f5-system-diagnostics-qkview:output": {
            "result": " {\"Busy\":false,\"Percent\":100,\"Status\":\"complete\",\"Message\":\"Completed collection.\",\"Filename\":\"my-qkview4.tgz\"}\n ",
            "resultint": 0
        }
    }

To upload the qkview file to iHealth using the API use the following POST AI call:

.. code-block:: bash

    POST https://{{Appliance4_IP}}:8888/restconf/data/openconfig-system:system/f5-system-diagnostics-qkview:diagnostics/f5-system-diagnostics-ihealth:ihealth/f5-system-diagnostics-ihealth:upload

Below is the body of the POST API call:

.. code-block:: json

    {
    "f5-system-diagnostics-ihealth:qkview-file": "my-qkview4.tgz",
    "f5-system-diagnostics-ihealth:description": "This is a test qkview",
    "f5-system-diagnostics-ihealth:service-request-number": ""
    }

In the output of tha API call the upload initiation is confirmed.

.. code-block:: json

    {
        "f5-system-diagnostics-ihealth:output": {
            "message": "HTTP/1.1 202 Accepted\r\nLocation: /support/ihealth/status/sthO7ieL\r\nDate: Tue, 18 Jan 2022 01:31:36 GMT\r\nContent-Length: 0\r\n\r\n",
            "errorcode": false
        }
    }


Logging
=======

Many functions inside the F5OS layer will log their events to the **platform.log** file that resides in the **/log/system/** path. You'll also notice there are other files for other types of logs.

Viewing Logs from the CLI
--------------------------

In the F5OS CLI the paths are simplified so that you don’t have know the underlying directory structure. You can use the **file list path** command to see the files inside the **log/system/** directory, use the tab complete to see the options:

.. code-block:: bash

    appliance-1# file list path log/
    Possible completions:
    confd/  host/  system/
    appliance-1# file list path log/system/
    Possible completions:
    audit.log                      confd.log          devel.log     devel.log.1    lcd.log           lcd.log.1           lcd.log.2.gz       
    lcd.log.3.gz                   lcd.log.4.gz       lcd.log.5.gz  logrotate.log  logrotate.log.1   logrotate.log.2.gz  platform.log       
    reprogram_chassis_network.log  rsyslogd_init.log  snmp.log      startup.log    startup.log.prev  trace/              vconsole_auth.log  
    vconsole_startup.log           velos.log          webui/        
    appliance-1# file list path log/system/

To view the contents of the **platform.log** file use the command **file show path /log/system/platform.log**. This will show the entire log file from the beginning, and may not be the best way to troubleshoot a recent event:

.. code-block:: bash

    appliance-1# file show log/system/platform.log 
    2021-10-18T20:53:28.620260+00:00 appliance-1 /usr/sbin/fips-service[9]: priority="Notice" version=1.0 msgid=0x5f01000000000001 msg="fips-service starting".
    2021-10-18T20:53:28.620289+00:00 appliance-1 utils-agent[9]: priority="Info" version=1.0 msgid=0x6602000000000005 msg="DB is not ready".
    2021-10-18T20:53:28.620392+00:00 appliance-1 /usr/sbin/fips-service[9]: priority="Info" version=1.0 msgid=0x6602000000000005 msg="DB is not ready".
    2021-10-18T20:53:28.620401+00:00 appliance-1 utils-agent[9]: priority="Info" version=1.0 msgid=0x6602000000000005 msg="DB is not ready".
    2021-10-18T20:53:28.620590+00:00 appliance-1 /usr/sbin/fips-service[9]: priority="Info" version=1.0 msgid=0x6602000000000005 msg="DB is not ready".
    2021-10-18T20:53:28.620591+00:00 appliance-1 ihealthd[8]: priority="Info" version=1.0 msgid=0x6602000000000005 msg="DB is not ready".
    2021-10-18T20:53:28.620593+00:00 appliance-1 utils-agent[9]: priority="Info" version=1.0 msgid=0x6602000000000006 msg="DB state monitor started".
    2021-10-18T20:53:28.620900+00:00 appliance-1 /usr/sbin/fips-service[9]: priority="Info" version=1.0 msgid=0x6602000000000006 msg="DB state monitor started".

There are options to manipulate the output of the file by adding **| ?**  to see the options.

.. code-block:: bash

    appliance-1# file show log/system/platform.log | ?
    Possible completions:
    append    Append output text to a file
    begin     Begin with the line that matches
    count     Count the number of lines in the output
    exclude   Exclude lines that match
    include   Include lines that match
    linnum    Enumerate lines in the output
    more      Paginate output
    nomore    Suppress pagination
    save      Save output text to a file
    until     End with the line that matches
    appliance-1# file show log/system/platform.log | 

There are also other file options to tail the log file using **file tail -f** for live tail of the file or **file tail -n <number of lines>**.

.. code-block:: bash

    appliance-1# file tail -f log/system/platform.log 
    2022-01-18T01:44:40.236691+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:44:40.255537+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    2022-01-18T01:45:40.213327+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:45:40.213596+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:45:40.226138+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:45:40.238555+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    2022-01-18T01:46:40.212159+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:46:40.212402+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:46:40.229909+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:46:40.247870+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    appliance-1# 



    appliance-1# file tail -n 20 log/system/platform.log
    2022-01-18T01:42:40.217019+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:42:40.217275+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:42:40.235046+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:42:40.254086+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    2022-01-18T01:43:40.332658+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:43:40.332900+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:43:40.352918+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:43:40.370488+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    2022-01-18T01:44:40.218159+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:44:40.218479+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:44:40.236691+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:44:40.255537+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    2022-01-18T01:45:40.213327+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:45:40.213596+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:45:40.226138+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:45:40.238555+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    2022-01-18T01:46:40.212159+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_init(confd_trans_ctx*)".
    2022-01-18T01:46:40.212402+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:46:40.229909+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::get_elem(confd_trans_ctx*, confd_hkeypath_t*)".
    2022-01-18T01:46:40.247870+00:00 appliance-1 sys-host-config[10328]: priority="Err" version=1.0 msgid=0x7001000000000031 msg="" func_name="static int SystemDateTimeOperHdlr::s_finish(confd_trans_ctx*)".
    appliance-1# 

Within the bash shell the path for the logging is different it is **/var/F5/system/log**. 

.. code-block:: bash

    [root@appliance-1 /]# ls -al /var/F5/system/log/
    total 1016748
    drwxr-xr-x.  4 root root      4096 Jan 17 19:38 .
    drwxr-xr-x. 21 root root      4096 Jan 17 20:30 ..
    -rw-r--r--.  1 root root  14123371 Jan 17 20:48 audit.log
    -rw-r--r--.  1 root root    588341 Jan 17 05:18 confd.log
    -rw-r--r--.  1 root root  41019035 Jan 17 20:48 devel.log
    -rw-r--r--.  1 root root 104858562 Dec 22 15:29 devel.log.1
    -rw-r--r--.  1 root root  64837421 Jan 17 20:49 lcd.log
    -rw-r--r--.  1 root root 104860300 Jan  5 18:04 lcd.log.1
    -rw-r--r--.  1 root root   6501388 Dec 17 08:23 lcd.log.2.gz
    -rw-r--r--.  1 root root   6532013 Nov 27 22:08 lcd.log.3.gz
    -rw-r--r--.  1 root root   6396563 Nov  8 12:27 lcd.log.4.gz
    -rw-r--r--.  1 root root   5101197 Oct 20 03:46 lcd.log.5.gz
    -rw-r--r--.  1 root root    110308 Jan 17 20:49 logrotate.log
    -rw-r--r--.  1 root root   5245071 Jan 17 19:38 logrotate.log.1
    -rw-r--r--.  1 root root     28600 Jan 15 11:15 logrotate.log.2.gz
    -rw-r--r--.  1 root root 471625299 Jan 17 20:48 platform.log
    -rw-r--r--.  1 root root         0 Sep 30 18:10 reprogram_chassis_network.log
    -rw-r--r--.  1 root root     14659 Jan 17 05:17 rsyslogd_init.log
    -rw-r--r--.  1 root root         0 Sep 24 16:41 snmp.log
    -rw-r--r--.  1 root root       118 Jan 17 05:17 startup.log
    -rw-r--r--.  1 root root       193 Jan 17 05:14 startup.log.prev
    drwxr-xr-x.  2 root root      4096 Sep 24 16:41 trace
    -rw-r--r--.  1 root root      3381 Jan 17 05:17 vconsole_auth.log
    -rw-r--r--.  1 root root     18817 Jan 17 05:17 vconsole_startup.log
    -rw-r--r--.  1 root root 209193620 Oct 18 16:46 velos.log
    drwxr-xr-x.  2 root root      4096 Jan 17 05:17 webui
    [root@appliance-1 /]# 

If you would like to change any of the logging levels via the CLI you must be in config mode. Use the **system logging sw-components sw-component <component name> config <logging severity>** command. You must **commit** for this change to take affect. Be sure to set logging levels back to normal after troubleshooting has completed.


.. code-block:: bash

    appliance-1(config)# system logging sw-components sw-component ?
    Possible completions:
    alert-service     api-svc-gateway         appliance-orchestration-agent  appliance-orchestration-manager  authd         confd-key-migrationd  
    dagd-service      datapath-cp-proxy       diag-agent                     disk-usage-statd                 dma-agent     fips-service          
    fpgamgr           ihealth-upload-service  ihealthd                       image-agent                      kubehelper    l2-agent              
    lacpd             license-service         line-dma-agent                 lldpd                            lopd          network-manager       
    nic-manager       optics-mgr              platform-diag                  platform-fwu                     platform-hal  platform-mgr          
    platform-monitor  platform-stats-bridge   qkviewd                        rsyslog-configd                  snmp-trapd    stpd                  
    sw-rbcast         sys-host-config         system-control                 tcpdumpd-manager                 tmstat-agent  tmstat-merged         
    upgrade-service   user-manager            vconsole                       
    appliance-1(config)# system logging sw-components sw-component lacpd ?
    Possible completions:
    config   Configuration data for platform sw-component logging
    <cr>     
    appliance-1(config)# system logging sw-components sw-component lacpd config ?
    Possible completions:
    description   Text that describes the platform sw-component (read-only)
    name          Name of the platform sw-component (read-only)
    severity      sw-component logging severity level.
    appliance-1(config)# system logging sw-components sw-component lacpd config severity ?
    Description: sw-component logging severity level. Default is INFORMATIONAL.
    Possible completions:
    [INFORMATIONAL]  ALERT  CRITICAL  DEBUG  EMERGENCY  ERROR  INFORMATIONAL  NOTICE  WARNING
    appliance-1(config)# system logging sw-components sw-component lacpd config severity DEBUG
    appliance-1(config-sw-component-lacpd)# commit
    Commit complete.
    appliance-1(config-sw-component-lacpd)# 
  
Viewing Logs from the GUI
--------------------------

In the intial release you cannot view the F5OS logs directly from the GUI, although you can download them from the GUI. To view the logs you can use the CLI or API, or download the files and then view, or use a remote syslog server. To download log files from the GUI go to the **System Settings -> File Utilities** page. Here there are various logs directories you can download files from. You have the option to **Export** files to a remote HTTPS server, or **Download the files directly to your client machine through the browser.

.. image:: images/rseries_diagnostics/image4.png
  :align: center
  :scale: 70%

If you want to download the main **platform.log** select the directoy **/log/system**.


.. image:: images/rseries_diagnostics/image5.png
  :align: center
  :scale: 70%


Currently F5OS GUI’s logging levels can be configured for local logging, and remote logging servers can be added. The **Software Component Log Levels** can be changed to have additional logging information sent to the local log.  The remote logging has its own **Severity** level which will ultimately control the maximum level of all messages going to a remote log server regardless of the individual Component Log Levels. This will allow for more information to be logged locally for debug purposes, while keeping remote logging to a minimum. If you would like to have more verbosity going to the remote logging host, you can raise its severity to see additional messages.

.. image:: images/rseries_diagnostics/image6.png
  :align: center
  :scale: 70%

Viewing Logs from the API
--------------------------

TBD


TCPDUMP
=======

You can use the **tcpdump** utility to capture traffic at the F5OS layer. The captured traffic can be saved as a file and analyzed to help troubleshoot network issues.

When you use the tcpdump utility to capture traffic on a VELOS system, traffic is captured based on the chassis partition in which the command was run. Only the traffic that occurs on that chassis partition is captured. This includes traffic traversing the front panel ports on the chassis blades in the chassis partition as well as backplane traffic for the chassis partition.

When you run tcpdump in a chassis partition, a secondary tcpdump operation runs on each member blade in the chassis partition. The packets captured by the secondary tcpdumps are collected together in the command output.

In addition to the normal tcpdump output, the following fields have been added that are specific to the VELOS system:

•	did - The Destination ID indicates the destination port for the frame.
•	sid - The Source ID indicates the source port for the frame.
•	svc - The Service ID indicates the destination tenant for the packet.
•	sep - The Service Endpoint indicates the service endpoint the packet is sent to.

You can see this in the following example output:

02:28:55.385343 IP 10.10.11.12 > 10.10.11.13: ICMP echo request, id 19463, seq 4, length 64 did:0F sid:04 sep:F svc:08 ld:1 rd:0
More detail on configuration and filtering of tcpdump is provide here:

https://support.f5.com/csp/article/K12313135


You can capture traffic for a specific interface on a blade using the interface keyword in the tcpdump command. The interface is specified as <blade>/<port>.<subport>. If the interface keyword is not supplied, or if 0/0.0 is specified for the interface, no interface filtering occurs and the command captures all interfaces in the partition.

Important: The interfaces on the VELOS system are capable of very high traffic rates. To prevent dropped packets during traffic capture, you should specify appropriate filters in order to capture only the intended traffic and reduce the total amount of captured traffic.

For example, the following command captures traffic on interface 1.0 on blade number 2:

.. code-block:: bash

    system diagnostics tcpdump interface "2/1.0"

The following command captures traffic-only packets in and out of the host of blade 2:

.. code-block:: bash
    system diagnostics tcpdump interface "2/0.0"

----------------
Specify a filter
----------------

Using the bpf keyword in the tcpdump command, you can specify a filter that limits the traffic capture based on the keywords you supply.

For example, the following command captures traffic only if the source or destination IP address is 10.10.10.100 and the source or destination port is 80:

system diagnostics tcpdump bpf "host 10.10.10.100 and port 80"

The following command captures traffic if the source IP address is 10.10.1.1 and the destination port is 443:

system diagnostics tcpdump bpf "src host 10.10.1.1 and dst port 443"

----------------------
Specify an output file
----------------------

To send the captured traffic to a file, specify the filename using the outfile keyword. The resulting file is placed in the /var/F5/<partiton>/ directory by default, or you can specify the directory in which to save the file.

For example, the following command sends the output of the tcpdump command to the /var/F5/partition/shared/example_capture.pcap file:

.. code-block:: bash

    system diagnostics tcpdump outfile /var/F5/partition/shared/example_capture.pcap

---------------
Combine options
---------------

The following example combines options to only capture traffic on interface 2.0 on blade 1 if the source IP address is 10.10.1.1 and the destination port is 80, and send the output to the /var/F5/partition/shared/example_capture.pcap file:

system diagnostics tcpdump interface "1/2.0" bpf "src host 10.10.1.1 and dst port 80" outfile /var/F5/partition/shared/example_capture.pcap    

Console Access to System Controllers and Blades via Built-In Terminal Server
============================================================================

You may have a need to access the console of a VELOS BX110 blade, one of the system controllers, or a tenant to diagnose a problem, or to watch it bootup. VELOS provides a built-in terminal server function that will proxy network connections to individual blades, system controller, & tenant console ports. Specific TCP ports on the system controller floating IP address have been reserved and mapped to console ports of as follows:

•	System controller ports 7001-7008 map to slots/blades 1-8
•	System controller ports 7100 & 7200 map to system controllers 1 & 2
•	Chassis partition ports 700x map to tenant ID’s (requires tenant name as username)

You can connect to any blade by SSH’ing to the floating IP address of the system controller and specifying the proper port for the blade you want to connect with. Port 7001 maps to blade1, 7002 to blade2 etc…. Once connected to the terminal server, you will need to login as root to the blade. The blade will have the default root password and will need to be changed on first reboot. The example below shows connecting to blade 2 ( port 7002) through a terminal server.

.. code-block:: bash

    FLD-ML-00054045:~ jmccarron$ ssh -l admin 10.255.0.147 -p 7002
    admin@10.255.0.147's password: 

    Terminal session established

    CentOS Linux 7 (Core)
    Kernel 3.10.0-862.14.4.el7.centos.plus.x86_64 on an x86_64

    blade-2 login: root
    Password: 
    You are required to change your password immediately (root enforced)
    Changing password for root.
    (current) UNIX password: 

Connecting to a system controller follows the same general process but uses ports 7100 for controller1 and 7200 for controller2. Below is an example connecting to system controller 1:

.. code-block:: bash

    FLD-ML-00054045:~ jmccarron$ ssh -l admin 10.255.0.147 -p 7100
    admin@10.255.0.147's password: 
    Terminal session established

    CentOS Linux 7 (Core)
    Kernel 3.10.0-862.14.4.el7.centos.plus.x86_64 on an x86_64

    controller-1 login: root
    Password: 
    Last failed login: Wed May 19 21:14:06 UTC 2021 on ttyS0
    There was 1 failed login attempt since the last successful login.
    Last login: Wed May 19 01:20:04 from controller-1.chassis.local
    [root@controller-1 ~]# 

Console Access to Tenant via Built-In Terminal Server
=====================================================


You may have a need to access the console of a tenant to diagnose a problem, or to watch it bootup. VELOS
provides a built-in terminal server function that will proxy network connections to a tenant console. VIPRION provided a **vconsole** capability which required a user to authenticate to VIPRION’s CLI first before they could run the vconsole command. 

When a VELOS tenant is created and deployed a listening ssh port will be configured on port 700x of the chassis partition (where x is the tenant instance ID). After a tenant is created, you will need to set the tenant password and tweak the expiry date to force a password change before a user can connect via the terminal server.

Once a tenant is created from the chassis partition CLI enter the command **show system aaa authentication**. Note that there is a **username** that corresponds to each tenant that has been created (tenant1, tenant2, tenant3 in this case, but will match the configured name of the tenant) and each of these have the role of **tenant-console**. Note the expiry date is set for **1**, which means expired.

.. code-block:: bash

    bigpartition-1# show system aaa authentication      
            LAST    TALLY  EXPIRY                  
    USERNAME  CHANGE  COUNT  DATE    ROLE            
    -------------------------------------------------
    admin     18667   0      -1      admin           
    root      18000   0      -1      root            
    tenant1   0       0      1       tenant-console  
    tenant2   0       0      1       tenant-console  
    tenant3   0       0      1       tenant-console  

    ROLENAME        GID   USERS  
    -----------------------------
    admin           9000  -      
    limited         9999  -      
    operator        9001  -      
    root            0     -      
    tenant-console  9100  -   


For tenant2 to have console access you must first set a password for that user using the command **system aaa authentication users user <tenant-name> config set-password password**. When prompted enter the desired password for this tenant’s console access. Next set the tenants **expiry-date** to **-1** (no expiration date) and then **commit** to enable the changes.

.. code-block:: bash

    bigpartition-1(config)# system aaa authentication users user tenant2 config set-password password      
    Value for 'password' (<string>): **************
    bigpartition-1(config)# system aaa authentication users user tenant2 config expiry-date                                                                                                                                    
    (<string>) (1): -1
    bigpartition-1(config-user-tenant2)# commit 
    Commit complete.

Now it will be possible to remotely ssh using a specific username and port pointed at the chassis partition IP address to connect directly to the console port of the tenant. The username will be the name of the tenant, and the port will be the instance tcp port 700x (where x is the instance ID of that tenant). Below is an example of the output from the **show tenants** command within the chassis partition. The tenant **tenant2** is running on two blades so it has two instance ID’s 1 & 2. You can connect to either one of these instances via the console using tenant2 as the username and either port 7001 or 7002. 

.. code-block:: bash

    tenants tenant tenant2
    state type          BIG-IP
    state mgmt-ip       10.255.0.205
    state prefix-length 24
    state gateway       10.255.0.1
    state vlans         [ 444 500 555 ]
    state cryptos       enabled
    state vcpu-cores-per-node 6
    state memory        22016
    state running-state deployed
    state mac-data base-mac 00:94:a1:8e:d0:1c
    state mac-data mac-pool-size 1
    state appliance-mode disabled
    state status        Running
    state primary-slot  1
    state image-version "BIG-IP 14.1.4 0.0.9"
    NDI      MAC                
    ----------------------------
    default  00:94:a1:8e:d0:1a  


        INSTANCE                                                                                                                                                  
    NODE  ID        PHASE    IMAGE NAME                                     CREATION TIME         READY TIME            STATUS                   MGMT MAC           
    ----------------------------------------------------------------------------------------------------------------------------------------------------------------
    1     1         Running  BIGIP-14.1.4-0.0.9.ALL-VELOS.qcow2.zip.bundle  2021-02-10T21:16:23Z  2021-02-10T21:16:38Z  Started tenant instance  a2:4a:b4:fd:85:81  
    2     2         Running  BIGIP-14.1.4-0.0.9.ALL-VELOS.qcow2.zip.bundle  2021-02-10T21:16:27Z  2021-02-10T21:16:24Z  Started tenant instance  16:1a:f6:87:be:07  


The built-in terminal server will switch the connection to the appropriate tenant terminal server port. Once connected, you will still need to login to the tenant. In the example below the username is tenant2 (matches the tenant name), and the port is 7001 meaning connect to instance ID 1 of that tenant. 

.. code-block:: bash

    FLD-ML-00054045:~ jmccarron$ ssh tenant2@10.255.0.148 -p 7001
    tenant1@10.255.0.148's password: 
    Successfully connected to tenant2-1 console. The escape sequence is ^]

    BIG-IP 14.1.4 Build 0.0.9
    Kernel 3.10.0-862.14.4.el7.x86_64 on an x86_64
    tenant1 login: root
    Password: 
    Last login: Thu Feb 11 22:43:43 on ttyS0
    [root@tenant2:/S1-green-P::Active:Standalone] config #



 
