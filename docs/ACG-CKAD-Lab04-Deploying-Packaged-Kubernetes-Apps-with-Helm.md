# ACG-CKAD-Lab04 - Deploying Packaged Kubernetes Apps with Helm

## Overview
> Helm is a powerful tool that can allow you to more easily manage applications in Kubernetes. This lab will give you a chance to get hands-on with Helm by installing Helm and deploying a Helm chart.


## Install Helm
Helm now provides an official installer script that you can use to grab the latest version of Helm and install it locally:
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Verify the installation:

```
helm version
```
Install a Helm Chart in the Cluster


Add the bitnami chart repository:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update the chart listing:

```
helm repo update
```
Install the chart in the cert-manager Namespace:



```
helm install -n cert-manager cert-manager bitnami/cert-manager
```

View the Pods created by the Helm installation:

```
kubectl get pods -n cert-manager
```

View the Deployments created by the Helm installation:

```
kubectl get deployments -n cert-manager
```

View the Services created by the Helm installation:

```
kubectl get svc -n cert-manager
```


# Logs and lab details

Download via curl
```
cloud_user@k8s-control:~$ time curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

real    0m0.026s
user    0m0.012s
sys     0m0.008s
cloud_user@k8s-control:~$ ls -l
total 592
-rw-r--r-- 1 root       root       591853 Jan 10  2022 aws-cfn-bootstrap-py3-latest.tar.gz
-rw-rw-r-- 1 cloud_user cloud_user  11664 Nov 11 20:51 get_helm.sh
```

and run
```
cloud_user@k8s-control:~$ chmod 700 get_helm.sh
cloud_user@k8s-control:~$ ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.13.1-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
[sudo] password for cloud_user:
helm installed into /usr/local/bin/helm
cloud_user@k8s-control:~$ helm version
version.BuildInfo{Version:"v3.13.1", GitCommit:"3547a4b5bf5edb5478ce352e18858d8a552a4110", GitTreeState:"clean", GoVersion:"go1.20.8"}
```

### helm repo add bitnami
```
cloud_user@k8s-control:~$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

###helm repo update
```
cloud_user@k8s-control:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
cloud_user@k8s-control:~$ helm install -n cert-manager cert-manager bitnami/cert-manager
NAME: cert-manager
LAST DEPLOYED: Sat Nov 11 20:53:00 2023
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: cert-manager
CHART VERSION: 0.13.3
APP VERSION: 1.13.2

** Please be patient while the chart is being deployed **

In order to begin using certificates, you will need to set up Issuer or ClustersIssuer resources.

https://cert-manager.io/docs/configuration/

To configure a new ingress to automatically provision certificates, you will find some information in the following link:

https://cert-manager.io/docs/usage/ingress/
```

### kubectl get pods
```
cloud_user@k8s-control:~$ kubectl get pods -n cert-manager
NAME                                       READY   STATUS              RESTARTS   AGE
cert-manager-cainjector-b4b45f7c8-khhvb    0/1     ContainerCreating   0          10s
cert-manager-controller-59df5c5b7c-6wmfq   0/1     ContainerCreating   0          10s
cert-manager-webhook-9fd595cc5-5ntn2       0/1     ContainerCreating   0          10s
cloud_user@k8s-control:~$ kubectl get deployments -n cert-manager
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
cert-manager-cainjector   0/1     1            0           19s
cert-manager-controller   0/1     1            0           19s
cert-manager-webhook      0/1     1            0           19s
```
### kubectl get svc
```
cloud_user@k8s-control:~$ kubectl get svc -n cert-manager
NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
cert-manager-controller-metrics   ClusterIP   10.111.234.229   <none>        9402/TCP   27s
cert-manager-webhook              ClusterIP   10.96.135.31     <none>        443/TCP    27s
```

### kubectl describe svc
```
cloud_user@k8s-control:~$ kubectl describe svc -n cert-manager
Name:              cert-manager-controller-metrics
Namespace:         cert-manager
Labels:            app.kubernetes.io/component=controller
                   app.kubernetes.io/instance=cert-manager
                   app.kubernetes.io/managed-by=Helm
                   app.kubernetes.io/name=cert-manager
                   app.kubernetes.io/version=1.13.2
                   helm.sh/chart=cert-manager-0.13.3
Annotations:       meta.helm.sh/release-name: cert-manager
                   meta.helm.sh/release-namespace: cert-manager
Selector:          app.kubernetes.io/component=controller,app.kubernetes.io/instance=cert-manager,app.kubernetes.io/name=cert-manager
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.111.234.229
IPs:               10.111.234.229
Port:              controller  9402/TCP
TargetPort:        9402/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>


Name:              cert-manager-webhook
Namespace:         cert-manager
Labels:            app.kubernetes.io/component=webhook
                   app.kubernetes.io/instance=cert-manager
                   app.kubernetes.io/managed-by=Helm
                   app.kubernetes.io/name=cert-manager
                   app.kubernetes.io/version=1.13.2
                   helm.sh/chart=cert-manager-0.13.3
Annotations:       meta.helm.sh/release-name: cert-manager
                   meta.helm.sh/release-namespace: cert-manager
Selector:          app.kubernetes.io/component=webhook,app.kubernetes.io/instance=cert-manager,app.kubernetes.io/name=cert-manager
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.135.31
IPs:               10.96.135.31
Port:              https  443/TCP
TargetPort:        10250/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```
### kubectl describe deployments
```
cloud_user@k8s-control:~$ kubectl describe  deployments -n cert-manager
Name:                   cert-manager-cainjector
Namespace:              cert-manager
CreationTimestamp:      Sat, 11 Nov 2023 20:53:04 +0000
Labels:                 app.kubernetes.io/component=cainjector
                        app.kubernetes.io/instance=cert-manager
                        app.kubernetes.io/managed-by=Helm
                        app.kubernetes.io/name=cert-manager
                        app.kubernetes.io/version=1.13.2
                        helm.sh/chart=cert-manager-0.13.3
Annotations:            deployment.kubernetes.io/revision: 1
                        meta.helm.sh/release-name: cert-manager
                        meta.helm.sh/release-namespace: cert-manager
Selector:               app.kubernetes.io/component=cainjector,app.kubernetes.io/instance=cert-manager,app.kubernetes.io/name=cert-manager
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app.kubernetes.io/component=cainjector
                    app.kubernetes.io/instance=cert-manager
                    app.kubernetes.io/managed-by=Helm
                    app.kubernetes.io/name=cert-manager
                    app.kubernetes.io/version=1.13.2
                    helm.sh/chart=cert-manager-0.13.3
  Service Account:  cert-manager-cainjector
  Containers:
   cainjector:
    Image:      docker.io/bitnami/cainjector:1.13.2-debian-11-r1
    Port:       <none>
    Host Port:  <none>
    Args:
      --v=2
      --leader-election-namespace=kube-system
    Environment:
      BITNAMI_DEBUG:  false
      POD_NAMESPACE:   (v1:metadata.namespace)
    Mounts:           <none>
  Volumes:            <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   cert-manager-cainjector-b4b45f7c8 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  65s   deployment-controller  Scaled up replica set cert-manager-cainjector-b4b45f7c8 to 1


Name:                   cert-manager-controller
Namespace:              cert-manager
CreationTimestamp:      Sat, 11 Nov 2023 20:53:04 +0000
Labels:                 app.kubernetes.io/component=controller
                        app.kubernetes.io/instance=cert-manager
                        app.kubernetes.io/managed-by=Helm
                        app.kubernetes.io/name=cert-manager
                        app.kubernetes.io/version=1.13.2
                        helm.sh/chart=cert-manager-0.13.3
Annotations:            deployment.kubernetes.io/revision: 1
                        meta.helm.sh/release-name: cert-manager
                        meta.helm.sh/release-namespace: cert-manager
Selector:               app.kubernetes.io/component=controller,app.kubernetes.io/instance=cert-manager,app.kubernetes.io/name=cert-manager
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app.kubernetes.io/component=controller
                    app.kubernetes.io/instance=cert-manager
                    app.kubernetes.io/managed-by=Helm
                    app.kubernetes.io/name=cert-manager
                    app.kubernetes.io/version=1.13.2
                    helm.sh/chart=cert-manager-0.13.3
  Annotations:      prometheus.io/path: /metrics
                    prometheus.io/port: 9402
                    prometheus.io/scrape: true
  Service Account:  cert-manager-controller
  Containers:
   cert-manager:
    Image:      docker.io/bitnami/cert-manager:1.13.2-debian-11-r1
    Port:       9402/TCP
    Host Port:  0/TCP
    Args:
      --v=2
      --cluster-resource-namespace=$(POD_NAMESPACE)
      --leader-election-namespace=kube-system
      --acme-http01-solver-image=docker.io/bitnami/acmesolver:1.13.2-debian-11-r1
    Environment:
      BITNAMI_DEBUG:  false
      POD_NAMESPACE:   (v1:metadata.namespace)
    Mounts:           <none>
  Volumes:            <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   cert-manager-controller-59df5c5b7c (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  65s   deployment-controller  Scaled up replica set cert-manager-controller-59df5c5b7c to 1


Name:                   cert-manager-webhook
Namespace:              cert-manager
CreationTimestamp:      Sat, 11 Nov 2023 20:53:04 +0000
Labels:                 app.kubernetes.io/component=webhook
                        app.kubernetes.io/instance=cert-manager
                        app.kubernetes.io/managed-by=Helm
                        app.kubernetes.io/name=cert-manager
                        app.kubernetes.io/version=1.13.2
                        helm.sh/chart=cert-manager-0.13.3
Annotations:            deployment.kubernetes.io/revision: 1
                        meta.helm.sh/release-name: cert-manager
                        meta.helm.sh/release-namespace: cert-manager
Selector:               app.kubernetes.io/component=webhook,app.kubernetes.io/instance=cert-manager,app.kubernetes.io/name=cert-manager
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app.kubernetes.io/component=webhook
                    app.kubernetes.io/instance=cert-manager
                    app.kubernetes.io/managed-by=Helm
                    app.kubernetes.io/name=cert-manager
                    app.kubernetes.io/version=1.13.2
                    helm.sh/chart=cert-manager-0.13.3
  Service Account:  cert-manager-webhook
  Containers:
   cert-manager-webhook:
    Image:      docker.io/bitnami/cert-manager-webhook:1.13.2-debian-11-r1
    Port:       10250/TCP
    Host Port:  0/TCP
    Args:
      --v=2
      --secure-port=10250
      --dynamic-serving-ca-secret-namespace=$(POD_NAMESPACE)
      --dynamic-serving-ca-secret-name=cert-manager-webhook-ca
      --dynamic-serving-dns-names=cert-manager-webhook,cert-manager-webhook.cert-manager,cert-manager-webhook.cert-manager.svc
    Liveness:   http-get http://:6080/livez delay=60s timeout=1s period=10s #success=1 #failure=3
    Readiness:  http-get http://:6080/healthz delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:
      BITNAMI_DEBUG:  false
      POD_NAMESPACE:   (v1:metadata.namespace)
    Mounts:           <none>
  Volumes:            <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   cert-manager-webhook-9fd595cc5 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  65s   deployment-controller  Scaled up replica set cert-manager-webhook-9fd595cc5 to 1
```
### kubectl describe pods
```
cloud_user@k8s-control:~$ kubectl describe pods -n cert-manager
Name:         cert-manager-cainjector-b4b45f7c8-khhvb
Namespace:    cert-manager
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Sat, 11 Nov 2023 20:53:04 +0000
Labels:       app.kubernetes.io/component=cainjector
              app.kubernetes.io/instance=cert-manager
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=cert-manager
              app.kubernetes.io/version=1.13.2
              helm.sh/chart=cert-manager-0.13.3
              pod-template-hash=b4b45f7c8
Annotations:  cni.projectcalico.org/containerID: 703b9fb49ebcec51cc86a15865b3aa6ac8c42b43c842c12cb476d0f777cee8d8
              cni.projectcalico.org/podIP: 192.168.194.68/32
              cni.projectcalico.org/podIPs: 192.168.194.68/32
              container.seccomp.security.alpha.kubernetes.io/cainjector: runtime/default
Status:       Running
IP:           192.168.194.68
IPs:
  IP:           192.168.194.68
Controlled By:  ReplicaSet/cert-manager-cainjector-b4b45f7c8
Containers:
  cainjector:
    Container ID:  containerd://d44fc1a51f23a41c377c99c293fcf2a077384fa3c3408692ef21986cc9efc1d5
    Image:         docker.io/bitnami/cainjector:1.13.2-debian-11-r1
    Image ID:      docker.io/bitnami/cainjector@sha256:0efda2e91ffacc4bf468275c42ca94132722aae7ef90f818038166bff4b0eb98
    Port:          <none>
    Host Port:     <none>
    Args:
      --v=2
      --leader-election-namespace=kube-system
    State:          Running
      Started:      Sat, 11 Nov 2023 20:54:06 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      BITNAMI_DEBUG:  false
      POD_NAMESPACE:  cert-manager (v1:metadata.namespace)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-d9n8h (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-d9n8h:
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
  Normal  Scheduled  91s   default-scheduler  Successfully assigned cert-manager/cert-manager-cainjector-b4b45f7c8-khhvb to k8s-worker1
  Normal  Pulling    89s   kubelet            Pulling image "docker.io/bitnami/cainjector:1.13.2-debian-11-r1"
  Normal  Pulled     81s   kubelet            Successfully pulled image "docker.io/bitnami/cainjector:1.13.2-debian-11-r1" in 7.919570698s
  Normal  Created    29s   kubelet            Created container cainjector
  Normal  Started    29s   kubelet            Started container cainjector


Name:         cert-manager-controller-59df5c5b7c-6wmfq
Namespace:    cert-manager
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Sat, 11 Nov 2023 20:53:04 +0000
Labels:       app.kubernetes.io/component=controller
              app.kubernetes.io/instance=cert-manager
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=cert-manager
              app.kubernetes.io/version=1.13.2
              helm.sh/chart=cert-manager-0.13.3
              pod-template-hash=59df5c5b7c
Annotations:  cni.projectcalico.org/containerID: 4e259a7ba677e8645d369e75301c89d12115d38f5c49f6b89834706a612d5e87
              cni.projectcalico.org/podIP: 192.168.194.69/32
              cni.projectcalico.org/podIPs: 192.168.194.69/32
              container.seccomp.security.alpha.kubernetes.io/cert-manager: runtime/default
              prometheus.io/path: /metrics
              prometheus.io/port: 9402
              prometheus.io/scrape: true
Status:       Running
IP:           192.168.194.69
IPs:
  IP:           192.168.194.69
Controlled By:  ReplicaSet/cert-manager-controller-59df5c5b7c
Containers:
  cert-manager:
    Container ID:  containerd://e136631ab21ddc2c891f7aec0e389610de82cc30d2315b04ed7aaaa35a07d2f2
    Image:         docker.io/bitnami/cert-manager:1.13.2-debian-11-r1
    Image ID:      docker.io/bitnami/cert-manager@sha256:f35c087bd4ee154d19184d96b992a790fe01a1f88d3747377179220ed0fec7b8
    Port:          9402/TCP
    Host Port:     0/TCP
    Args:
      --v=2
      --cluster-resource-namespace=$(POD_NAMESPACE)
      --leader-election-namespace=kube-system
      --acme-http01-solver-image=docker.io/bitnami/acmesolver:1.13.2-debian-11-r1
    State:          Running
      Started:      Sat, 11 Nov 2023 20:54:06 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      BITNAMI_DEBUG:  false
      POD_NAMESPACE:  cert-manager (v1:metadata.namespace)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kmb2t (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-kmb2t:
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
  Normal  Scheduled  91s   default-scheduler  Successfully assigned cert-manager/cert-manager-controller-59df5c5b7c-6wmfq to k8s-worker1
  Normal  Pulling    89s   kubelet            Pulling image "docker.io/bitnami/cert-manager:1.13.2-debian-11-r1"
  Normal  Pulled     63s   kubelet            Successfully pulled image "docker.io/bitnami/cert-manager:1.13.2-debian-11-r1" in 25.330186261s
  Normal  Created    29s   kubelet            Created container cert-manager
  Normal  Started    29s   kubelet            Started container cert-manager


Name:         cert-manager-webhook-9fd595cc5-5ntn2
Namespace:    cert-manager
Priority:     0
Node:         k8s-worker1/10.0.1.102
Start Time:   Sat, 11 Nov 2023 20:53:04 +0000
Labels:       app.kubernetes.io/component=webhook
              app.kubernetes.io/instance=cert-manager
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=cert-manager
              app.kubernetes.io/version=1.13.2
              helm.sh/chart=cert-manager-0.13.3
              pod-template-hash=9fd595cc5
Annotations:  cni.projectcalico.org/containerID: c55acf9add7d467e2b0903cddc6e23ff36b2d5eb1afc1923a07e1d0ed2bf9ae5
              cni.projectcalico.org/podIP: 192.168.194.70/32
              cni.projectcalico.org/podIPs: 192.168.194.70/32
              container.seccomp.security.alpha.kubernetes.io/cert-manager-webhook: runtime/default
Status:       Running
IP:           192.168.194.70
IPs:
  IP:           192.168.194.70
Controlled By:  ReplicaSet/cert-manager-webhook-9fd595cc5
Containers:
  cert-manager-webhook:
    Container ID:  containerd://628af1276fbdce7fad4bddcf866f77d4ff0f4beaaf8bf2bf6a4252141774436c
    Image:         docker.io/bitnami/cert-manager-webhook:1.13.2-debian-11-r1
    Image ID:      docker.io/bitnami/cert-manager-webhook@sha256:173f3c796b3c4dde78bbd19e0833337786ca46a97563ce095cb660496add0b85
    Port:          10250/TCP
    Host Port:     0/TCP
    Args:
      --v=2
      --secure-port=10250
      --dynamic-serving-ca-secret-namespace=$(POD_NAMESPACE)
      --dynamic-serving-ca-secret-name=cert-manager-webhook-ca
      --dynamic-serving-dns-names=cert-manager-webhook,cert-manager-webhook.cert-manager,cert-manager-webhook.cert-manager.svc
    State:          Running
      Started:      Sat, 11 Nov 2023 20:54:06 +0000
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:6080/livez delay=60s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:6080/healthz delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:
      BITNAMI_DEBUG:  false
      POD_NAMESPACE:  cert-manager (v1:metadata.namespace)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-np4nq (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-np4nq:
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
  Normal  Scheduled  91s   default-scheduler  Successfully assigned cert-manager/cert-manager-webhook-9fd595cc5-5ntn2 to k8s-worker1
  Normal  Pulling    89s   kubelet            Pulling image "docker.io/bitnami/cert-manager-webhook:1.13.2-debian-11-r1"
  Normal  Pulled     41s   kubelet            Successfully pulled image "docker.io/bitnami/cert-manager-webhook:1.13.2-debian-11-r1" in 47.350486176s
  Normal  Created    29s   kubelet            Created container cert-manager-webhook
  Normal  Started    29s   kubelet            Started container cert-manager-webhook
```

  