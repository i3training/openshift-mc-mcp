# OpenShift Machine Config to set NTP client on all node's (Permanent)

1. We had setting chrony.conf  ==>  /etc/chrony.conf
```
[root@pl-srv1 ~]# cat  /etc/chrony.conf
server ntp1.plai-jkt.id iburst
server 0.id.pool.ntp.org iburst
server 10.253.0.5 iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony

```

2. Encode the chrony.conf using base64
```

[root@pl-srv1 ~]# cat chrony.conf | base64
c2VydmVyIG50cDEucGxhaS1qa3QuaWQgaWJ1cnN0CnNlcnZlciAwLmlkLnBvb2wubnRwLm9yZyBp
YnVyc3QKc2VydmVyIDEwLjI1My4wLjUgaWJ1cnN0CgojIFJlY29yZCB0aGUgcmF0ZSBhdCB3aGlj
aCB0aGUgc3lzdGVtIGNsb2NrIGdhaW5zL2xvc3NlcyB0aW1lLgpkcmlmdGZpbGUgL3Zhci9saWIv
Y2hyb255L2RyaWZ0CgojIEFsbG93IHRoZSBzeXN0ZW0gY2xvY2sgdG8gYmUgc3RlcHBlZCBpbiB0
aGUgZmlyc3QgdGhyZWUgdXBkYXRlcwojIGlmIGl0cyBvZmZzZXQgaXMgbGFyZ2VyIHRoYW4gMSBz
ZWNvbmQuCm1ha2VzdGVwIDEuMCAzCgojIEVuYWJsZSBrZXJuZWwgc3luY2hyb25pemF0aW9uIG9m
IHRoZSByZWFsLXRpbWUgY2xvY2sgKFJUQykuCnJ0Y3N5bmMKCiMgU3BlY2lmeSBmaWxlIGNvbnRh
aW5pbmcga2V5cyBmb3IgTlRQIGF1dGhlbnRpY2F0aW9uLgprZXlmaWxlIC9ldGMvY2hyb255Lmtl
eXMKCiMgU3BlY2lmeSBkaXJlY3RvcnkgZm9yIGxvZyBmaWxlcy4KbG9nZGlyIC92YXIvbG9nL2No
cm9ueQo=

```

3. Look on ignition version, see the latest MachineConfig (MC) update to MachineConfigPool worker and after that see the latest ignition version of MC 

```
 [root@pl-srv1 ~]# oc get mcp | grep worker
worker   rendered-worker-b9dbkd850bbea5c02320b855112c3cd   True      False      False       22            22                  22                    0

 [root@pl-srv1 ~]# oc get mc | grep rendered-worker-b9dbkd850bbea5c02320b855112c3cd
rendered-worker-b9dbkd850bbea5c02320b855112c3cd   1becd9e7d82be693f86g14798ce524d05d921fde   3.2.0             45d 

```
The ignition current version on  3.2.0 



4. Create MachineConfig file chrony config for worker node, select machineconfiguration role on worker for worker node, for control-plane node using master

```
[root@pl-srv1 ~]# vi mc-worker-chrony.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 01-worker-chrony-configuration
spec:
  config:
    ignition:
      config: {}
      security:
        tls: {}
      timeouts: {}
      version: 3.2.0
    networkd: {}
    passwd: {}
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,c2VydmVyIG50cDEucGxhaS1qa3QuaWQgaWJ1cnN0CnNlcnZlciAwLmlkLnBvb2wubnRwLm9yZyBp
          YnVyc3QKc2VydmVyIDEwLjI1My4wLjUgaWJ1cnN0CgojIFJlY29yZCB0aGUgcmF0ZSBhdCB3aGlj
          aCB0aGUgc3lzdGVtIGNsb2NrIGdhaW5zL2xvc3NlcyB0aW1lLgpkcmlmdGZpbGUgL3Zhci9saWIv
          Y2hyb255L2RyaWZ0CgojIEFsbG93IHRoZSBzeXN0ZW0gY2xvY2sgdG8gYmUgc3RlcHBlZCBpbiB0
          aGUgZmlyc3QgdGhyZWUgdXBkYXRlcwojIGlmIGl0cyBvZmZzZXQgaXMgbGFyZ2VyIHRoYW4gMSBz
          ZWNvbmQuCm1ha2VzdGVwIDEuMCAzCgojIEVuYWJsZSBrZXJuZWwgc3luY2hyb25pemF0aW9uIG9m
          IHRoZSByZWFsLXRpbWUgY2xvY2sgKFJUQykuCnJ0Y3N5bmMKCiMgU3BlY2lmeSBmaWxlIGNvbnRh
          aW5pbmcga2V5cyBmb3IgTlRQIGF1dGhlbnRpY2F0aW9uLgprZXlmaWxlIC9ldGMvY2hyb255Lmtl
          eXMKCiMgU3BlY2lmeSBkaXJlY3RvcnkgZm9yIGxvZyBmaWxlcy4KbG9nZGlyIC92YXIvbG9nL2No
          cm9ueQo=
        mode: 420 
        overwrite: true
        path: /etc/chrony.conf
  osImageURL: ""

  ```


5. Create MachineConfig configuration to worker node, for master/ control-plane node u can do step 3-5 but change worker to master

```
[root@pl-srv1 ~]# oc create -f  mc-worker-chrony.yaml

```
