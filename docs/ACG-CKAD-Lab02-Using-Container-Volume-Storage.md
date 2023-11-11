# ACG-CKAD-Lab02 - Using Container Volume Storage in Kubernetes

## Background

* https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/
> A Container's file system lives only as long as the Container does. So when a Container terminates and restarts, filesystem changes are lost. For more consistent storage that is independent of the Container, you can use a Volume.

* https://kubernetes.io/docs/concepts/storage/persistent-volumes/
> While PersistentVolumeClaims allow a user to consume abstract storage resources, it is common that users need PersistentVolumes with varying properties, such as performance, for different problems.


## Objective


Introduction
Kubernetes offers a variety of tools to help you manage external storage for your containers. 
In this lab, you will have a chance to work with Kubernetes storage volumes, 
in the form of both ephemeral volumes and Persistent Volumes.


Begin by editing the application's Deployment:

```
kubectl edit deployment app-processing
```


Under the Pod template, note the Pod specification ...create the ephemeral storage volume with the volume mount.

```
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do cat /data/hivekey.txt > /tempdata/hivekey.txt; cat /tempdata/hivekey.txt;
          sleep 5; done
        image: radial/busyboxplus:curl
        imagePullPolicy: IfNotPresent
        name: busybox
        resources: {}
        securityContext:
          readOnlyRootFilesystem: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /tempdata
          name: temp
        - mountPath: /data
          name: hostdata
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - emptyDir: {}
        name: temp
      - name: hostdata
        persistentVolumeClaim:
          claimName: hostdata-pvc
```



Create the PersistentVolume.

```
vi hostdata-pv.yml
...
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostdata-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: host
  hostPath:
    path: /etc/voldata
    type: Directory
```


```
kubectl apply -f hostdata-pv.yml
```

## Create a PersistentVolumeClaim:

```
vi hostdata-pvc.yml
...
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostdata-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: host
```

Create the claim:

```
kubectl apply -f hostdata-pvc.yml
```

Verify that the PersistentVolumeClaim is bound to the PersistentVolume:
```
kubectl get pvc hostdata-pvc
```

## Edit the Deployment, and mount the PersistentVolumeClaim to the container:

```
kubectl edit deployment app-processing
```

 Under volumes, add the hostdata volume:

```
volumes:
- name: temp
  emptyDir: {}
- name: hostdata
  persistentVolumeClaim:
    claimName: hostdata-pvc
```

Scroll up, and under volumeMounts, mount the new volume to the container:
```
        volumeMounts:
        - name: temp
          mountPath: /tempdata
        - name: hostdata
          mountPath: /data
```          

Check the Pod status:

```
kubectl get pods
```

Copy the name of the active Deployment Pod. 

```
kubectl logs $POD_NAME
```

If everything is set up correctly, you should see the Hive Key data in the Pod's logs.



# logs

## persistentvolume/hostdata-pv created


```
cloud_user@k8s-control:~$ vi hostdata-pv.yml
cloud_user@k8s-control:~$ cat hostdata-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostdata-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: host
  hostPath:
    path: /etc/voldata
cloud_user@k8s-control:~$ kube
kubeadm  kubectl  kubelet
cloud_user@k8s-control:~$ kubectl apply -f hostdata-pv.yml
persistentvolume/hostdata-pv created
cloud_user@k8s-control:~$ kubectl  get pv
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
hostdata-pv   1Gi        RWO            Retain           Available           host                    16s
```

```
cloud_user@k8s-control:~$ vi hostdata-pvc.yml
cloud_user@k8s-control:~$ cat hostdata-pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostdata-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: host
cloud_user@k8s-control:~$ kubectl apply -f hostdata-pvc.yml
persistentvolumeclaim/hostdata-pvc created
cloud_user@k8s-control:~$ kubectl  get pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hostdata-pvc   Bound    hostdata-pv   1Gi        RWO            host           5s
```


```
cloud_user@k8s-control:~$ kubectl edit deployment app-processing
error: deployments.apps "app-processing" is invalid
deployment.apps/app-processing edited
cloud_user@k8s-control:~$ kubectl edit deployment app-processing
Edit cancelled, no changes made.
cloud_user@k8s-control:~$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
app-processing-5565b9f7fb-whktx   1/1     Running   0          88s
cloud_user@k8s-control:~$ kubectl logs app-processing-5565b9f7fb-whktx
Hive Key:IQP7c6dTxOQA80Cf4UAk
Hive Key:IQP7c6dTxOQA80Cf4UAk
Hive Key:IQP7c6dTxOQA80Cf4UAk
...
cloud_user@k8s-control:~$ kubectl logs app-processing-5565b9f7fb-whktx | wc -l
25
cloud_user@k8s-control:~$ sleep 10;kubectl logs app-processing-5565b9f7fb-whktx | wc -l
28
```


```
cloud_user@k8s-control:~$ kubectl describe deployment app-processing
Name:                   app-processing
Namespace:              default
CreationTimestamp:      Sat, 11 Nov 2023 14:40:53 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=processing
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=processing
  Containers:
   busybox:
    Image:      radial/busyboxplus:curl
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      while true; do cat /data/hivekey.txt > /tempdata/hivekey.txt; cat /tempdata/hivekey.txt; sleep 5; done
    Environment:  <none>
    Mounts:
      /data from hostdata (rw)
      /tempdata from temp (rw)
  Volumes:
   temp:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
   hostdata:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hostdata-pvc
    ReadOnly:   false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   app-processing-5565b9f7fb (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m48s  deployment-controller  Scaled up replica set app-processing-5565b9f7fb to 1
  Normal  ScalingReplicaSet  2m46s  deployment-controller  Scaled down replica set app-processing-75bb64f447 to 0
cloud_user@k8s-control:~$ kubectl describe  pods
Name:         app-processing-5565b9f7fb-whktx
Namespace:    default
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Sat, 11 Nov 2023 15:47:00 +0000
Labels:       app=processing
              pod-template-hash=5565b9f7fb
Annotations:  cni.projectcalico.org/containerID: 4642b120dc37b3f40a142fb81ced95a20126f0bf3fd8b89f10c4ae3245d2e3ad
              cni.projectcalico.org/podIP: 192.168.194.67/32
              cni.projectcalico.org/podIPs: 192.168.194.67/32
Status:       Running
IP:           192.168.194.67
IPs:
  IP:           192.168.194.67
Controlled By:  ReplicaSet/app-processing-5565b9f7fb
Containers:
  busybox:
    Container ID:  containerd://3a579613a2352f277a4699b7a53632828ef7deb99ac1ff6750cf5850aaf82554
    Image:         radial/busyboxplus:curl
    Image ID:      sha256:4776f1f7d1f625c8c5173a969fdc9ae6b62655a2746aba989784bb2b7edbfe9b
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do cat /data/hivekey.txt > /tempdata/hivekey.txt; cat /tempdata/hivekey.txt; sleep 5; done
    State:          Running
      Started:      Sat, 11 Nov 2023 15:47:01 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from hostdata (rw)
      /tempdata from temp (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9pp58 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  temp:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  hostdata:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hostdata-pvc
    ReadOnly:   false
  kube-api-access-9pp58:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m15s  default-scheduler  Successfully assigned default/app-processing-5565b9f7fb-whktx to k8s-worker1
  Normal  Pulled     3m14s  kubelet            Container image "radial/busyboxplus:curl" already present on machine
  Normal  Created    3m14s  kubelet            Created container busybox
  Normal  Started    3m13s  kubelet            Started container busybox
```


```
cloud_user@k8s-control:~$ kubectl describe  pvc
Name:          hostdata-pvc
Namespace:     default
StorageClass:  host
Status:        Bound
Volume:        hostdata-pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       app-processing-5565b9f7fb-whktx
Events:        <none>
cloud_user@k8s-control:~$ kubectl describe  pv
Name:            hostdata-pv
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    host
Status:          Bound
Claim:           default/hostdata-pvc
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/voldata
    HostPathType:
Events:            <none>
cloud_user@k8s-control:~$ kubectl edit deployment app-processing
Edit cancelled, no changes made.
```
