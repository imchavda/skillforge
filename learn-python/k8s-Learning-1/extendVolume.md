
***Storage Expansion to the AWS EKS***

---

### Highlevel Overview

Here's why and what you need to do:

01. Decoupling of Layers: Kubernetes PVs are an abstraction over the underlying storage. Simply resizing the storage at the infrastructure level (like EBS) doesn't automatically communicate that change to the Kubernetes control plane and the associated PV object. Kubernetes needs to be explicitly informed about the size change.   

02. PersistentVolumeClaim (PVC) and Resizing: The mechanism for resizing persistent storage in Kubernetes involves the PersistentVolumeClaim (PVC). When you need more space, you typically edit the PVC's spec.resources.requests.storage field to request the new size.

03. Provisioner Responsibility: Once you update the PVC, the responsibility of actually resizing the underlying volume falls on the volume provisioner (in this case, the AWS EBS CSI driver). The driver will then interact with the AWS API to resize the EBS volume to match the new size requested in the PVC.


Important Considerations:

01. Volume Expansion Feature: Ensure that the volume expansion feature is enabled in your Kubernetes cluster. This feature allows resizing of persistent volumes after they have been provisioned.

02. StorageClass: The allowVolumeExpansion field in your StorageClass must be set to true for PVC resizing to be possible.

03. Filesystem Resize: After the PV and the underlying EBS volume are resized, you might also need to resize the filesystem within your pod to utilize the newly available space. This often involves using tools like resize2fs (for ext filesystems) or xfs_growfs (for XFS filesystems) within your pod.

---

### Steps to Extend an EBS Volume and Reflect Changes in Kubernetes:

Step 1: Check the below details
    1. K8s version
    2. SC
    3. PV
    4. PVC
    5. Pods

Step 2: Check the dots between PV, PVC and PODs. Make sure the "allowVolumeExpansion" is true to your storage class

```
ajay@ip-172-31-43-178 ~(context:eventhorizon-dev) $ k get storageclass -n dsodb
NAME            PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
efs-sc          efs.csi.aws.com   Retain          WaitForFirstConsumer   true                   445d
gp3 (default)   ebs.csi.aws.com   Retain          WaitForFirstConsumer   true                   269d
```

`kubectl get storageclass gp3 -o yaml | grep allowVolumeExpansion`

If it's not true, you'll need to patch your StorageClass:

`kubectl patch storageclass gp3 -p '{"allowVolumeExpansion": true}'`

---

Step 3: Extend the EBS Volume from the AWS Console. This will take time.

`aws ec2 modify-volume --volume-id <volume-id> --size <new-size-in-GB>`


Step 4: Extend the Storage to the K8s resource:

```
ajay@ip-172-31-43-178 ~(context:eventhorizon-dev) $ kubectl get pv dsodb-psql16-1c-dev
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
dsodb-psql16-1c-dev   100Gi      RWO            Retain           Bound    dsodb/dsodb-psql16-1c-dev   gp3            <unset>                          138d
```

```
ajay@ip-172-31-43-178 ~/manifest(context:eventhorizon-dev) $ kubectl get pvc dsodb-psql16-1c-dev -n dsodb
NAME                  STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
dsodb-psql16-1c-dev   Bound    dsodb-psql16-1c-dev   100Gi      RWO            gp3            <unset>                 138d
```

**Update storage value details:**

`kubectl edit pvc dsodb-psql16-1c-dev -n dsodb`

```yaml
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Gi
```

Once it is updated, check pvc events 

```
> kubectl describe pvc dsodb-psql16-1c-dev -n dsodb
Name:          dsodb-psql16-1c-dev
Namespace:     dsodb
StorageClass:  gp3
Status:        Bound
Volume:        dsodb-psql16-1c-dev
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               volume.kubernetes.io/storage-resizer: ebs.csi.aws.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      100Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       dsodb-postgres-16-0
Conditions:
  Type                      Status  LastProbeTime                     LastTransitionTime                Reason  Message
  ----                      ------  -----------------                 ------------------                ------  -------
  Resizing                  True    Mon, 01 Jan 0001 00:00:00 +0000   Mon, 28 Apr 2025 09:23:23 +0000           
  FileSystemResizePending   True    Mon, 01 Jan 0001 00:00:00 +0000   Mon, 28 Apr 2025 09:23:27 +0000           Waiting for user to (re-)start a pod to finish file system resize of volume on node.
Events:
  Type    Reason                    Age   From                              Message
  ----    ------                    ----  ----                              -------
  Normal  ExternalExpanding         5s    volume_expand                     CSI migration enabled for kubernetes.io/aws-ebs; waiting for external resizer to expand the pvc
  Normal  Resizing                  5s    external-resizer ebs.csi.aws.com  External resizer is resizing volume dsodb-psql16-1c-dev
  Normal  FileSystemResizeRequired  1s    external-resizer ebs.csi.aws.com  Require file system resize of volume on node

```


Check pods events:

```
$ k describe pod dsodb-postgres-16-0 -n dsodb
Name:             dsodb-postgres-16-0
Namespace:        dsodb
Priority:         0
Service Account:  dsodb-postgres-16
Node:             ip-172-31-40-154.ec2.internal/172.31.40.154
Start Time:       Wed, 26 Mar 2025 11:17:29 +0000
Labels:           app.kubernetes.io/component=primary
                  app.kubernetes.io/instance=dsodb
                  app.kubernetes.io/managed-by=Helm
                  app.kubernetes.io/name=postgres-16
                  app.kubernetes.io/version=16.3.0
                  apps.kubernetes.io/pod-index=0
                  controller-revision-hash=dsodb-postgres-16-79cf7669d8
                  helm.sh/chart=postgres-16-15.5.17
                  statefulset.kubernetes.io/pod-name=dsodb-postgres-16-0
Annotations:      <none>
Status:           Running
IP:               172.31.35.214
IPs:
  IP:           172.31.35.214
Controlled By:  StatefulSet/dsodb-postgres-16
Containers:
  postgresql:
    Container ID:    containerd://a4059de18e6864334a6ffb7b7e8e3cb5fadc1264d7eca25257e51d868964ad17
    Image:           docker.io/bitnami/postgresql:16.3.0-debian-12-r19
    Image ID:        docker.io/bitnami/postgresql@sha256:b0248a5e2bf4fda5208183d4a6203287828666823a7a57431cfa4d31688bae97
    Port:            5432/TCP
    Host Port:       0/TCP
    SeccompProfile:  RuntimeDefault
    State:           Running
      Started:       Wed, 26 Mar 2025 11:17:43 +0000
    Ready:           True
    Restart Count:   0
    Limits:
      cpu:     2
      memory:  5Gi
    Requests:
      cpu:      1
      memory:   3Gi
    Liveness:   exec [/bin/sh -c exec pg_isready -U "postgres" -h 127.0.0.1 -p 5432] delay=30s timeout=5s period=10s #success=1 #failure=6
    Readiness:  exec [/bin/sh -c -e exec pg_isready -U "postgres" -h 127.0.0.1 -p 5432
[ -f /opt/bitnami/postgresql/tmp/.initialized ] || [ -f /bitnami/postgresql/.initialized ]
] delay=5s timeout=5s period=10s #success=1 #failure=6
    Environment:
      BITNAMI_DEBUG:                        false
      POSTGRESQL_PORT_NUMBER:               5432
      POSTGRESQL_VOLUME_DIR:                /bitnami/postgresql
      PGDATA:                               /bitnami/postgresql/data
      POSTGRES_PASSWORD:                    <set to the key 'postgresql-postgres-password' in secret 'dsodb-postgresql-16-specific'>  Optional: false
      POSTGRESQL_ENABLE_LDAP:               no
      POSTGRESQL_ENABLE_TLS:                no
      POSTGRESQL_LOG_HOSTNAME:              false
      POSTGRESQL_LOG_CONNECTIONS:           false
      POSTGRESQL_LOG_DISCONNECTIONS:        false
      POSTGRESQL_PGAUDIT_LOG_CATALOG:       off
      POSTGRESQL_CLIENT_MIN_MESSAGES:       error
      POSTGRESQL_SHARED_PRELOAD_LIBRARIES:  pgaudit
    Mounts:
      /bitnami/postgresql from data (rw)
      /dev/shm from dshm (rw)
      /opt/bitnami/postgresql/conf from empty-dir (rw,path="app-conf-dir")
      /opt/bitnami/postgresql/tmp from empty-dir (rw,path="app-tmp-dir")
      /tmp from empty-dir (rw,path="tmp-dir")
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  empty-dir:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  dshm:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  data:
    Type:        PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:   dsodb-psql16-1c-dev
    ReadOnly:    false
QoS Class:       Burstable
Node-Selectors:  node-group=aspm-db
                 topology.ebs.csi.aws.com/zone=us-east-1c
Tolerations:     aspm-db=allow:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason                      Age   From     Message
  ----    ------                      ----  ----     -------
  Normal  FileSystemResizeSuccessful  106s  kubelet  MountVolume.NodeExpandVolume succeeded for volume "dsodb-psql16-1c-dev" ip-172-31-40-154.ec2.internal
```


