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
