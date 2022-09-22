# OpenShift Setting ISCID & Multipathd for Netapp Trident Storage / Nutanix CSI
This is absolute requirements if use storage nfs client such as NetApp Trident / Nutanix CSI [must enable on all worker nodes]

1. Create iscsid.conf 
```
[root@pl-srv1 ~]# vi  /root/iscsid/iscsid.conf 

```

2. Create multipath.conf  ==> allows you to configure multiple I/O paths between server nodes and storage arrays into a single device

```
[root@pl-srv1 ~]# vi  /root/iscsid/multipath.conf 

```

3. Encode iscsid.conf using urlencode https://www.utilities-online.info/urlencode and copy it the  data value


4. Encode multipath.conf using base64 and copy the data value

```
[root@pl-srv1 ~]# cat  /root/iscsid/multipath.conf  | base64

```

5. Look on ignition version, see the latest MachineConfig (MC) update to MachineConfigPool worker and after that see the latest ignition version of MC 

```
 [root@pl-srv1 ~]# oc get mcp | grep worker
worker   rendered-worker-b9dbkd850bbea5c02320b855112c3cd   True      False      False       22            22                  22                    0

 [root@pl-srv1 ~]# oc get mc | grep rendered-worker-b9dbkd850bbea5c02320b855112c3cd
rendered-worker-b9dbkd850bbea5c02320b855112c3cd   1becd9e7d82be693f86g14798ce524d05d921fde   3.2.0             45d 

```
The ignition current version on  3.2.0 


6. Create MachineConfig file for enable iscsid service - multipath service and setting the configuration files, Dont forget to set ignition version on 3.2.0 and set label on MC :  worker

```
 [root@pl-srv1 ~]# vi worker-iscsi-mpath.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 30-worker-iscsid-mpath
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - contents:
          source: data:COPY-OF-ISCSID-VALUE-DATA
          verification: {}
        filesystem: root
        mode: 384
        path: /etc/iscsi/iscsid.conf
      - contents:
          source: data:text/plain;charset=utf-8;base64,COPY-OF-MULTIPATHD-VALUE-DATA
          verification: {}
        filesystem: root
        mode: 384
        path: /etc/multipath.conf
    systemd:
      units:
      - enabled: true
        name: multipathd.service
      - enabled: true
        name: iscsid.service
      - enabled: true
        name: iscsi.service
```

7. Create MachineConfig configuration to apply config on our cluster

```
[root@pl-srv1 ~]# oc create -f  worker-iscsi-mpath.yaml

```