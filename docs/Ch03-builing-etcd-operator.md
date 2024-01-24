# Ch03 builing etcd-operator cluster

## First new cluster via kind for ch03

You need docker desktop (for kind).

First ensure everything else is stopped

```
ch03 % docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

setup a new cluster via kind

```
ch03 % kind create cluster --name ch03
Creating cluster "ch03" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-ch03"
You can now use your cluster with:

kubectl cluster-info --context kind-ch03

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
ch03 % kubectl cluster-info --context kind-ch03
Kubernetes control plane is running at https://127.0.0.1:59365
CoreDNS is running at https://127.0.0.1:59365/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


##  Start with the CRD for the new operator

```
ch03 % cat etcd-operator-crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: etcdclusters.etcd.database.coreos.com
spec:
  group: etcd.database.coreos.com
  names:
    kind: EtcdCluster
    listKind: EtcdClusterList
    plural: etcdclusters
    shortNames:
    - etcdclus
    - etcd
    singular: etcdcluster
  scope: Namespaced
  versions:
  - name: v1beta2
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              size:
                type: integer
              version:
                type: string
```
and
```
ch03 % kubectl create -f etcd-operator-crd.yaml
customresourcedefinition.apiextensions.k8s.io/etcdclusters.etcd.database.coreos.com created
ch03 % kubectl get crd
NAME                                    CREATED AT
etcdclusters.etcd.database.coreos.com   2024-01-24T17:03:50Z
```


## Create new service account

```
ch03 % cat etcd-operator-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: etcd-operator-sa
```
and
```
ch03 % kubectl create -f etcd-operator-sa.yaml
serviceaccount/etcd-operator-sa created
ch03 % kubectl get serviceaccounts
NAME               SECRETS   AGE
default            0         4m17s
etcd-operator-sa   0         30s
```
## Create new role

```
ch03 % cat etcd-operator-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: etcd-operator-role
rules:
- apiGroups:
  - etcd.database.coreos.com
  resources:
  - etcdclusters
  - etcdbackups
  - etcdrestores
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - events
  verbs:
  - '*'
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
```
and
```
ch03 % kubectl create -f etcd-operator-role.yaml
role.rbac.authorization.k8s.io/etcd-operator-role created
ch03 % kubectl get roles
NAME                 CREATED AT
etcd-operator-role   2024-01-24T17:15:22Z
```
## Create new service account

```
ch03 % cat etcd-operator-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: etcd-operator-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: etcd-operator-role
subjects:
- kind: ServiceAccount
  name: etcd-operator-sa
  namespace: default
```
and
```
ch03 % kubectl create -f etcd-operator-rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/etcd-operator-rolebinding created
```

## Create new service account

```
ch03 % cat etcd-operator-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: etcd-operator
spec:
  selector:
    matchLabels:
      app: etcd-operator
  replicas: 1
  template:
    metadata:
      labels:
        app: etcd-operator
    spec:
      containers:
      - name: etcd-operator
        image: quay.io/coreos/etcd-operator:v0.9.4
        command:
        - etcd-operator
        - --create-crd=false
        env:
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        imagePullPolicy: IfNotPresent
      serviceAccountName: etcd-operator-sa
```
and
```
ch03 % kubectl create -f  etcd-operator-deployment.yaml
deployment.apps/etcd-operator created
ch03 % kubectl get deployments
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
etcd-operator   1/1     1            1           27s
ch03 % kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
etcd-operator-5f98c6c6c5-7992p   1/1     Running   0          39s
```

## Create etcd-cluster

```
ch03 % cat etcd-cluster-cr.yaml
apiVersion: etcd.database.coreos.com/v1beta2
kind: EtcdCluster
metadata:
  name: example-etcd-cluster
spec:
  size: 3
  version: 3.1.10
ch03 % kubectl create -f  etcd-cluster-cr.yaml
etcdcluster.etcd.database.coreos.com/example-etcd-cluster created
ch03 % kubectl get pods
NAME                              READY   STATUS     RESTARTS   AGE
etcd-operator-5f98c6c6c5-7992p    1/1     Running    0          98s
example-etcd-cluster-bczmh22ph8   0/1     Init:0/1   0          20s
```
and
```
davidpitts@Davids-MacBook-Pro ch03 % for i in {1..10};do uptime;kubectl get pods;sleep 30;done
18:30  up 20 days, 43 mins, 23 users, load averages: 1.89 1.87 1.76
NAME                              READY   STATUS     RESTARTS   AGE
etcd-operator-5f98c6c6c5-7992p    1/1     Running    0          3m30s
example-etcd-cluster-bczmh22ph8   0/1     Init:0/1   0          2m12s
18:30  up 20 days, 44 mins, 23 users, load averages: 1.53 1.78 1.73
NAME                              READY   STATUS     RESTARTS   AGE
etcd-operator-5f98c6c6c5-7992p    1/1     Running    0          4m
example-etcd-cluster-bczmh22ph8   0/1     Init:0/1   0          2m42s
...
18:34  up 20 days, 47 mins, 23 users, load averages: 1.95 1.58 1.63
NAME                              READY   STATUS     RESTARTS   AGE
etcd-operator-5f98c6c6c5-7992p    1/1     Running    0          7m32s
example-etcd-cluster-bczmh22ph8   0/1     Init:0/1   0          6m14s
18:34  up 20 days, 48 mins, 23 users, load averages: 2.32 1.71 1.67
NAME                              READY   STATUS     RESTARTS   AGE
etcd-operator-5f98c6c6c5-7992p    1/1     Running    0          8m2s
example-etcd-cluster-bczmh22ph8   0/1     Init:0/1   0          6m44s
```


## Debugging PodInitializing problem

Nothing obviously wrong here:
```
ch03 % kubectl describe pods
Name:             etcd-operator-5f98c6c6c5-7992p
Namespace:        default
Priority:         0
Service Account:  etcd-operator-sa
Node:             ch03-control-plane/172.19.0.2
Start Time:       Wed, 24 Jan 2024 18:26:53 +0100
Labels:           app=etcd-operator
                  pod-template-hash=5f98c6c6c5
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:           10.244.0.5
Controlled By:  ReplicaSet/etcd-operator-5f98c6c6c5
Containers:
  etcd-operator:
    Container ID:  containerd://7fcac7c7f34b038e8a923a3b5f96764a205f3c00efad2fdfb373e3f58f3733a0
    Image:         quay.io/coreos/etcd-operator:v0.9.4
    Image ID:      sha256:d72ca3c4ecd4570ae8f5e9377b12a6988e48302853b372b447c4738b5ef595b4
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd-operator
      --create-crd=false
    State:          Running
      Started:      Wed, 24 Jan 2024 18:27:06 +0100
    Ready:          True
    Restart Count:  0
    Environment:
      MY_POD_NAMESPACE:  default (v1:metadata.namespace)
      MY_POD_NAME:       etcd-operator-5f98c6c6c5-7992p (v1:metadata.name)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-djxlm (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-djxlm:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  18m   default-scheduler  Successfully assigned default/etcd-operator-5f98c6c6c5-7992p to ch03-control-plane
  Normal  Pulling    18m   kubelet            Pulling image "quay.io/coreos/etcd-operator:v0.9.4"
  Normal  Pulled     18m   kubelet            Successfully pulled image "quay.io/coreos/etcd-operator:v0.9.4" in 13.394475797s (13.394491339s including waiting)
  Normal  Created    18m   kubelet            Created container etcd-operator
  Normal  Started    18m   kubelet            Started container etcd-operator


Name:             example-etcd-cluster-bczmh22ph8
Namespace:        default
Priority:         0
Service Account:  default
Node:             ch03-control-plane/172.19.0.2
Start Time:       Wed, 24 Jan 2024 18:28:11 +0100
Labels:           app=etcd
                  etcd_cluster=example-etcd-cluster
                  etcd_node=example-etcd-cluster-bczmh22ph8
Annotations:      etcd.version: 3.1.10
Status:           Pending
IP:               10.244.0.6
IPs:
  IP:           10.244.0.6
Controlled By:  EtcdCluster/example-etcd-cluster
Init Containers:
  check-dns:
    Container ID:  containerd://6b42c1178cd552401dca0c6e180379d17b3e55d4462c3cca5cd7b316c26ff124
    Image:         busybox:1.28.0-glibc
    Image ID:      docker.io/library/busybox@sha256:0b55a30394294ab23b9afd58fab94e61a923f5834fba7ddbae7f8e0c11ba85e6
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c

                TIMEOUT_READY=0
                while ( ! nslookup example-etcd-cluster-bczmh22ph8.example-etcd-cluster.default.svc )
                do
                  # If TIMEOUT_READY is 0 we should never time out and exit
                  TIMEOUT_READY=$(( TIMEOUT_READY-1 ))
                              if [ $TIMEOUT_READY -eq 0 ];
                                  then
                                      echo "Timed out waiting for DNS entry"
                                      exit 1
                                  fi
                              sleep 1
                            done
    State:          Running
      Started:      Wed, 24 Jan 2024 18:28:15 +0100
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
Containers:
  etcd:
    Container ID:
    Image:         :v3.1.10
    Image ID:
    Ports:         2380/TCP, 2379/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/bin/etcd
      --data-dir=/var/etcd/data
      --name=example-etcd-cluster-bczmh22ph8
      --initial-advertise-peer-urls=http://example-etcd-cluster-bczmh22ph8.example-etcd-cluster.default.svc:2380
      --listen-peer-urls=http://0.0.0.0:2380
      --listen-client-urls=http://0.0.0.0:2379
      --advertise-client-urls=http://example-etcd-cluster-bczmh22ph8.example-etcd-cluster.default.svc:2379
      --initial-cluster=example-etcd-cluster-bczmh22ph8=http://example-etcd-cluster-bczmh22ph8.example-etcd-cluster.default.svc:2380
      --initial-cluster-state=new
      --initial-cluster-token=487efdeb-7b74-47c3-91de-3abe26e882b7
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Liveness:       exec [/bin/sh -ec ETCDCTL_API=3 etcdctl endpoint status] delay=10s timeout=10s period=60s #success=1 #failure=3
    Readiness:      exec [/bin/sh -ec ETCDCTL_API=3 etcdctl endpoint status] delay=1s timeout=5s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       False
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  etcd-data:
    Type:        EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:   <unset>
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  17m   default-scheduler  Successfully assigned default/example-etcd-cluster-bczmh22ph8 to ch03-control-plane
  Normal  Pulling    17m   kubelet            Pulling image "busybox:1.28.0-glibc"
  Normal  Pulled     17m   kubelet            Successfully pulled image "busybox:1.28.0-glibc" in 3.287030085s (3.287048251s including waiting)
  Normal  Created    17m   kubelet            Created container check-dns
  Normal  Started    17m   kubelet            Started container check-dns
```

but checking logs

```
davidpitts@Davids-MacBook-Pro ch03 % kubectl logs example-etcd-cluster-bczmh22ph8
Defaulted container "etcd" out of: etcd, check-dns (init)
Error from server (BadRequest): container "etcd" in pod "example-etcd-cluster-bczmh22ph8" is waiting to start: PodInitializing
```

and this appears to match [this coreos/etcd-operator known issue](https://github.com/coreos/etcd-operator/issues/2077):

> After change from flannel network to calico, this not happen more. Try switch network.

and

> please investigate events in kubectl cluster, especially from etcd pods, there should be an info why pod is still in initializing state. Usually it's related to insufficient resources (too high cpu/memory requests per pod), or incorrectly configured storage (for example pod in in zone A while PV was created in zone B, thus you should create new storageclass with volumeBindingMode: WaitForFirstConsumer).