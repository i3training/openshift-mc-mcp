# OpenShift Registries Image Policy Hardening - Blocking Specific registries

To block image registry from Specific host

Semi - Permanent 
oc edit image.config.openshift.io/cluster


apiVersion: config.openshift.io/v1
kind: Image
metadata:
  annotations:
    release.openshift.io/create-only: "true"
  generation: 1
  name: cluster
  selfLink: /apis/config.openshift.io/v1/images/cluster
spec:
  registrySources: 
    blockedRegistries: 
    - hub.docker.com
status:
  internalRegistryHostname: image-registry.openshift-image-registry.svc:5000

===========================================================================================

Permanent

1. Create registries.conf 
```
[root@pl-srv1 ~]# cat  registries.conf
unqualified-search-registries = ['registry.access.redhat.com', 'docker.io']

[[registry]]
  prefix = ""
  location = "hub.docker.com"
  blocked = true

```

2. Encode file registries.conf to base 64 and copy the value date

```
[root@pl-srv1 ~]# cat  registries.conf  | base64


```

3. Create MachineConfig file 
```
[root@pl-srv1 ~]# vi registries-block.yaml
[root@pl-srv1 ~]# cat registries-block.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 55-registries-block
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,DATA-REGISTRY-CONF
          verification: {}
        mode: 420
        overwrite: true
        path: /etc/containers/registries.conf

4. Apply the configuration to our cluster
```
[root@pl-srv1 ~]# oc create -f  registries-block.yaml

```