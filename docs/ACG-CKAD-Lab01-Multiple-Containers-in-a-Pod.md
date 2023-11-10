
# ACG-CKAD-Lab01 - Using Multiple Containers in a Kubernetes Pod


## Objective
- Update service from 9090 to 9076
- Delay startup by via Init Container (busybox:stable sh -c sleep 10)
- Add Ambassador container (haproxy) to accept traffic on original port 9090



## Update service from 9090 to 9076

We already have hive-svc service

```
cloud_user@k8s-control:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
hive-svc     ClusterIP   10.99.86.209   <none>        9076/TCP   56m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    57m
```

### editing svc hive-svc

Using

```
kubectl edit svc hive-svc
```

update the port
```
ports:
- port: 9076
  ...
```


## Init Container startup-delay


We already have app-gateway deployment

```
cloud_user@k8s-control:~$ kubectl get deployment
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
app-gateway   1/1     1            1           59m
```



Using

```
kubectl edit deployment app-gateway 
```

add the initContainers (busybox:stable sh -c sleep 10)

```
     spec:
       containers:
       - ...
       initContainers:
       - name: startup-delay
         image: busybox:stable
         command: ['sh', '-c', 'sleep 10']
```

## Add Ambassador container (haproxy) to accept traffic on original port 9090

This part was still a bit too complex for me...

```
1) Edit the app-gateway Deployment again:

kubectl edit deployment app-gateway

2) Scroll down to locate the Pod template and specification. Change the main container's command to point to localhost instead of hive-svc:

        command:
          - sh
          - -c
          - while true; do curl localhost:9090; sleep 5; done

3) Add a new ambassador container that uses the haproxy:2.4 image. Mount the existing configMap to haproxy:

  containers:
  - command:
    - sh
    - -c
    - while true; do curl localhost:9090; sleep 5; done
    image: radial/busyboxplus:curl
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
  - name: ambassador
    image: haproxy:2.4
    volumeMounts:
    - name: haproxy-config
      mountPath: /usr/local/etc/haproxy/
  volumes:
  - name: haproxy-config
    configMap:
      name: haproxy-config
```

## Full logs


```
cloud_user@k8s-control:~$ kubectl describe svc
Name:              hive-svc
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=hive
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.86.209
IPs:               10.99.86.209
Port:              <unset>  9090/TCP
TargetPort:        80/TCP
Endpoints:         192.168.194.65:80
Session Affinity:  None
Events:            <none>


Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.0.1
IPs:               10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.0.1.101:6443
Session Affinity:  None
Events:            <none>
cloud_user@k8s-control:~$ kubectl edit svc hive-svc
service/hive-svc edited
cloud_user@k8s-control:~$ kubectl describe svc
Name:              hive-svc
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=hive
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.86.209
IPs:               10.99.86.209
Port:              <unset>  9076/TCP
TargetPort:        80/TCP
Endpoints:         192.168.194.65:80
Session Affinity:  None
Events:            <none>


Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.0.1
IPs:               10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.0.1.101:6443
Session Affinity:  None
Events:            <none>
cloud_user@k8s-control:~$ kubectl describe svc
Name:              hive-svc
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=hive
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.86.209
IPs:               10.99.86.209
Port:              <unset>  9076/TCP
TargetPort:        80/TCP
Endpoints:         192.168.194.65:80
Session Affinity:  None
Events:            <none>


Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.0.1
IPs:               10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.0.1.101:6443
Session Affinity:  None
Events:            <none>
cloud_user@k8s-control:~$ kubectl get deployment
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
app-gateway   1/1     1            1           59m
cloud_user@k8s-control:~$ kubectl describe deployment
Name:                   app-gateway
Namespace:              default
CreationTimestamp:      Thu, 09 Nov 2023 19:09:04 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=gateway
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=gateway
  Containers:
   busybox:
    Image:      radial/busyboxplus:curl
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      while true; do curl hive-svc:9090; sleep 5; done
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   app-gateway-85d8c6bb4d (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  59m   deployment-controller  Scaled up replica set app-gateway-85d8c6bb4d to 1
cloud_user@k8s-control:~$ kubectl edit deployment app-gateway
error: deployments.apps "app-gateway" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-3326875864.yaml"
error: Edit cancelled, no valid changes were saved.
cloud_user@k8s-control:~$ kubectl edit deployment app-gateway
Edit cancelled, no changes made.
cloud_user@k8s-control:~$ cat /tmp/kubectl-edit-3326875864.yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
# deployments.apps "app-gateway" was not valid:
# * <nil>: Invalid value: "The edited file failed validation": yaml: line 45: mapping values are not allowed in this context
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"app-gateway","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"gateway"}},"template":{"metadata":{"labels":{"app":"gateway"}},"spec":{"containers":[{"command":["sh","-c","while true; do curl hive-svc:9090; sleep 5; done"],"image":"radial/busyboxplus:curl","name":"busybox"}]}}}}
  creationTimestamp: "2023-11-09T19:09:04Z"
  generation: 1
  name: app-gateway
  namespace: default
  resourceVersion: "862"
  uid: ec422c19-2e67-4812-a529-af6c451a970a
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: gateway
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: gateway
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl hive-svc:9090; sleep 5; done
        image: radial/busyboxplus:curl
        imagePullPolicy: IfNotPresent
        name: busybox
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      initContainers:
      - name startup-delay
        image: busybox:stable
        command: ['sh','-c','sleep 10']
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2023-11-09T19:11:19Z"
    lastUpdateTime: "2023-11-09T19:11:19Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2023-11-09T19:09:04Z"
    lastUpdateTime: "2023-11-09T19:11:19Z"
    message: ReplicaSet "app-gateway-85d8c6bb4d" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
cloud_user@k8s-control:~$ kubectl edit deployment app-gateway
error: deployments.apps "app-gateway" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1636085232.yaml"
error: Edit cancelled, no valid changes were saved.
cloud_user@k8s-control:~$ kubectl edit deployment app-gateway
Edit cancelled, no changes made.
cloud_user@k8s-control:~$ kubectl edit deployment app-gateway
deployment.apps/app-gateway edited
cloud_user@k8s-control:~$ kubectl get pods
NAME                           READY   STATUS        RESTARTS   AGE
app-gateway-757bcd5456-brfvc   1/1     Running       0          38s
app-gateway-85d8c6bb4d-2pw2k   1/1     Terminating   0          71m
hive-web                       1/1     Running       0          71m
cloud_user@k8s-control:~$ kubectl edit deployment
error: deployments.apps "app-gateway" is invalid
deployment.apps/app-gateway edited
cloud_user@k8s-control:~$ kubectl get pods
NAME                           READY   STATUS        RESTARTS   AGE
app-gateway-5bf5554d8d-4cb5w   2/2     Running       0          14s
app-gateway-757bcd5456-brfvc   1/1     Terminating   0          7m28s
hive-web                       1/1     Running       0          78m
cloud_user@k8s-control:~$ kubectl get pods
NAME                           READY   STATUS        RESTARTS   AGE
app-gateway-5bf5554d8d-4cb5w   2/2     Running       0          19s
app-gateway-757bcd5456-brfvc   1/1     Terminating   0          7m33s
hive-web                       1/1     Running       0          78m
cloud_user@k8s-control:~$ kubectl edit deployment
error: deployments.apps "app-gateway" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-2100050158.yaml"
error: Edit cancelled, no valid changes were saved.
cloud_user@k8s-control:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
app-gateway-5bf5554d8d-4cb5w   2/2     Running   0          93s
hive-web                       1/1     Running   0          80m
cloud_user@k8s-control:~$ kubectl logs app-gateway-5bf5554d8d-4cb5w -c busybox
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0curl: (7) Failed to connect to localhost port 9090: Connection refused
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    40  100    40    0     0  15729      0 --:--:-- --:--:-- --:--:-- 40000
{"message": "Welcome to the Hive API!"}
...

100    40  100    40    0     0  39643      0 --:--:-- --:--:-- --:--:-- 40000
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
{"message": "Welcome to the Hive API!"}
100    40  100    40    0     0  24829      0 --:--:-- --:--:-- --:--:-- 40000
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    40  100    40    0     0  32626      0 --:--:-- --:--:-- --:--:-- 40000
{"message": "Welcome to the Hive API!"}
cloud_user@k8s-control:~$ kubectl logs app-gateway-5bf5554d8d-4cb5w -c busybox | wc -l
139
cloud_user@k8s-control:~$ sleep 10,kubectl logs app-gateway-5bf5554d8d-4cb5w -c busybox | wc -l
sleep: invalid option -- 'c'
Try 'sleep --help' for more information.
0
cloud_user@k8s-control:~$ sleep 10;kubectl logs app-gateway-5bf5554d8d-4cb5w -c busybox | wc -l
163
cloud_user@k8s-control:~$ sleep 10;kubectl logs app-gateway-5bf5554d8d-4cb5w -c busybox | wc -l
175
cloud_user@k8s-control:~$ kubectl describe pods
Name:         app-gateway-5bf5554d8d-4cb5w
Namespace:    default
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Thu, 09 Nov 2023 20:27:35 +0000
Labels:       app=gateway
              pod-template-hash=5bf5554d8d
Annotations:  cni.projectcalico.org/containerID: 21bb62f86fb868ba22f1c858dc10ff191eb9e5bd83b48862724c159c6f730d7c
              cni.projectcalico.org/podIP: 192.168.194.69/32
              cni.projectcalico.org/podIPs: 192.168.194.69/32
Status:       Running
IP:           192.168.194.69
IPs:
  IP:           192.168.194.69
Controlled By:  ReplicaSet/app-gateway-5bf5554d8d
Init Containers:
  startup-delay:
    Container ID:  containerd://5f86630cda3c6c600988ce66a226e8693a80bea94ccc38f4ecaef2e0b882ce35
    Image:         busybox:stable
    Image ID:      docker.io/library/busybox@sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 10
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 09 Nov 2023 20:27:36 +0000
      Finished:     Thu, 09 Nov 2023 20:27:46 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kq57p (ro)
Containers:
  busybox:
    Container ID:  containerd://ab8de86a895c9e17e9277b3e9f64449c1d617de4565a514118758c9ff2ab1b6f
    Image:         radial/busyboxplus:curl
    Image ID:      sha256:4776f1f7d1f625c8c5173a969fdc9ae6b62655a2746aba989784bb2b7edbfe9b
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do curl localhost:9090; sleep 5; done
    State:          Running
      Started:      Thu, 09 Nov 2023 20:27:46 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kq57p (ro)
  ambassador:
    Container ID:   containerd://b6fff10fe6f70a872d39bfa8317aba120cafa6a41ae23fe9912b139cf4a3000b
    Image:          haproxy:2.4
    Image ID:       docker.io/library/haproxy@sha256:c78fe878cc877eb27e514f15bb7773b5e01aa5415d089d694e00cf809e020e8e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 09 Nov 2023 20:27:48 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/local/etc/haproxy/ from haproxy-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kq57p (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  haproxy-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      haproxy-config
    Optional:  false
  kube-api-access-kq57p:
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
  Normal  Scheduled  4m47s  default-scheduler  Successfully assigned default/app-gateway-5bf5554d8d-4cb5w to k8s-worker1
  Normal  Pulled     4m46s  kubelet            Container image "busybox:stable" already present on machine
  Normal  Created    4m46s  kubelet            Created container startup-delay
  Normal  Started    4m46s  kubelet            Started container startup-delay
  Normal  Pulled     4m36s  kubelet            Container image "radial/busyboxplus:curl" already present on machine
  Normal  Created    4m36s  kubelet            Created container busybox
  Normal  Started    4m36s  kubelet            Started container busybox
  Normal  Pulling    4m36s  kubelet            Pulling image "haproxy:2.4"
  Normal  Pulled     4m34s  kubelet            Successfully pulled image "haproxy:2.4" in 1.316100968s
  Normal  Created    4m34s  kubelet            Created container ambassador
  Normal  Started    4m34s  kubelet            Started container ambassador


Name:         hive-web
Namespace:    default
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Thu, 09 Nov 2023 19:10:24 +0000
Labels:       app=hive
Annotations:  cni.projectcalico.org/containerID: 8993e407da579c4e9f0200f46ae63b411b93c6e844021294496e40eecb20abb7
              cni.projectcalico.org/podIP: 192.168.194.65/32
              cni.projectcalico.org/podIPs: 192.168.194.65/32
Status:       Running
IP:           192.168.194.65
IPs:
  IP:  192.168.194.65
Containers:
  nginx:
    Container ID:   containerd://fccc62e364bc0819368e009481e05f6bc1b19bcc449cf2fdc55a9e69fc54c858
    Image:          nginx:stable
    Image ID:       docker.io/library/nginx@sha256:8091c5f722b5060431042b000a742df96a586c6ecc3dcb440fbbdbdc3c261f3e
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 09 Nov 2023 19:11:04 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html/ from content (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xlmvd (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  content:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      hive-content
    Optional:  false
  kube-api-access-xlmvd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
cloud_user@k8s-control:~$
```

