

## 002-my-first-pod

I suspect there is a general work around for this
```
[~/projects/learning-kubernetes] # kubectl run --image=nginx web
WARN[2023-11-05T21:59:03.641219703+01:00] Unable to read /etc/rancher/k3s/k3s.yaml, please start server with --write-kubeconfig-mode to modify kube config permissions 
error: error loading config file "/etc/rancher/k3s/k3s.yaml": open /etc/rancher/k3s/k3s.yaml: permission denied
```

but for now let´s brute force this i.e. use sudo
```
[~/projects/learning-kubernetes] # sudo kubectl run --image=nginx web
pod/web created
[~/projects/learning-kubernetes] # sudo kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          24s
```

and to get the full details of the pod I have just deployed:
```
[~/projects/learning-kubernetes] # sudo kubectl get pods web -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2023-11-05T20:59:17Z"
  labels:
    run: web
  name: web
  namespace: default
  resourceVersion: "7704642"
  uid: 05139e6b-e3ed-4eb3-90c5-b55a2c23fdc8
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: web
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-8jfgp
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: ijsselstein
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-8jfgp
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-11-05T20:59:17Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-11-05T20:59:30Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-11-05T20:59:30Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-11-05T20:59:17Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://2b86e45196a6e110c7f015484f7323f0fbccdf6bab540780d722da4846e92a96
    image: docker.io/library/nginx:latest
    imageID: docker.io/library/nginx@sha256:86e53c4c16a6a276b204b0fd3a8143d86547c967dc8258b3d47c3a21bb68d3c6
    lastState: {}
    name: web
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2023-11-05T20:59:29Z"
  hostIP: 192.168.1.137
  phase: Running
  podIP: 10.42.0.69
  podIPs:
  - ip: 10.42.0.69
  qosClass: BestEffort
  startTime: "2023-11-05T20:59:17Z"
[~/projects/learning-kubernetes] # 
```

