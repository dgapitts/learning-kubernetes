# CrashLoopBackOff for Container image "docker.io/mtinside/blue-green:blue" 

```
[docker@c7minikube 01_03]$ kubectl describe pods
Name:             blue
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Thu, 09 Nov 2023 11:20:29 +0000
Labels:           app=blue-green
Annotations:      <none>
Status:           Running
IP:               10.244.0.3
IPs:
  IP:  10.244.0.3
Containers:
  blue:
    Container ID:   docker://72f1a39251f10062ccdcd2157571b1bdcd1cf54716431374d8aa1d56e94e5db1
    Image:          docker.io/mtinside/blue-green:blue
    Image ID:       docker-pullable://mtinside/blue-green@sha256:5fb83f49dd7001591fb1591a0d74596bf8264e527702dc78c0a06651fffd0c60
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Thu, 09 Nov 2023 11:21:21 +0000
      Finished:     Thu, 09 Nov 2023 11:21:21 +0000
    Ready:          False
    Restart Count:  3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-c4bn7 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-c4bn7:
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
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  70s                default-scheduler  Successfully assigned default/blue to minikube
  Normal   Pulling    67s                kubelet            Pulling image "docker.io/mtinside/blue-green:blue"
  Normal   Pulled     62s                kubelet            Successfully pulled image "docker.io/mtinside/blue-green:blue" in 2.074s (5.211s including waiting)
  Normal   Created    18s (x4 over 62s)  kubelet            Created container blue
  Normal   Started    18s (x4 over 62s)  kubelet            Started container blue
  Normal   Pulled     18s (x3 over 61s)  kubelet            Container image "docker.io/mtinside/blue-green:blue" already present on machine
  Warning  BackOff    5s (x6 over 60s)   kubelet            Back-off restarting failed container blue in pod blue_default(24a1114b-9c2e-4032-8fd7-9fbffe806419)
```


Looking into docker buildx as this appears to be arm vs x86 issue... tbc
