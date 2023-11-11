# ACG-CKAD-Lab03 - Advanced Rollout with Kubernetes Deployments


## Objectives

> The basic features of Kubernetes can be used to implement advanced Deployment strategies such as blue/green Deployments.

```
Check the Deployment configuration:

kubectl describe deployment web-frontend -n hive
Take note of the container's name, in this case, nginx.

Perform a rolling update on the Deployment. You will include the container name you noted before:

kubectl set image deployment.v1.apps/web-frontend -n hive nginx=nginx:1.16.1
Check the status of the rollout:

kubectl rollout status deployment/web-frontend -n hive
You should be able to see the status of the rollout while it occurs.

Check the Deployment's Pods in the hive Namespace:

kubectl get pods -n hive
You should see that the Deployment's Pods are now running the new image version.
```


## Logs

```
cloud_user@k8s-control:~$ kubectl describe deployment web-frontend -n hive
Name:                   web-frontend
Namespace:              hive
CreationTimestamp:      Sat, 11 Nov 2023 17:03:44 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web-frontend
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web-frontend
  Containers:
   nginx:
    Image:        nginx:1.14.2
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   web-frontend-5ff599d8bf (5/5 replicas created)
Events:          <none>
```

```
cloud_user@k8s-control:~$ kubectl set image deployment.v1.apps/web-frontend -n hive nginx=nginx:1.16.1
deployment.apps/web-frontend image updated
cloud_user@k8s-control:~$ kubectl describe deployment web-frontend -n hive
Name:                   web-frontend
Namespace:              hive
CreationTimestamp:      Sat, 11 Nov 2023 17:03:44 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=web-frontend
Replicas:               5 desired | 3 updated | 7 total | 4 available | 3 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web-frontend
  Containers:
   nginx:
    Image:        nginx:1.16.1
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  web-frontend-5ff599d8bf (4/4 replicas created)
NewReplicaSet:   web-frontend-778c8fd558 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  5s    deployment-controller  Scaled up replica set web-frontend-778c8fd558 to 2
  Normal  ScalingReplicaSet  5s    deployment-controller  Scaled down replica set web-frontend-5ff599d8bf to 4
  Normal  ScalingReplicaSet  5s    deployment-controller  Scaled up replica set web-frontend-778c8fd558 to 3
cloud_user@k8s-control:~$ kubectl rollout status deployment/web-frontend -n hive
deployment "web-frontend" successfully rolled out
cloud_user@k8s-control:~$ kubectl get pods -n hive
NAME                                 READY   STATUS    RESTARTS   AGE
internal-api-blue-84957f7888-87rbd   1/1     Running   0          3h34m
web-frontend-778c8fd558-6vlhg        1/1     Running   0          33s
web-frontend-778c8fd558-mgr5t        1/1     Running   0          33s
web-frontend-778c8fd558-mnkml        1/1     Running   0          14s
web-frontend-778c8fd558-pjhnb        1/1     Running   0          14s
web-frontend-778c8fd558-w4lq8        1/1     Running   0          33s
```

```
cloud_user@k8s-control:~$ cp internal-api-blue.yml internal-api-green.yml
cloud_user@k8s-control:~$ cat internal-api-blue.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: internal-api-blue
  namespace: hive
spec:
  replicas: 1
  selector:
    matchLabels:
      app: internal-api
      color: blue
  template:
    metadata:
      labels:
        app: internal-api
        color: blue
    spec:
      containers:
      - name: nginx
        image: linuxacademycontent/ckad-nginx:blue
        ports:
        - containerPort: 80
```

```
cloud_user@k8s-control:~$ vi internal-api-green.yml
cloud_user@k8s-control:~$ diff internal-api-blue.yml internal-api-green.yml
4c4
<   name: internal-api-blue
---
>   name: internal-api-green
11c11
<       color: blue
---
>       color: green
16c16
<         color: blue
---
>         color: green
20c20
<         image: linuxacademycontent/ckad-nginx:blue
---
>         image: linuxacademycontent/ckad-nginx:green
cloud_user@k8s-control:~$ sdiff internal-api-blue.yml internal-api-green.yml
apiVersion: apps/v1                                             apiVersion: apps/v1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  name: internal-api-blue                                     |   name: internal-api-green
  namespace: hive                                                 namespace: hive
spec:                                                           spec:
  replicas: 1                                                     replicas: 1
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: internal-api                                               app: internal-api
      color: blue                                             |       color: green
  template:                                                       template:
    metadata:                                                       metadata:
      labels:                                                         labels:
        app: internal-api                                               app: internal-api
        color: blue                                           |         color: green
    spec:                                                           spec:
      containers:                                                     containers:
      - name: nginx                                                   - name: nginx
        image: linuxacademycontent/ckad-nginx:blue            |         image: linuxacademycontent/ckad-nginx:green
        ports:                                                          ports:
        - containerPort: 80                                             - containerPort: 80
cloud_user@k8s-control:~$ kubectl apply -f internal-api-green.yml
deployment.apps/internal-api-green created
```

```
cloud_user@k8s-control:~$ kubectl get deployments -n hive
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
internal-api-blue    1/1     1            1           3h36m
internal-api-green   1/1     1            1           8s
web-frontend         5/5     5            5           3h36m
cloud_user@k8s-control:~$ kubectl get pods -n hive -o wide
NAME                                  READY   STATUS    RESTARTS   AGE     IP               NODE          NOMINATED NODE   READINESS GATES
internal-api-blue-84957f7888-87rbd    1/1     Running   0          3h36m   192.168.194.71   k8s-worker1   <none>           <none>
internal-api-green-697bc6fb8b-wm6q2   1/1     Running   0          17s     192.168.194.77   k8s-worker1   <none>           <none>
web-frontend-778c8fd558-6vlhg         1/1     Running   0          2m36s   192.168.194.74   k8s-worker1   <none>           <none>
web-frontend-778c8fd558-mgr5t         1/1     Running   0          2m36s   192.168.194.72   k8s-worker1   <none>           <none>
web-frontend-778c8fd558-mnkml         1/1     Running   0          2m17s   192.168.194.76   k8s-worker1   <none>           <none>
web-frontend-778c8fd558-pjhnb         1/1     Running   0          2m17s   192.168.194.75   k8s-worker1   <none>           <none>
web-frontend-778c8fd558-w4lq8         1/1     Running   0          2m36s   192.168.194.73   k8s-worker1   <none>           <none>
```

```
cloud_user@k8s-control:~$ curl 192.168.194.77
I'm green!
cloud_user@k8s-control:~$ curl 192.168.194.71
I'm Blue!
```

```
cloud_user@k8s-control:~$ kubectl edit svc api-svc -n hive
service/api-svc edited
cloud_user@k8s-control:~$ kubectl describe  svc api-svc -n hive
Name:              api-svc
Namespace:         hive
Labels:            <none>
Annotations:       <none>
Selector:          app=internal-api,color=green
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.6.43
IPs:               10.103.6.43
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.194.77:80
Session Affinity:  None
Events:            <none>
```

```
cloud_user@k8s-control:~$ curl 192.168.194.77
I'm green!
cloud_user@k8s-control:~$ curl 192.168.194.77
I'm green!
cloud_user@k8s-control:~$ kubectl describe  svc api-svc -n hive
Name:              api-svc
Namespace:         hive
Labels:            <none>
Annotations:       <none>
Selector:          app=internal-api,color=green
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.6.43
IPs:               10.103.6.43
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.194.77:80
Session Affinity:  None
Events:            <none>
```

```
cloud_user@k8s-control:~$ kubectl edit svc api-svc -n hive
service/api-svc edited
cloud_user@k8s-control:~$ curl 192.168.194.77
I'm green!
cloud_user@k8s-control:~$ curl 192.168.194.77
I'm green!
cloud_user@k8s-control:~$ kubectl describe  svc api-svc -n hive
Name:              api-svc
Namespace:         hive
Labels:            <none>
Annotations:       <none>
Selector:          app=internal-api,color=blue
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.6.43
IPs:               10.103.6.43
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.194.71:80
Session Affinity:  None
Events:            <none>
cloud_user@k8s-control:~$ curl 192.168.194.77
I'm green!
cloud_user@k8s-control:~$ curl 192.168.194.71
I'm Blue!
cloud_user@k8s-control:~$
```