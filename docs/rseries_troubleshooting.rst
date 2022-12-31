=======================
rSeries Troubleshooting
=======================

Storage / Disk
==============

Some rSeries appliances have a single disk, while other models have dual disks that are RAID-1 mirrored. 

How the Disk is Partitioned
---------------------------

Determining Free Space

F5OS Configuration Backups
--------------------------

Backups of the F5OS configuration are stored in the path **/var/F5/system/configs/**. 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) log]# cd /var/F5/system/configs/
  [root@appliance-1(r10900.f5demo.net) configs]# ls
  backup1  F5OS-BACKUP2022-01-20  F5OS-BACKUP-APPLIANCE12022-04-19  jim-backup  jim-backup.db  jim-july  kfo-bkp  kfo-bkup  new-backup


F5OS Images
-----------

F5OS-A ISO images are uploaded into the system where they are intially written to disk in the following path **/var/import/staging/**. 

.. code-block:: bash


  [root@appliance-1(r10900.f5demo.net) /]# ls /var/import/staging/
  F5OS-A-1.0.0-8722.R2R4.NSIT.iso        F5OS-A-1.4.0-2631.R5R10.CANDIDATE.iso  F5OS-A-1.4.0-5242.R5R10.CANDIDATE.iso
  F5OS-A-1.4.0-2129.R5R10.CANDIDATE.iso  F5OS-A-1.4.0-3402.R5R10.CANDIDATE.iso  F5OS-A-1.4.0-6325.R5R10.DEV.iso
  [root@appliance-1(r10900.f5demo.net) /]#

Once the file upload completes the file is then imported and extracted into one of the following paths depending on which type of rSeries system it is being installed onto. For r5000 or r10000 appliances, the path of the import is **/var/images/R5R10**. For r2000 and r4000 appliances, the imported path is **/var/images/R2R4**. Below is an example showing various images on an r10000 system.


.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) /]# ls var/images/R5R10/
  1.0.0-10170  1.0.0-11080  1.0.0-8303  1.0.0-8566  1.0.0-8830  1.0.0-9194  1.0.0-9396  1.4.0-2129  1.4.0-2631  1.4.0-3402  1.4.0-5242  1.4.0-6325
  [root@appliance-1(r10900.f5demo.net) /]# 

Tenant Images
-------------

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) system]# ls IMAGES/
  BIGIP-15.1.4-0.0.26.ALL-VELOS.qcow2.zip.bundle  BIGIP-bigip15.1.x-europa-15.1.5-0.0.210.ALL-F5OS.qcow2.zip.bundle    BIGIP-bigip151x-miranda-15.1.4.1-0.0.171.ALL-VELOS.qcow2.zip.bundle
  BIGIP-15.1.5-0.0.3.ALL-F5OS.qcow2.zip.bundle    BIGIP-bigip15.1.x-europa-15.1.5-0.0.222.ALL-F5OS.qcow2.zip.bundle    BIGIP-bigip151x-miranda-15.1.4.1-0.0.173.ALL-VELOS.qcow2.zip.bundle
  BIGIP-15.1.5-0.0.8.ALL-F5OS.qcow2.zip.bundle    BIGIP-bigip15.1.x-europa-15.1.5-0.0.225.ALL-F5OS.qcow2.zip.bundle    BIGIP-bigip151x-miranda-15.1.4.1-0.0.176.ALL-VELOS.qcow2.zip.bundle
  BIGIP-15.1.6.1-0.0.6.ALL-F5OS.qcow2.zip.bundle  BIGIP-bigip15.1.x-europa-15.1.5.1-0.0.276.ALL-F5OS.qcow2.zip.bundle
  [root@appliance-1(r10900.f5demo.net) system]# pwd
  /var/F5/system
  [root@appliance-1(r10900.f5demo.net) system]#

Tenant Virtual Disks
--------------------

The virtual disks for the tenants are stored in the path **/var/F5/system/cbip-disks/**. You'll see a directory with the configured tenant name for each tenant. Inside that directory will be a file with a .raw extension. As an example the rSeries appliance below has two tenants configured which are named **tenant1** and **tenant2**. There is a directory for each tenant, and inside the tenant1 directory is a **tenant1.raw** file which is the tenants virtual disk.

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) mibs]# cd /var/F5/system/cbip-disks/
  [root@appliance-1(r10900.f5demo.net) cbip-disks]# ls
  lost+found  tenant1  tenant2
  [root@appliance-1(r10900.f5demo.net) cbip-disks]# ls tenant1
  tenant1.raw
  [root@appliance-1(r10900.f5demo.net) cbip-disks]# ls -al tenant1
  total 6793212
  drwxr-xr-x. 2 root root        4096 Oct  7 07:55 .
  drwxr-xr-x. 5 root root        4096 Oct  7 10:12 ..
  -rw-r--r--. 1  107  107 82678120448 Dec 22 12:39 tenant1.raw
  [root@appliance-1(r10900.f5demo.net) cbip-disks]# 

Note that the tenant virtual disks use sparse provisioning, so the size seen is not representative to the actual storage space consumed on the disk. As an example, the tenant1 virtual disk image appears to be consuming 77GB, but that is what it has reserved, not consumed. In this case, the **ls -lsh** output confirms only 6.5GB of 77GB is actually consumed.

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) cbip-disks]# pwd
  /var/F5/system/cbip-disks
  [root@appliance-1(r10900.f5demo.net) cbip-disks]# cd tenant1
  
  [root@appliance-1(r10900.f5demo.net) tenant1]# ls -alh
  total 6.5G
  drwxr-xr-x. 2 root root 4.0K Oct  7 07:55 .
  drwxr-xr-x. 6 root root 4.0K Dec 22 12:41 ..
  -rw-r--r--. 1  107  107  77G Dec 22 12:54 tenant1.raw
  
  [root@appliance-1(r10900.f5demo.net) tenant1]# ls -lsh
  total 6.5G
  6.5G -rw-r--r--. 1 107 107 77G Dec 22 12:50 tenant1.raw
  [root@appliance-1(r10900.f5demo.net) tenant1]# 

Logging
-------

Important F5OS logs are stored in the path **/var/F5/system/log**. 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) log]# pwd
  /var/F5/system/log

  [root@appliance-1(r10900.f5demo.net) log]# ls
  audit.log       confd.log.1     devel.log.2.gz          k3s_events.log       lcd.log.2.gz     logrotate.log.2.gz  platform.log.5.gz              snmp.log.1        trace                 velos.log.3.gz
  audit.log.1     confd.log.2.gz  devel.log.3.gz          k3s_events.log.1     lcd.log.3.gz     platform.log        platform.log.6.gz              snmp.log.2.gz     vconsole_auth.log     webui
  audit.log.2.gz  confd.log.3.gz  devel.log.4.gz          k3s_events.log.2.gz  lcd.log.4.gz     platform.log.1      platform.log.7.gz              snmp.log.3.gz     vconsole_startup.log
  audit.log.3.gz  confd.log.4.gz  devel.log.5.gz          lacp_out_132         lcd.log.5.gz     platform.log.2.gz   reprogram_chassis_network.log  snmp.log.4.gz     velos.log
  audit.log.4.gz  devel.log       dma-agent-launcher.log  lcd.log              logrotate.log    platform.log.3.gz   rsyslogd_init.log              startup.log       velos.log.1
  confd.log       devel.log.1     dma-agent.log           lcd.log.1            logrotate.log.1  platform.log.4.gz   snmp.log                       startup.log.prev  velos.log.2.gz
  [root@appliance-1(r10900.f5demo.net) log]# 

SNMP MIBs
---------

F5OS SNMP MIBs are stored in the path **/var/F5/system/mibs**.

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) configs]# cd /var/F5/system/mibs
  [root@appliance-1(r10900.f5demo.net) mibs]# ls
  mibs_f5os_appliance.tar.gz  mibs_netsnmp.tar.gz
  [root@appliance-1(r10900.f5demo.net) mibs]# 


Kubernetes Environment
=======================

KubeVirt
--------

Useful Commands
---------------

https://docs.f5net.com/pages/viewpage.action?pageId=738733466

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl get nodes
  NAME                        STATUS   ROLES                  AGE    VERSION
  appliance-1.chassis.local   Ready    control-plane,master   387d   v1.21.1+k3s-9fb22ec1-dirty
  [root@appliance-1(r10900.f5demo.net) ~]# 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl get pods
  NAME                            READY   STATUS    RESTARTS   AGE
  virt-launcher-dummy-1-5z55d     1/1     Running   0          5h44m
  virt-launcher-tenant1-1-vrxzq   1/1     Running   0          5h44m
  virt-launcher-tenant2-1-99b54   1/1     Running   0          5h44m
  [root@appliance-1(r10900.f5demo.net) ~]# 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl cluster-info
  Kubernetes control plane is running at https://100.75.3.71:6443
  CoreDNS is running at https://100.75.3.71:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
  Metrics-server is running at https://100.75.3.71:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

  To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
  [root@appliance-1(r10900.f5demo.net) ~]# 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl get virtualmachineinstances --all-namespaces
  NAMESPACE   NAME        AGE    PHASE     IP    NODENAME
  default     dummy-1     6h6m   Running         appliance-1.chassis.local
  default     tenant1-1   6h6m   Running         appliance-1.chassis.local
  default     tenant2-1   6h5m   Running         appliance-1.chassis.local
  [root@appliance-1(r10900.f5demo.net) ~]# 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# systemctl status k3s
  ● k3s.service - Lightweight Kubernetes
    Loaded: loaded (/etc/systemd/system/k3s.service; enabled; vendor preset: disabled)
    Active: active (running) since Fri 2022-12-30 14:09:55 EST; 6h ago
      Docs: https://k3s.io
  Main PID: 46598 (k3s-server)
      Tasks: 390
    Memory: 1.6G
    CGroup: /system.slice/k3s.service
            ├─ 2139 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 66e5710bca80132aeab...
            ├─ 2370 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 6c26f1e3955342240a2...
            ├─ 2392 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 376353a1aaa5370233b...
            ├─ 2705 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id d256b2bfa149ef1de42...
            ├─ 3412 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 055b4c0a77bf5d181d9...
            ├─ 3880 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 8db01957810e4ab53b3...
            ├─ 8757 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 03ebd03fb56d4506e42...
            ├─18115 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 3b4344b8946f59a4975...
            ├─19787 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 48e3298f94a5726398e...
            ├─23731 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id a7cfcdbf20f79f1d7eb...
            ├─23830 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 4f157690361b20c6aae...
            ├─27287 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id a897258cff01945e32a...
            ├─31996 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 5a91317c6de79decdfc...
            ├─32629 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 28cae46f7d995dd941a...
            ├─37837 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 16e94a287f2c8e83b2c...
            ├─42559 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 2358f15465b9fe99693...
            ├─42963 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 080df187588dc4058ee...
            ├─46598 /usr/local/bin/k3s server
            ├─46720 containerd 
            └─47111 /var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 7fa94b414bc81a425b1...

  Dec 30 14:13:19 appliance-1.chassis.local k3s[46598]: I1230 14:13:19.384818   46598 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"conta...
  Dec 30 14:13:19 appliance-1.chassis.local k3s[46598]: I1230 14:13:19.384833   46598 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"hugep...
  Dec 30 14:13:19 appliance-1.chassis.local k3s[46598]: I1230 14:13:19.384869   46598 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"infra...
  Dec 30 14:13:19 appliance-1.chassis.local bridge[46946]: OpenSSL is initialized in FIPS mode.
  Dec 30 14:13:19 appliance-1.chassis.local host-local[46960]: OpenSSL is initialized in FIPS mode.
  Dec 30 14:13:19 appliance-1.chassis.local portmap[46986]: OpenSSL is initialized in FIPS mode.
  Dec 30 14:13:19 appliance-1.chassis.local macvlan[46995]: OpenSSL is initialized in FIPS mode.
  Dec 30 14:13:20 appliance-1.chassis.local macvlan[47036]: OpenSSL is initialized in FIPS mode.
  Dec 30 14:13:20 appliance-1.chassis.local macvlan[47070]: OpenSSL is initialized in FIPS mode.
  Dec 30 14:13:53 appliance-1.chassis.local k3s[46598]: I1230 14:13:53.860476   46598 scope.go:111] "RemoveContainer" containerID="81295bb2fde450e9cbfe3a54f2b181d84d7c277e...a1146f189"
  Hint: Some lines were ellipsized, use -l to show in full.
  [root@appliance-1(r10900.f5demo.net) ~]# 

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl describe node
  Name:               appliance-1.chassis.local
  Roles:              control-plane,master
  Labels:             beta.kubernetes.io/arch=amd64
                      beta.kubernetes.io/instance-type=k3s
                      beta.kubernetes.io/os=linux
                      bladeready=true
                      kubernetes.io/arch=amd64
                      kubernetes.io/hostname=appliance-1.chassis.local
                      kubernetes.io/os=linux
                      kubevirt.io/schedulable=true
                      node-role.kubernetes.io/control-plane=true
                      node-role.kubernetes.io/master=true
                      node.kubernetes.io/instance-type=k3s
  Annotations:        alpha.kubernetes.io/provided-node-ip: 100.75.3.71
                      flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"ae:54:13:01:d6:ad"}
                      flannel.alpha.coreos.com/backend-type: vxlan
                      flannel.alpha.coreos.com/kube-subnet-manager: true
                      flannel.alpha.coreos.com/public-ip: 10.255.0.132
                      k3s.io/hostname: appliance-1.chassis.local
                      k3s.io/internal-ip: 100.75.3.71
                      k3s.io/node-args:
                        ["server","--bind-address","100.75.3.71","--node-ip","100.75.3.71","--flannel-backend","none","--service-cidr","100.75.0.0/16","--cluster-...
                      k3s.io/node-config-hash: S3KWNN6DDC2BGWFS3YFPAXO5XGXKB7KUWRAWLE2DBNV42UK7WTMQ====
                      k3s.io/node-env: {"K3S_DATA_DIR":"/var/lib/rancher/k3s/data/1d7cbc1481bebe220ff42957a88280f5eb5eda5b382aef7a8835e00df1fc4c3f"}
                      kubevirt.io/heartbeat: 2022-12-31T01:21:39Z
                      node.alpha.kubernetes.io/ttl: 0
                      volumes.kubernetes.io/controller-managed-attach-detach: true
  CreationTimestamp:  Tue, 07 Dec 2021 21:29:49 -0500
  Taints:             <none>
  Unschedulable:      false
  Lease:
    HolderIdentity:  appliance-1.chassis.local
    AcquireTime:     <unset>
    RenewTime:       Fri, 30 Dec 2022 20:21:44 -0500
  Conditions:
    Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
    ----                 ------  -----------------                 ------------------                ------                       -------
    NetworkUnavailable   False   Fri, 30 Dec 2022 14:10:14 -0500   Fri, 30 Dec 2022 14:10:14 -0500   FlannelIsUp                  Flannel is running on this node
    MemoryPressure       False   Fri, 30 Dec 2022 20:20:24 -0500   Mon, 19 Sep 2022 10:43:11 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
    DiskPressure         False   Fri, 30 Dec 2022 20:20:24 -0500   Mon, 19 Sep 2022 10:43:11 -0400   KubeletHasNoDiskPressure     kubelet has no disk pressure
    PIDPressure          False   Fri, 30 Dec 2022 20:20:24 -0500   Mon, 19 Sep 2022 10:43:11 -0400   KubeletHasSufficientPID      kubelet has sufficient PID available
    Ready                True    Fri, 30 Dec 2022 20:20:24 -0500   Fri, 30 Dec 2022 14:09:53 -0500   KubeletReady                 kubelet is posting ready status
  Addresses:
    InternalIP:  100.75.3.71
    Hostname:    appliance-1.chassis.local
  Capacity:
    cpu:                            48
    devices.kubevirt.io/kvm:        110
    devices.kubevirt.io/tun:        110
    devices.kubevirt.io/vhost-net:  110
    ephemeral-storage:              115046548Ki
    f5.com/qat:                     96
    hugepages-1Gi:                  0
    hugepages-2Mi:                  231906Mi
    memory:                         263687192Ki
    pods:                           110
  Allocatable:
    cpu:                            48
    devices.kubevirt.io/kvm:        110
    devices.kubevirt.io/tun:        110
    devices.kubevirt.io/vhost-net:  110
    ephemeral-storage:              111917281807
    f5.com/qat:                     96
    hugepages-1Gi:                  0
    hugepages-2Mi:                  231906Mi
    memory:                         26215448Ki
    pods:                           110
  System Info:
    Machine ID:                 d940c29299f0477bb7b112a0a07cb787
    System UUID:                10016451-002F-0023-1551-393130353438
    Boot ID:                    e7cc2911-1921-4597-85ea-1d5e65e08083
    Kernel Version:             3.10.0-1160.62.1.F5.1.el7_8.x86_64
    OS Image:                   CentOS Linux 7 (Core)
    Operating System:           linux
    Architecture:               amd64
    Container Runtime Version:  containerd://1.4.4-k3s2
    Kubelet Version:            v1.21.1+k3s-9fb22ec1-dirty
    Kube-Proxy Version:         v1.21.1+k3s-9fb22ec1-dirty
  PodCIDR:                      100.76.0.0/24
  PodCIDRs:                     100.76.0.0/24
  ProviderID:                   k3s://appliance-1.chassis.local
  Non-terminated Pods:          (18 in total)
    Namespace                   Name                                           CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
    ---------                   ----                                           ------------  ----------  ---------------  -------------  ---
    kube-system                 traefik-ingress-controller-54c67bc84c-29pq4    0 (0%)        0 (0%)      0 (0%)           0 (0%)         240d
    kube-system                 local-path-provisioner-9d987bf48-rhhxm         0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h11m
    kube-system                 pause-58948b59d6-gchsb                         0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h11m
    kube-system                 coredns-8dc4c7b6d-kzlrb                        0 (0%)        0 (0%)      70Mi (0%)        170Mi (0%)     6h11m
    kube-system                 metrics-server-79cf557fc9-5b6g9                0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h11m
    kube-system                 klipper-lb-xm6qj                               0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h11m
    kube-system                 kube-flannel-ds-877v4                          0 (0%)        0 (0%)      50Mi (0%)        50Mi (0%)      6h11m
    kube-system                 kube-multus-ds-amd64-xltct                     0 (0%)        0 (0%)      50Mi (0%)        50Mi (0%)      6h11m
    kubevirt                    virt-operator-6b8985ccd-224tc                  0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h10m
    kubevirt                    virt-operator-6b8985ccd-pxfrb                  0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h10m
    kubevirt                    virt-api-8955d745c-kt7wt                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h10m
    kubevirt                    virt-api-8955d745c-mkrn4                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h9m
    kubevirt                    virt-controller-64c77bf5dd-rmpst               0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h9m
    kubevirt                    virt-handler-9tkzt                             0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h9m
    kubevirt                    virt-controller-64c77bf5dd-7mwcz               0 (0%)        0 (0%)      0 (0%)           0 (0%)         6h9m
    default                     virt-launcher-dummy-1-5z55d                    4 (8%)        0 (0%)      485574529 (1%)   0 (0%)         6h8m
    default                     virt-launcher-tenant1-1-vrxzq                  6 (12%)       0 (0%)      540254593 (2%)   0 (0%)         6h8m
    default                     virt-launcher-tenant2-1-99b54                  6 (12%)       0 (0%)      540254593 (2%)   0 (0%)         6h8m
  Allocated resources:
    (Total limits may be over 100 percent, i.e., overcommitted.)
    Resource                       Requests           Limits
    --------                       --------           ------
    cpu                            16 (33%)           0 (0%)
    memory                         1744341635 (6%)    270Mi (1%)
    ephemeral-storage              0 (0%)             0 (0%)
    hugepages-1Gi                  0 (0%)             0 (0%)
    hugepages-2Mi                  61740154880 (25%)  61740154880 (25%)
    devices.kubevirt.io/kvm        3                  3
    devices.kubevirt.io/tun        3                  3
    devices.kubevirt.io/vhost-net  3                  3
    f5.com/qat                     24                 24
  Events:                          <none>
  [root@appliance-1(r10900.f5demo.net) ~]#

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl get events
  LAST SEEN   TYPE     REASON    OBJECT                             MESSAGE
  4m55s       Normal   Created   virtualmachineinstance/tenant1-1   VirtualMachineInstance defined.
  4m55s       Normal   Created   virtualmachineinstance/tenant2-1   VirtualMachineInstance defined.
  4m55s       Normal   Created   virtualmachineinstance/dummy-1     VirtualMachineInstance defined.
  [root@appliance-1(r10900.f5demo.net) ~]#

To view console logs of a pod issue the command **kubectl logs <podname> -n <namespace>. First list the current pods to get their names, and namespaces.

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl get pods -A
  NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
  kube-system   traefik-ingress-controller-54c67bc84c-29pq4   1/1     Running   15         240d
  kube-system   local-path-provisioner-9d987bf48-rhhxm        1/1     Running   0          6h12m
  kube-system   pause-58948b59d6-gchsb                        1/1     Running   0          6h12m
  kube-system   coredns-8dc4c7b6d-kzlrb                       1/1     Running   0          6h12m
  kube-system   metrics-server-79cf557fc9-5b6g9               1/1     Running   0          6h12m
  kube-system   klipper-lb-xm6qj                              2/2     Running   0          6h12m
  kube-system   kube-flannel-ds-877v4                         1/1     Running   0          6h12m
  kube-system   kube-multus-ds-amd64-xltct                    1/1     Running   0          6h12m
  kubevirt      virt-operator-6b8985ccd-224tc                 1/1     Running   0          6h11m
  kubevirt      virt-operator-6b8985ccd-pxfrb                 1/1     Running   0          6h11m
  kubevirt      virt-api-8955d745c-kt7wt                      1/1     Running   0          6h11m
  kubevirt      virt-api-8955d745c-mkrn4                      1/1     Running   0          6h10m
  kubevirt      virt-controller-64c77bf5dd-rmpst              1/1     Running   0          6h10m
  kubevirt      virt-handler-9tkzt                            1/1     Running   0          6h10m
  kubevirt      virt-controller-64c77bf5dd-7mwcz              1/1     Running   0          6h10m
  default       virt-launcher-dummy-1-5z55d                   1/1     Running   0          6h9m
  default       virt-launcher-tenant1-1-vrxzq                 1/1     Running   0          6h9m
  default       virt-launcher-tenant2-1-99b54                 1/1     Running   0          6h9m
  [root@appliance-1(r10900.f5demo.net) ~]#

An example gettings logs for the pod **virt-launcher-dummy-1-5z55d** in the **default** namespace.

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl logs virt-launcher-dummy-1-5z55d -n default
  {"component":"virt-launcher","level":"info","msg":"VELOS: CPU Topology: hyperthreads are enabled","pos":"cpumanager.go:210","timestamp":"2022-12-30T19:13:04.048064Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: try to set cpu affinity (4 cpus) for VM dummy-1","pos":"virt-launcher.go:642","timestamp":"2022-12-30T19:13:04.048111Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: Domain dummy-1 requesting 4 cpus from allocator","pos":"virt-launcher.go:552","timestamp":"2022-12-30T19:13:04.048130Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: Domain dummy-1 acquired cpus [35 11 44 20]","pos":"virt-launcher.go:561","timestamp":"2022-12-30T19:13:04.048678Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: adjust affinity for pid 1 from [281474976710655 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] to [17626546833408 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]","pos":"virt-launcher.go:356","timestamp":"2022-12-30T19:13:04.048703Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: affinity after adjust [17626546833408 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]","pos":"virt-launcher.go:368","timestamp":"2022-12-30T19:13:04.048735Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: Installing deferred cpu release handler for pid 1","pos":"virt-launcher.go:649","timestamp":"2022-12-30T19:13:04.048774Z"}
  {"component":"virt-launcher","level":"info","msg":"VELOS: pinVCPUs -\u003e [35 11 44 20], [17626546833408 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]","pos":"virt-launcher.go:398","timestamp":"2022-12-30T19:13:04.048864Z"}
  {"component":"virt-launcher","level":"info","msg":"Collected all requested hook sidecar sockets","pos":"manager.go:68","timestamp":"2022-12-30T19:13:04.146736Z"}
  {"component":"virt-launcher","level":"info","msg":"Sorted all collected sidecar sockets per hook point based on their priority and name: map[]","pos":"manager.go:71","timestamp":"2022-12-30T19:13:04.146785Z"}
  {"component":"virt-launcher","level":"info","msg":"Connecting to libvirt daemon: qemu:///system","pos":"libvirt.go:417","timestamp":"2022-12-30T19:13:04.148487Z"}
  {"component":"virt-launcher","level":"info","msg":"Connecting to libvirt daemon failed: virError(Code=38, Domain=7, Message='Failed to connect socket to '/var/run/libvirt/libvirt-sock': No such file or directory')","pos":"libvirt.go:425","timestamp":"2022-12-30T19:13:04.149368Z"}
  {"component":"virt-launcher","level":"info","msg":"Connected to libvirt daemon","pos":"libvirt.go:433","timestamp":"2022-12-30T19:13:04.650399Z"}
  {"component":"virt-launcher","level":"info","msg":"Registered libvirt event notify callback","pos":"client.go:432","timestamp":"2022-12-30T19:13:04.652048Z"}
  {"component":"virt-launcher","level":"info","msg":"Marked as ready","pos":"virt-launcher.go:85","timestamp":"2022-12-30T19:13:04.652203Z"}
  {"component":"virt-launcher","kind":"","level":"info","msg":"VLCTY: before cpu swizzle, podCPUset [0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47]","name":"dummy-1","namespace":"default","pos":"manager.go:1263","timestamp":"2022-12-30T19:13:18.466218Z","uid":"2eb19a43-0e88-4431-bbe3-fa4511e1e479"}
  {"component":"virt-launcher","kind":"","level":"info","msg":"VLCTY: after  cpu swizzle, podCPUset [0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47]","name":"dummy-1","namespace":"default","pos":"manager.go:1265","timestamp":"2022-12-30T19:13:18.466261Z","uid":"2eb19a43-0e88-4431-bbe3-fa4511e1e479"}


To view the details of a specific pod enter the command ** kubectl describe pod <podname> -n <namespace>.

.. code-block:: bash

  [root@appliance-1(r10900.f5demo.net) ~]# kubectl describe pod virt-launcher-dummy-1-5z55d -n default
  Name:         virt-launcher-dummy-1-5z55d
  Namespace:    default
  Priority:     0
  Node:         appliance-1.chassis.local/100.75.3.71
  Start Time:   Fri, 30 Dec 2022 14:12:59 -0500
  Labels:       configby=TPOB-VM
                cpumanager=true
                guest=dummy-1
                kubevirt.io=virt-launcher
                kubevirt.io/created-by=2eb19a43-0e88-4431-bbe3-fa4511e1e479
                name=dummy
                project=default
                zone=node1
  Annotations:  k8s.v1.cni.cncf.io/network-status:
                  [{
                      "name": "",
                      "interface": "eth0",
                      "ips": [
                          "100.76.0.30"
                      ],
                      "mac": "22:16:b6:3d:35:67",
                      "default": true,
                      "dns": {}
                  },{
                      "name": "default/mgmt-conf-dummy",
                      "interface": "net1",
                      "mac": "00:94:a1:69:59:2b",
                      "dns": {}
                  },{
                      "name": "default/macvlan-conf-dummy",
                      "interface": "net2",
                      "ips": [
                          "127.3.0.0"
                      ],
                      "mac": "ce:ae:a0:02:18:86",
                      "dns": {}
                  },{
                      "name": "default/hnet-conf-dummy",
                      "interface": "net3",
                      "ips": [
                          "100.69.7.1"
                      ],
                      "mac": "d6:a9:28:d0:b7:dc",
                      "dns": {}
                  }]
                k8s.v1.cni.cncf.io/networks:
                  [{"interface":"net1","mac":"00:94:a1:69:59:2b","name":"mgmt-conf-dummy","namespace":"default"},{"interface":"net2","name":"macvlan-conf-du...
                k8s.v1.cni.cncf.io/networks-status:
                  [{
                      "name": "",
                      "interface": "eth0",
                      "ips": [
                          "100.76.0.30"
                      ],
                      "mac": "22:16:b6:3d:35:67",
                      "default": true,
                      "dns": {}
                  },{
                      "name": "default/mgmt-conf-dummy",
                      "interface": "net1",
                      "mac": "00:94:a1:69:59:2b",
                      "dns": {}
                  },{
                      "name": "default/macvlan-conf-dummy",
                      "interface": "net2",
                      "ips": [
                          "127.3.0.0"
                      ],
                      "mac": "ce:ae:a0:02:18:86",
                      "dns": {}
                  },{
                      "name": "default/hnet-conf-dummy",
                      "interface": "net3",
                      "ips": [
                          "100.69.7.1"
                      ],
                      "mac": "d6:a9:28:d0:b7:dc",
                      "dns": {}
                  }]
                kubevirt.io/domain: dummy-1
  Status:       Running
  IP:           100.76.0.30
  IPs:
    IP:           100.76.0.30
  Controlled By:  VirtualMachineInstance/dummy-1
  Init Containers:
    kubehelper:
      Container ID:   containerd://77c71603dd7c02a1c7670cf2d6e9fdeffb3065438fbc3f212078b21a25346c6c
      Image:          localhost:2014/kubehelper:5.13.4
      Image ID:       localhost:2014/kubehelper@sha256:66f3673df77b6b53b4ee15eab6e548d87cca4979ce739956b3d36fabceb805ff
      Port:           <none>
      Host Port:      <none>
      State:          Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Fri, 30 Dec 2022 14:13:00 -0500
        Finished:     Fri, 30 Dec 2022 14:13:01 -0500
      Ready:          True
      Restart Count:  0
      Environment:
        TMM_DESCSOCK_SVC_ID:  10
        TENANT_ID:            7
        KVM_OPERATION:        4
        KVM_MEMORY:           15569256448
        TENANT_OP:            BIGIP
        HA_IP:                BIGIP
        HA_MASK:              BIGIP
        TENANT_NAME:          dummy
        TENANT_TYPE:          BIGIP
        TENANT_SIZE:          77
        VLOG_INFO:            sevbound
        SLOT_NUM:             1
        VMID:                 dummy-1
        VMCPUS:               4
        HTSPLIT:              on
        F5_HOST_HP_PATH:      /var/huge_pages/2048kB/dummy/1/dummy
      Mounts:
        /dev/hugepages from hugepages (rw)
  Containers:
    compute:
      Container ID:  containerd://5e79134ac1c78044225eb813b926f4609615ed3261f5dd6b537d72b42996e9f3
      Image:         localhost:2014/kubevirt-appliance/virt-launcher:2.6.1
      Image ID:      localhost:2014/kubevirt-appliance/virt-launcher@sha256:2a46e8988acc9c636af31ea5c0a8a1a9e81add6acc393ea2ec8f8243f4e5f18c
      Port:          <none>
      Host Port:     <none>
      Command:
        /usr/bin/virt-launcher
        --qemu-timeout
        5m
        --name
        dummy-1
        --uid
        2eb19a43-0e88-4431-bbe3-fa4511e1e479
        --namespace
        default
        --kubevirt-share-dir
        /var/run/kubevirt
        --ephemeral-disk-dir
        /var/run/kubevirt-ephemeral-disks
        --container-disk-dir
        /var/run/kubevirt/container-disks
        --grace-period-seconds
        195
        --hook-sidecars
        0
        --less-pvc-space-toleration
        10
        --ovmf-path
        /usr/share/OVMF
      State:          Running
        Started:      Fri, 30 Dec 2022 14:13:03 -0500
      Ready:          True
      Restart Count:  0
      Limits:
        devices.kubevirt.io/kvm:        1
        devices.kubevirt.io/tun:        1
        devices.kubevirt.io/vhost-net:  1
        f5.com/qat:                     6
        hugepages-2Mi:                  15569256448
      Requests:
        cpu:                            4
        devices.kubevirt.io/kvm:        1
        devices.kubevirt.io/tun:        1
        devices.kubevirt.io/vhost-net:  1
        f5.com/qat:                     6
        hugepages-2Mi:                  15569256448
        memory:                         485574529
      Liveness:                         exec [/alive.sh] delay=20s timeout=1s period=30s #success=1 #failure=4
      Readiness:                        exec [/ready.sh] delay=14s timeout=5s period=30s #success=1 #failure=3
      Environment:
        KUBEVIRT_RESOURCE_NAME_mgmt-conf-dummy:     
        KUBEVIRT_RESOURCE_NAME_macvlan-conf-dummy:  
        KUBEVIRT_RESOURCE_NAME_hnet-conf-dummy:     
        TMM_DESCSOCK_SVC_ID:                        10
        TENANT_ID:                                  7
        KVM_OPERATION:                              4
        KVM_MEMORY:                                 15569256448
        TENANT_OP:                                  BIGIP
        HA_IP:                                      BIGIP
        HA_MASK:                                    BIGIP
        TENANT_NAME:                                dummy
        TENANT_TYPE:                                BIGIP
        TENANT_SIZE:                                77
        VLOG_INFO:                                  sevbound
        SLOT_NUM:                                   1
        VMID:                                       dummy-1
        VMCPUS:                                     4
        KUBEVIRT_SHARE_DIR:                         /var/run/kubevirt
        HTSPLIT:                                    on
        F5_HOST_HP_PATH:                            /var/huge_pages/2048kB/dummy/1/dummy
      Mounts:
        /dev/hugepages from hugepages (rw)
        /dev/vfio/ from dev-vfio (rw)
        /sys/bus/pci/ from pci-bus (rw)
        /sys/devices/ from pci-devices (rw)
        /var/run/kubevirt from virt-share-dir (rw)
        /var/run/kubevirt-ephemeral-disks from ephemeral-disks (rw)
        /var/run/kubevirt-infra from infra-ready-mount (rw)
        /var/run/kubevirt-private/config-map/dummy-1-configmap from dummy-1-configmap (ro)
        /var/run/kubevirt-private/secret/dummy-1-secrets from dummy-1-secrets (ro)
        /var/run/kubevirt-private/vmi-disks/dummy-host-volume from dummy-host-volume (rw)
        /var/run/kubevirt/container-disks from container-disks (rw)
        /var/run/kubevirt/hotplug-disks from hotplug-disks (rw)
        /var/run/kubevirt/sockets from sockets (rw)
        /var/run/libvirt from libvirt-runtime (rw)
        /var/run/unix-sockets/dma.sock from unix-socket (rw)
  Conditions:
    Type              Status
    Initialized       True 
    Ready             True 
    ContainersReady   True 
    PodScheduled      True 
  Volumes:
    sockets:
      Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
      Medium:     
      SizeLimit:  <unset>
    pci-bus:
      Type:          HostPath (bare host directory volume)
      Path:          /sys/bus/pci/
      HostPathType:  
    pci-devices:
      Type:          HostPath (bare host directory volume)
      Path:          /sys/devices/
      HostPathType:  
    dev-vfio:
      Type:          HostPath (bare host directory volume)
      Path:          /dev/vfio/
      HostPathType:  
    dummy-host-volume:
      Type:          HostPath (bare host directory volume)
      Path:          /var/F5/system/cbip-disks/dummy
      HostPathType:  DirectoryOrCreate
    dummy-1-configmap:
      Type:      ConfigMap (a volume populated by a ConfigMap)
      Name:      dummy-1-configmap
      Optional:  false
    dummy-1-secrets:
      Type:        Secret (a volume populated by a Secret)
      SecretName:  dummy-1-secrets
      Optional:    false
    unix-socket:
      Type:          HostPath (bare host directory volume)
      Path:          /var/run/platform/tenant_doorbell.sock
      HostPathType:  Socket
    hugepages:
      Type:          HostPath (bare host directory volume)
      Path:          /var/huge_pages/2048kB/dummy/1
      HostPathType:  
    infra-ready-mount:
      Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
      Medium:     
      SizeLimit:  <unset>
    virt-share-dir:
      Type:          HostPath (bare host directory volume)
      Path:          /var/run/kubevirt
      HostPathType:  
    virt-bin-share-dir:
      Type:          HostPath (bare host directory volume)
      Path:          /var/lib/kubevirt/init/usr/bin
      HostPathType:  
    libvirt-runtime:
      Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
      Medium:     
      SizeLimit:  <unset>
    ephemeral-disks:
      Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
      Medium:     
      SizeLimit:  <unset>
    container-disks:
      Type:          HostPath (bare host directory volume)
      Path:          /var/run/kubevirt/container-disks/2eb19a43-0e88-4431-bbe3-fa4511e1e479
      HostPathType:  
    hotplug-disks:
      Type:        EmptyDir (a temporary directory that shares a pod's lifetime)
      Medium:      
      SizeLimit:   <unset>
  QoS Class:       Burstable
  Node-Selectors:  bladeready=true
                  kubevirt.io/schedulable=true
  Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                  node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:          <none>
  [root@appliance-1(r10900.f5demo.net) ~]#




Networking
==========

TCPDUMP 
-------


LLDP 
----



Tenants
=======


Logging
=======

Log Rotation
------------

Log rotation is set per the normal rsyslog mechanisms.  The only supported method is to edit the rotation parameters directly in the configuration files, but this is not recommended for customers.  Only F5 PD and Support (under direction from PD or ENE's when trying to help to debug problems) should modify the log rotation parameters.

Log rotation is currently hard coded and handled via **/var/F5/system/etc/logrotate.d/platform.conf**.

.. code-block:: bash

  [root@appliance-1(r10900-2.f5demo.net) logrotate.d]# pwd
  /var/F5/system/etc/logrotate.d
  [root@appliance-1(r10900-2.f5demo.net) logrotate.d]# more platform.conf 
  /var/log/audit.log
  /var/log/confd.log
  /var/log/devel.log
  /var/log/lcd.log
  /var/log/snmp.log {
      rotate 5
      size 100M
      copytruncate
  }
  /var/log/k3s_events.log {
      rotate 2
      size 100M
      copytruncate
  }
  /var/log/platform.log {
      rotate 10
      size 1G
      sharedscripts
      postrotate
          pkill -HUP rsyslogd
      endscript
  }
  /var/log/logrotate.log 
  /var/log/rsyslogd.log {
      rotate 2
      size 5M
      copytruncate
  }
