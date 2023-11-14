## KC001 - Pod with Resource Requests and Limits

### Objective

This looks quite simple 
Create a Pod with Resource Requests and Limits
Create a new Namespace limit .
In that Namespace create a Pod named resource-checker of image httpd:alpine .
The container should be named my-container .
It should request 30m CPU and be limited to 300m CPU.
It should request 30Mi memory and be limited to 30Mi memory.


### Solution

First we create the Namespace yaml:

```
kubectl create ns limit
```

Then we generate a Pod yaml:

```
kubectl -n limit run resource-checker --image=httpd:alpine -oyaml --dry-run=client > pod.yaml
```
Next we adjust it to the requirements:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: my-container
    resources:
      requests:
        memory: "30Mi"
        cpu: "30m"
      limits:
        memory: "30Mi"
        cpu: "300m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

and worked very smoothly on the ACG playground

```
cloud_user@k8s-control:~$ kubectl create ns limit
namespace/limit created
cloud_user@k8s-control:~$ kubectl -n limit run resource-checker --image=httpd:alpine -oyaml --dry-run=client > pod.yaml
cloud_user@k8s-control:~$ kubectl describe  ns limit
Name:         limit
Labels:       kubernetes.io/metadata.name=limit
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
cloud_user@k8s-control:~$ vi test.yaml
cloud_user@k8s-control:~$ cat test.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: my-container
    resources:
      requests:
        memory: "30Mi"
        cpu: "30m"
      limits:
        memory: "30Mi"
        cpu: "300m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
cloud_user@k8s-control:~$ kubectl apply -f test.yaml
pod/resource-checker created
cloud_user@k8s-control:~$ kubectl get pods
No resources found in default namespace.
cloud_user@k8s-control:~$ kubectl get pods -n limit
NAME               READY   STATUS    RESTARTS   AGE
resource-checker   1/1     Running   0          26s
```

```
cloud_user@k8s-control:~$ kubectl describe pods -n limit
Name:         resource-checker
Namespace:    limit
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Sun, 12 Nov 2023 07:05:11 +0000
Labels:       run=resource-checker
Annotations:  cni.projectcalico.org/containerID: 67b90b9ffefc8913c9b59e8609482a79f3acdb4dcb484c8ee186f0bf5dee4c93
              cni.projectcalico.org/podIP: 192.168.194.68/32
              cni.projectcalico.org/podIPs: 192.168.194.68/32
Status:       Running
IP:           192.168.194.68
IPs:
  IP:  192.168.194.68
Containers:
  my-container:
    Container ID:   containerd://806f7c3cc49e78b6b781a66985cc4dff22d6a6bb6103e4f1f581f796210610fd
    Image:          httpd:alpine
    Image ID:       docker.io/library/httpd@sha256:5b3cd2c5faf4da636709e7b71ce9daca24c03166125e11a10b58e276b8b0469d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 12 Nov 2023 07:05:15 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     300m
      memory:  30Mi
    Requests:
      cpu:        30m
      memory:     30Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wkvkg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-wkvkg:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  36s   default-scheduler  Successfully assigned limit/resource-checker to k8s-worker1
  Normal  Pulling    35s   kubelet            Pulling image "httpd:alpine"
  Normal  Pulled     33s   kubelet            Successfully pulled image "httpd:alpine" in 2.267084836s
  Normal  Created    32s   kubelet            Created container my-container
  Normal  Started    32s   kubelet            Started container my-container
```


### killercoda playground issue


* Haven't been able to debug this yet?
* Suspect I need to careful review background settings in https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

```
  ########################
##                     ##
##   $ Killer Shell    ##
##                     ##
#########################
Initialising Kubernetes... done
Initialising Scenario... done

controlplane $ create ns limit

Command 'create' not found, did you mean:

  command 'pcreate' from deb pbuilder-scripts (22)

Try: apt install <deb name>

controlplane $ k create ns limit
namespace/limit created
controlplane $ kubectl  -n limit run resource-checker --image=httpd:alpine -oyaml --dry-run=client > pod.yaml
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: resource-checker
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
controlplane $ kubectl apply -f pod.yaml
Error from server (Forbidden): error when creating "pod.yaml": pods "resource-checker" is forbidden: error looking up service account limit/default: serviceaccount "default" not found
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: my-container
    resources:
      requests:
        memory: "30Mi"
        cpu: "30m"
      limits:
        memory: "30Mi"
        cpu: "300m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
controlplane $ vi pod.yaml 
controlplane $ kubectl apply -f pod.yaml
Error from server (Forbidden): error when creating "pod.yaml": pods "resource-checker" is forbidden: error looking up service account limit/default: serviceaccount "default" not found
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: my-container
    resources:
      requests:
        memory: "30Mi"
        cpu: "30m"
      limits:
        memory: "30Mi"
        cpu: "300m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
controlplane $ k apply -f pod.yaml
Error from server (Forbidden): error when creating "pod.yaml": pods "resource-checker" is forbidden: error looking up service account limit/default: serviceaccount "default" not found
controlplane $ alias k
alias k='kubectl'
controlplane $ k get ns
NAME                 STATUS   AGE
default              Active   25d
kube-node-lease      Active   25d
kube-public          Active   25d
kube-system          Active   25d
limit                Active   11m
local-path-storage   Active   25d
controlplane $ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: my-container
    resources:
      requests:
        memory: "30Mi"
        cpu: "30m"
      limits:
        memory: "30Mi"
        cpu: "300m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
controlplane $  


ntrolplane' and this object
controlplane $ k get pods --all-namespaces
NAMESPACE            NAME                                      READY   STATUS             RESTARTS       AGE
kube-system          calico-kube-controllers-9d57d8f49-d26pd   1/1     Running            4 (47m ago)    25d
kube-system          canal-nsbcq                               2/2     Running            0              51m
kube-system          coredns-7cbb7cccb8-w4cps                  1/1     Running            0              25d
kube-system          coredns-7cbb7cccb8-zck8q                  1/1     Running            0              25d
kube-system          etcd-controlplane                         1/1     Running            0              25d
kube-system          kube-apiserver-controlplane               1/1     Running            1              25d
kube-system          kube-controller-manager-controlplane      0/1     CrashLoopBackOff   12 (99s ago)   25d
kube-system          kube-proxy-mk759                          1/1     Running            0              25d
kube-system          kube-scheduler-controlplane               0/1     CrashLoopBackOff   12 (96s ago)   25d
local-path-storage   local-path-provisioner-5d854bc5c4-mgmbx   1/1     Running            0              25d


controlplane $ k describe pods --all-namespaces | more
Name:                 calico-kube-controllers-9d57d8f49-d26pd
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Service Account:      calico-kube-controllers
Node:                 controlplane/172.30.1.2
Start Time:           Tue, 17 Oct 2023 14:33:48 +0000
Labels:               k8s-app=calico-kube-controllers
                      pod-template-hash=9d57d8f49
Annotations:          cni.projectcalico.org/containerID: bb1c659ff347de5a3fd3e09396f205817e8228c1800adaea365baf2178d13943
                      cni.projectcalico.org/podIP: 192.168.0.4/32
                      cni.projectcalico.org/podIPs: 192.168.0.4/32
Status:               Running
IP:                   192.168.0.4
IPs:
  IP:           192.168.0.4
Controlled By:  ReplicaSet/calico-kube-controllers-9d57d8f49
Containers:
  calico-kube-controllers:
    Container ID:   containerd://20fc4d90200300c43827dfd555658a941f3d201468d43000617c5f1e6eb5c38b
    Image:          docker.io/calico/kube-controllers:v3.24.1
    Image ID:       docker.io/calico/kube-controllers@sha256:4010b2739792ae5e77a750be909939c0a0a372e378f3c81020754efcf4a91efa
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 11 Nov 2023 16:12:43 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Sat, 11 Nov 2023 16:09:43 +0000
      Finished:     Sat, 11 Nov 2023 16:12:22 +0000
    Ready:          True
    Restart Count:  4
    Liveness:       exec [/usr/bin/check-status -l] delay=10s timeout=10s period=10s #success=1 #failure=6
    Readiness:      exec [/usr/bin/check-status -r] delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:
      ENABLED_CONTROLLERS:  node
      DATASTORE_TYPE:       kubernetes
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kmb2q (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-kmb2q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age                  From               Message
  ----     ------                  ----                 ----               -------
  Warning  FailedScheduling        25d                  default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node.
kubernetes.io/not-ready: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling..
  Normal   Scheduled               25d                  default-scheduler  Successfully assigned kube-system/calico-kube-controllers-9d57d
8f49-d26pd to controlplane
  Warning  FailedCreatePodSandBox  25d                  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = 
failed to setup network for sandbox "c2e42dda6cd62b8add470f237d7e523f67a256ef09f51d1fc374572adca33156": plugin type="calico" failed (add):
 stat /var/lib/calico/nodename: no such file or directory: check that the calico/node container is running and has mounted /var/lib/calico
/
  Normal   SandboxChanged          25d (x3 over 25d)    kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal   Pulling                 25d                  kubelet            Pulling image "docker.io/calico/kube-controllers:v3.24.1"
  Normal   Pulled                  25d                  kubelet            Successfully pulled image "docker.io/calico/kube-controllers:v3
.24.1" in 5.384s (7.292s including waiting)
  Normal   Started                 25d (x2 over 25d)    kubelet            Started container calico-kube-controllers
  Warning  Unhealthy               25d (x4 over 25d)    kubelet            Readiness probe failed: Error initializing datastore: Get "http
s://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default": dial tcp 10.96.0.1:443: connect: connection refused
  Warning  Unhealthy               25d                  kubelet            Liveness probe failed: Error initializing datastore: Get "https
://10.96.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default": dial tcp 10.96.0.1:443: connect: connection refused
  Warning  Unhealthy               25d (x3 over 25d)    kubelet            Readiness probe failed: initialized to false
  Warning  Unhealthy               25d                  kubelet            Liveness probe failed: Failed to read status file /status/statu
s.json: unexpected end of JSON input
  Normal   Pulled                  51m (x2 over 25d)    kubelet            Container image "docker.io/calico/kube-controllers:v3.24.1" alr
eady present on machine
  Normal   Created                 51m (x3 over 25d)    kubelet            Created container calico-kube-controllers
  Warning  FailedMount             51m (x2 over 51m)    kubelet            MountVolume.SetUp failed for volume "kube-api-access-kmb2q" : t
oken "calico-kube-controllers"/"kube-system"/[]string(nil)/3607/v1.BoundObjectReference{Kind:"Pod", APIVersion:"v1", Name:"calico-kube-con
trollers-9d57d8f49-d26pd", UID:"5988050c-7b2f-4998-984b-2615e29794dc"} expired and refresh failed: Post "https://172.30.1.2:6443/api/v1/na
mespaces/kube-system/serviceaccounts/calico-kube-controllers/token": dial tcp 172.30.1.2:6443: connect: connection refused
  Warning  FailedMount             51m                  kubelet            MountVolume.SetUp failed for volume "kube-api-access-kmb2q" : t
oken "calico-kube-controllers"/"kube-system"/[]string(nil)/3607/v1.BoundObjectReference{Kind:"Pod", APIVersion:"v1", Name:"calico-kube-con
trollers-9d57d8f49-d26pd", UID:"5988050c-7b2f-4998-984b-2615e29794dc"} expired and refresh failed: Post "https://172.30.1.2:6443/api/v1/na
mespaces/kube-system/serviceaccounts/calico-kube-controllers/token": net/http: TLS handshake timeout
  Warning  Unhealthy               13m (x48 over 50m)   kubelet            (combined from similar events): Readiness probe failed: Failed 
to read status file /status/status.json: unexpected end of JSON input
  Warning  Unhealthy               4m7s (x63 over 50m)  kubelet            Readiness probe failed: Error reaching apiserver: <nil> with ht
tp status code: 500
  Warning  Unhealthy               4m7s (x56 over 50m)  kubelet            Liveness probe failed: Error reaching apiserver: <nil> with htt
p status code: 500


Name:                 canal-nsbcq
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Service Account:      canal
Node:                 controlplane/172.30.1.2
Start Time:           Sat, 11 Nov 2023 16:10:01 +0000
Labels:               controller-revision-hash=5fbb664668
                      k8s-app=canal
                      pod-template-generation=1
Annotations:          <none>
Status:               Running
IP:                   172.30.1.2
IPs:
  IP:           172.30.1.2
Controlled By:  DaemonSet/canal
Init Containers:
  install-cni:
    Container ID:  containerd://5e6136dd19ed80899875c77c209187f606dc9942671a354ae930f7b40ebeacad
    Image:         docker.io/calico/cni:v3.24.1
    Image ID:      docker.io/calico/cni@sha256:e60b90d7861e872efa720ead575008bc6eca7bee41656735dcaa8210b688fcd9
    Port:          <none>
    Host Port:     <none>
    Command:
      /opt/cni/bin/install
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 11 Nov 2023 16:10:36 +0000
      Finished:     Sat, 11 Nov 2023 16:10:43 +0000
    Ready:          True
    Restart Count:  0
    Environment Variables from:
      kubernetes-services-endpoint  ConfigMap  Optional: true
    Environment:
      CALICO_CNI_SERVICE_ACCOUNT:   (v1:spec.serviceAccountName)
      CNI_CONF_NAME:               10-canal.conflist
      CNI_NETWORK_CONFIG:          <set to the key 'cni_network_config' of config map 'canal-config'>  Optional: false
      KUBERNETES_NODE_NAME:         (v1:spec.nodeName)
      CNI_MTU:                     <set to the key 'veth_mtu' of config map 'canal-config'>  Optional: false
      SLEEP:                       false
    Mounts:
      /host/etc/cni/net.d from cni-net-dir (rw)
      /host/opt/cni/bin from cni-bin-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l6gqx (ro)
  mount-bpffs:
    Container ID:  containerd://dfcc707a311aa62f86bce753e69a13b5fea5ce60dfe2d112452beb45ced11503
    Image:         docker.io/calico/node:v3.24.1
    Image ID:      docker.io/calico/node@sha256:43f6cee5ca002505ea142b3821a76d585aa0c8d22bc58b7e48589ca7deb48c13
    Port:          <none>
    Host Port:     <none>
    Command:
      calico-node
      -init
      -best-effort
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 11 Nov 2023 16:11:25 +0000
      Finished:     Sat, 11 Nov 2023 16:11:26 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /nodeproc from nodeproc (ro)
      /sys/fs from sys-fs (rw)
      /var/run/calico from var-run-calico (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l6gqx (ro)
Containers:
  calico-node:
    Container ID:   containerd://da1aa35d62e6a6d5ffaebb1397d48ef42d1ecdc7fbe8b658f3e11af065b8e333
    Image:          docker.io/calico/node:v3.24.1
    Image ID:       docker.io/calico/node@sha256:43f6cee5ca002505ea142b3821a76d585aa0c8d22bc58b7e48589ca7deb48c13
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 11 Nov 2023 16:11:28 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:      25m
    Liveness:   exec [/bin/calico-node -felix-live] delay=10s timeout=10s period=10s #success=1 #failure=6
    Readiness:  http-get http://localhost:9099/readiness delay=0s timeout=10s period=10s #success=1 #failure=3
    Environment Variables from:
      kubernetes-services-endpoint  ConfigMap  Optional: true
    Environment:
      DATASTORE_TYPE:                     kubernetes
      USE_POD_CIDR:                       true
      WAIT_FOR_DATASTORE:                 true
      NODENAME:                            (v1:spec.nodeName)
      CALICO_CNI_SERVICE_ACCOUNT:          (v1:spec.serviceAccountName)
      CALICO_NETWORKING_BACKEND:          none
      CLUSTER_TYPE:                       k8s,canal
      FELIX_IPTABLESREFRESHINTERVAL:      60
      IP:                                 
      CALICO_DISABLE_FILE_LOGGING:        true
      FELIX_DEFAULTENDPOINTTOHOSTACTION:  ACCEPT
      FELIX_IPV6SUPPORT:                  false
      FELIX_HEALTHENABLED:                true
    Mounts:
      /host/etc/cni/net.d from cni-net-dir (rw)
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /sys/fs/bpf from bpffs (rw)
      /var/lib/calico from var-lib-calico (rw)
      /var/log/calico/cni from cni-log-dir (ro)
      /var/run/calico from var-run-calico (rw)
      /var/run/nodeagent from policysync (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l6gqx (ro)
  kube-flannel:
    Container ID:  containerd://eca5fb095aeb94990281b86644d645ad58f6cbef22642b89b9e85c7cbfe47b0c
    Image:         quay.io/coreos/flannel:v0.15.1
    Image ID:      quay.io/coreos/flannel@sha256:9a296fbb67790659adc3701e287adde3c59803b7fcefe354f1fc482840cdb3d9
    Port:          <none>
    Host Port:     <none>
    Command:
      /opt/bin/flanneld
      --ip-masq
      --kube-subnet-mgr
    State:          Running
      Started:      Sat, 11 Nov 2023 16:11:29 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:          canal-nsbcq (v1:metadata.name)
      POD_NAMESPACE:     kube-system (v1:metadata.namespace)
      FLANNELD_IFACE:    <set to the key 'canal_iface' of config map 'canal-config'>  Optional: false
      FLANNELD_IP_MASQ:  <set to the key 'masquerade' of config map 'canal-config'>   Optional: false
    Mounts:
      /etc/kube-flannel/ from flannel-cfg (rw)
      /run/xtables.lock from xtables-lock (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l6gqx (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
```  
