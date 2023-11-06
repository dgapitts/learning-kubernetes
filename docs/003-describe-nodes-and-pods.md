# 003-describe-nodes-and-pods

## kubectl describe nodes

"the basics of monitoring and maintaining node status to ensure a healthy and stable cluster"
https://kubernetes.io/docs/reference/node/node-status/


and the memory and cpu constraints are at the bottom

```
[~/projects/learning-kubernetes] # sudo kubectl describe nodes
Name:               ijsselstein
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=k3s
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=ijsselstein
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=true
                    node-role.kubernetes.io/master=true
                    node.kubernetes.io/instance-type=k3s
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"12:3f:8b:7d:ac:95"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.1.137
                    k3s.io/hostname: ijsselstein
                    k3s.io/internal-ip: 192.168.1.137
                    k3s.io/node-args: ["server"]
                    k3s.io/node-config-hash: ZG257ICDIZ7PEQFNS6LNUKWNCK3SEWJLTDY22RDPHKWIG3BXUOXQ====
                    k3s.io/node-env: {"K3S_DATA_DIR":"/var/lib/rancher/k3s/data/9d8f9670e1bff08a901bc7bc270202323f7c2c716a89a73d776c363ac1971018"}
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 25 Nov 2021 22:09:13 +0100
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  ijsselstein
  AcquireTime:     <unset>
  RenewTime:       Mon, 06 Nov 2023 19:38:19 +0100
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Fri, 03 Nov 2023 20:09:15 +0100   Fri, 03 Nov 2023 20:09:15 +0100   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Mon, 06 Nov 2023 19:36:13 +0100   Thu, 25 Nov 2021 22:09:12 +0100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Mon, 06 Nov 2023 19:36:13 +0100   Thu, 25 Nov 2021 22:09:12 +0100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Mon, 06 Nov 2023 19:36:13 +0100   Thu, 25 Nov 2021 22:09:12 +0100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Mon, 06 Nov 2023 19:36:13 +0100   Sun, 20 Aug 2023 17:19:39 +0200   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  192.168.1.137
  Hostname:    ijsselstein
Capacity:
  cpu:                4
  ephemeral-storage:  475136144Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16260592Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  462212440521
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16260592Ki
  pods:               110
System Info:
  Machine ID:                 7c3fd60f5b324a488fa9a0bb7dc64532
  System UUID:                325bfa80-0975-0000-0000-000000000000
  Boot ID:                    4a600709-d5ba-4f02-96a0-3b6b68d3f420
  Kernel Version:             5.15.0-86-generic
  OS Image:                   Ubuntu 22.04.3 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.4.11-k3s1
  Kubelet Version:            v1.21.5+k3s2
  Kube-Proxy Version:         v1.21.5+k3s2
PodCIDR:                      10.42.0.0/24
PodCIDRs:                     10.42.0.0/24
ProviderID:                   k3s://ijsselstein
Non-terminated Pods:          (6 in total)
  Namespace                   Name                                       CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                       ------------  ----------  ---------------  -------------  ---
  kube-system                 metrics-server-86cbb8457f-zcbvp            0 (0%)        0 (0%)      0 (0%)           0 (0%)         710d
  kube-system                 coredns-7448499f4d-b7wt6                   100m (2%)     0 (0%)      70Mi (0%)        170Mi (1%)     710d
  kube-system                 svclb-traefik-jlk8j                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         710d
  kube-system                 traefik-97b44b794-xvg7m                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         710d
  default                     web                                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         21h
  kube-system                 local-path-provisioner-5ff76fc89d-wtwvl    0 (0%)        0 (0%)      0 (0%)           0 (0%)         710d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (2%)  0 (0%)
  memory             70Mi (0%)  170Mi (1%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:              <none>
[~/projects/learning-kubernetes] # 
```



## kubectl describe pods

"The `describe` subcommand displays more information about a specific pod than `get` does.
`kubectl describe pod constraintpod`
There is quite a bit of information provided, with the resource limits displayed …"
https://kubebyexample.com/concept/pods



and the event history is at the bottom

```
[~/projects/learning-kubernetes] # sudo kubectl describe pods
Name:         web
Namespace:    default
Priority:     0
Node:         ijsselstein/192.168.1.137
Start Time:   Sun, 05 Nov 2023 21:59:17 +0100
Labels:       run=web
Annotations:  <none>
Status:       Running
IP:           10.42.0.69
IPs:
  IP:  10.42.0.69
Containers:
  web:
    Container ID:   containerd://2b86e45196a6e110c7f015484f7323f0fbccdf6bab540780d722da4846e92a96
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:86e53c4c16a6a276b204b0fd3a8143d86547c967dc8258b3d47c3a21bb68d3c6
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 05 Nov 2023 21:59:29 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8jfgp (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-8jfgp:
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
  Normal  Scheduled  21h   default-scheduler  Successfully assigned default/web to ijsselstein
  Normal  Pulling    21h   kubelet            Pulling image "nginx"
  Normal  Pulled     21h   kubelet            Successfully pulled image "nginx" in 10.088908173s
  Normal  Created    21h   kubelet            Created container web
  Normal  Started    21h   kubelet            Started container web
```

